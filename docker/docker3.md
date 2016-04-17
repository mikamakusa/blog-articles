+++
date = "2015-09-17T16:57:49+02:00"
draft = false
title = "Docker - Les Dockerfiles"

+++
# 1 - Introduction
Docker est un logiciel Open Source permettant de gérer des conteneurs.  
Contrairement à la virtualisation classique comprenant un système hôte sur lequel il est possible d'installer tout ce qui le serait sur un système classique, les conteneurs contiennent au maximum, que les applications/bibliothèques.

Dans le premier tutoriel, nous avions abordé uniquement l'installation de Docker sur les systèmes Linux et Windows.  
Dans le suivant, nous nous sommes concentrés sur les commandes de base de Docker.  
Dans celui-ci, nous allons aborder des notions plus avancées telles que la création de Dockerfiles.

# 2 - Qu'est ce qu'un Dockerfile ?
Les dockerfiles sont des fichiers textes décrivant les différentes étapes de création d'un conteneur totalement personnalisé.
	
L’équivalent d’un Dockerfile pour une application Console affichant l’universel « Hello World » pourrait être le suivant :      

		FROM busybox
		ENTRYPOINT ["echo"]
		CMD ["hello world"]

# 3 - Comment créer un Dockerfile ?
Le dockerfile doit :  
- Utiliser un OS de base  
- Installer les pré-requis système  
- Installer les packages nécessaires

Les commandes que doit contenir un dockerfile sont les suivantes :  
		`ADD` 			Copie un fichier de l'hôte vers le conteneur  
		`CMD` 			Définit les commandes par défaut à exécuter, ou à passer à ENTRYPOINT  
		`ENTRYPOINT` 		Définit le point d'entrée d'une application dans le conteneur  
		`ENV` 			Définit les variables d'environnement (Exemple : "clé = valeur")  
		`EXPOSE` 			Expose un port du conteneur vers l'hôte (fonctionne de la même manière que la commande "docker run -p")  
		`FROM` 			Définit l'image à utiliser  
		`MAINTAINER` 		Définit l'auteur du fichier  
		`RUN` 			Démarre une commande à l'intérieur du conteneur  
		`USER` 			Définit l'utilisateur pouvant exécuter l'image  
		`VOLUME` 			Monte un dossier de l'hôte sur le conteneur  
		`WORKDIR` 		Définit le dossier pour les directives CMD à exécuter  
		`LABEL` 			Ajoute des metadata à une image Docker  
		`COPY`			Copie les fichier de [src] et les ajoute au chemin [dest]  
		`ONBUILD`			Ajoute un déclencheur pour que l'instruction soit exécutée plus tard.

Toutes les commandes à lancer, pour une installation classique de serveur, sont précédées par la commande docker RUN.
Une fois le fichier de configuration créé, il faut lancer la construction du conteneur ainsi que l'exportation vers une nouvelle image à l'aide de la commande suivante :   
`docker build -t [nom de l'image] - < [nom du fichier]`  
Ou sur un dépôt GitHub : `docker build -t [nom de l'image] [dépot GitHub/nom du fichier]`

Exemples de Dockerfile : 

**Dockerfile nginx**  

		# Nginx    
		#  
		# VERSION               0.0.1  
		FROM      ubuntu  
		MAINTAINER ######  
		LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"  
		RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server

**Dockerfile Firefox over VNC**  

		# Firefox over VNC  
		#  
		# VERSION               0.3  
		FROM ubuntu  
		# Install vnc, xvfb in order to create a 'fake' display and firefox  
		RUN apt-get update && apt-get install -y x11vnc xvfb firefox  
		RUN mkdir ~/.vnc  
		# Setup a password  
		RUN x11vnc -storepasswd 1234 ~/.vnc/passwd  
		# Autostart firefox (might not be the best way, but it does the trick)  
		RUN bash -c 'echo "firefox" >> /.bashrc'  
		EXPOSE 5900  
		CMD    ["x11vnc", "-forever", "-usepw", "-create"]

**Dockerfile Cherokee Webserver** 
  
		############################################################  
		# Dockerfile to build Cherokee Webserver Containers  
		# Based on CentOS  
		############################################################  
		# Set the base image to CentOS  
		FROM tutum/centos:latest  
		MAINTAINER Arubacloud.fr  
		# Add the application resources URL  
		RUN yum install wget gcc gcc-c++ libtool bind mysql-server mysql-devel php-mysql gettext git psmisc -y  
		RUN yum groupinstall "Development Tools" -y  
		# Environment  
		RUN mkdir /home/cherokee  
		WORKDIR /home/cherokee  
		# Download & clone Cherokee from git repository  
		RUN cd /home/cherokee && git clone http://github.com/cherokee/webserver.git  
		WORKDIR /home/cherokee/webserver  
		RUN /autogen.sh
		RUN make && make install  
		EXPOSE 22 80 9090  
		RUN cherokee-admin -b
		
	
Une fois le traitement du dockerfile terminé, il faut lancer la commande suivante pour créer le conteneur :   
`docker run --name [nom du conteneur] -i -t [nom de l'image]`  
Ou, si le dockerfile se nomme "Dockerfile" : `docker run --name [nom du conteneur] .`

# 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder le fonctionnement de Docker et de ses outils Compose et Swarm.  
Nous nous sommes concentrés sur l'installation de Docker et sur les commandes de base de Docker dans les précédents documents, celui-ci était orienté sur les Dockerfiles.  
Dans le prochain document, nous aborderons les outils Docker Machine, Compose et Swarm.
