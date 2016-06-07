+++
date = "2016-06-05T15:29:49+02:00"
draft = false
title = "Le Labo #13 1/2 | Provisionning d'infrastructure à l'aide de Terraform"

+++

# Provisionning d'infrastructure à l'aide de Terraform - 1ère partie


## Petite introduction  
Il y a plusieurs mois, j'avais fais un tour d'horizon de **Terraform** mais je n'avais jamais rien tenté dans ce sens là...tout simplement parce que ce n'est pas compatible avec l'infrastructure Cloud de mon ancien employeur : **ArubaCloud**.  
Entre temps, je me suis ouvert plusieurs comptes chez plusieurs Cloud Providers pour étudier leur API et développer un script (en Python)effectuant plus ou moins le même travail que **Terraform**.  

Depuis, j'ai pu observer en détail le fonctionnement de Terraform (en production) sur **Goougle Cloud Engine** et j'ai décidé de tenter l'expérience avec mon compte **Digital Ocean**.


## Installation de Terraform
L'installation est très simple, il suffit juste de télécharger l'archive sur le site **terraform.io** et est disponible pour plusieurs systèmes d'exploitations :

- **MacOS X**
- **FreeBSD**
- **OpenBSD**
- **Linux**
- **Solaris**
- **Windows**

Pour l'installer sur Linux, exécutez juste les commandes suivantes :  
`wget https://releases.hashicorp.com/terraform/0.6.16/terraform_0.6.16_linux_<version>.zip`  
*version* correspond au type de système sur lequel **Terraform** sera installé :  
- 32bits : 386  
- 64bits : amd64  
- arm : arm  

`unzip terraform_0.6.8_linux_amd64.zip -d /usr/local/bin/terraform`  
`echo "PATH=$PATH:/usr/local/bin/terraform" >> .bashrc && exec bash`
 

## Avant de commencer
Pour ce Lab, nous n'allons pas déployer de nombreux serveurs. Je me suis fixé une limite de 10 qui me semble raisonnable.

**Détails importants** :

- Le *réseau privé* sera activé ainsi qu'une clé privé SSH.

- Je vais déployer un serveur avec CoreOS, contrairement aux autres distributions, il nécessite une configuration particulière.  

- Je vais déployer, au moins, une 'One-Click Applications' parmi celle disponible sur leur site.

- Par la suite, je vais aussi déployer plusieurs paquets sur ces serveurs...dans ce but, je vais étudier plusieurs pistes (Puppet, Gitlab-CI, Ansible). Ce qui fera l'objet d'un prochain article.


## La voie "Terraform"
### Les variables
Pour pouvoir utiliser les ressources de son compte Digital Ocean, sans avoir à se rendre sur leur site, il nous le token. Ce token est généré sur le site de Digital Ocean.  
On peut aussi faire appel à la méthode Oauth2 ou à la libraire **doto** en python pour générer le token automatiquement.

Ensuite, on crée un fichier **digitalocean-provider.tf** ce qui suit :  

	variable "do_token" {}  
	variable "pub_key" {}  
	variable "pvt_key" {}  
	variable "ssh_fingerprint" {}  
	variable "user_data" {}
	provider "digitalocean" {  
	    token = "${var.do_token}"  
	}

Et ensuite, on crée le fichier **terraform.tfvars** dans lequel on insère les variables :  

	do_token = "<token>"
	pub_key = "<clé_publique>"
	pvt_key = "clé_privée"  
	ssh_fingerprint = "<fingerprint_ssh>"

Il est possible de rencontrer un message d'erreur du type : 

	* digitalocean_ssh_key.default: Error creating SSH Key: POST https://api.digitalocean.com/v2/account/keys: 422 Key invalid, key should be of the format `type key [comment]`

Dans ce cas, on peut créer la clé ssh dans le panneau de contrôle de Digital Ocean.

### La clé SSH
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


