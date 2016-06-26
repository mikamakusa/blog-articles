+++
date = "2016-06-25T15:29:49+02:00"
draft = false
title = "Le Labo #17 | Packer, Terraform & Docker"

+++

# Petite introduction
Il y a quelque mois, je me lançais sur un projet "né sous la douche" (véridique...l'idée m'est venue un matin alors que je prenais ma douche) nommé "Powershell Docker Deployment" et consistait en du déploiement de conteneurs Docker sur des hôtes linux via un script en Powershell.  
Le pire, c'est que ça fonctionnait très bien...Mais, il faut dire que Terraform le fait aussi bien.  
Ce sera justement l'objet de ce labo : le déploiement de conteneurs Docker via Terraform.  
Je me suis concentré sur un Cloud Provider (**Digital Ocean**), mais c'est tout à fait faisable avec n'importe quel autre (**AWS**, **GCE** ou **Azure**).


# De quoi avons nous besoin ?
Pour avancer sur ce lab, je vais avoir besoin des éléments suivants : 

- Le token API de **Digital Ocean**
- **Docker** installé sur le serveur distant
- Un port ouvert rattaché au Docker Daemon
- **Terraform** et **Packer** installé en local

Pour ais-je besoin d'un port ouvert pour le démon Docker ? Parce que Terraform va effectuer des actions sur l'API de Docker.


# Du côté de Packer
Sur **Digital Ocean** la version d'**Ubuntu** disponible pour **Docker** n'est pas la plus à jour.  
Certes, cela reste une LTS (la 14.04) mais je souhaitais benéficier de la toute dernière (la 16.04). C'est pour cela que j'ai commencé par créer l'image que j'allais utiliser par la suite.

Pour cela, j'ai créé les fichiers do.json (le template **Packer**) et docker.sh (le script d'installation de **Docker**).

## Le fichier do.json

	{
		"builders": [{
			"type": "digitalocean",
			"api_token": "<API_token>",
			"image": "ubuntu-16-04-x64",
			"region": "ams2",
			"size": "512mb",
			"droplet_name": "docker"
		}],
		"provisioners": [{
		    "type": "file",
		    "source": "docker.sh",
		    "destination": "/root/docker.sh"
	  	}]
	}

## Le script docker.sh

	#!/bin/bash -e

	sleep 30
	sudo apt-get update -y
	sudo touch /etc/apt/sources.list.d/docker.list
	sudo echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" > /etc/apt/sources.list.d/docker.list
	sudo apt-get update -
	sudo apt-get install docker-engine -y --allow-unauthenticated


## Le résultat de packer build <file>

	digitalocean output will be in this color.

	==> digitalocean: Creating temporary ssh key for droplet...
	==> digitalocean: Creating droplet...
	==> digitalocean: Waiting for droplet to become active...
	==> digitalocean: Waiting for SSH to become available...
	==> digitalocean: Connected to SSH!
	==> digitalocean: Uploading docker.sh => /root/docker.sh
	==> digitalocean: Provisioning with shell script: /tmp/packer-shell629686493
	==> digitalocean: Gracefully shutting down droplet...
	==> digitalocean: Creating snapshot: packer-1466866245
	==> digitalocean: Waiting for snapshot to complete...
	==> digitalocean: Destroying droplet...
	==> digitalocean: Deleting temporary ssh key...
	Build 'digitalocean' finished.

Dans la partie qui suit, je retrouve l'ID du snapshot avec lequel je vais déployer le serveur qui me servira par la suite : 

	==> Builds finished. The artifacts of successful builds are:
	--> digitalocean: A snapshot was created: 'packer-1466866245' (ID: 18097708) in region 'ams2'


