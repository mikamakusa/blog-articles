+++
date = "2015-10-29T18:21:49+02:00"
draft = false
title = "Le Labo #3 | Cluster Zookeeper Dockerisé"

+++

# 1 - Introduction
Dans un précédent tutoriel, nous avions abordé la création d'un cluster Hadoop multinode réalisé à l'aide de deux conteneurs Docker utilisant les fonctionnalités de Weave pour la partie réseau.
Dans ce tutoriel, nous allons créer un cluster Zookeeper à l'aide de plusieurs conteneurs Docker, le tout orchestré par Swarm.

# 2 - Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 2 serveurs linux (CentOS/Ubuntu) 64bits : **mida-clm1 (IP 81.2.240.198)** et **mida-clm2 (IP 81.2.248.101)**
- Docker : Gestion de conteneur
- Swarm : Gestion de cluster de conteneurs Docker
- Weave : Gestion du réseau Docker
- Zookeeper : Gestionnaire de cluster de l'écosystème Hadoop/Mesos

# 3 - Configuration des deux serveurs hôtes
## Installation des outils nécessaires sur les deux serveurs
### Installation de Docker
Dans le premier document relatif à Docker, nous avions observé l'installation de ce logiciel.  
Si vous souhaitez installer la toute dernière version, lancez la commande suivante :  
`curl -sSL https://get.docker.com/ | sh`  

### Installation de Weave
Weave ne fait pas partie des dépôts des distributions, pour l'installer il faudra taper les commandes suivantes :  
`curl -L git.io/weave -o /usr/local/bin/weave`    
`chmod a+x /usr/local/bin/weave`  

### Installation de Swarm
`docker pull swarm:latest`

## Configuration des outils sur les deux serveurs
### Configuration de Docker 
Afin de pouvoir monter notre cluster Swarm et Weave, nous allons devoir ouvrir plusieurs ports d'écoute sur les deux serveurs à l'aide de la commande suivante :  
`docker -H tcp://0.0.0.0:<port> -H unix:///var/run/docker.sock -d`

Il est d'ailleurs possible d'ouvrir plusieurs ports en même temps, pour utiliser plusieurs outils Docker (**Swarm** et **Weave**, ainsi que pour l'utilisation de l'API).

Exemple, sur les deux hôtes :  
`docker -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -d`

Le port 2375 serait utilisé pour le cluster Swarm.  
Le port 4243 serait utilisé pour l'API Docker (Que nous présenterons dans un prochain tutoriel).  
Les ports 53, 6783 et 6784 sont utilisés par Weave.

