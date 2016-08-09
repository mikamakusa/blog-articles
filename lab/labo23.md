+++
date = "2016-08-09T15:29:49+02:00"
draft = false
title = "Le Labo #19 | La stack ELK et SaltStack"

+++

# Formattage de logs de SaltStack (Master et Minion) à l'aide de la stack ELK

# Avant de commencer
Lors de mon dernier lab, je m'étais lancé sur de l'automatisation de création de conteneur Docker à l'aide de SaltStack et rappelant que les logs d'érreur, bien que facilement compréhensible par un humain, pouvaient devenir assez indigeste.  
J'avais même évoqué la stack ELK comme éventualité pour formatter ces journaux de logs.  
Voici donc le résultat de deux semaines de travail et de nombreuses tentatives pour trouver une solution au problèmes rencontrés.


# Prérequis
Nous allons avoir besoin de plusieurs serveurs sur lesquels il faudra installer les services suivants :  

- Salt-Master : En plus d'installer **Salt**, il faudra y installer **Logstash**
- Salt-Minion : La configuration sera identique à **Salt-Master**
- Redis : Afin de pouvoir récupérer les logs Salt et les transmettre au serveur ELK
- ELK : Logstash (Extraction de logs), Elasticsearch (indexation) et Kibana (présentation) seront installés.

## Installation
L'instalation de Salt (Master et Minion) ayant été expliqué lors du lab précédent, je ne reviendrais pas dessus. Du coup, seuls les installations de Redis et de la stack ELK sera expliquée ici.

### Redis 
Pour l'installer sur CentOS 7, il faut exécuter la commande suivante : `yum install -y redis`

### La stack ELK
Pour l'installer sur CentOS 7, il faut exécuter la commande suivante : `yum install -y logstash elasticsearch kibana` ou `yum groupinstall -y ELK`

## Configuration
la configuration va se faire en plusieurs étapes qui seront décrites pas à pas.

### 1 - Redis
Pour **Redis**, il y a très peu de modifications à effectuer, mais il faut commencer par démarrer le service : `systemctl start redis`  
Puis, il faut éditer le fichier */etc/redis.conf* et y chercher la ligne en rapport avec l'adresse d'écoute : *bind 127.0.0.1* et remplacer l'adresse locale par l'adresse IP externe (ou interne, selon la configuration souhaitée).
Une fois ceci effectué, il faut redémarrer le service.

### 2 - Les serveurs Salt
Vu que **Logstash** est installé aussi sur les serveurs **Salt**, la configuration adequate sera abordée après celle de **Salt**.

#### Le fichier de configuration Salt (Master/Minion)
Afin de rediriger les logs de Salt vers l'instance **Logstash** tournant sur ce serveur, il faudra ajouter ce qui suit dans le fichier */etc/salt/master* et */etc/salt/minion*

	logstash_udp_handler:
	  host: 127.0.0.1
	  port: 6379
	  version: 1

	logstash_zmq_handler:
	  address: tcp://127.0.0.1:6379
	  version: 1

J'avais pu faire un test : l'installation et la configuration d'ElasticSearch et de Kibana directment sur le serveur Salt-Master, mais cela présentait un inconvenient majeur : **ElasticSearch est un enorme consommateur de ressources** (Donc, c'est fortement déconseillé de l'installer sur un serveur qui fait tourner de nombreux autres services).

De plus, il faudra aussi modifier la valeur relative au niveau de log *log_level* et la remplacer par *all* (ou tout autre valeur, cela dépendra du type d'évenement que vous souhaiterez référencer dans Kibana), afin d'avoir le plus d'éléments possible à intégrer par la suite à **Kibana**.

#### Le fichier de configuration de Logstash
Le fichier de configuration de **logstash** doit, au minimum, intégrer un *input* et un *output*, dans notre cas, nous allons ajouter ce qui suit : 

	input {
	  udp {
	    port => 6379
	    codec => json
	  }
	  zeromq {
	    topology => "pubsub"
	    address => "tcp://0.0.0.0:6379"
	    codec => json
	  }
	}
	output {
	  stdout {
	    codec => rubydebug
	  }
	  redis {
	    host => "<@ip du serveur Redis>"
	    data_type => "list"
	    key => "salt-master"/"salt-minion"
	  }
	}

Dans la partie *input*, il est possible de remplacer *udp* par *tcp*, mais il faudra faire la même modification dans le fichier de configuration de **Salt**.

A partir de ce moment, on peut déjà redémarrer les services **Salt**, **Logstash** et **Redis** qui viennent d'être configurrés.  
Pour vérifier que tout fonctionne sans erreur, voici les chemins des fichiers de logs de chaque services : 

- **Salt** : /var/log/salt/master et /var/log/salt/minion
- **logstash** : /var/log/logstash/logstash.log
- **Redis** : /var/log/redis/redis.log


### Le serveur ELK
#### Configuration de Logstash
Ici, **Logstash** va récupérer les logs centralisés sur le serveur **Redis**, voici la configuration qui sera appliquée dans ce cas : 

	input {
	  redis {
	    host => "<@ip du serveur Redis>"
	    type => "redis-input"
	    data_type => "list"
	    key => "salt"
	  }
	}

	filter { }

	output {
	  elasticsearch {
	    hosts => [ "127.0.0.1" ]
	  }
	}

Une fois le fichier de configuration enregistré dans le dossier */etc/logstash/conf.d*, il est temps de démarrer chaque service à l'aide de la commande suivante : 
`systemctl start logstash elasticsearch kibana`

Dès que les services ont été démarrés, il sera possible de visualiser les logs dans Kibana après avoir sélectionné la source et rafraîchi la page.  

# Pour finir
Je dois avouer qu'une fois la configuration maîtrisée, la stack ELK est d'une absolue simplicitée.  
Néanmoins, bien que les logs soient visible et facilement exploitable dans Kibana, il est tout à fait possible de les présenter d'une manière la plus sexy possible (ce qui est d'ailleurs l'une des utilités de Kibana).  
Mais cela, je vous le laisse...