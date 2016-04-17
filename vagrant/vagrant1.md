+++
date = "2015-11-23T18:21:49+02:00"
draft = false
title = "Hashicorp | Présentation de l'écosystème"

+++

# Qu'est-ce que l'écosystème d'Hashicorp ?
L'écosystème d'Hashicorp se compose des applications suivantes : 

- **Vagrant** : Virtualisation
- **Packer** : Automatisation de création d'image
- **Serf** : Clustering
- **Consul** : Service Discovery
- **Terraform** : Outil de construction, de modification et de versioning d'infrastructure.
- **Vault** : Gestion de clé/mot de passe/certificats

# Comment installer chaque outils ?

## Vagrant
- Ensuite, téléchargez le paquet "deb" ou "rpm" depuis le site officiel :  
`wget https://dl.bintray.com/mitchellh/vagrant/vagrant_[version].[extension .deb ou .rpm]`  
- Ensuite, installez le paquet avec la commande suivante :  
*Debian/Ubuntu* : `dpkg -i vagrant_[version].deb`  
*CentOS* : `rpm -iv vagrant_[version].rpm`  
[Documentation sur Vagrant](http://www.ageekslab.com/vagrant/vagrant2/)

## Packer
Pour l'installer, il suffit de taper les commandes suivantes :  

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/packer/0.8.6/packer_0.8.6_linux_amd64.zip`  
- **Décompresser l'archive** : `unzip packer_0.8.6_linux_amd64.zip -d /usr/local/bin/packer`  
- **Déclarer le chemin de Packer** : `echo "PATH=$PATH:/usr/local/bin/packer" >> .bashrc && exec bash `
- **Vérifier l'installation** : `packer`  
[Documentation sur Packer](http://www.ageekslab.com/vagrant/packer1/)


## Serf
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/serf/0.6.4/serf_0.6.4_linux_amd64.zip`
- **Décompresser l'archive** : `unzip serf_0.6.4_linux_amd64.zip -d /usr/local/bin/serf`
- **Déclarer le chemin de Serf** : `echo "PATH=$PATH:/usr/local/bin/serf" >> .bashrc && exec bash`
- **Vérifier l'installation** : `serf`  
[Documentation sur Serf](http://www.ageekslab.com/vagrant/serf1/)  

## Consul
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_linux_amd64.zip`
- **Décompresser l'archive** : `unzip consul_0.6.0_linux_amd64.zip -d /usr/local/bin/consul`
- **Déclarer le chemin de Consul** : `echo "PATH=$PATH:/usr/local/bin/consul" >> .bashrc && exec bash`
- **Vérifier l'installation** : `consul`  
[Documentation sur Consul](http://www.ageekslab.com/vagrant/consul1/)

## Terraform
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/terraform/0.6.8/terraform_0.6.8_linux_amd64.zip`
- **Décompresser l'archive** : `unzip terraform_0.6.8_linux_amd64.zip -d /usr/local/bin/terraform`
- **Déclarer le chemin de Terraform** : `echo "PATH=$PATH:/usr/local/bin/terraform" >> .bashrc && exec bash`
- **Vérifier l'installation** : `terraform`

## Vault
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/vault/0.4.0/vault_0.4.0_linux_amd64.zip`
- **Décompresser l'archive** : `unzip vault_0.4.0_linux_amd64.zip -d /usr/local/bin/vault`
- **Déclarer le chemin de Vault** : `echo "PATH=$PATH:/usr/local/bin/vault" >> .bashrc && exec bash`
- **Vérifier l'installation** : `vault`


