+++
date = "2015-09-25T11:37:49+02:00"
draft = false
title = "Hadoop - Cluster | High Availability & Manual Failover"

+++

# 1 - Qu'est-ce que Hadoop ? #
Hadoop est un framework destiné à faciliter la création d'applications distribuées et échelonnables permettant aux applications de travailler avec de nombreux noeuds et des données de grandes tailles.  
Dans le [précédent document](http://localhost:1313/bigdata/hadoop1/), nous avions étudié un cas de cluster Hadoop ne comprenant que deux noeuds, un maître et un esclave.  
Dans celui-ci, nous allons nous pencher sur un cluster disposant de plusieurs noeux maîtres.

# 2 - Contexte initial #
Nous avons déployé les serveurs suivants, installé Java et Hadoop sur chacun d'eux :  
- **mida-clmaster1** : <u>192.168.1.1</u> - CentOS 7  
- **mida-clslave1** : <u>192.168.1.50</u> - CentOS 7

# 3 - Situation finale #
Hadoop sera installé de la manière suivante :  
Les noeuds maîtres disposeront des rôles suivants :
  
- <u>Namenode</u>,  
- <u>Secondary Namenode</u>,  
- <u>Resource Manager</u>

Le noeud esclave disposera des rôles suivants :
  
- <u>Datanode</u>  
- <u>Node Manager</u>

# 4 - Comment installer et configurer Hadoop dans cet environnement. #
## Déploiement et configuration (Hadoop et Java) du nouveau serveur.
Dans notre exemple, nous allons avoir besoin de déployer un serveur maître en plus :
  
- **mida-clmaster2** : <u>192.168.1.2</u> - CentOS 7

Nous allons aussi installer Java et Hadoop en suivant les mêmes étapes que lors du tutoriel précédent, à l'aide de la commande scp depuis le serveur mida-clmaster1 : 
### Déclarer le serveur des deux premiers serveurs : 
Ajoutez la ligne suivante dans le fichier **hosts* des deux premiers serveurs :
  
	192.168.1.2 	mida-clmaster2

Dans le fichier **hosts* de ce nouveau serveur, ajoutez les deux premiers serveurs :

	192.168.1.1		mida-clmaster1
	192.168.1.50	mida-clslave1

### Copier la clé SSH du premier serveur maître vers le second à l'aide de la commande "ssh-copy-id".
Lancer la commande suivante depuis le serveur **mida-clmaster1** :  
`ssh-copy-id -i ~/.ssh/id_rsa.pub root@mida-clmaster2`

### Copier les dossiers java et hadoop sur le nouveau serveur : 
Lancer les commandes suivantes depuis le serveur **mida-clmaster1** :  
`scp -r [chemin/vers/java] root@mida-clmaster2:[dossier/vers/java]`
`scp -r [chemin/vers/hadoop] root@mida-clmaster2:[dossier/vers/hadoop]`

### Configurer l'environnement à l'aide de la commande "export", "alternatives --install" et "alternatives --set" . 
Pour configurer l'environnement afin qu'il prene en compte l'installation de java, <u>tapez les commandes suivantes dans le terminal sur le nouveau serveur maître</u> :  
`alternatives --install [/Chemin/vers/java] java [/Chemin/vers/java]/bin/java 1`  
`alternatives --install [/Chemin/vers/java]/bin/jar jar [/Chemin/vers/java]/bin/jar 1`  
`alternatives --install [/Chemin/vers/java]/bin/javac javac [/Chemin/vers/java]/bin/javac 1`  
`alternatives --set jar [/Chemin/vers/java]/bin/jar`  
`alternatives --set javac [/Chemin/vers/java]/bin/javac`  
Puis ajouter les lignes suivantes dans le fichier .bashrc :
 
	export JAVA_HOME=[/Chemin/vers/java]  
	export JRE_HOME=[/Chemin/vers/java]/jre  
	export PATH=$PATH:[/Chemin/vers/java]/bin:[/Chemin/vers/java]/jre/bin  
	export HADOOP_HOME=[/Chemin/vers/hadoop]   
	export HADOOP_INSTALL=$HADOOP_HOME   
	export HADOOP_MAPRED_HOME=$HADOOP_HOME   
	export HADOOP_COMMON_HOME=$HADOOP_HOME   
	export HADOOP_HDFS_HOME=$HADOOP_HOME   
	export YARN_HOME=$HADOOP_HOME  
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native   
	export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

Et lancer la commande suivante : `exec bash`

### Modifier les fichiers de configurations de Hadoop
#### Le fichier core-site.xml
Afin que la configuration "haute disponibilité" de Hadoop soit fonctionnelle, ce fichier doit être le plus dépouillé possible.  
La seule propriété indispensable est la suivante :
 
	<property>  
		<name>fs.default.name</name>   
		<value>hdfs://hdfs://[nom du cluster]/</value>   
	</property>

La propriété <u>dfs.permissions</u> peut être déplacée dans le fichier **hdfs-site.xml**. 

#### Le fichier hdfs-site.xml
Afin que la configuration "haute disponibilité" de Hadoop soit fonctionnelle, il faut ajouter les propriétés suivantes au fichier hdfs-site.xml (en incluant les propriétés indispensables pour un cluster hadoop multinodes). :   
  
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
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
		<value>[Chemin vers le dossier "shared edits"]</value>
	</property>
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	</property>
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>[chemin/vers/clé/ssh]</value>
	</property>

### Configuration du pare-feu
**Ne surtout pas oublier d'ouvrir les ports 8020 et 50070**

### Démarrer les services
Sur le serveur principal : `hadoop namenode -format && hadoop-daemon.sh start namenode`

Sur le serveur secondaire : `hadoop namenode -bootstrapStandby && hadoop-daemon.sh start namenode`

Actuellement, tous les serveurs sont en attente, pour en activer un il faudra lancer la commande suivante : 
`hdfs haadmin -transitionToActive [alias namenode1]`  

Si une erreur survient lors du lancement de la commande, ré-essayez avec l'option --forceactive, voir ci-dessous : 
`hdfs haadmin -transitionToActive --forceactive [alias namenode1]`

# 5 - Conclusion #
Dans ce document, nous avons pu observer en détail l'installation et la configuration d'un cluster Hadoop en mode "haute disponibilité" à basculement manuel à l'aide de trois serveurs comprenant deux noeuds maitres et un esclave.  
Ce type de configuration permet d'éliminer certains points de blocage et d'assurer plus de sécurité (si un des noeuds tombaient).  
Dans le [prochain document](http://localhost:1313/bigdata/hadoop2/), nous observerons comment automatiser le basculement à l'aide de Zookeeper.

