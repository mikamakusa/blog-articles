+++
date = "2015-09-18T13:10:49+02:00"
draft = false
title = "Mesos - Installation et configuration"

+++

# Mesos - installation et configuration#

## 1 - Qu'est-ce que Mesos ? ##
Apache Mesos est un outil Open Source de gestion de cluster.  
Il permet de mettre les ressources en commun tout en isolant les applications/framework à l'aide d'API de gestion de ressources et de planification.  
Les applications/framework possibles sont les suivants : *Marathon*, *Chronos*, *Hadoop*, *Spark*, etc...

## 2 - Comment l'installer ? ##
Il est possible de l'installer depuis les sources disponible sur le site du projet ou depuis les dépots des distrubtions linux.  

- Installation depuis les sources :  
<u>Il faut tout d'abord installer les paquets nécessaire à la compilation.</u>  
*Ubuntu/Debian* :  
`apt-get upgrade -y`
`apt-get install -y build-essential python-dev python-boto libcurl4-nss-dev libsasl2-dev libapr1-* libsvn* maven openjdk-7-jdk autoconf libtool`
	*CentOS* :  
	`yum groupinstall -y 'Development Tools' && yum install -y apache-maven python-devel java-1.7.0-openjdk-devel zlib-devel libcurl-devel openssl-devel cyrus-sasl-devel cyrus-sasl-md5 apr-devel subversion-devel apr-util-devel maven wget`  
	`pip install -U boto`  
<u>Ensuite, téléchargez l'archive et décompressez la :</u>   
		`wget http://www.apache.org/dist/mesos/0.24.0/mesos-0.24.0.tar.gz`
		`tar xzf mesos-0.24.0.tar.gz`  
<u>Enfin lancez la compilation depuis le dossier dans lequel la décompression à été effectuée :</u>   
`cd mesos-0.24.0`
`./configure && make && make check`  
Afin de pouvoir lancer les commandes sans avoir à se déplacer dans le dossier correspondant, vous pouvez insérer ce qui suit dans le fichier .bashrc  
`PATH=$PATH:/bin:/usr/local/bin:/usr/bin:/sbin:/[MESOS_INSTALL]/bin`
puis lancer la commande : `exec bash`

- Installation depuis les dépots :  
<u>Ajout des dépots</u>  
Debian/Ubuntu : `sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF`  
`DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')`  
`CODENAME=$(lsb_release -cs)`  
`echo "deb http://repos.mesosphere.com/${DISTRO} ${CODENAME} main" | sudo tee /etc/apt/sources.list.d/mesosphere.list`  
`sudo apt-get -y update`  
CentOS : `sudo rpm -Uvh http://repos.mesosphere.com/el/[X]/noarch/RPMS/mesosphere-el-repo-[X]-2.noarch.rpm`  
(X représente la version de CentOS utilisée)  
<u>Installation des paquets</u> :   
Debian/Ubuntu : `apt-get -y install mesos python-mesos mesos-devel mesos-java`  
CentOS : `yum install -y python-mesos mesos-devel mesos-java mesos mesosphere-zookeeper`

## 3 - Quelques commandes - Livrées uniquement lors de l'installation depuis les sources ##
Mesos est livré avec deux nombreuses commandes, mais les deux suivantes sont utiles pour créer un cluster :  

- mesos-master.sh - dispose des options suivantes :  
`--advertise_ip=VALUE` | Publie l'adresse IP "VALUE" a atteindre par les noeuds esclaves  
`--advertise_port=VALUE` | Publie le port "VALUE" (lié à --advertise_ip)  
`--quorum=VALUE` | Défini la taille du quorum = Nombre de maîtres/2  
`--work_dir=VALUE` | Où sont stockées les informations persistantes dans le Registre  
`--zk=VALUE` | URL Zookeeper (si installé) de type : zk://hostx:portx/path

- mesos-slave.sh - dispose des options suivantes : 
`--master=VALUE` | Défini l'adresse du maître (--master=localhost:5050) ou un quorum de maîtres (--master=zk://host1:port1,host2:port2,.../path)

