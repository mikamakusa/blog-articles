+++
date = "2015-10-27T18:21:49+02:00"
draft = false
title = "Le Labo #2 | Cluster Hadoop multinodes Dockerisé"

+++

# 1 - Introduction   
Après avoir présenté les fonctionnalités de Docker et Weave, nous allons vous présenter un cas pratique de création de Cluster Hadoop dockerisé.

# 2 - Avant d'aller plus loin...  
Avant toutes choses, nous avons besoin d'un serveur linux 64bits (CentOS ou Ubuntu), à jour (Kernel 3.8 iédalement).

# 3 - Configuration de la machine hôte
## Installation des outils nécessaires
### Installation et configuration de Docker
Dans le premier document relatif à Docker, nous avions observé l'installation de ce logiciel.  
Si vous souhaitez installer la toute dernière version, lancez la commande suivante :  
`wget -qO http://get.docker.com/ | sh`

### Installation de Weave
Weave ne fait pas partie des dépôts des distributions, pour l'installer il faudra taper les commandes suivantes :  
`curl -L git.io/weave -o /usr/local/bin/weave`  
`chmod a+x /usr/local/bin/weave`

## Déroulement de ce cas pratique
### Lancement des conteneurs Docker
Pour ce cas pratique, nous allons avoir besoin de Hadoop.  
Afin de pouvoir monter un cluster Hadoop, nous allons avoir besoin d'une image Hadoop Docker "namenode" et d'une autre en tant que "datanode".

- Création du conteneur **namenode** :  
`docker run -e WEAVE_CIDR=10.2.1.1/24 --name namenode -d -P -p 50011:50010 -p 50021:50020 -p 50071:50070 -p 50076:50075 -p 50091:50090 -p 19889:19888 -p 7030:8030 -p 7031:8031 -p 7032:8032 -p 7033:8033 -p 7040:8040 -p 7042:8042 -p 7088:8088 -p 49607:49707 -it rastasheep/ubuntu-sshd:14.04`

- Création du conteneur **datanode** :  
`docker run -e WEAVE_CIDR=10.2.1.2/24 --name datanode -d -P -p 50012:50010 -p 50022:50020 -p 50072:50070 -p 50077:50075 -p 50092:50090 -p 19890:19888 -p 9030:8030 -p 9031:8031 -p 9032:8032 -p 9033:8033 -p 9040:8040 -p 9042:8042 -p 9088:8088 -p 49807:49707 -it rastasheep/ubuntu-sshd:14.04` 

- Affichage des conteneurs créés : 
`docker ps -a`

		CONTAINER ID        IMAGE                          COMMAND                  CREATED              STATUS                    PORTS                                                                                                                                                                                                                                                                                                                                                                                 NAMES
		7e2be7188055        rastasheep/ubuntu-sshd:14.04   "/w/w /usr/sbin/sshd "   About a minute ago   Up 59 seconds             0.0.0.0:32820->22/tcp, 0.0.0.0:9030->8030/tcp, 0.0.0.0:9031->8031/tcp, 0.0.0.0:9032->8032/tcp, 0.0.0.0:9033->8033/tcp, 0.0.0.0:9040->8040/tcp, 0.0.0.0:9042->8042/tcp, 0.0.0.0:9088->8088/tcp, 0.0.0.0:19890->19888/tcp, 0.0.0.0:49807->49707/tcp, 0.0.0.0:50012->50010/tcp, 0.0.0.0:50022->50020/tcp, 0.0.0.0:50072->50070/tcp, 0.0.0.0:50077->50075/tcp, 0.0.0.0:50092->50090/tcp   datanode
		a517e09e71a9        rastasheep/ubuntu-sshd:14.04   "/w/w /usr/sbin/sshd "   About a minute ago   Up About a minute         0.0.0.0:32819->22/tcp, 0.0.0.0:7030->8030/tcp, 0.0.0.0:7031->8031/tcp, 0.0.0.0:7032->8032/tcp, 0.0.0.0:7033->8033/tcp, 0.0.0.0:7040->8040/tcp, 0.0.0.0:7042->8042/tcp, 0.0.0.0:7088->8088/tcp, 0.0.0.0:19889->19888/tcp, 0.0.0.0:49607->49707/tcp, 0.0.0.0:50011->50010/tcp, 0.0.0.0:50021->50020/tcp, 0.0.0.0:50071->50070/tcp, 0.0.0.0:50076->50075/tcp, 0.0.0.0:50091->50090/tcp   namenode

Cette commande nous permet de connaître les conteneurs créés, leur ID, leur nom et les ports ouvert sur l'hôte rattachés à ceux des conteneurs.

Maintenant, nous allons pouvoir nous connecter en SSH sur les conteneurs afin d'installer et configurer Java et Hadoop. 

### Configuration des conteneurs
Puisque nous partons d'un conteneur vide, nous allons devoir se connecter en SSH puis y installer Java et Hadoop.

- Connexion en SSH à **namenode** : `ssh localhost -p 32819`  
le mot de passe root pour ce conteneur est : root

- installation des outils nécessaires : wget, curl, tar, vim  
`apt-get install -y wget curl tar vim`

