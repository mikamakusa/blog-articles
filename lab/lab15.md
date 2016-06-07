+++
date = "2016-06-07T15:29:49+02:00"
draft = false
title = "Le Labo #13 2/2 | Provisionning d'infrastructure à l'aide de Terraform"

+++

# Provisionning d'infrastructure à l'aide de Terraform - 2eme partie

## Petite introduction
J'avais déjà effectué un premier essai avec Terraform, sur mon compte Digital Ocean, dans le but de déployer un dizaine de serveurs...sans aller très loin.  
C'est à dire utiliser un outil de **Configuration Management** tel que **Puppet**, **Ansible**, **Chef** ou n'importe quel autre **Provisionner** évoqué dans la documentation officielle pour déployer des paquets de manière totalement automatisée.  
Dans cet article, il sera question d'**Ansible**.


## Avant de commencer...
Je vais partir du principe que **Terraform** et **Ansible** sont déja installés.  
Dans le cas contraire, de nombreux tutoriels sont disponible (ainsi que sur ce blog).

Je vais avoir besoin des packages et fonctionnalités suivantes :

- **Consul**
- **Load Balancing**
- **MySQL**
- **Serveur Web**

Puisque je vais utiliser **Ansible** pour déployer les services dont je vais avoir besoin, je ne vais utiliser que des images classiques et non des "One-Click Applications".


## Préparons le déploiement...
### Du côté de Terraform
#### Les variables
Je vais avoir besoin des fichiers **digitalocean-provider.tf** et **terraform.tfvars** pour stocker les variables relatives au provider, à la clé SSH ainsi que certaines autres variables.

**digitalocean-provider.tf** :

	variable "do_token" {}  
	variable "pub_key" {}  
	variable "pvt_key" {}  
	variable "ssh_fingerprint" {}  
	variable "user_data" {}
	provider "digitalocean" {  
	    token = "${var.do_token}"  
	}
	variable "www_count" {
	  description = "Number of web servers"
	  default = "2"
	}
	variable "mysql_count" {
		description = "Number of mysql servers"
		default = "2"
	}
	variable "do_region" {
		default = "ams2"
	}
	variable "do_image" {
		default = "ubuntu-14-04-x64"
	}

**terraform.tfvars**

	do_token = "<token>"
	pub_key = "<clé_publique>"
	pvt_key = "clé_privée"  
	ssh_fingerprint = "<fingerprint_ssh>"


#### La clé SSH
Il est possible de se passer de clé SSH pour créer les serveurs, mais dans le cas de **DigitalOcean**, les mots de passe *root* sont transmis par mail.  

Tout d'abord, on crée la clé à l'aide de la commande suivante :  
`ssh-keygen -t rsa`  
Et on intègre la clé au fichier de déploiement :

	resource "digitalocean_ssh_key" "default" {  
	    name = "ssh_key"  
	    public_key = "${var.pub_key}"  
	}

L'output de la commande suivante est à intégrer dans la variable "ssh_fingerprint" :  
`ssh-keygen -lf ~/.ssh/id_rsa.pub | awk '{print $2}'`


#### Le Load Balancer
Je vais maintenant aborder le fichier principal de **Terraform** qui va servir à définir les instances Cloud à déployer...en commençant par le **Load Balancer** : 

	# Create Load Balancer
	resource "digitalocean_droplet" "do_lb" {
	    image = "{var.do_image}"
	    name = "do_lb"
	    region = "{var.do_region}"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	    connection {
			user = "root"
			type = "ssh"
			key_file = "${var.pvt_key}"
			timeout = "2m"
		}
	}
	resource "digitalocean_floating_ip" "do_lb" {
	    droplet_id = "${digitalocean_droplet.do_lb.id}"
	    region = "${digitalocean_droplet.do_lb.region}"
	}

