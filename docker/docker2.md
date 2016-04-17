+++
date = "2015-08-28T15:37:49+02:00"
draft = false
title = "Docker - Les commandes de base"

+++

# 1 - Introduction
Docker est un logiciel Open Source permettant de gérer des conteneurs.  
Contrairement à la virtualisation classique comprenant un système hôte sur lequel il est possible d'installer tout ce qui le serait sur un système classique, les conteneurs contiennent au maximum, que les applications/bibliothèques.

Dans le précédent tutoriel, nous avions abordé uniquement l'installation de Docker sur les systèmes Linux et Windows. Dans celui-ci, nous allons nous concentrer sur les commandes de base de Docker.

# 2 - les commandes de Docker
Les commandes permettant l'intéraction avec le moteur Docker sont les suivantes :   
	- Recherche d'image :								`docker search [mot clé]`  
	- Téléchargement d'image (depuis le docker Hub) :	`docker pull [mot clé]`  
	- Téléchargement d'image :							`docker push [dossier]/[image]`  
	- Connaître le nombre d'images téléchargées :		`docker info`  
	- Afficher la liste d'images disponibles :			`docker images`  
	- Stopper un conteneur :								`docker stop [containerID]`  
	- Effacer un conteneur :								`docker rm [containerID]`  
	- Redémarrer un conteneur :							`docker restart [containerID]`  
	- Envoyer des signaux à un conteneur :				`docker kill [containerID]`  
	- Afficher les conteneurs actifs :					`docker ps`  
	- Afficher les conteneurs en tâche de fond :		`docker ps -a`  
	- Créer une image docker à partir d'un Dockerfile :	`docker build`

La prochaine commande contient des options importantes que nous détaillerons :  
- Démarrer un conteneur  
`docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]`

Les options sont les suivantes :  
	`-i` 			Garder STDIN ouvert, même si pas attaché  
	`-t` 			Allouer un pseudo-terminal  
	`-p` 			Permet de publier un conteneur, et accessible via une translation de port depuis le serveur.  
	`-d` 			Active le mode daemon. Le conteneur sera actif et en tâche de fond  
	`-e` 			Force Docker a utiliser un driver exec sépcifique  
	`--name` 		Pour identifier un conteneur par un nom au lieu d'un ID  
	`--dns` 		Pour définir un DNS personnalisé  
	`--net=""`	Pour définir le mode réseau entre "Bridge", "none", "container:" et "host"  
	`--add-host`	Modifie le contenu du fichier "hosts" en y ajoutant les valeurs spécifiées  
	`--link`		Permet d'ajouter un lien vers un autre conteneur  
	`-c` 			Permet de créer un partage  
	`-v` 			Permet de lier un un dossier sur le serveur au conteneur

Pour plus d'informations sur les options dispnibles pour la commande "docker run", nous vous invitons à vous rendre ici : [https://docs.docker.com/reference/run/](https://docs.docker.com/reference/run/ "https://docs.docker.com/reference/run/")

Exemples :  
`docker run -i -t ubuntu:latest bash`  
ou  
`docker run -i -t -p 8080:80 ubuntu:latest /bin/bash`  
ou  
`docker run -d -p 2222:22 -e ROOT_PASS="mypass" tutum/ubuntu:trusty`

# 3 - Conclusion
Dans cette série de tutoriels, nous allons aborder le fonctionnement de Docker et de ses outils Compose et Swarm.  
Nous nous sommes concentrés sur l'installation de Docker dans le document précédent, mais celui-ci était orienté sur les commandes de base de Docker.  
Dans le prochain document, nous aborderons la création des Dockerfiles avec un exemple de Dockerfile.