- Modification du fichier **hosts** : 
Le fichier **hosts** de chaque conteneur se présente ainsi : 

		# created by Weave - BEGIN
		# container hostname
		[Adresse IP]        [nom du conteneur].weave.local [nom du conteneur]

		# static names added with --add-host
		
		# default localhost entries
		127.0.0.1       localhost
		::1             ip6-localhost ip6-loopback
		fe00::0         ip6-localnet
		ff00::0         ip6-mcastprefix
		ff02::1         ip6-allnodes
		ff02::2         ip6-allrouters
		# created by Weave - END
Il faut ajouter, sous la ligne **--add-host** une entrée faisant référence à l'autre conteneur, telle que : 

		10.2.1.1        namenode.weave.local namenode

- Création d'une clé SSH afin que le **namenode** puisse se connecter au **datanode** sans mot de passe.  
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@datanode`  
`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`

### Installation de Java
Sur les deux conteneurs, nous allons installer Java à l'aide de la commande suivante :  
`apt-get install -y default-jre default-jdk`

Une fois tous les paquets installé, tapez la commande `java -version` afin de vérifier que l'installation se soit bien passée.

### Installation de Hadoop
Une fois Java installé et configuré, il faut :  

- <u>télécharger l'archive</u> Hadoop à l'aide de la commande suivante :  
`wget http://mirrors.ircam.fr/pub/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz`

- <u>La décompresser</u> : `tar xzf hadoop-2.7.1.tar.gz`  
- <u>Renommer le dossier</u> : `mv hadoop-2.7.1 hadoop`

Pour configurer l'environnement afin que l'installation de Hadoop soit prise en compte par le système, <u>entrez les lignes suivantes dans le fichier .bashrc</u> :
  
	export HADOOP_HOME=[/Chemin/vers/hadoop]   
	export HADOOP_INSTALL=$HADOOP_HOME  
	export HADOOP_MAPRED_HOME=$HADOOP_HOME   
	export HADOOP_COMMON_HOME=$HADOOP_HOME  
	export HADOOP_HDFS_HOME=$HADOOP_HOME  
	export YARN_HOME=$HADOOP_HOME  
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native  
	export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
	  
Et tapez `exec bash`

### Configuration de Hadoop - Namenode et Datanode
Nous allons expliquer la configuration minimale de Hadoop via les six fichiers principaux à remplir :

- **$HADOOP_HOME/etc/hadoop/core-site.xml**

		<property>  
			<name>fs.default.name</name>   
			<value>hdfs://[IP ou alias du noeud maître]:9000/</value>   
		</property>   
		<property>  
			<name>dfs.permissions</name>   
			<value>false</value>   
		</property>

- **$HADOOP_HOME/etc/hadoop/hdfs-site.xml**

		<property>  
			<name>dfs.data.dir</name>   
			<value>[chemin/vers/]/datanode</value>   
			<final>true</final>  
		</property>  
		<property>   
			<name>dfs.name.dir</name>  
			<value>[chemin/vers/]/namenode</value>   
			<final>true</final>  
		</property>   
		<property>   
			<name>dfs.replication</name>   
			<value>1</value>  
		</property>

- **$HADOOP_HOME/etc/hadoop/mapred-site.xml**
<u>**Avant de pouvoir éditer le fichier mapred-site.xml, il faut copier le fichier mapred-site.xml.template vers mapred-site.xml.**</u>

		<property>  
			<name>mapred.job.tracker</name>  
			<value>[IP ou alias du noeud maître]:9001</value>  
		</property>

- **$HADOOP_HOME/etc/hadoop/yarn-site.xml**

		<property>  
			<name>yarn.nodemanager.aux-services</name>  
			<value>mapreduce_shuffle</value>  
		</property>  
		<property>  
			<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>  
			<value>org.apache.hadoop.mapred.ShuffleHandler</value>  
		</property>  
		<property>  
			<name>yarn.resourcemanager.resource-tracker.address</name>  
			<value>[IP ou alias du noeud maître]:8025</value>  
		</property>  
		<property>  
			<name>yarn.resourcemanager.scheduler.address</name>  
			<value>[IP ou alias du noeud maître]:8030</value>  
		</property>  
		<property>  
			<name>yarn.resourcemanager.address</name>  
			<value>[IP ou alias du noeud maître]:8040</value>  
		</property>

- **$HADOOP_HOME/etc/hadoop/hadoop-env.sh**  
il faudra modifier {$JAVA_HOME} par le chemin exact de java.

### Configuration de Hadoop - Namenode uniquement
- **$HADOOP_HOME/etc/hadoop/slaves**  
Il faut remplacer la valeur par défaut par l'alias du datanode.

### Configuration de Hadoop - Datanode uniquement
- **$HADOOP_HOME/etc/hadoop/slaves**  
Il faut laisser la valeur **localhost** par défaut.

### Démarrer les services
- Formatage en hdfs : 
`hadoop namenode -format`

- Démarrage des services **dfs** et **yarn** :  
`start-dfs.sh`  
`start-yarn.sh`  

- Démarrage du service hadoop **datanode** sur datanode : 
`hadoop-daemon.sh start datanode`

# 4 - Conclusion
Nous vous avons présenté un cas de création de Cluster Hadoop multinode Dockerisé faisant appel à plusieurs outils de Docker.