#### Les serveurs web
Vous avez remarqué ?  
Parmi les variables, j'en ai défini une nommée **www_count** à **2**.  
Ce qui signifie que je vais déployer deux serveurs web.

	# Create Web Servers
	resource "digitalocean_droplet" "do_web" {
	    image = "{var.do_image}"
	    name = "do_web_${count.index}"
	    region = "{var.do_region}"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    count = "${var.www_count}"
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	    connection {
			user = "root"
			type = "ssh"
			key_file = "${var.pvt_key}"
			timeout = "2m"
		}
	}
	resource "digitalocean_floating_ip" "do_web_${count.index}" {
	    droplet_id = "${digitalocean_droplet.do_web_${count.index}.id}"
	    region = "${digitalocean_droplet.do_web_${count.index}.region}"
	}

#### Les serveurs MySQL
A l'instar des serveurs Web, j'ai également défini une variable nommée **mysql_count** à **2**. Je vais donc utiliser le même paramètre *count* qui permet de renseigner le nombre d'instances à démarrer.

	# Create Web Servers
	resource "digitalocean_droplet" "do_mysql" {
	    image = "{var.do_image}"
	    name = "do_mysql_${count.index}"
	    region = "{var.do_region}"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    count = "${var.mysql_count}"
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	    connection {
			user = "root"
			type = "ssh"
			key_file = "${var.pvt_key}"
			timeout = "2m"
		}
	}
	resource "digitalocean_floating_ip" "do_web_${count.index}" {
	    droplet_id = "${digitalocean_droplet.do_web_${count.index}.id}"
	    region = "${digitalocean_droplet.do_mysql_${count.index}.region}"
	}


### Du côté d'Ansible
Maintenant que les hôtes sont définis dans **Terraform**, je vais pouvoir me concentrer sur le playbook et les rôles à déployer à l'aide d'**Ansible**.

#### Le playbook
Je vais créer deux playbooks, le premier va me permettre de générer automatiquement le fichier d'inventaire...un comme celui que j'avais créé pour le Labo n°9 (**Déploiement automatisé et surveillance de microservices**).
Par contre, mon expérience sur le script de déploiement en python me sera utile.

##### Playbook n°1

	---
	- hosts: localhost
	  sudo: yes
	  tasks:
	    - name: Plan Terraform deployment
	      shell: terraform plan
	    - name: Launch terraform deployment
	      shell: terraform apply
	    - wait_for: path=/path/to/terraform.tfstate/file
	    - name: Inventory generation
	      shell: ./inventory.py
	    - wait_for: path=/path/to/ansible_host
	    - name: Launch second ansible playbook
	      shell: ansible-playbook /path/to/second_playbook.yml

##### le script inventory.py
	
	import requests
	token = "<do_token>"
	request = requests.get("https://api.digitalocean.com/v2/droplets", headers={"Authorization": "Bearer %s" % token})
	data = request.json()
	for i in data['droplets']:
		i['name'] + i['networks']['v4']['ip_address'] >> ansible_host

##### Playbook n°2

	---
	- hosts: localhost
	  sudo: yes
	  roles:
	    - consul
	  tasks:
	    - name: Launch Consul Master
	      shell: consul agent -data-dir consul -server -bootstrap -client <ip.address> -advertise <local.ip.address> -ui-dir /home/web_ui/ &
	- hosts: do_lb
	  sudo: yes
	  roles:
	    - load_balancer
	  	- consul
	  tasks:
	    - name: Launch Consul Agent
	      shell : consul agent -data-dir /root/consul -client <local.ip.address> -advertise <local.ip.address> -node webserver -config-dir /home/consul/service -join <master.ip.address>
	- hosts: do_web_${count.index}
	  sudo: yes
	  roles:
	    - webserver
	    - consul
	  tasks:
	    - name: Launch Consul Agent
	      shell : consul agent -data-dir /root/consul -client <local.ip.address> -advertise <local.ip.address> -node webserver -config-dir /home/consul/service -join <master.ip.address>
	- hosts: do_mysql_${count.index}
	  sudo: yes
	  roles:
	    - database
	    - consul
	  tasks:
	    - name: Launch Consul Agent
	      shell : consul agent -data-dir /root/consul -client <local.ip.address> -advertise <local.ip.address> -node webserver -config-dir /home/consul/service -join <master.ip.address>

