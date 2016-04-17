+++
date = "2015-09-18T12:32:49+02:00"
draft = false
title = "Docker - les outils Compose et Swarm"

+++

# 1 - Introduction
Docker est un logiciel Open Source permettant de gérer des conteneurs.  
Contrairement à la virtualisation classique comprenant un système hôte sur lequel il est possible d'installer tout ce qui le serait sur un système classique, les conteneurs contiennent au maximum, que les applications/bibliothèques.

Dans le premier tutoriel, nous avions abordé uniquement l'installation de Docker sur les systèmes Linux et Windows.   
Dans le suivant, nous nous sommes concentrés sur les commandes de base de Docker.  
Dans le dernièr, nous avons abordé les Dockerfiles.

Dans ce tutoriel, nous allons observer l'utilisation et le fonctionnement des outils Compose et Swarm.

# 2 - Docker Compose
## 2.1 - Qu'est-ce que Compose ?
Docker Compose est un outil permettant de définir des applications composée de plusieurs conteneurs et de les déployer (ou exécuter) avec des commandes ayant un impact sur le cycle de vie des composantes de ces applications, telles que :  
- Démarrage, arrêt et reconstruction des services  
- Affichage du statut des services  
- Capture d'un flux d'information   
- Execution d'un commande sur un service  

## 2.2 - Comment l'utiliser ?
Pour pouvoir l'utiliser, deux étapes sont nécessaires : 
- Il faut d'abord l'installer à l'aide de la commande suivante : 
`curl -L https://github.com/docker/compose/releases/download/1.4.0/docker-compose-uname -s-uname -m > /usr/local/bin/docker-compose`

Ou via l'outil "pip" :  
`pip install -U docker-compose`

Puis, rendez-le exécutable à l'aide de la commande suivante :  
`chmod +x /usr/local/bin/docker-compose`

- Ensuite, il faut créer le fichier **docker-compose.yml** décrivant chaque conteneur incluant les informations suivantes : image, commande de lancement, volume, ports, etc...Voir l'exemple ci-dessous : 
			
			redis:
			  image: redis
			  command: redis-server --appendonly yes
			  volumes:
			  - /root/rss2twitter/data:/data
			  ports:
			  - "6379:6379"

			rss2twitter:
			  build: app
			  links:
			  - redis

Docker-compose dispose lui aussi de commande bien distincte de Docker, elles sont listées ci-dessous : 

- build : Construit ou reconstruit les services  
`docker-compose build`
- help : Affiche l'aide de compose		
- kill : Force l'arrêt des conteneurs  
`docker-compose kill -s SIGINT`
- logs : Affiche les logs
- port : Affiche les ports publics liés aux ports du conteneur
- ps : Affiche la liste des conteneurs
- pull : Télécharge une image
- restart : Redémarre un service
- rm : Supprimer un conteneur
- run : Exécute une commande sur un service  
`docker-compose run web python manage.py shell`
- scale	: Défini un nombre de conteneur pour un service  
`docker-compose scale web=2 worker=3`
- start	: Démarrer les conteneurs existant
- stop : Stoppe les conteneurs existant
- up : Lance les conteneurs (l'option "-d" permet de démarrer le service en tâche de fond)
- links : Liste les liens entre chaque conteneurs
- external_links : Liste les liens entre les conteneurs et des applications/serveurs externe
			
# 3 - Docker Swarm
## 3.1 - Qu'est-ce que Docker Swarm ?
Docker Swarm est un outil permettant de gérer un cluster de conteneurs Docker afin de créer des pools de serveurs hôtes exécutant Docker et de les exposer comme un seul hôte Docker virtuel, puis de configurer des containers Docker en gérant automatiquement la répartition de la charge et la gestion de l'état du cluster.

## 3.2 - Comment l'utiliser ?
Pour créer un cluster swarm, il faut commencer par obtenir l’image officielle proposée dans le référentiel Docker Hub.  
`docker pull swarm:latest`
		 
Ensuite, créez un conteneur swarm pour générer le token qui servira, par la suite, à créer le cluster entier (avec le maître et les clients swarm)
`docker run swarm create`

Une fois que nous avons généré le token du cluster swarm, nous allons redémarrer docker en ouvrant le port qui servira à ajouter les noeuds au cluster :  
Ubuntu/Debian/CentOS (jusqu'à 6.x) :  
`service docker.io stop && docker -H tcp://0.0.0.0:<port> -H unix:///var/run/docker.sock -d`

CentOS 7 :  
`systemctl stop docker.io && docker -H tcp://0.0.0.0:<port> -H unix:///var/run/docker.sock -d`

Puis, toujours depuis la machine maître, nous allons démarrer l'agent Swarm sur chacun de nos nodes et leur intégration dans le cluster en reprenant le token obtenu précédemment via la commande :   
`docker run swarm join --addr=[@IP du node]:[port] token://[token]`

exemple :  
`docker run -d swarm join --addr=81.2.240.113:2375 token://eff2e1fb0660f82dd4d69ca639b514ca`

La commande suivante permet de lister les nodes du cluster :   
`docker run swarm list token://[token]`

Exemple :  
`docker run --rm swarm list token://eff2e1fb0660f82dd4d69ca639b514ca`

	81.2.240.198:2375
	81.2.240.113:2375


# 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder le fonctionnement de Docker et de ses outils Machine, Compose, Swarm et Weave.  
Nous nous sommes concentrés sur l'installation de Docker et sur les commandes de base de Docker ainsi que les Dockerfiles dans les précédents documents, celui-ci était orienté sur les outils Compose et Swarm.  
Dans le prochain document, nous aborderons un cas pratique : [le déploiement automatisé de paquets sur un conteneur Docker à l'aide d'Ansible](http://localhost:1313/docker/docker5/).  
Cependant, si vous souhaitez plus d'information sur Docker, nous vous invitons à vous rendre sur le site officiel de Docker : [https://docs.docker.com/](https://docs.docker.com/ "https://docs.docker.com/")