### Le fichier "User Data"
Comme expliqué plus haut, pour le serveur CoreOS, une configuration particulière est obligatoire. Celle-ci inclue des **user data**.  
Il est possible de les intégrer directement dans la déclaration du serveur, mais je vais créer un fichier juste pour eux et les intégrer comme je le fais pour les clés SSH.  

Voici un fichier **user_data.yml** exemple, libre à vous de créer le votre avec votre configuration : 

	#cloud-config

	coreos:
	  units:
	    - name: docker.service
	      command: start
	    - name: fleet.service
	      command: start


### Les serveurs
Comme expliqué plus haut, je vais déployer dix serveurs, dont un (CoreOS) qui requiert des paramètres particuliers et deux 'One-Click Applications'.

**Server1** à **Server7**
	
	resource "digitalocean_droplet" "ServerX" {
	    image = "ubuntu-14-04-x64"
	    name = "server(x)"
	    region = "ams2"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	}
	resource "digitalocean_floating_ip" "ServerX" {
	    droplet_id = "${digitalocean_droplet.ServerX.id}"
	    region = "${digitalocean_droplet.ServerX.region}"
	}


**Server8**

	resource "digitalocean_droplet" "Server8" {
	    image = "coreos-stable"
	    name = "server8"
	    region = "ams2"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    user_data = "${var.user_data}"
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	}
	resource "digitalocean_floating_ip" "Server8" {
	    droplet_id = "${digitalocean_droplet.Server8.id}"
	    region = "${digitalocean_droplet.Server8.region}"
	}

**Server9** : Gitlab

	resource "digitalocean_droplet" "Server9" {
	    image = "17689953"
	    name = "server9"
	    region = "ams2"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	}
	resource "digitalocean_floating_ip" "Server9" {
	    droplet_id = "${digitalocean_droplet.Server9.id}"
	    region = "${digitalocean_droplet.Server9.region}"
	}

**Server10** : Nginx  

	resource "digitalocean_droplet" "Server10" {
	    image = "16539180"
	    name = "server10"
	    region = "ams2"
	    size = "512mb"
	    private_networking = true
	    ipv6 = true
	    ssh_keys = [
	      "${var.ssh_fingerprint}"
	    ]
	}
	resource "digitalocean_floating_ip" "Server10" {
	    droplet_id = "${digitalocean_droplet.Server10.id}"
	    region = "${digitalocean_droplet.Server10.region}"
	}

### Les commandes de Terraform
Une fois le fichier de déploiement créé, il est temps de se lancer. Pour cela, on exécute la commande `terraform plan` pour vérifier que le fichier ne comporte aucune erreur.  
Si tout est bon, on peut véritablement se lancer avec la commande `terraform apply`.

On devrait obtenir un résultat similaire à ce qui suit : 

	digitalocean_droplet.Server1: Still creating... (10s elapsed)
	digitalocean_droplet.Server1: Still creating... (20s elapsed)
	digitalocean_droplet.Server1: Still creating... (30s elapsed)
	digitalocean_droplet.Server1: Creation complete
	digitalocean_floating_ip.Server1: Creating...
	  droplet_id: "" => "16892410"
	  ip_address: "" => "<computed>"
	  region:     "" => "ams2"
	digitalocean_floating_ip.Server1: Still creating... (10s elapsed)
	digitalocean_floating_ip.Server1: Creation complete

Pour vérifier que tout est bien déployé : `terraform show terraform.tfstate`

Pour détruire ce que l'on vient de créer : `terraform destroy`


## Pour aller plus loin
Terraform s'intègre à tout : Consul, Serf, Googgle Cloud, AWS...même Azure.  
Pour ce qui est du **Configuration Management**, j'ai pu observer Puppet en action sur un infrastructure déployée via Terraform, et ce sera l'objet d'un prochain article.  
Pas forcément **Puppet**, j'hésite encore entre celui-ci, **Chef**, **Ansible** ou un autre (**Gitlab-CI** peut-être).