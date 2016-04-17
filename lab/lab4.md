+++
date = "2015-11-18T18:21:49+02:00"
draft = false
title = "Le Labo #4 | Déploiement de conteneurs Docker automatisé sur plusieurs hôtes"

+++

# 1 - Introduction
Dans un précédent tutoriel, nous avions abordé la création d'un cluster sur deux niveaux à l'aide de deux hôtes, ainsi que Swarm pour relier les hôtes, Weave pour la partie réseau entre les conteneurs et Zookeeper pour la gestion du cluster de conteneurs.
Dans celui ci, nous aborderons le déploiement automatisé de conteneurs Docker sur plusieurs hôtes à l'aide de Docker, Weave, Swarm et Ansible.

# 2 - Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 3 serveurs linux (CentOS/Ubuntu) 64bits : **mida-clm1 (IP 81.2.240.198)**, **mida-clm2 (IP 81.2.248.101)** et **mida-cls1 (IP 80.10.243.100)**
- <u>Ansible</u> : Automatisation de configuration
- <u>Docker</u> : Gestion de conteneur
- <u>Docker Compose</u>: Orchestration de déploiement de conteneurs
- <u>Swarm</u> : Gestion de cluster de l'écosystème Docker
- <u>Weave</u> : Gestion du réseau Docker
- <u>Weave Scope</u> : Mappage, détection et surveillance de conteneurs Docker

# 3 - Installation des outils nécessaires
Afin d'installer les outils nécessaires à ce cas pratique, nous allons lancer les commandes suivantes sur **mida-dockh1** :  

- Installation de **Ansible** : 
`apt-get install software-properties-common -y`   
`add-apt-repository ppa:ansible/ansible`  
`apt-get update -y`  
`apt-get install ansible phyton-pip -y`  
`pip install paramiko PyYAML Jinja2 httplib2 six`

Puis les commandes suivantes sur chaque hôtes : 

- Installation de **Docker** (sur les 3 hôtes) :  
`curl -sSL https://get.docker.com/ | sh`

- Installation de **docker Swarm** (sur les 3 hôtes) :  
`docker pull swarm:latest`
  
- Installation de **Weave** (sur les 3 hôtes) :  
`curl -L git.io/weave -o /usr/local/bin/weave`    
`chmod a+x /usr/local/bin/weave`  

- Installation de **Weave Scope** (sur les 3 hôtes) :  
`wget -O scope git.io/scope && cp scope /usr/local/bin/`  
`chmod a+x /usr/local/bin/scope`

# 4 - Configuration des outils
### Génération de clé ssh et copie de la clé sur les noeuds esclaves
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@mida-clm2 && ssh-copy-id -i ~/.ssh/id_rsa.pub root@mida-cls1`

Pour connexion ssh sans mot de passe sur le noeud maître :  
`cat ~/.ssh/id_rsa.pub » ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`

### Configuration d'Ansible
Nous allons commencer par ajouter les hôtes dans le fichier /etc/ansible/hosts : 

  	mida-clm1
  	mida-clm2
  	mida-cls1

Ansi que dans le fichier /etc/hosts : 

  	81.2.240.198 mida-clm1
  	81.2.248.101 mida-clm2
  	80.10.243.100 mida-cls1

### Configuration de Docker 
Afin de pouvoir monter notre cluster Swarm et Weave, nous allons devoir le port d'écoute 2375 sur les trois serveurs à l'aide de la ligne suivant dans le fichier /etc/default/docker :  
`DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"`

### Configuration de Swarm

- Generer le token : `docker run swarm create`
- Démarrer **Swarm Manager** : `docker run swarm manage -H tcp://81.2.240.198:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`
- Ajout des serveurs au cluster Swarm :   
Sur **mida-clm1** : `docker run swarm join --addr=81.2.240.198:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`  
Sur **mida-clm2** : `docker run swarm join --addr=81.2.248.101:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`  
Sur **mida-cls1** : `docker run swarm join --addr=80.10.240.100:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`


### Configuration de Weave Scope
Pour celui-ci, nous avons juste besoin de le démarrer, sur les trois hôtes, à l'aide de la commande suivante :  
`scope launch scope1:4030 mida-clm1:4030 mida-clm2:4030 mida-clms1:4030`

Une fois démarré, le terminal affichera le message suivant :  

	Weave Scope is reachable at the following URL(s):
	* http://[adresse IP de l'hôte]:4040

### Configuration de Weave Net
Nous allons démarrer Weave sur chaque hôtes à l'aide de la commande suivante : 

- Sur **mida-clm1** : `weave launch --init-peer-count 3` puis `weave connect mida-clm2` et enfin `weave connect mida-cls1`  
- Sur **mida-clm2** : `weave launch --init-peer-count 3`  
- Sur **mida-cls1** : `weave launch --init-peer-count 3`  