### Configuration de Weave 
Puisque nous allons créer un cluster Zookeeper dockerisé, nous allons avoir besoin de deux conteneurs (c'est le minimum).  
Grâce à weave, nous allons pouvoir les créer sur deux hôtes différents.  
Dans notre cas, nous allons avoir besoin d'ouvrir des ports spécifique à zookeeper (2181, 3888, 2888 - relatif à une configuration classique de Zookeeper), et pour cela nous allons lancer les commandes suivantes sur les hôtes Docker : 

- Sur **mida-clm1** :  
`weave launch`  
`weave run 10.0.0.1/24 --name namenode1 -d -P -p 32181:2181 -p 33888:3888 -p 22888:2888 -it rastasheep/ubuntu-sshd:14.04`

- Sur **mida-clm2** :  
`weave launch mida-clm1`  
`weave run 10.0.0.2/24 --name namenode2 -d -P -p 32181:2181 -p 33888:3888 -p 22888:2888 -it rastasheep/ubuntu-sshd:14.04`

### Configuration de Swarm 
A cette étape, nous allons monter le cluster Swarm.
Swarm peut être utilisé aussi bien monter un cluster de conteneur ou un cluster d'hôtes Docker. Dans ce tutoriel, nous allons créer un cluster d'Hôtes.
Pour cela, il faudra lancer les commandes suivantes :  

- Pour générer le token :  
`docker run swarm create`  
- Docker ayant déjà été redémarré avec un port d'écoute nécessaire à Swarm, nous allons adjoindre chaque hôte au cluster Swarm à l'aide de la commande suivante :  
`docker run swarm join --addr=[@IP du node]:[port] token://[token]`  
**[@IP du node] se réfère à l'addresse IP publique du node.**

Exemple :  
- Sur **mida-clm1** : `docker run swarm join --addr=81.2.240.198:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`  
- Sur **mida-clm2** : `docker run swarm join --addr=81.2.248.101:2375 token://eff2e1fb0660f82dd4d69ca639d514ca`

Et démarrer le swarm manager sur mida-clm1 : 
`docker run -d -p 22375:2375 swarm manager token://eff2e1fb0660f82dd4d69ca639d514ca`

Une fois les deux hôtes inclus dans le cluster Swarm, nous allons pouvoir configurer les conteneurs.

# 4 - Configuration des conteneurs
Les conteneurs ayant été créés sur deux hôtes différents, il faut qu'il puissent communiquer ensemble afin de pouvoir monter un cluster Zookeeper dessus. Cependant, nous avons utilisé les fonctionnalités réseau de Weave afin qu'ils soient sur le même réseau.
Afin de vérifier que Weave fonctionne bien, nous allons effectuer une requête ICMP de **namenode1** vers **namenode2** :  
`ping -c 4 -q namenode2`

	--- namenode2.weave.local ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/mdev = 0.156/0.889/2.248/0.811 ms

Maintenant que nous savons que tout fonctionne correctement, nous allons pouvoir installer les paquets nécessaires sur les deux conteneurs
## Création d'une clé SSH sur namenode1  
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@namenode2`  
`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`

## Installation des paquets
Installation des outils nécessaires : wget, curl, tar, vim, java à l'aide de la commande suivante :    
`apt-get install -y wget curl tar vim default-jre default-jdk`

Une fois tous les paquets installé, tapez la commande `java -version` afin de vérifier que l'installation se soit bien passée.

## Installation et configuration de Zookeeper
### Installation de Zookeeper
Pour télécharger, décompresser l'archive et renommer le dossier, tapez les commandes suivantes :  
`wget http://mirror.cc.columbia.edu/pub/software/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz`  
`tar xzf zookeeper-3.4.6.tar.gz`  
`mv zookeeper-3.4.6 zookeeper`

Pour configurer l'environnement afin qu'il prenne en compte l'installation de Zookeeper, <u>tapez les lignes suivantes dans le fichier .bashrc</u> : 
 
	export ZOOKEEPER_HOME=[/chemin/vers/zookeeper]  
	export PATH=$PATH:[chemin/vers/zookeeper]/bin

Puis lancez la commande suivante : `exec bash`
 
### Configurer Zookeeper  
Pour configurer Zookeeper, il faut commencer par lancer la commande permettant d'inquer l'ID du serveur :   

- sur **namenode1** :  
		`echo 1 > [chemin/vers/zookeeper/DataDir]/myid`  

- sur **namenode2** :  
		`echo 2 > [chemin/vers/zookeeper/DataDir]/myid`

Ensuite, il faut ajouter les serveurs sur les deux conteneurs en ajoutant les lignes suivantes dans le fichier zoo.conf se trouvant dans /etc/zookeeper/conf:

	server.1=namenode1:2888:3888  
	server.2=namenode2:2888:3888

Enfin, démarrez zookeeper via la commande suivante sur les deux conteneurs :  
`zkServer.sh start`  
`zkServer.sh status` permet de savoir si zookeeper est bien actif et quel rôle a été attribué au serveur : "Leader" ou "Follower".

# 5 - Conclusion
Voici un cas de cluster Zookeeper Dockerisé à l'aide de conteneurs créés sur deux hôtes différents et réliés à l'aide de Weave (pour la partie réseau) et de Swarm (pour la partie cluster).
Il est possible d'aller plus loin au niveau du cluster Swarm en ajoutant les fonctions de réplication et de haute disponibilité, expliquées dans le lien suivant : https://docs.docker.com/swarm/multi-manager-setup/