#### Le rôle Consul

	---
	- name: install wget
	  apt: name=wget state=present
	- name: Install unzip
	  apt: name=unzip state=present
	- name: Install Consul
	  shell: cd /root && wget https://releases.hashicorp.com/consul/0.6.1/consul_0.6.1_linux_amd64.zip && unzip consul_0.6.1_linux_amd64.zip -d /usr/local/bin/consul
	- name: 4. Add file to environment
	  shell: echo "PATH=$PATH:/usr/local/bin/consul" >> /root/.bashrc

#### Le rôle Webserver

	---
	- name: install Apache
	  apt: name=apache2 state=present
	- name: install PHP module for Apache
	  apt: name=libapache2-mod-php5 state=present
	- name: start Apache
	  service: name=apache2 state=running enabled=yes
	- name: install Hello World PHP script
	  copy: src=index.php dest=/var/www/index.php mode=0664
	- name: Check file directory
	  shell: mkdir -p /home/consul/service
	- name: File Consul checks
	  copy: src=services.json dest=/home/consul/service/services.json mode=0664

#### Le rôle Database

	---
	- name: Install mysql server package
	  apt: name=mysql-server state=present
	- name: Start Mysql Service
	  service: name=mysql state=started enabled=true
	- name: Install python Mysql package #required for mysql_db tasks
	  apt: name=python-mysqldb state=present
	- name: Check file directory
	  shell: mkdir -p /home/consul/service
	- name: File Consul checks
	  copy: src=services.json dest=/home/consul/service/services.json mode=0664

#### Le rôle Load Balancer
##### Le role Ansible

	---
	- name: Install HA Proxy package
	  apt: name=haproxy state=present
	- name: Enable haproxy
	  shell: echo ENABLED=1 > /etc/default/haproxy
	- name: move old config file
	  shell: mv /etc/haproxy/haproxy.cfg /etc/haproxy/old_haproxy.cfg
	- name: copy new config file
	  copy: src=haproxy.cfg /etc/haproxy/haproxy.cfg
	- name: Start HAProxy service
	  service: name=haproxy state=started enabled=true

##### Le fichier haproxy.cfg

	global
	        log 127.0.0.1    local0 notice
	        log 127.0.0.1   local1 notice
	        chroot /var/lib/haproxy
	        user haproxy
	        group haproxy
	        daemon
	defaults
	        log     global
	        mode    http
	        option  httplog
	        option  dontlognull
	        contimeout 5000
	        clitimeout 50000
	        srvtimeout 50000
	        errorfile 400 /etc/haproxy/errors/400.http
	        errorfile 403 /etc/haproxy/errors/403.http
	        errorfile 408 /etc/haproxy/errors/408.http
	        errorfile 500 /etc/haproxy/errors/500.http
	        errorfile 502 /etc/haproxy/errors/502.http
	        errorfile 503 /etc/haproxy/errors/503.http
	        errorfile 504 /etc/haproxy/errors/504.http
	
	listen appname 0.0.0.0:80
		mode http
		stats enable
		stats uri /haproxy?stats
		stats realm Strictly\ Private
		stats auth <A_Username:YourPassword>
		stats auth <Another_User:passwd>
		balance roundrobin
		option httpclose
		option forwardfor
		server lamp1 <WebServer_1_Ipaddress>:80 check
		server lamp2 <WebServer_2_Ipaddress>:80 check

## Pour aller plus loin
Dans mon premier article sur Terraform, j'avais évoqué le fait que l'on pouvait l'intégrer à tout.  
Il y a, certes, encore pas mal de *fine tunning* a effectuer pour que les deux serveurs mysql fonctionnent en mode réplication ou en mode cluster.  
Il est possible de gérer ce type de configuration à l'aide d'Ansible...Mais ça, ce sera certainement pour un autre article.