### Pour créer le noeud maître :
`mkdir -p [chemin "work_dir"]`  
`./mesos-master.sh --ip=[@IP du maître] --work_dir=[chemin/work_dir]`

### Pour créer le noeud esclave :    
`./mesos-slave.sh --ip=[@IP du maître]:5050`

## 4 - La configuration - Uniquement pour une installation depuis les dépôts. ##
### Configuration des noeuds maîtres  
Il y a deux services à configurer : **Mesos** et **Zookeeper**.
	
Zookeeper est un outil permettant d'effectuer une rotation entre les leaders du cluster si l'un d'eux ne répond plus.  
#### La configuration de Zookeeper s'effectue ainsi : 
##### ID du serveur pour Zookeeper
Debian/Ubuntu :  
Ajoutez le numéro du serveur dans le fichier /etc/zookeeper/conf/myid
			
CentOS 6 :  
Lancez la commande `zookeeper-server-initialize --myid=[N°]`  
(N° représente le numéro du serveur)

CentOS 7 :   
Ajoutez le numéro du serveur dans le fichier /var/lib/zookeeper/myid

##### Adresses des serveurs
Ajoutez les adresses des serveurs dans le fichier /etc/zookeeper/conf/zoo.cfg, de la manière suivante :
  
	server.1=[Adresse IP ou nom FQDN du serveur]:2888:3888  
	server.2=[Adresse IP ou nom FQDN du serveur]:2888:3888  
	server.3=[Adresse IP ou nom FQDN du serveur]:2888:3888  
	server.x=[Adresse IP ou nom FQDN du serveur]:2888:3888  

##### Redémarrez le sevice zookeeper
Debian/Ubuntu : `service zookeeper restart`  
CentOS 6 : `zookeeper-server restart`  
CentOS 7 : `systemctl restart zookeeper`

#### La configuration de Mesos s'effectue ainsi : 
##### Zookeeper
Sur chaque noeuds maître, il faut ajouter la ligne suivante au fichier /etc/mesos/zk :
  
	zk://[Adresse IP ou nom FQDN du serveur 1]:2181,[Adresse IP ou nom FQDN du serveur 2]:2181,[Adresse IP ou nom FQDN du serveur 3]:2181,[Adresse IP ou nom FQDN du serveur x]:2181/mesos

##### Quorum
Sur chaque noeuds maître, il faut modifier le fichier quorum en y insérant le nombre de maîtres divisé par 2. Dans le cas d'un cluster de trois maître, il fauda un quorum égale à un.  
Le fichier à modifier est le suivant : /etc/mesos-master/quorum

##### Redémarrez le service mesos-master
Debian/Ubuntu/CentOS 6 : `service mesos-master restart`  
CentOS 7 : `systemctl restart mesos-master`

#### Configuration des noeuds esclave
Il n'y a que deux fichiers à modifier :  
**Zookeeper**  
Sur chaque noeuds esclave, il faut ajouter la ligne suivante au fichier /etc/mesos/zk :
  
	zk://[Adresse IP ou nom FQDN du serveur 1]:2181,[Adresse IP ou nom FQDN du serveur 2]:2181,[Adresse IP ou nom FQDN du serveur 3]:2181,[Adresse IP ou nom FQDN du serveur x]:2181/mesos

**Hostname**  
Il faut aussi renseigner le nom FQDN de l'esclave dans le fichier /etc/mesos-slave/hostname. 
 
**Redémarrer le service mesos-slave**  
Debian/Ubuntu/CentOS 6 : `service mesos-slave restart`  
CentOS 7 : `systemctl restart mesos-slave`

## 5 - Conclusion ##
Nous avons pu découvrir comment installer et créer un cluster à l'aide de Mesos.  
Néanmoins, un cluster sans framework ou applications installées par dessus n'est pas très utile.  
Dans un prochain document, nous observerons l'installation d'un framework sur les hôtes esclaves du cluster Mesos.

Voici un lien utile : 
Configuration de Mesos : [http://mesos.apache.org/documentation/latest/configuration/](http://mesos.apache.org/documentation/latest/configuration/ "http://mesos.apache.org/documentation/latest/configuration/")

