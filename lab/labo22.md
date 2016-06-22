+++
date = "2016-06-15T15:29:49+02:00"
draft = false
title = "Le Labo #15 - 3eme partie | Industrialisation d'infrastructure Cloud sur AWS"

+++

# Industrialisation d'infrastructure Cloud sur AWS - 3eme partie

## Petite introduction
Après avoir créé ma première image (Wouhouuuuu, alleluia...) sur la plateforme Cloud d'AWS à l'aide de **Packer** et d'**Ansible**, je vais en générer une nouvelle.  
Cette fois-ci, au lieu d'utiliser **Ansible**, je vais opter pour **Puppet**.  
Ayant déjà utilisé ce dernier dans le cadre d'un de mes derniers lab (le Lab #14 - avec **Jenkins** et **Gitlab**), l'expérience engrangée me servira aussi durant celui-ci.

## Avant d'aller plus loin
Puisque j'ai déjà effectué un travail préparatoire sur **Packer** dans la première partie de ce Lab, je vais pouvoir passer directement sur la partie **provisioners** (côté **Packer**) et sur le/les **manifests** (côté **Puppet**).

## Du côté du script d'installation de Puppet
En fait, tout comme le provisionning via Ansible, il faut tout d'abord installer **Puppet** avant de pouvoir bénéficier du *provisionner* **puppet-masterless**

	#!/bin/bash -e
	wget -O /tmp/puppetlabs.deb http://apt.puppetlabs.com/puppetlabs-release-`lsb_release -cs`.deb
	dpkg -i /tmp/puppetlabs.deb
	apt-get update
	apt-get -y install puppet
	echo START=yes > /etc/default/puppet
	service puppet restart


## Du côté de Packer 
Sur le site officiel, on peut voir **Puppet Masterless** et **Puppet Server** parmi les **Provisionners**. Je ne vais utiliser que le premier.  
De la même manière que lors du **Lab #14**, il est possible d'utiliser les fonctionnalités de **Hiera** et de **Facter**.

	"provisioners": [
		{
			"type": "file",
			"source": "puppet_install.sh",
			"destination": "/tmp/puppet_install.sh"
		},
		{
			"type": "shell",
			"inline": [
				"chmod +x /tmp/puppet_install.sh",
				"sudo /tmp/./puppet_install.sh",
				"sudo rm -f /tmp/puppet_install.sh"
			]	
		},
		{
		  "type": "puppet-masterless",
		  "manifest_file": "site.pp"
		}
	]

## Du côté de Puppet
Je vais juste déployer un Docker Registry + UCP à l'aide de **Puppet**.

	exec { 'Add Docker public key':
		command      => 'curl -s "https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e" | sudo apt-key add --import',
		user        => 'sudo',
	}
	exec { 'Add Docker public key':
		command      => 'echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list',
		user        => 'sudo',
	}
	exec { 'apt refresh':
		command      => 'apt-get update -y',
		user        => 'sudo',
	}
	package { 'apt-transport-https':
		ensure => installed,
	}
	package { 'docker-engine':
		ensure => installed,
	}
	exec { 'download archive':
		command      => 'wget https://packages.docker.com/caas/ucp-1.1.1_dtr-2.0.1.tar.gz',
		user        => 'sudo',
	}
	exec { 'load archive':
		command      => 'docker load < /tmp/ucp-1.1.1_dtr-2.0.1.tar.gz',
		user        => 'sudo',
	}
	exec { 'Install the registry':
		command      => 'docker run docker/trusted-registry:1.4.3 install | bash',
		user        => 'sudo',
	}

## Du côté de Packer
Retournons du côté de **Packer**.  
Maintenant que les playbooks pour chaque images ont été créés, je vais pouvoir démarrer la création des images.  
Avant cela, à l'aide de la commande **packer inspect**, je vais pouvoir vérifier qu'il n'y a aucune erreur : 

	Variables:
	  <No variables>
	Builders:
	  amazon-ebs
	Provisioners:
	  file
	  shell
	  puppet-masterless
	Note: If your build names contain user variables or template
	functions such as 'timestamp', these are processed at build time,
	and therefore only show in their raw form here.

Ensuite, à l'aide de la commande **packer build**, la création des images démarre.  
Si la création des images est un succès, on peut obtenir quelque chose comme ce qui suit en fin de création: 

	==> amazon-ebs: Stopping the source instance
	==> amazon-ebs: Waiting for the instance to stop
	==> amazon-ebs: Creating the AMI: pack_registry_1465999202
	    amazon-ebs: AMI: ami-0734dd68
	==> amazon-ebs: Waiting for AMI to become ready
	==> amazon-ebs: Adding tags to AMI (ami-0734dd68)
	    amazon-ebs: adding tag: "OS_Version": "Ubuntu"
	    amazon-ebs: adding tag: "Release": "Latest"
	==> amazon-ebs: Tagging snapshot: snap-694af482
	==> amazon-ebs: Terminating the source AWS instance...
	==> amazon-ebs: Cleaning up any extra volumes...
	==> amazon-ebs: Deleting temporary security groups...
	Build 'amazon-ebs' finished.

	==> Build finished. The artifacts of successful builds are:
	--> amazon-ebs: AMIs were created