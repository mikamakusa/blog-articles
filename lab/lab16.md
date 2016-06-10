+++
date = "2016-06-10T15:29:49+02:00"
draft = false
title = "Le Labo #14 | Intégration Continue - Jenkins/Puppet/Gitlab"

+++

# Intégration Continue - Jenkins/Puppet/Gitlab


## Petite introduction
Lors d'un de mes labos précédent, j'avais fais connaissance avec Gitlab-CI qui, à l'aide de Gitlab permet de bénéficier d'un environnement de déploiement continue unifié.
Ce sur quoi j'ai travaillé ici est adaptable sur Gitlab-CI/Gitlab.
C'est tout d'abord un environnement *Puppet masterless* dont les recettes Puppet sont stockées dans des dépots Gitlab, dont chaque nouveau commit fera l'objet d'un démarrage de tâche sur chaque agent Puppet.


## Mon environnement
J'avoue...Je suis litteralement tombé sous le charme de Terraform...Si bien que je l'ai encore utilisé pour déployer mon environnement de Lab.

Je vais avoir besoin des éléments suivants : 

- Un serveur Jenkins
- Un Serveur Gitlab
- Quatre agents Puppet

### Déploiement à l'aide de Terraform

#### Les variables

**digitalocean-provider.tf** :

	variable "do_token" {}  
	variable "pub_key" {}  
	variable "pvt_key" {}  
	variable "ssh_fingerprint" {}  
	variable "user_data_puppet" {}
	variable "user_data_jenkins" {}
	provider "digitalocean" {  
	    token = "${var.do_token}"  
	}
	variable "puppet_count" {
	  description = "Number of web servers"
	  default = "4"
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
	user_data_puppet = "puppet.sh"
	user_data_jenkins = "jenkins.sh"


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

#### Les serveurs

	resource "digitalocean_droplet" "Jenkins" {
		    image = "ubuntu-14-04-x64"
		    name = "Jenkins"
		    region = "ams2"
		    size = "1gb"
		    private_networking = true
		    ipv6 = true
		    user_data = "jenkins.sh"
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
	resource "digitalocean_floating_ip" "Jenkins" {
	    droplet_id = "${digitalocean_droplet.Jenkins.id}"
	    region = "${digitalocean_droplet.Jenkins.region}"
	}
	resource "digitalocean_droplet" "gitlab" {
		    image = "17689953"
		    name = "gitlab"
		    region = "ams2"
		    size = "1gb"
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
	resource "digitalocean_floating_ip" "gitlab" {
	    droplet_id = "${digitalocean_droplet.gitlab.id}"
	    region = "${digitalocean_droplet.gitlab.region}"
	}
	resource "digitalocean_droplet" "Node4" {
		    image = "ubuntu-14-04-x64"
		    name = "Puppet"
		    region = "ams2"
		    size = "1gb"
		    private_networking = true
		    ipv6 = true
		    user_data = "puppet.sh"
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
	resource "digitalocean_floating_ip" "Node4" {
	    droplet_id = "${digitalocean_droplet.Node4.id}"
	    region = "${digitalocean_droplet.Node4.region}"
	}
	resource "digitalocean_droplet" "Node1" {
		    image = "ubuntu-14-04-x64"
		    name = "Node1"
		    region = "ams2"
		    size = "1gb"
		    private_networking = true
		    ipv6 = true
		    user_data = "puppet.sh"
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
	resource "digitalocean_floating_ip" "Node1" {
	    droplet_id = "${digitalocean_droplet.Node1.id}"
	    region = "${digitalocean_droplet.Node1.region}"
	}
	resource "digitalocean_droplet" "Node2" {
		    image = "ubuntu-14-04-x64"
		    name = "Node2"
		    region = "ams2"
		    size = "1gb"
		    private_networking = true
		    ipv6 = true
		    user_data = "puppet.sh"
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
	resource "digitalocean_floating_ip" "Node2" {
	    droplet_id = "${digitalocean_droplet.Node2.id}"
	    region = "${digitalocean_droplet.Node2.region}"
	}
	resource "digitalocean_droplet" "Node3" {
		    image = "ubuntu-14-04-x64"
		    name = "Node3"
		    region = "ams2"
		    size = "1gb"
		    private_networking = true
		    ipv6 = true
		    user_data = "puppet.sh"
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
	resource "digitalocean_floating_ip" "Node3" {
	    droplet_id = "${digitalocean_droplet.Node3.id}"
	    region = "${digitalocean_droplet.Node3.region}"
	}

#### Les scripts

**puppet.sh**

	#!/bin/bash -e

	## Install Git and Puppet
	wget -O /tmp/puppetlabs.deb http://apt.puppetlabs.com/puppetlabs-release-`lsb_release -cs`.deb
	dpkg -i /tmp/puppetlabs.deb
	apt-get update
	apt-get -y install git-core puppet


**jenkins.sh**  

	#!/bin/bash
	wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	sudo apt-get update
	sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
	sudo apt-get update -y
	sudo apt-get install -y jenkins
	service jenkins start


## Configuration des outils
### Puppet (1ere partie)

Sur l'agent Puppet *Node1*, je commence par générer une clé ssh à l'aide de la commande suivante : `ssh-keygen -t rsa`  
Une fois généré, je le copie sur chaque agent **puppet**.  

### Gitlab

La configuration de Gitlab est moins compliquée que celle de jenkins.  
J'ai créé un dépôt nommé **puppet** et copié la clé ssh (générée sur *Node1* et copiée sur chacun des agents **puppet**)

### Puppet (2eme partie)
#### Git
Ensuite, je génère la configuration **Git** qui permettra d'uploader la configuration de **puppet** ainsi que les *manifests* vers **Git** pour, par la suite, les récupérer sur chaque agent **puppet**.
Voici les commandes qui permettront d'effectuer tout cela :  
*Générer la configuration Git*  
`cd /etc/puppet && git init`  
`git config --global user.name "Administrator"`  
`git config --global user.email "admin@example.com"`  
*Insérer l'adresse IP du serveur gitlab dans le fichier /etc/hosts*  
`echo xxx.xxx.xxx.xxx gitlab >> /etc/hosts`  
*Ajouter le contenur du dossier et valider*  
`git add .`  
`git commit -m "Initial commit of Puppet files"`  
*Ajouter le projet Git à la configuration locale et uploader les modifications*  
`git remote add origin git@gitlab:root/<depot_puppet>.git`  
`git push -u origin master`

#### Configuration de Puppet
**puppet.conf**  
De base, le fichier *puppet.conf* contient ce qui suit : 

	[main]
	logdir=/var/log/puppet
	vardir=/var/lib/puppet
	ssldir=/var/lib/puppet/ssl
	rundir=/var/run/puppet
	factpath=$vardir/lib/facter
	templatedir=$confdir/templates

	[master]
	# These are needed when the puppetmaster is run by passenger
	# and can safely be removed if webrick is used.
	ssl_client_header = SSL_CLIENT_S_DN 
	ssl_client_verify_header = SSL_CLIENT_VERIFY

Il faut supprimer la ligne suivante : **templatedir=$confdir/templates**  

Je vais créer les fichiers suivants : **init.pp**, **site.pp** et **post-merge**  

*cron-puppet/manifests/init.pp*  

	class cron-puppet {
	    file { 'post-hook':
	        ensure  => file,
	        path    => '/etc/puppet/.git/hooks/post-merge',
	        source  => 'puppet:///modules/cron-puppet/post-merge',
	        mode    => 0755,
	        owner   => root,
	        group   => root,
	    }
	    cron { 'puppet-apply':
	        ensure  => present,
	        command => "cd /etc/puppet ; /usr/bin/git pull",
	        user    => root,
	        minute  => '*/30',
	        require => File['post-hook'],
	    }
	}

*cron-puppet/files/post-merge*

	#!/bin/bash -e
	## Run Puppet locally using puppet apply
	/usr/bin/puppet apply /etc/puppet/manifests/site.pp

	## Log status of the Puppet run
	if [ $? -eq 0 ]
	then
	    /usr/bin/logger -i "Puppet has run successfully" -t "puppet-run"
	    exit 0
	else
	    /usr/bin/logger -i "Puppet has ran into an error, please run Puppet manually" -t "puppet-run"
	    exit 1
	fi

*/etc/puppet/manifests/site.pp*

	node default {
	    include cron-puppet
	}

Une fois ces fichiers créés, j'upload le tout sur gitlab à l'aide de la commande suivante :  
`git add . && git commit -m "Added the cron-puppet module"`  
`git push -u origin master`

Sur chacun des autres agents **Puppet**, il faudra lancer les commandes suivantes :  
*Générer la configuration Git*  
`cd /etc/puppet && git init`  
`git config --global user.name "Administrator"`  
`git config --global user.email "admin@example.com"`  
*Insérer l'adresse IP du serveur gitlab dans le fichier /etc/hosts*  
`echo xxx.xxx.xxx.xxx gitlab >> /etc/hosts`  
`git remote add origin git@gitlab:root/puppet-project.git`  
*Récupérer la configuration Puppet stockée sur le dépot Gitlab*  
`git pull`

### Jenkins

La première fois que l'on accède à l'UI de Jenkins, il nous est possible de laisser Jenkins installer les plugins recommandés ou de choisir ceux que l'on veut installer.  
J'ai préféré installer ceux que je souhaitais...c'est à dire tous ceux qui étaient proposés...cela m'a permit de bénéficier des plugins suivants :  

- Gitlab
- SSH
- PAM authentication

Ces plugins seront très utiles par la suite.

#### Les esclaves
Il y a deux manières de configurer les *esclaves* :  

- Soit en ajoutant des esclaves, via **Administrer Jenkins**/**Gérer les noeuds**
- Soit en ajoutant des hôtes SSH, via **Administrer Jenkins**/**Configurer le système**

Il ne me semble pas vraiment utile d'ajouter des *noeuds*, car il est possible de démarrer des builds sur chaque noeud en ajoutant des hôtes SSH.  
Par contre, il faut ajouter la configuration *gitlab*, en modifiant : 

- Connection name 
- Gitlab host URL 
- API token : Secret Text

Dans Identifiants, il faudra entrer le token affiché dans la configuration de Gitlab.

#### Les jobs
Pour créer un job, on clique sur *Nouveau item* (vive la traduction), on nomme le projet et on selectionne le type parmi les suivants : 

- free-style
- maven
- pipeline
- multi-configuration
- multijob
- External job
- folder
- Multibranch

Dans le cas qui m'intérese, j'ai choisi de créer plusieurs jobs **free-style** et de les configurer ainsi : 

**General** : Je n'ai rien sélectionné dans cette partie.  

**Gestion de code source** : Malgré le fait que le plugin gitlab ait été installé, il ne semble pas figurer parmi les outils de versionning. Il n'empêche qu'en sélectionnant **git** il est possible de lier **Jenkins** à **Gitlab**.  
Pour cela, je renseigne l'url du dépot **Gitlab** dans le champs *Repository URL*, les identifiants de connexion et la branche depuis laquel seront lancés les builds.

**Ce qui déclenche les builds** : J'ai sélectionné l'option suivante : *Build when a change is pushed to GitLab*.  
Et *On push to source or targe branch* sur **Rebuild open Merge Requests**.

**Environnement de build** : J'ai sélectionné l'option suivante : *Execute shell script on remote host using ssh*.  
Je sélectionne l'agent **Puppet** concerné par le job en question et je défini des actions *pré* et *post* build :  

 - Pré build

		apt-get update -y

- Post build

		cd /etc/puppet/
		git pull
		puppet apply /etc/puppet/manifests/site.pp

**Build** et **Actions à la suite du build** : Je n'ai rien modifié dans ces deux sections...mais ce sera à vous de voir ce que vous souhaiterez définir en fonction des jobs que vous créerez.

Une fois le job sauvegardé...il est temps de démarrer le job et de vérifier que tout fonctionne bien.


## Et du côté des recettes Puppet ?
Elles sont disponible sur de nombreux sites.  
Ce sera à vous de les intégrer dans votre projet.