# Du côté de Terraform
## Provider Digital Ocean
J'avais déjà fais un tour du côté de **Digital Ocean** avec **Terraform** il y a quelques semaines, donc je ne vais pas avoir besoin de réexpliquer les bases du déploiement (les *variables* et le *builder*), par contre je vais détailler la partie *resource* : 

	resource "digitalocean_droplet" "docker-master" {
		image = "18097708"
		name = "docker-master"
		region = "ams2"
		size = "512mb"
		ssh_keys = [
			"${var.ssh_fingerprint}"
		]
		provisioner "remote-exec" {
			connection {
				user = "root"
				type = "ssh"
				key_file = "${var.pvt_key}"
			}
			inline = [
				"sudo service docker stop",
				"sudo docker daemon -H=0.0.0.0:1234"
			]
		}
	}

## Provider Docker
Dans le même fichier template, j'ajoute le provider **docker** pour la raison suivante : je vais pouvoir utiliser l'option *ipv4_address* et, ainsi, automatiser la connexion à l'hôte **Docker** dans le but de déployer des conteneurs par la suite.

	provider "docker" {
		host = "tcp://${digitalocean_droplet.docker-master.ipv4_address}:1234/"
	}

Je peux déployer de multiples conteneurs en utilisant une seule image et donc utiliser la *resource* **docker_image** ou utiliser plusieurs images différentes : 

### Avec une seule image 

	resource "docker_image" "ubuntu" {
		name = "ubuntu:latest"
	}
	resource "docker_container" "test" {
		name = "test1"
		image = "${docker_image.ubuntu.latest}"
	}

### Avec plusieurs images

	resource "docker_container" "test" {
		name = "test1"
		image = "ubuntu:latest"
	}
	resource "docker_container" "test" {
		name = "test2"
		image = "centos:latest"
	}
	resource "docker_container" "test" {
		name = "test3"
		image = "rastasheep/ubuntu-sshd:14.04"
	}

Le déploiement des conteneurs est relativement classique et fonctionne de la même manière que si l'on utilisait uniquement le CLI **docker**
par exemple, pour déployer un conteneur avec les spécification suivantes : 

- port 80 et 445 ouverts
- lien avec un conteneur mysql
- publication du dossier var/www vers /home/conteneur/www
- hostname du conteneur : web

---
	resource "docker_container" "web" {
		name = "web"
		image = "ubuntu:latest"
		hostname = "web"
		ports = [{
			"internal": 80,
			"external": 80
		},
		{
			"internal": 443,
			"external": 443
		}]
		volume = [{
			"host_path": "/home/conteneur/www",
			"container_path": /var/www
		}]
		"links": "mysql"
	}

## Le résultat de terraform plan

	+ digitalocean_droplet.docker-master
	    image:                "" => "18097708"
	    ipv4_address:         "" => "<computed>"
	    ipv4_address_private: "" => "<computed>"
	    ipv6_address:         "" => "<computed>"
	    ipv6_address_private: "" => "<computed>"
	    locked:               "" => "<computed>"
	    name:                 "" => "docker-master"
	    region:               "" => "ams2"
	    size:                 "" => "512mb"
	    ssh_keys.#:           "" => "1"
	    ssh_keys.0:           "" => "8d:68:36:ed:6a:5e:37:84:89:3b:a0:c7:f0:88:5b:7d"
	    status:               "" => "<computed>"

	+ docker_container.web
	    bridge:           "" => "<computed>"
	    gateway:          "" => "<computed>"
	    image:            "" => "${docker_image.ubuntu.latest}"
	    ip_address:       "" => "<computed>"
	    ip_prefix_length: "" => "<computed>"
	    log_driver:       "" => "json-file"
	    must_run:         "" => "true"
	    name:             "" => "web"
	    restart:          "" => "no"

	+ docker_image.ubuntu
	    latest: "" => "<computed>"
	    name:   "" => "ubuntu:latest"

	Plan: 2 to add, 0 to change, 0 to destroy.


# Pour le prochain ?
Soyons fou...mais j'hésite encore.  
Ce sera la surprise.