+++
date = "2016-03-25"
draft = false
title = "Le Labo #10 | Rkt - Installation et premier conteneur"
+++

# Qu'est-ce que Rkt ?  
Ca, c'est la très très bonne question. Rkt, ou Rocket, est un système de conteneur proche de Docker développé par l'équipe derrière CoreOS.  
La création d'un conteneur Rkt peut se faire à l'aide des images disponible sur Docker Hub ou à l'aide des images propres à Rkt.  
Rkt peut s'installer partout, il suffit juste de télécharger l'archive et de la décompresser.  
Par contre, et c'est là que ça devient intéressant, Rkt semble être plus sécurisé que Docker par un principe d'isolation plus poussé et de séparation des privilèges entre les utilisateurs de l'hôtes et de ceux des conteneurs.  

# Comment on l'installe ?
Comme expliqué précédemment, Rkt est une simple archive téléchargeable et peut être installé sur n'importe quelle distribution.  
Voici la procédure :  

- Télécharger l'archive : wget https://github.com/coreos/rkt/releases/download/v0.9.0/rkt-v0.9.0.tar.gz 
- Décompresser l'archive : tar xzf rkt-v0.9.0.tar.gz

Une fois ces étapes suivies, rkt est déjà prêt à l'emploi.

# Comment créer un conteneur ?
Pour ce petit use case, nous allons avoir besoin des éléments suivants : 

- Une image Docker Ubuntu  
- Un driver réseau (Weave, par exemple)  
- Un hostname  
- un volume partagé entre l'hôte et le conteneur rkt  

La commande sera la suivante : 

	rkt run --insecure-option=image docker://ubuntu --net=weave --hostname=c_rkt_test --volume data,kind=host,source=/srv/data,readOnly=false

Pour afficher la liste des conteneurs créés à l'aide de rkt, on lance la commande suivante : 

	rkt list

	UUID        APP     		IMAGE NAME               STATE      CREATED        STARTED         NETWORKS
	1fg5r00r	c_rkt_test		Ubuntu 					 running    2 minutes ago  41 seconds ago  weave:ip4=10.0.1.10

Pour supprimer le conteneur ainsi créé, il faudra taper la commande suivante : 

	rkt gc --grace-preiod=0

Dès que j'ai plus de temps, je publierai un article plus conséquent sur le sujet.
Mais, ce n'est qu'une entrée en matière.
