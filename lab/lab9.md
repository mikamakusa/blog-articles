+++
date = "2016-01-14T17:39:00"
draft = false
title = "Le Labo #9 | Déploiement automatisé et surveillance de microservices"
+++

# Introduction
Dans de précédents articles, nous avions abordé les technologies suivantes : 

- **Docker**
- **Weave**
- **Consul**
- **Ansible**

Ces technologies permettent de créer des micro-services et de surveiller leur bon fonctionnement.
Ceci s'inscit dans une démarche 'DevOps', lorsque du code permet, comme nous allons le démontrer, de créer toute une infrastructure fonctionnelle.

Nous allons effectuer les tâches suivantes : 

- Description des conteneurs à déployer dans un fichier utilisable par **docker-compose**,
- Déploiement d'un réseau pour conteneurs **Docker** à l'aide de **Weave**,
- Déploiement des conteneurs puis installation des paquets automatisée à l'aide d'**Ansible**,
- Découverte et surveillance des services à l'aide de **Consul**

# Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 1 serveurs : **mida-dockh1 - Ubuntu - (IP : 81.2.243.226)**  
- **Docker-Compose**, **Weave**, **Consul** et **Ansible** installés sur le serveur **mida-dockh1**

# Docker - Préparation et déploiement
## L'image docker
Dans cet exercice, nous allons utiliser l'image docker **rastasheep/ubuntu-ssh:14.04**.  
L'avantage de cette image est qu'un serveur SSH est préinstallé, ce qui se révèlera utile lors du déploiement des paquets à l'aide d'Ansible ainsi que lors du démarrage des services sur les conteneurs hadoop.

## Docker Compose
Nous allons avoir besoin des éléments suivants pour mener à bien cet exercice : 

- Les ports **8300**, **8301**, **8302**, **8400**, **8500** et **8600** permettent à **Consul** de fonctionner (communication entre serveurs et interface web),
- Le port **80** pour le conteneur serveur web,
- Le port **3306** pour le conteneur serveur mysql,
- les ports **50010**, **50020**, **50070**, **50075**, **50090**, **19888**, **8030**, **8031**, **8032**, **8033**, **8040**, **8042**, **8088** et **9000**: Tous ces ports sont nécessaire au bon fonctionnement d'un serveur Hadoop, au déploiement de HDFS, de MapReduce et de Yarn. 
- Le port **22** pour SSH.

Si ces ports ne sont pas précisés dans le fichier docker-compose.yml lors du déploiement des conteneurs, les conteneurs ne disposeront pas de translation de ports vers les ports du serveur hôte.
De plus, nous aurons besoin du dossier /root/.ssh contenant les clé SSH et des liens suivants entre les serveurs : 

- **web -> db**
- **hadoopnn -> hadoopdn**

Enfin, lors du déploiement des *namenode* et *datanode* Hadoop, nous aurons aussi besoin des **hostname** des conteneurs.

## Déploiement de l'infrastructure
Le déploiement se fait à l'aide de la commande `docker-compose up -d`  
Pour afficher l'ensemble des conteneurs déployés, exécutons la commande `docker-compose ps`, ce qui permettra de connaître les ports ssh translatés de chaque conteneurs.  

# Ansible - Préparation et déploiement
Nous allons effectuer le déploiement des paquets, archives et fichiers de configuration nécessaires à l'aide d'Ansible, ce qui permettra d'automatiser cette étape et, par conséquent, de gagner du temps.
Lors de cette étape, nous aurons obligatoirement besoin des deux éléments suivants : 

- Le fichier d'inventaire d'Ansible (**hosts**), il se trouve dans /etc/ansible.
- Le playbook. Dans notre cas, nous allons créer plusieurs rôles puisque nous allons déployer quatres types de serveurs : Consul, Web, MySQL et Hadoop.

## Le fichier d'inventaire
Le fichier d'inventaire Ansible doit contenir, au moins l'alias du serveur et son adresse IP.  
Dans notre cas, le déploiement des paquets à l'aide d'Ansible se fera sur des conteneurs Docker reliés en réseau via Weave. En sachant que le serveur hôte dispose de l'adresse IP 172.17.0.1, les conteneurs obtiennent les IP qui suivent en fonction de leur ordre de création (et non suivant leur ordre de déclaration dans le fichier **docker-compose.yml**).

Il est d'ailleurs possible d'automatiser la création du fichier **/etc/ansible/hosts** nécessaire à l'aide de la commande suivante :  
`for x in $(docker ps --format "{{.Names}}" --filter name="root"); do echo "$x " | sed 's/^.....//' | sed 's/...$//' | sed 's/^/[/' | sed 's/$/]/'  && docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $x ; done > /etc/ansible/hosts`

