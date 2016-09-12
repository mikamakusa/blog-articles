+++
date = "2016-07-30T15:29:49+02:00"
draft = "false"
title = "Le Labo #18 | Jouons avec Docker et SaltStack (update 12/09/2016)"

+++


# Pour commencer...
Cela fait un mois depuis mon dernier article et je commençais aussi à trouver le temps assez long.  
En fait, au dela de chercher une idée d'article pertinente pour vous, je travaillais d'arrache pieds à trouver des solutions sur les tâches qui m'ont été confiées.  

Tout cela pour dire que mes articles les plus récents m'ont été très utile...et celui-la aura pour objectif d'expliquer comment provisionner des conteneurs **Docker** à l'aide d'un outil de **Config Management** et j'ai décidé d'utiliser **SaltStack** au lieu d'**Ansible** (pour lequel j'avais déjà publier quelques articles).


# Pourquoi Salt ?
En fait, **Salt** et **Ansible** sont très similaire dans la syntaxe des fichiers...ils sont tout les deux basés sur Python.  
Mais **Salt** se décline en deux entités : 

- le maître, appelé salt-master
- l'esclave, appelé salt-minion

Les **minions** publient des clés qu'ils transmettent au **maître** et ce dernier doit les accepter pour pouvoir appliquer les **states** ou les **pillars** (selon la configuration définie à votre niveau). 

Et propose d'utiliser les plugins de **Python** pour la configuration des minions.


# Comment ça s'installe ?
Sur le site officiel du développeur/mainteneur de la solution, il a un guide expliquant comment l'installer.  
J'utilise souvent le mode **bootstrap** pour installer aussi bien le **master** que le **minion**

- Pour le master :  
`curl -o bootstrap_salt.sh -L https://bootstrap.saltstack.com`
`sudo sh bootstrap_salt.sh -M -N git develop`

- Pour le minion :  
`curl -o bootstrap_salt.sh -L https://bootstrap.saltstack.com`
`sudo sh bootstrap_salt.sh git develop`


# Comment les configurer ?
En ce qui concerne la configuration, pour cet article, je ferai dans le plus basique :

- modification du /etc/hosts pour y ajouter l'adresse IP du master (voir ci-dessous)

	xxx.xxx.xxx.xxx 		salt

- Création d'un dossier /srv/salt qui contiendra les *states* sur le **saltmaster**

Une fois ceci effectué, je vais déjà pouvoir lancer les commandes suivantes, celles-ci me permettront par la suite d'intégrer la clé du minion au magasin de clé acceptées par le maître.

- Sur le minion, pour démarrer le service  
`systemctl start salt-minion`  

- Sur le master, pour démarrer le service  
`systemctl start salt-master`

- Sur le master, pour visualiser les clés en attente et les accepter  
`salt-key` et `salt-key -a * -y`


# Les states (update 08/09/2016)
La configuration des deux serveurs **Salt** étant terminée, nous allons pouvoir nous concentrer sur les **states**.  
Je vais en créer  : 

- Python : pour les besoins de ce lab, je vais avoir besoin d'installer Python 3.5 (qui est un prérequis à docker-py).

- Docker et la librairie docker-py (et les dépendances qui vont bien).

## Le State Python
Voici à quoi ressemble ce state :  

	epel-release:
	  pkg.installed:
    	- name: epel-release

	plugin-priorities:
	  pkg.installed:
	    - name: yum-plugin-priorities

	scl-rh:
	  pkg.installed:
	    - name: centos-release-scl-rh

	scl:
	  pkg.installed:
	    - name: centos-release-scl

	scl-utils:
	  pkg.installed:
	    - name: scl-utils

	python:
	  pkg.installed:
	    - name: rh-python35

	python-enable:
	  cmd.run:
	    - name: scl enable rh-python35 bash

	python-pip:
	  pkg.installed:
	    - name: python-pip

## Le State Docker
Voici à quoi il ressemble :

	docker:
	  file.managed:
	    - name: /etc/yum.repos.d/docker.repo
	    - contents:
	      - '[dockerrepo]'
	      - name=Docker Repository
	      - baseurl=https://yum.dockerproject.org/repo/main/centos/7/
	      - enabled=1
	      - gpgcheck=1
	      - gpgkey=https://yum.dockerproject.org/gpg
	  pkg.installed:
	    - pkgs:
	      - iptables
	      - ca-certificates
	      - lxc
	      - docker-engine
	  service.running:
	    - name: docker

	dockerpy-install:
	  pip.installed:
	    - name: docker-py

