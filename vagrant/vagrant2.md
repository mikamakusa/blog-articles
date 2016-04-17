+++
date = "2015-11-24T15:29:49+02:00"
draft = false
title = "Vagrant - Présentation et Installation"

+++

# 1 - Qu'est-ce que Vagrant ?
Vagrant est un logiciel libre et open-source pour la création et la configuration des environnements de développement virtuel.  
A l'origine lié à VirtualBox, il est maintenant compatible avec d'autres environnements de virtualisation tels que VMware, Amazon EC2 et Docker.  
De plus, il est écrit en Ruby mais compatible avec de nombreux langages tels que PHP, Python, Java, etc...

# 2 - Comment l'installer ?
Vagrant peut être installé de deux façons différentes :  

- <u>via les sources disponibles sur le site de Vagrant</u>  
>Pour installer via les sources, soyez certains que les deux paquets suivants sont déjà installés :  
 <code>dpkg-dev</code>  
 <code>virtualbox-dkms</code>
  
>Ensuite, téléchargez le paquet "deb" ou "rpm" depuis le site officiel :  
<code>wget https://dl.bintray.com/mitchellh/vagrant/vagrant_[version].[extension .deb ou .rpm]</code>

> Ensuite, installez le paquet avec la commande suivante :  
*Debian/Ubuntu* : <code>dpkg -i vagrant_[version].deb</code>  
*CentOS* : <code>rpm -iv vagrant_[version].rpm</code>

- <u>Via RubyGems</u>  
<code>gem install vagrant</code>

# 3 - Comment l'utiliser ?
Une fois installé, il faut ajouter des "boxes" pour, par la suite, créer les environnements.  
Les commandes suivantes permettent de manipuler les boxes : 

- ajouter une box : <code>vagrant box add</code> 

- supprimer une box : <code>vagrant box remove</code> 

- lister les box existantes : <code>vagrant box list</code>

- Redémarrer une box : <code>vagrant halt</code>

- transformer une box en package : <code>vagrant package</code>

- Destruction d'une box : <code>vagrant destroy</code>

- Effectue un snapshot de la box et la suspend ensuite : <code>vagrant suspend</code>

- Redémarre une box suspendue : <code>vagrant resume</code>

- Affiche le statut d'une box : <code>vagrant status</code>

- Démarre le provisionnement d'une box : <code>vagrant provision</code>

Une fois la box créée à l'aide de la commande **vagrant box add**, il faut taper la commande **vagrant init** pour générer le **vagrantfile**.  
Le **Vagrantfile** permet de configurer l'environnement sur lequel vous allez travailler.

Le **Vagrantfile** se présente ainsi :  
<code>Vagrant::Config.run do |config|</code>  
<code>config.vm.box = "base"</code>  
<code># config.vm.boot_mode = :gui</code>  
<code>end</code>

Il faudra remplacer la ligne **config.vm.box = "base"** Par la box qui a été installée.

Par exemple, si vous avez entré la commande suivante :  
<code>vagrant box add hashicorp/precise32</code>  
Il faudra préciser **hashicorp/precise32** après **config.vm.box =**.

Une fois ces étapes effectuées, la commande <code>vagrant up</code> lancer la création de la machine virtuelle.

Une fois la machine virtuelle prête, vous pourrez vous y connecter à l'aide de la commande <code>vagrant ssh</code>.

Important : Si vous rencontrez une erreur lors de la création de la VM, nous vous suggérons de dé-commenter la ligne **config.vm.boot_mode = :gui**.

# 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder le fonctionnement de Vagrant en nous concentrant sur divers aspect tels que les dossiers partagés, le réseau et la configuration des images.  
Dans ce document, nous avons expliqué comment installer Vagrant ainsi que les commandes de base liées à la manipulation des box.  
Dans le prochain, nous aborderons les différents modes de provisionnement.

