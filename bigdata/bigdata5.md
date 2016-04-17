+++
date = "2015-10-26T15:00:49+02:00"
draft = false
title = "Hbase - Installation de type 'Fully Distributed'"

+++

# 1 - Qu'est-ce que HBase ? #
HBase est un système de gestion de base de données non-relationnelles distribué, écrit en Java, disposant d'un stockage structuré pour les grandes tables.

HBase est un sous-projet d'Hadoop, un framework d'architecture distribuée. La base de données HBase s'installe généralement sur le système de fichiers HDFS d'Hadoop pour faciliter la distribution.

Cet article fait suite à celui traitant d'un cluster Hadoop en mode "Haute Disponibilité et à Basculement Automatique", il reprend les mêmes éléments, à savoir Zookeeper, Java et Hadoop.

# 2 - Avant d'aller plus loin... #
Comme expliqué précédemment, Hbase fait partie de l'écosystème hadoop et à donc besoin d'être installé sur un cluster Hadoop fonctionnel.  
Dans le mode **fully distributed**, nous aurons besoin des trois serveurs suivants :  

- 2 serveurs Linux avec Zookeeper et Hadoop installés en mode HA/AF : les namenodes
- 1 serveur Linux avec Hadoop installé : le datanode

Ce sont les serveurs que nous avons configurés à l'aide du [tutoriel précédent](http://localhost:1313/bigdata/hadoop2/)

# 3 - Comment installer Hbase dans cet environnement ? #

## Télécharger Hbase
Si le tutoriel précédent à bien été suivi, Java, Zookeeper et Hadoop ont été correctement installés.  
Nous allons pouvoir nous concentrés sur Hbase, son installation et sa configuration.

Tapez les commandes suivantes pour télécharger, décompresser et renommer :  

- Télécharger : `wget http://apache.crihan.fr/dist/hbase/1.1.2/hbase-1.1.2-bin.tar.gz`
- Décompresser l'archive : `tar xzf hbase-1.1.2-bin.tar.gz`
- Renommer le dossier : `mv hbase-1.1.2 hbase`

Ensuite, ajouter les lignes suivantes au fichier .bashrc afin de configurer l'environnement pour que hbase soit pris en compte : 

	export HBASE_HOME=/root/hbase
	export HBASE_CONF=$HBASE_HOME/conf
	export HBASE_WEBAPPS=$HABE_HOME/hbase-webapps
	export PATH=$PATH:/root/hbase/bin

Et tapez ensuite : `exec bash`

## Configurer Hbase
Hbase dispose de trois types de configuration :  

- Mode **Standalone**
- Mode **Pseudo distributed**
- Mode **Fully distributed**

##### Un peu de vocabulaire lié à Hbase
**Hbase Master** : Il s'agit du serveur Hbase maître responsable de la surveillance des des serveurs "esclaves" (Region servers).
**Region Servers** : Ce sont les serveurs "esclaves" responsable de la gestion et du stockage des régions.

##### Particularité d'un cluster Hbase "Fully distributed"
Un cluster Hbase peut gérer la gestion des ressources de deux manières : 

- **Interne**, le cluster Zookeeper est géré directement par Hbase, c'est ce dernier qui gère les ressources ainsi que l'état du cluster.
- **Externe**, lorsque Zookeeper est autonome et pleinement opérationnel.

### Configuration d'Hbase sur les serveurs "maître"
La configuration de Hbase nécessite de modifier les fichiers suivants :

- $HBASE_CONF/hbase-env.sh
- $HBASE_CONF/hbase-site.xml
- $HBASE_CONF/regionservers

Le contenu du fichier **$HBASE_CONF/hbase-env.sh** sera identique pour chaque serveur, il faudra juste modifier les lignes suivantes : 

	export JAVA_HOME=[/Chemin/vers/java]
	export HBASE_MANAGES_ZK=false

Ensuite, il faudra modifier le contenu du fichier **$HBASE_CONF/hbase-site.xml**. Le fichier sera biensûr identique sur les serveurs "maîtres (voir ci-dessous) : 

Ci dessous, il s'agit du port que le hbase Master utilisera pour communiquer avec les autres serveurs. Il faudra, biensûr, ouvrir ce port du pare-feu. 

	<property>
	    <name>hbase.master.port</name>
	    <value>[N° du port]</value>
	</property>

Ci-dessous, il s'agit du serveur maître principal - Leader, selon la terminologie Zookeeper. 

	<property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://[alias du serveur "leader" zookeeper]:8020/hbase</value>
	</property>

La propriété ci-dessous n'est à utiliser que lors des modes **pseudo distributed** et **fully distributed**

	<property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	</property>

Les propriétés suivantes sont à utiliser uniquement si le cluster Zookeeper est un cluster autonome et non géré par Hbase.  
Il faut biensûr vérifier que Zookeeper fonctionne correctement, que le port défini dans le fichier de configuration de Zookeeper soit identique a celle défini dans la propriété **hbase.zookeeper.property.clientPort** (et que le port en question soit ouvert dans le pare-feu).  
La propriété **hbase.zookeeper.quorum** doit contenir la même valeur que la propriété **ha.zookeeper.quorum** présente dans le fichier de configuration **hdfs-site.xml** de Hadoop.

	<property>
	    <name>hbase.zookeeper.property.clientPort</name>
	    <value>2181</value>
	</property>
	<property>
    	<name>hbase.zookeeper.quorum</name>
    	<value>[alias du serveur maître 1],[alias du serveur maître 2]</value>
	</property> 
	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>[/chemin/vers/zookeeper/datadir]</value>
	</property>

Le fichier **$HBASE_CONF/regionservers** devra contenir les alias des serveurs sur lesquels le rôle **HRegionServer** sera déployé.  
Il est possible de déployer ce rôle sur un serveur disposant aussi du rôle **HMaster**.


### Configuration d'Hbase sur les serveurs "esclave"
Suite à la partie précédente, nous ne verrons que le contenu du fichier **$HBASE_CONF/hbase-site.xml**.

Ci-dessous, il s'agit du serveur maître principal - Leader, selon la terminologie Zookeeper.

	<property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://[alias du serveur "leader" zookeeper]/hbase</value>
	</property>

Il faut également lui signaler que ce serveur est rattaché à un claster géré par Zookeeper.

	<property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	</property>

## Les commandes de Hbase
Démarrer Hbase : <code>$HBASE_HOME/bin/start-hbase.sh</code>  

La commande précédente permet de démarrer Hbase sur toutes les serveurs contenus dans le fichier **$HBASE_CONF/regionservers** et de déployer les rôles suivants : 

- HMaster
- HRegionServer
  
Stopper Hbase : <code>$HBASE_HOME/bin/stop-hbase.sh</code>

Entrer dans le shell Hbase : <code>$HBASE_HOME/bin/hbase shell</code>

# Conclusion
Dans ce document, nous avons pu observer en détail l'installation et la configuration de Hbase en mode **fully distributed**.
Nous vous suggérons le lien suivant sur lequel vous trouverez plus d'informations sur les commandes du shell hbase : [http://hbase.apache.org/book.html#shell](http://hbase.apache.org/book.html#shell)
