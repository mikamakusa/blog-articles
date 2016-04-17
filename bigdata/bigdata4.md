+++
date = "2015-10-05T15:39:49+02:00"
draft = false
title = "Hadoop - Cluster | High Availability & Auto Failover"

+++

# 1 - Qu'est-ce que Hadoop ? #
Hadoop est un framework destiné à faciliter la création d'applications distribuées et échelonnables permettant aux applications de travailler avec de nombreux noeuds et des données de grandes tailles.  
Dans le [premier document](http://localhost:1313/bigdata/hadoop1/), nous avions étudié un cas de cluster Hadoop ne comprenant que deux noeuds, un maître et un esclave.  
Dans celui-ci, nous allons nous concentrer sur un cluster disposant des fonctions "Haute disponibilité" et "Basculement de charge automatique", grâce à l'utilisation de zookeeper.  

# 2 - Contexte initial #
Nous avons déployé les serveurs suivants, installé Java et Hadoop sur chacun d'eux :
  
- **mida-clmaster1** : <u>192.168.1.1</u> - CentOS 7  
- **mida-clmaster2** : <u>192.168.1.2</u> - CentOS 7  
- **mida-clslave1** : <u>192.168.1.50</u> - CentOS 7

# 3 - Situation finale #
Dans ce cas, nous devrons installer zookeeper sur chacun des serveurs maîtres, en plus de Hadoop.  

- Les noeuds maîtres disposeront des rôles suivants relatif à Hadoop :  
<u>JournalNode</u>  
<u>Namenode</u>   
<u>DFSZKFailover Controller</u>  
Et suite à l'installation de Zookeeper, ces mêmes serveurs seront soit "principal" soit "en attente" et bénéficieront du rôle "QuorumPeerMain".

- Les noeuds esclaves disposeront du rôle suivant :  
<u>Datanode</u>

# 4 - Comment installer et configurer Hadoop dans cet environnement. #
## Définition du fichier /etc/hosts
Pour que les fichiers de configuration de Hadoop puissent reconnaitre quelles machines sont les esclaves et quels sont les maîtres, il faut renseigner ce fichier :
   
	192.168.1.1		mida-clmaster1  
	192.168.1.2		mida-clmaster2  
	192.168.1.50	mida-clslave1  

## Génération de clé ssh et copie de la clé sur les noeuds esclaves
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@mida-clmaster2 && ssh-copy-id -i ~/.ssh/id_rsa.pub root@mida-clslave1`

Pour connexion ssh sans mot de passe sur le noeud maître :  
`cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`

## Installation et configuration de Java
Java peut s'installer de deux manières différentes : 

- Via les paquets :  
Debian/Ubuntu : `apt-get install -y default-jdk default-jre wget`  
CentOS : `yum install -y java-1.7.0-openjdk java-1.7.0-openjdk-devel wget`  
(Pour vérifier la version installée : `java -version`)

- Via l'archive  
La téléchager :  
`wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.tar.gz"`  
Décompresser l'archive et renommer le dossier :  
`tar xzf jdk-8u60-linux-x64.tar.gz`  
`mv jdk1.8.0_60 java`  
Pour configurer l'environnement afin qu'il prene en compte l'installation de java, <u>tapez les commandes suivantes dans le terminal</u> :  
`alternatives --install [/Chemin/vers/java] java [/Chemin/vers/java]/bin/java 1`  
`alternatives --install [/Chemin/vers/java]/bin/jar jar [/Chemin/vers/java]/bin/jar 1`  
`alternatives --install [/Chemin/vers/java]/bin/javac javac [/Chemin/vers/java]/bin/javac 1`  
`alternatives --set jar [/Chemin/vers/java]/bin/jar`  
`alternatives --set javac [/Chemin/vers/java]/bin/javac`  
Puis ajouter les lignes suivantes dans le fichier .bashrc
  
		export JAVA_HOME=[/Chemin/vers/java]  
		export JRE_HOME=[/Chemin/vers/java]/jre  
		export PATH=$PATH:[/Chemin/vers/java]/bin:[/Chemin/vers/java]/jre/bin
  
Et lancer la commande suivante : `exec bash`

Nous allons ensuite installer et configurer Zookeeper, c'est cet outil qui permettra d'effectuer le basculement automatique de charge entre les serveurs.

## Installation et configuration de Zookeeper

- Télécharger et décompresser l'archive et renommer le dossier :  
`wget http://mirror.cc.columbia.edu/pub/software/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz`  
`tar xzf zookeeper-3.4.6.tar.gz`  
`mv zookeeper-3.4.6 zookeeper`

- Pour configurer l'environnement afin qu'il prenne en compte l'installation de Zookeeper, <u>tapez les lignes suivantes dans le fichier .bashrc</u> : 
 
		export ZOOKEEPER_HOME=[/chemin/vers/zookeeper]  
		export PATH=$PATH:[chemin/vers/zookeeper]/bin  
Puis lancez la commande suivante : `exec bash`
 
- Configurer Zookeeper  
Pour configurer Zookeeper, il faut commencer par lancer la commande permettant d'inquer l'ID du serveur :   
`echo [n° du serveur] > [chemin/vers/zookeeper/DataDir]/myid`  
Ensuite, dans le fichier zoo.conf se trouvant dans /etc/zookeeper/conf, il faut ajouter les serveurs ainsi :   
`server.[n°]=[Adresse IP ou nom FQDN du serveur]:2888:3888`  
Enfin, démarrez zookeeper via la commande suivante :  
`zkServer.sh start`  
`zkServer.sh status` permet de savoir si zookeeper est bien actif et quel rôle a été attribué au serveur : "Leader" ou "Follower".

## Installation et configuration de Hadoop

- Télécharger l'archive, la décompresser et renommer le dossier :  
`wget http://mirrors.ircam.fr/pub/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz`  
`tar xzf hadoop-2.7.1.tar.gz`  
`mv hadoop-2.7.1 hadoop`

- Configurer l'environnement en ajoutant les lignes suivantes dans le fichier .bashrc :
   
		export HADOOP_HOME=[/Chemin/vers/hadoop]   
		export HADOOP_INSTALL=$HADOOP_HOME   
		export HADOOP_MAPRED_HOME=$HADOOP_HOME   
		export HADOOP_COMMON_HOME=$HADOOP_HOME   
		export HADOOP_HDFS_HOME=$HADOOP_HOME   
		export YARN_HOME=$HADOOP_HOME  
		export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native   
		export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin  
Et lancer la commande suivante : `exec bash`

- Définir le nom du service dans le fichier **$HADOOP_HOME/etc/hadoop/core-site.xml** à l'aide de la propriété "fs.defaultFS" :
  
		<property>  
			<name>fs.defaultFS</name>  
			<value>hdfs://[nom du service]/</value>  
		</property>

- Dans le fichier **$HADOOP_HOME/etc/hadoop/hdfs-site.xml**, il faudra définir les éléments suivants :  

	- Le mode "**Haute Disponibilité**" réutilise les identifiants "**NameService**" afin d'identifier une instance HDFS unique qui peut être composée de plusieurs serveurs Namenodes.  
<u>**Très important**</u> : Seuls deux namenodes sont possible même si trois serveurs maîtres sont configurés dans zookeeper.  
les propriétés à inclure sont les suivantes :

			<property>
				<name>dfs.nameservices</name>
				<value>[nom du cluster]</value>
			</property>
			<property>
				<name>dfs.ha.namenodes.[nom du cluster]</name>
				<value>[alias namenode #1],[alias namenode #2]</value>
			</property>
			<property>
				<name>dfs.namenode.rpc-address.[nom du cluster].[alias namenode #1]</name>
				<value>[alias du serveur maître 1]:8020</value>
			</property>
			<property>
				<name>dfs.namenode.rpc-address.[nom du cluster].[alias namenode #2]</name>
				<value>[alias du serveur maître 2]:8020</value>
			</property>
			<property>
				<name>dfs.namenode.http-address.[nom du cluster].[alias namenode #1]</name>
				<value>[alias du serveur maître 1]:50070</value>
			</property>
			<property>
				<name>dfs.namenode.http-address.[nom du cluster].[alias namenode #2]</name>
				<value>[alias du serveur maître 2]:50070</value>
			</property>

	- Hadoop dispose de deux types de mécanismes fournissant un stockage de fichiers modifiés:   
	**Quorum Journal Manager**  
	En ajoutant les propriétés suivantes dans le fichier $HADOOP_HOME/etc/hadoop/hdfs-site.xml :   
	`dfs.namenode.shared.edits.dir` : Doit contenir les noms FQDN tels qu'enregistrés dans le fichier /etc/hosts ainsi qu'un port bien défini (à ouvrir dans le pare-feu).  
	`dfs.journalnode.edits.dir` : Doit contenir le chemin vers un dossier, **ne surtout pas oublier de créer les dossiers**.

			<property>
				<name>dfs.namenode.shared.edits.dir</name>
				<value>qjournal://mida-clmaster1:8485;mida-clmaster2:8485;mida-clmaster3:8485/auto-ha</value>
			</property>
			<property>
				<name>dfs.journalnode.edits.dir</name>
				<value>[Chemin vers le dossier "shared edits"]</value>
			</property>


	**Conventional Shared Storage**  
	Ce mode ne nécessite que la propriété suivante :   
	`dfs.journalnode.edits.dir` : Doit contenir le chemin vers un dossier, ne surtout pas oublier de créer les dossiers.

		<property>
			<name>dfs.journalnode.edits.dir</name>
			<value>[Chemin vers le dossier "shared edits"]</value>
		</property>

- Si le mode "**Haute Disponibilité**" est configuré, Le client Hadoop se connectera au Namenode actif. Pendant le basculement de charge, l'état "en attente" change pour l'état "actif". Pour cela, il faudra ajouter la propriété suivante :
 
		<property>  
			<name>dfs.client.failover.proxy.provider.testHadoop</name>
			<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>  
		</property>

- Avec le mode "**Haute Disponibilité**" configuré, lorsque le Namenode actif devient indisponible, le Namenode en attente prend le relai. Si le premier Namenode revient sur le réseau, il pourrait ne pas être au courant que l'autre Namenode à pris le relai. Afin de palier à cette éventualité, Hadoop fournit un mécanisme de fencing permettant d'éviter un scénario contenant deux namenodes actifs. Pour cela, il faudra ajouter les options suivantes :  
	`dfs.ha.fencing.methods` : Cette propriété fourni un mécanisme à ssh permettant d'activer le namenode et d'arrêter le processus en cours.  
	Valeur : sshfence
	`dfs.ha.fencing.ssh.private-key-files` : Doit contenir le chemin de la clé publique.

		<property>
			<name>dfs.ha.fencing.methods</name>
			<value>sshfence</value>
		</property>
		<property>
			<name>dfs.ha.fencing.ssh.private-key-files</name>
			<value>[chemin/vers/clé/ssh]</value>
		</property>

- Afin de fournir une solution de sauvegarde à chaud pendant le basculement, Hadoop dispose d'un mécanisme de basculement automatique. Pour cela, il faudra ajouter les propriétés suivantes : 
	`dfs.ha.automatic-failover.enabled.[nom du service]` : True/false  
	`ha.zookeeper.quorum` : il faudra lister les noms FQDN de chaques noeuds configurés dans le fichier zoo.conf ainsi que le port défini dans ce même fichier.  

		<property>
  			<name>dfs.ha.automatic-failover.enabled.auto-ha</name>
  			<value>true</value>
 			</property>
 		<property>
   			<name>ha.zookeeper.quorum</name>
   			<value>[alias du serveur maître 1]:2181,[alias du serveur maître 2]:2181</value>
 		</property>

## Ouverture des ports du pare-feu
Il ne faut surtout pas oublier d'ouvrir les ports suivants :
 
- Pour **Zookeeper** : 2181, 3888, 2888
- Pour **Hadoop** : 8485, 8019, 8020, 50070, 8480, 39502

## Démarrage des services
Une fois les fichiers "core-site.xml" et "hdfs-site.xml" rempli, nous allons pouvoir démarrer les services nécessaires :  

1. Dans le cas d'une utilisation du mécanisme "Quorum Journal Manager" :   
`hadoop-daemon.sh start journalnode`  

2. Formattage et démarrage du Namenode actif   
`hdfs namenode -format`
`hadoop-daemon.sh start namenode`

3. formattage et démarrage du Namenode passif  
`hdfs namenode -bootstrapStandby`  
`hadoop-daemon.sh start namenode`

Une fois ces commandes lancées, les deux namenodes sont actif et pour passer le second en passif, lancer la commande suivante sur le namenode actif :   
`hdfs haadmin -transitionToActive --forceactive [namenode n°X]` 
 
<u>Le n° du namenode a démarrer en tant qu'actif est a déterminer en fonction du résultat de la commande zkServer.sh status.</u>

4. Démarrer le mode "Haute Disponibilité" de Zookeeper  
`hdfs zkfc -formatZK`

5. Démarrer les services "zookeeper failover controller"  
`hadoop-daemon.sh start zkfc`

6. Démarrer les datanodes  
`hadoop-daemons.sh start datanode`

# 5 - Conclusion #
Dans ce document, nous avons pu observer en détail l'installation et la configuration d'un cluster Hadoop à l'aide de zookeeper afin d'assurer une continuité de service et un basculement de charge automatique si l'un des deux serveur namenode venait à tomber en panne.  
Dans un prochain document, nous nous concentrerons sur l'installation de HBase sur ce cluster.
