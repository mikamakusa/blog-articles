+++
date = "2015-09-22T14:29:49+02:00"
draft = false
title = "Hadoop - Cluster multinode"

+++

# 1 - Qu'est-ce que Hadoop ? #
Hadoop est un framework destiné à faciliter la création d'applications distribuées et échelonnables permettant aux applications de travailler avec de nombreux noeuds et des données de grandes tailles.

# 2 - Contexte initial #
Nous avons déployé les serveurs suivants :  
- **clmaster1** : <u>192.168.1.1</u> - CentOS 7  
- **clslave1** : <u>192.168.1.50</u> - CentOS 7

# 3 - Situation finale #
Hadoop sera installé de la manière suivante :  

- Le noeud maître disposera des rôles suivants :   
Namenode,  
Secondary Namenode,  
Resource Manager

- Le noeud esclave disposera du rôle suivant :  
Datanode  
Node Manager

# 4 - Comment installer Hadoop dans cet environnement ? #
#### Génération de clé ssh et copie de la clé sur les noeuds esclaves
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@[alias des noeuds esclaves]`

Pour connexion ssh sans mot de passe sur le noeud maître : 
`cat ~/.ssh/id_rsa.pub » ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`

#### Définition du fichier /etc/hosts
Pour que les fichiers de configuration de Hadoop puissent reconnaitre quelles machines sont les esclaves et quels sont les maîtres, il faut renseigner ce fichier :
  
	192.168.1.1 	clmaster1  
	192.168.1.50 	clslave1  
	  


#### Installation de Java
Java peut s'installer de deux manières différentes :
  
- Via les paquets :  
<u>Debian/Ubuntu</u> : `apt-get install -y default-jdk default-jre wget`  
<u>CentOS</u> : `yum install -y java-1.7.0-openjdk java-1.7.0-openjdk-devel wget`  
(Pour vérifier la version installée : `java -version`)

- Via l'archive  
<u>La téléchager</u> :  
`wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.tar.gz"`
<u>Décompresser l'archive et renommer le dossier</u> :  
`tar xzf jdk-8u60-linux-x64.tar.gz`
`mv jdk1.8.0_60 java`  
Pour configurer l'environnement afin qu'il prene en compte l'installation de java, <u>tapez les commandes suivantes dans le terminal</u> :  
`alternatives --install [/Chemin/vers/java] java [/Chemin/vers/java]/bin/java 1`  
`alternatives --install [/Chemin/vers/java]/bin/jar jar [/Chemin/vers/java]/bin/jar 1`   
`alternatives --install [/Chemin/vers/java]/bin/javac javac [/Chemin/vers/java]/bin/javac 1`   
`alternatives --set jar [/Chemin/vers/java]/bin/jar`   
`alternatives --set javac [/Chemin/vers/java]/bin/javac`
Puis entrez les lignes suivantes dans le fichier .bashrc :  
`export JAVA_HOME=[/Chemin/vers/java]`   
`export JRE_HOME=[/Chemin/vers/java]/jre`   
`export PATH=$PATH:[/Chemin/vers/java]/bin:[/Chemin/vers/java]/jre/bin`
Et tapez `exec bash`

#### Installation de Hadoop
Une fois Java installé et configuré, il faut télécharger l'archive Hadoop à l'aide de la commande suivante :  
`wget http://mirrors.ircam.fr/pub/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz`

<u>La décompresser</u> : `tar xzf hadoop-2.7.1.tar.gz`  
<u>Renommer le dossier</u> : `mv hadoop-2.7.1 hadoop`

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

#### Configuration de Hadoop
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
<u>**Ne surtout pas oublier de créer les dossier namenode et datanode en référence à dfs.data.dir et à dfs.name.dir.**</u>

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

Le fichier **$HADOOP_HOME/etc/hadoop/slaves** sert à déclarer les noeuds esclaves.  
Par défaut, le seul noeud inscrit est "localhost".  
Il faut remplacer la valeur par défaut par les alias des noeuds esclaves.

Le fichier **$HADOOP_HOME/etc/hadoop/masters** est optionnel dans le cas d'un <u>Cluster Hadoop single node</u>, mais dans le cas actuel, il faudra y renseigner l'alias du maître.

Dans le fichier **$HADOOP_HOME/etc/hadoop/hadoop-env.sh**, il faudra modifier {$JAVA_HOME} par le chemin exact de java.

Hadoop est maintenant configuré sur le noeud maître, nous allons passer à l'installation d'Hadoop sur le noeud esclave à l'aide de la commande suivante :  
`scp -r [chemin/vers/hadoop] root@[alias de l'esclave]:[dossier/vers/hadoop]`
		
<u>Sans oublier de configurer l'environnement de la même manière que pour le serveur maîte, en ajoutant les lignes "export" dans le fichier .bashrc (voir **Installation de Java** et **Installation de Hadoop**)</u>
	
#### Démarrage des services 
Une fois Hadoop configuré et copié sur les noeuds esclaves, il faudra lancer les commandes suivantes sur le noeud maître : 

##### Formatage en hdfs
`hadoop namenode -format`

##### Démarrage des services dfs et yarn (démarrage de Namenode, Secondary Namenode et Resource Manager) sur le noeud maître.
`start-dfs.sh`  
`start-yarn.sh`  

##### Démarrage du service hadoop datanode sur le noeud esclave : 
`hadoop-daemon.sh start datanode`

# 5 - Conclusion #
Dans ce document, nous avons pu observer en détail l'installation et la configuration d'un cluster Hadoop à l'aide de deux serveurs comprenant un noeud maitre et un esclave.  
Ce type de configuration est le plus simple, il est possible d'y ajouter de la haute disponibilité à l'aide de Zookeeper en intégrant plusieurs serveurs maîtres, ce que nous observerons dans le prochain document.