A l'aide de la commande `weave status connections` nous pourrons vérifier que chaque hôtes est bien lié.

# 5 - Déploiement de conteneurs isolés
Pour cet exercice, nous allons utiliser l'image Docker suivante : rastasheep/ubuntu-sshd:14.04.  
Nous allons générer le fichier de déploiement nécessaire à Ansible ci-dessous : 

	- hosts: all
    	tasks:
        - name: Deploy docker containers
          command: docker run -d -P --name test_sshd rastasheep/ubuntu-sshd:14.04

Ensuite, tapez la commande suivante : 
`ansible-playbook playbook.yml`

En démarrant le playbook, le terminal affichera ce qui suit : 

	PLAY [all] ***********************************************************

	GATHERING FACTS ******************************************************
	ok: [mida-cls1]
	ok: [mida-clm2]
	ok: [mida-clm1]

	TASKS: [Deploy docker containers] ************************************
 	changed: [mida-clm1]
  	changed: [mida-clm2]
  	changed: [mida-cls1]

  	PLAY RECAP ***********************************************************
  	mida-clm1       : ok=3    changed=1     unreachable=0     failed=0
  	mida-clm2       : ok=3    changed=1     unreachable=0     failed=0
  	mida-cls1       : ok=3    changed=1     unreachable=0     failed=0

Une fois ceci effectué, vous pouvez lancer la commande sur chaque hôte pour vérifier que le conteneur a bien été déployé : 

  	root@mida-clm1:~# docker ps -a
  	CONTAINER ID        IMAGE                          COMMAND               CREATED             STATUS              PORTS                   NAMES
  	abd14f29cbe9        rastasheep/ubuntu-sshd:14.04   "/usr/sbin/sshd -D"   14 seconds ago      Up 14 seconds       0.0.0.0:32768->22/tcp   test_sshd

  	root@mida-clm2:~# docker ps -a
  	CONTAINER ID        IMAGE                          COMMAND               CREATED             STATUS              PORTS                   NAMES
  	z0de12r95dcq        rastasheep/ubuntu-sshd:14.04   "/usr/sbin/sshd -D"   14 seconds ago      Up 14 seconds       0.0.0.0:32768->22/tcp   test_sshd

  	root@mida-cls1:~# docker ps -a
  	CONTAINER ID        IMAGE                          COMMAND               CREATED             STATUS              PORTS                   NAMES
  	hil5692fb1n0        rastasheep/ubuntu-sshd:14.04   "/usr/sbin/sshd -D"   14 seconds ago      Up 14 seconds       0.0.0.0:32768->22/tcp   test_sshd

# 6 - Déploiement de conteneurs liés
## le fichier playbook.yml
Il est tout a fait possible d'utiliser Docker Compose pour effectuer un déploiement automatisé de plusieurs conteneurs, mais Docker compose reste limité à un seul hôte.  
Cependant, il est possible de contourner cela afin que le fichier de déploiement de Compose puisse ête utilisé sur plusieurs hôtes et ceci toujours à l'aide de Ansible.  
Pour cela, nous allons générer le playbook suivant afin de l'utiliser par la suite avec la commande `ansible-playbook playbook.yml` : 

	---
	- hosts: all
	  tasks:
        - name: Install python-pip
      	  apt: pkg=python-pip state=installed update_cache=true
    	- name: Install docker Compose
          command: pip install -U docker-compose
    	- name: Copy docker compose file
          copy: src=docker-compose.yml dest=/root/docker-compose.yml
    	- name: Deploy containers
          shell: docker-compose up -d

## le fichier docker-compose.yml
Dans notre exemple, nous allons utiliser le fichier suivant : 

	db1:
	  image: peterbourgon/tns-db
	db2:
	  image: peterbourgon/tns-db
	  links:
	    - db1
	db3:
	  image: peterbourgon/tns-db
	  links:
	    - db1
	    - db2

	app1:
	  image: peterbourgon/tns-app
	  links:
	    - db1
	    - db2
	    - db3
	app2:
	  image: peterbourgon/tns-app
	  links:
	    - db1
	    - db2
	    - db3

	lb1:
	  image: peterbourgon/tns-lb
	  links:
	    - app1
	    - app2
	  ports:
	    - 0.0.0.0:8001:80
	lb2:
	  image: peterbourgon/tns-lb
	  links:
	    - app1
	    - app2
	  ports:
	    - 0.0.0.0:8002:80

Les modifications effectuées par rapport au playbook précédent permettent au fichier de déploiement de fonctionner afin d'obtenir le résultat recherché : le déploiement automatisé d'un ou plusieurs conteneurs, probablement liés entre eux, décrit dans le fichier docker-compose.yml.

# 7 - Conclusion
Voici un cas pratique de déploiement automatisé de conteneur Docker sur plusieurs hôtes à l'aide de Ansible, chaque conteneurs étant réliés sur un même réseau afin de pouvoir communiquer entre eux.
