+++
date = "2015-12-10T18:21:49+02:00"
draft = false
title = "Le Labo #6 | Packer, Docker et Go"

+++

# Introduction
Dans le précédent document, nous avions effectué une rapide présentation des fonctionnalités de Packer et des différentes clés nécessaires à la création de templates.
Dans celui-ci, nous allons observer comment faire fonctionner Packer, Go et Docker ensemble.

# Le "Builder" Docker
Le driver Docker de Packer permet de créer des images et conteneurs Docker sans utiliser de Dockerfile.
De cette façon, Packer peut provisioner des conteneurs à l'aide scripts ou de systèmes de gestion de configurations (tels que Chef ou Ansible) de la même manière qu'un serveur physique.

# Le "Provisioner" Shell
Le "provisioner" `Shell` utilise le shell pour lancer les commandes souhaitées sur la machine sur laquelle Packer est installé.

# Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 1 serveur linux (CentOS/Ubuntu) 64 bits : **mida-dockh1 (IP : 81.2.243.226)**  
- **Go** : Langage de programmation
- **Docker** : Gestion de conteneur
- **Packer**

# Installation des outils nécessaires
Afin d'installer les outils nécéssaires à ce cas pratique, nous allons lancer les commandes suivantes sur le serveur mida-dockh1 : 

- Installation de **Packer** :  
`wget https://releases.hashicorp.com/packer/0.8.6/packer_0.8.6_linux_amd64.zip`  
`apt-get install -y unzip`  
`unzip packer_0.8.6_linux_amd64.zip -d /usr/local/bin/packer`  
`echo "PATH=$PATH:/usr/local/bin/packer" >> .bashrc && exec bash `

# Création du template Packer
Packer utilise un simple fichier JSON dans lequel ous allons avoir besoin de "Provisioner" (Pour l'utilisation du shell) et de "Builder" (pour Docker).

	{
      "builders": [{
        "type": "docker",
        "image": "rastasheep/ubuntu-sshd:14.04",
        "export_path": "image.tar"
      }],
	"provisioners": [{
		"type": "shell",
		"inline": [
			"apt-get update -y",
			"apt-get install -y curl git mercurial make binutils bison gcc build-essential python-software-properties golang vim",
			"export GOROOT=/usr/lib/go >> .bashrc",
			"export GOBIN=/usr/bin/g >> .bashrc",
      		"export GOPATH=/root/gocode >> .bashrc",
      		"export PATH=$PATH:$GOPATH/bin >> .bashrc",
      		"mkdir /root/gocode",
      		]
	  	}]
  	}

# Déploiement 
Lancez tout d'abord la commande `packer inspect [fichier.json]` afin de vérifier qu'il n'y a aucune erreur dans le fichier.  
Le terminal affichera ce qui suit : 

	Variables:

	  <no variables>

	Builders:

	  docker

	Provisioners:

	  shell

	Note: If your Build names contain user variables or template functions such as 'timestamp', these are processed at build time, and therefore only show in their raw form here.

Ensuite, lancez la commande `packer build [fichier.json]` afin de démarrer la création de l'image.  
Le terminal affichera les étapes de création, voir ci-dessous : 
	
	==> docker: Creating a temporary directory for sharing data...
	...
	==> docker: Pulling Docker image: rastasheep/ubuntu-sshd:14.04
	...
	==> docker: Starting docker container...
	...
	==> docker: Provisioning with shell script: /tmp/packer-shell432615347
	...
	==> docker: Exporting the container
	...
	==> docker: Killing the container: 962f1da87dfd2049f4233debb36227560715307a8757441da29febca1a27236e
	Build 'docker' finished.