## Le playbook et les rôles
Notre fichier playbook devra contenir les paquets, archives et fichiers de configurations.  
Chaque type de configuration sera organisée en **rôle** dans le playbook d'Ansible.  

Nous allons créer 4 rôles : **webserver**, **database**, **hadoop** et **consul**.
Notre fichier */etc/ansible/hosts* contient les informations nécessaires au déploiement de paquets sur tous les conteneurs ainsi que sur le serveur hôte, dans le but de démarrer le déploiement des conteneurs directement à l'aide d'ansible.

Les rôles **webserver** et **database** ne nécessitent aucune intervention post-deploiment, contrairement aux autres rôles. Nous détaillerons plus tard les interventions à effectuer. 

**Le playbook**

	---
	- hosts: web
	  roles:
	    - webserver
	    - consul
	- hosts: db
	  roles:
	    - database
	    - consul
	- hosts: hadoopnn
	  roles:
	    - hadoop
	    - consul
	  tasks: 
	    - name: 1. Hadoop NameNode Config
	      shell: echo hadoopdn > /root/hadoop/etc/hadoop/slaves
	    - name: 2. Hadoop Namenode Config
	      shell: echo localhost > /root/hadoop/etc/hadoop/masters
	- hosts: hadoopdn
	  roles:
	    - hadoop
	    - consul
	  tasks: 
	    - name: Hadoop DataNode Config
	      shell: echo "localhost" > /root/hadoop/etc/hadoop/slaves

Concerant le rôle **hadoop**, nous avons deux situations différentes : 

- Un serveur hadoop <u>NameNode</u> sur lequel sera déployé le formattage HDFS, ainsi que les rôles **dfs** et **yarn**,  
- Un serveur hadoop <u>DataNode</u> sur lequel sera déployé le rôle **datanode**.

C'est pour cela que le playbook contient des tâches supplémentaires pour le conteneur **hadoopnn** relatives au fichier de configuration *slaves* (ainsi que le fichier /etc/hosts).

**Le rôle Consul**

	---
	- name: install wget
	  apt: name=wget state=present
	- name: Install unzip
	  apt: name=unzip state=present
	- name: Install Consul
	  shell: cd /root && wget https://releases.hashicorp.com/consul/0.6.1/consul_0.6.1_linux_amd64.zip && unzip consul_0.6.1_linux_amd64.zip -d /usr/local/bin/consul
	- name: 4. Add file to environment
	  shell: echo "PATH=$PATH:/usr/local/bin/consul" >> /root/.bashrc

**Le rôle webserver**

	---
	- name: install Apache
	  apt: name=apache2 state=present
	- name: install PHP module for Apache
	  apt: name=libapache2-mod-php5 state=present
	- name: start Apache
	  service: name=apache2 state=running enabled=yes
	- name: install Hello World PHP script
	  copy: src=index.php dest=/var/www/index.php mode=0664
	- name: Check file directory
	  shell: mkdir -p /home/consul/service
	- name: File Consul checks
	  copy: src=services.json dest=/home/consul/service/services.json mode=0664

**Le rôle database**

	---
	- name: Install mysql server package
	  apt: name=mysql-server state=present
	- name: Start Mysql Service
	  service: name=mysql state=started enabled=true
	- name: Install python Mysql package #required for mysql_db tasks
	  apt: name=python-mysqldb state=present
	- name: Check file directory
	  shell: mkdir -p /home/consul/service
	- name: File Consul checks
	  copy: src=services.json dest=/home/consul/service/services.json mode=0664

Le rôle hadoop en lui même contient toutes les tâches relatives aux fichiers de configuration core-site, hdfs-site, mapred-site et yarn-site que vous devrez modifier par la suite.

**Le rôle hadoop**

	---
	- name: Install Java-jre
	  apt: name=default-jre state=present
	- name: Install Java-jdk
	  apt: name=default-jdk state=present
	- name: Install tar
	  apt: name=tar state=present
	- name: Download hadoop 
	  shell: cd /root && wget http://mirrors.ircam.fr/pub/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz && tar xzf /root/hadoop-2.7.1.tar.gz && mv /root/hadoop-2.7.1 /root/hadoop
	- name: Environment file
	  template: src=.bashrc dest=/root/.bashrc
	- name: Check file directory
	  shell: mkdir -p /home/consul/service
	- name: File Consul checks
	  copy: src=services.json dest=/home/consul/service/services.json mode=0664
	- name: Hadoop Config files - core-site.xml
	  copy: src=hadoop_config/core-site.xml dest=/root/hadoop/etc/hadoop/core-site.xml  
	- name: Hadoop Config files - hdfs-site.xml
	  copy: src=hadoop_config/hdfs-site.xml dest=/root/hadoop/etc/hadoop/hdfs-site.xml
	- name: Hadoop Config files - mapred-site.xml
	  copy: src=hadoop_config/mapred-site.xml dest=/root/hadoop/etc/hadoop/mapred-site.xml
	- name: Hadoop Config files - yarn-site.xml
	  copy: src=hadoop_config/yarn-site.xml dest=/root/hadoop/etc/hadoop/yarn-site.xml
	- name: hadoop tmp directory
	  shell: mkdir -p /root/hadoop/namenode && mkdir -p /root/hadoop/datanode
	- name: Modify hadoop-env.sh
	  shell: sed -i '/JAVA_HOME/d' /root/hadoop/etc/hadoop/hadoop-env.sh && echo 'export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64' >> /root/hadoop/etc/hadoop/hadoop-env.sh