// **Update 12/09/2016** //

### Ajout au state Docker
Puisque le module **Docker** pour **Salt** fonctionne grâce à l'API, il faut donc ouvrir le port **Docker Remote** afin de pouvoir utiliser **Docker** à distance.
Pour cela, il est recommandé d'ajouter ce qui suit au state relatif à l'installation de Docker : 

    docker-api-open:
      cmd.run:
        - name: dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &

// //


## Le fichier d'orchestration
Le fonctionnement de **Salt** diffère très peu de celui d'**Ansible** et de ses *rôles*, mais pour que le déploiement se fasse sans erreur, il faut un fichier d'orchestration appelé **top.sls** dans lequel on va ajouter les étapes du déploiement.  
Dans le cas qui m'intéresse ici, le fichier contiendra ce qui suit : 

	base:
	  '*':
	    - python
	    - docker
	    - apache

Pour lancer le déploiement, deux voies sont à notre disposition : 

- L'automatique
- La semi automatique

La commande de déploiement *automatique* est la suivante :  
`salt <minion> state.highstate`  
Celle-ci lance les commandes les unes après les autres et affiche le résultat une fois que la toute dernière commande est passée.

La commande de déploiement *semi automatique* est la suivante :  
`salt <minion> state.sls <etape>`  
Avant cette commande; je peux découper le processus de déploiement en étape (correspondante, biensûr, à celles inscrite dans le **top.sls**) et par conséquent, d'éviter de perdre trop de temps à rechercher les éventuelles erreurs.


# Docker et Salt (update 12/09/2016)
Après avoir effectué de nombreux tests de création de conteneurs à l'aide de **Salt**, je suis arrivé à la conclusion suivante :  
**C'est tout sauf une bonne idée**...surtout si on souhaite garder la main sur la création/suppression de conteneurs par la suite.

### Création d'images et de conteneurs à la main
En fait, créer un conteneur via **Salt** n'offre pas le même niveau de contrôle que si le conteneur était créé directement à l'aide des commandes de **Docker**, tout simplement parce que le module *docker-py* (utilisé par **Salt**) ne s'appuie pas sur la dernière version de l'API de Docker.  
A la place, il est recommandé de suivre les étapes suivantes :  

- Builder l'image : `docker build -t <tag> .`  
- créer le conteneur : `docker build -ditP --add-host salt:<ip master> --hostname <hostname> --name <name> <image>:<tag> salt-minion &`  
- Accepter la clé sur le **salt-master**: `salt-key -a "*" -y`

### Création d'images et de conteneurs via Salt
Dans cette optique, j'ai préféré créer plusieurs formules afin de découper les tâches

**docker.build**

	{% set name = salt['pillar.get']('name') %}
	{% set tag = salt['pillar.get']('tag') %}
	{% set path = salt['pillar.get']('path') %}

	build-image:
	  docker.built:
	    - name: '{{name}}'
	    - tag: '{{tag}}'
	    - path: '{{path}}'

**docker.pull**
	
	{% set tag = salt['pillar.get']('tag') %}

	pull-image:
	  docker.pulled:
	    - tag: '{{tag}}'

**docker.run**

	{ % set cnt = salt['pillar.get']('cnt') %}
	{ % set host = salt['pillar.get']('host') %}
	{ % set img = salt['pillar.get']('img') %}

	build-container:
	  docker.installed:
	    - container: '{{cnt}}'
	    - hostname: '{{host}}'
	    - image: '{{img}}'
	    - tty: True
	    - detach: True
	    - mem_limit: !!null
	    - start: True

	cmd.run:
	    - name: docker start '{{cnt}}'


Pour chacun d'entre, la commande incluera les pillar, voir ci-dessous : 
`salt "*" state.sls docker.build pillar="{'name':'test','tag':'latest','path':'/root/'}"`  
`salt "*" state.sls docker.pull pillar="{'tag':'ubuntu:latest'}"`  
`salt "*" state.sls docker.run pillar="{'cnt':'test','host':'test','img':'app:latest'}"`

# Conclusion
Même si les erreurs sont assez parlante, les journaux de logs d'erreurs sont parfois assez indigeste...et plus encore ceux de **Salt**...tout ça pour dire que mon prochain article aura probablement un lien avec la stack **ELK**.