## Le déploiement
Pour déployer tout cela, nous allons employer la commande <code>ansible-playbook playbook.yml</code>.  
Ce qui permettra d'automatiser l'installation des paquets et des archives nécessaires. 

Une fois le déploiement terminé, le terminal affichera ce qui suit :  
 
	PLAY RECAP ********************************************************************
	172.17.0.2                 : ok=10   changed=8    unreachable=0    failed=0
	172.17.0.3                 : ok=21   changed=18   unreachable=0    failed=0
	172.17.0.4                 : ok=10   changed=7    unreachable=0    failed=0
	172.17.0.5                 : ok=19   changed=16   unreachable=0    failed=0

Signifiant que tout s'est bien passé et que notre infrastructure est fonctionnelle...ou presque.
Il ne manque que quelques commandes à exécuter sur les conteneurs **hadoopnn** et **hadoopdn**.

### Sur le Namenode Hadoop
Il faudra exécuter les commandes suivantes : 

- Démarrer le formattage en HDFS :  
`hadoop/bin/hadoop namenode -format`
- Démarrage des services dfs et yarn :  
`hadoop/sbin/start-dfs.sh`  
`hadoop/sbin/start-yarn.sh`
- Vérifier l'état des services à l'aide de la commande `jps`

		root@localhost:~# jps  
		7101 SecondaryNameNode
		7256 ResourceManager
		6928 NameNode
		7323 Jps

### Sur le Datanode Hadoop
Si tout s'est bien passé sur le conteneur Namenode Hadoop, l'exécution de la commande jps devrait afficher le service Datanode, cependant il est toujours possible de le vérifier d'une autre manière grâce à la commande :  
`hadoop/sbin/hadoop-daemon.sh start datanode`  

Afin d'afficher ce qui suit dans le terminal :  

	datanode running as process 6764. Stop it first.

# Consul
Dans notre cas, **Consul** va nous servir à surveiller le fonctionnement des conteneurs grâce aux fichiers de services copiés sur chacun d'eux lors du déploiement.  
La première étape va consister à démarrer **Consul** et à servir l'interface web sur le serveur hôte à l'aide de la commande suivante : 
`consul agent -data-dir consul -server -bootstrap -client 81.2.243.226 -advertise 172.17.0.1 -ui-dir /home/web_ui/ &`

Ensuite, sur chaque conteneur, nous rejoindrons le service **Consul** démarré sur le serveur hôte à l'aide des commandes suivantes : 

- Sur **webserver** :  
`consul agent -data-dir /root/consul -client 172.17.0.2 -advertise 172.17.0.2 -node webserver -config-dir /home/consul/service -join 172.17.0.1`
- Sur **database** :  
`consul agent -data-dir /root/consul -client 172.17.0.4 -advertise 172.17.0.4 -node database -config-dir /home/consul/service -join 172.17.0.1`
- Sur **hadoopnn** :  
`consul agent -data-dir /root/consul -client 172.17.0.3 -advertise 172.17.0.3 -node hadoopnn -config-dir /home/consul/service -join 172.17.0.1`
- Sur **hadoopdn** :  
`consul agent -data-dir /root/consul -client 172.17.0.5 -advertise 172.17.0.5 -node hadoopdn -config-dir /home/consul/service -join 172.17.0.1`

Une fois cette étape effectuée, nous allons pouvoir consulter l'interface web et vérifier que les quatre conteneurs y sont bien enregistrés.  
Pour y accéder, il faudra ouvrir l'adresse IP du serveur, suivi du port <u>*8500*</u> et du dossier <u>*ui*</u>.  

# Conclusion
Nous vous avons présenté un exemple de Service Discovery à l'aide de **Consul**, **Ansible** et **Docker**.  
Comme nous l'avions expliqué dans un précédent document, **Consul** est extensible et s'adapte à son environnement, il est tout à fait possible, dans l'état actuel de notre environnement de test, d'ajouter plusieurs service et de les faire reconnaître par Consul afin de surveiller leur bon fonctionnement.
