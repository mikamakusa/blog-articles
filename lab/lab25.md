+++
date = "2016-12-01T15:29:49+02:00"
draft = "false"
title = "Le Labo #19 | A la découverte de Nomad"

+++

# Pour commencer
A un mois de Nöel...ca fait un peu plus de 4 mois que je n'avais rien publié...et je n'avais pas le temps de préparer d'article (ni d'inspiration).  
C'est alors que l'on m'a proposé de travailler sur de l'orchestration de conteneurs.  
Et...j'ai pensé à vous.


## Petit Historique de Nomad
Le développement de Nomad a débuté en Juin 2015, en compagnie de Otto (projet qui fut simplement abandonné quelques mois avant la release de Nomad).  
Cet outil devait devenir le concurrent de Kubernetes, Swarm et Mesos (pour ne citer que les plus connus).  
La release initiale date de Juin ou Juillet, moment auquel il fut mis à disposition du public.


## Le cas concret
Dans le cas d'utilisation qui va servir de démonstration, je vais déployer plusieurs conteneurs Docker sur 6 hôtes différents avec du réseau de conteneurs et du monitoring.

## Ce qu'il nous faut
Dans ce labo, je vais avoir besoin de plusieurs serveurs : 

- 6 serveurs *Minions* sur lesquels seront installés Docker et Nomad 
- 1 serveur *Master* sur lequel Nomad sera installé
- 1 serveur *Nexus* pour le *Docker Private Registry*
- 1 Serveur *Monitor* sur lequel sera installé et configuré *Consul*

Vu que chaque serveur sera déployé à l'aide de terraform, j'ai aussi prévu les scripts de provisionning et de déploiement (afin d'eviter de passer par **Salt** ou **Ansible**...ce qui s'avèrerait plus long).

La configuration de la partie *Private Registry* de **Nexus** étant disponible sur le site de **Sonatype**, je ne la détaillerais pas ici.


## Configuration des serveurs Master et Minions
La configuration de ces serveurs fut assez longue (environ 2 semaines), le plus dur fut de trouver les bonnes infos (qui sont assez éparpillées). Mais une fois le tout rassemblé, tout va très vite.  
On va commencer par le plus simple : **Docker**  

### La configuration de Docker
Dans mon cas, j'ai deux détails à intégrer dans mon fichier *docker.conf* : le proxy (parce que oui...je me suis compliqué la tâche) et le *Private Registry* configuré sur le serveur *Nexus*.  
Le fichier *docker.conf* devra se trouver dans le dossier */etc/systemd/system/docker.service.d* (si vous etes sur un server linux équipé de *systemd* comme CentOS 7 ou Ubuntu 16.04/16.10) et ajouter les infos ci dessous :  

	[Service]
	ExecStart=
	ExecStart=/usr/bin/dockerd --insecure-registry=mon.docker.registry:port
	Environment="HTTP_PROXY=http://mon.adresse.proxy:port"
	Environment="HTTPS_PROXY=https://mon.adresse.proxy:port"
	Environment="NO_PROXY=localhost,127.0.0.1,mon.docker.registry"

N'oubliez surtout pas d'ajouter `insecure-registry`, surtout si votre private registry est configuré en http et non en https (sinon docker aura du mal à le contacter).

Ensuite, exécutez la commande suivante : `systemctl daemon-reload && systemctl restart docker` (si vous créez le fichier docker.conf avant d'installer docker, vous n'aurez pas besoin d'exécuter a commande précédente).

### La configuration du système
Vu que l'infra sur laquelle j'ai travaille comportait un proxy, il fallait faire en sorte que docker l'évite pour accéder au *Private Registry".  
Donc, j'ai exécuté la commande suivante pour compléter les variables d'environnement:  

`export NO_PROXY="localhost,127.0.0.1,mon.docker.registry"`

Une fis cette étape effectuée, j'ai pu m'authentifier sur le *Private Registry* avec un `docker login`

### La configuration de Nomad
La configuration de Nomad (master et client) est relativement simple, il faut surtout déclarer les éléments suivants : 

- log_level
- bind_addr
- addresses
- advertise
- data_dir
- server
- ports

Les clients ont des keywords particulier, comme *datacenters* qui permet de définir l'endroit exact où la ressource sera déployée.

#### Le master

	log_level = "DEBUG"
	bind_addr = "adresse.nomad.master"
	addresses {
		rpc = "adresse.nomad.master"
		http = "adresse.nomad.master"
	}
	advertise {
		rpc = "adresse.nomad.master:4647"
		http = "adresse.nomad.master:4646"
	}
	data_dir = "/tmp/server"
	server {
		enabled = true
		bootstrap_expect = 1
	}
	ports {
		rpc = 4647
		http = 4646
	}

#### Le client

	log_level = "DEBUG"
	bind_addr = "adresse.nomad.client"
	datacenter "dc[x]"
	addresses {
		rpc = "adresse.nomad.client"
		http = "adresse.nomad.client"
	}
	advertise {
		rpc = "adresse.nomad.client:4647"
		http = "adresse.nomad.client:4646"
	}
	data_dir = "/tmp/server"
	client {
		enabled = true
		options = {
			"docker.auth.config" = "/root/.docker/config.json"
			"driver.raw_exec.enabled" = "1"
		}
		server = ["adresse.nomad.master:4647"]
	}
	ports {
		rpc = 4647
		http = 4646
	}

### Démarrage des services Master et Client
La commande est très simple, pour démarrer chaque service, il faut excuter la commande suivante : `nomad agent -config /chemin/vers/fichier/config.hcl &`  
Pour vérifier que tout s'est bien lancé, exécutons la commande suivante sur le serveur *Master* : `nomad node-status -address=http://ip.serveur.master:4646`  
Ce qui suite va s'afficher :  
	
	ID 			DC 		Name 		Class 		Drain 		False
	45deb5e		dc1		host1 		<none> 		false 		ready  
	98f55d7		dc2		host2 		<none> 		false 		ready
	4cf0da1		dc3		host3 		<none> 		false 		ready
	65ah75e		dc4		host4 		<none> 		false 		ready

### Les jobs
Sur le site de Nomad, tout est expliqué au sujet des *jobs specifications*, mais ce qui va suivre n'est expliqué nulle part, donc faites bien attention à ce qui suit :  

- Vous pouvez définir un seul *job* avec un seul *group* et inclure de nombreuses *task*, mais vous n'aurez aucun contrôle sur l'endroit ou seront déployées ces *task*.
- En utilisant les *constraints*, vous pouvez restreindre en fonction d'un certain pattern l'exécution des jobs/groups/tasks.

Je vous invite à consulter la documentation officielle.

De mon côté, j'ai déployé 3 conteneurs *zookeeper* et 3 conteneurs *tomcat* de cette manière : 

#### Les conteneurs Zookeeper

	job "zookeeper" {
		datacenter = ["dc1", "dc2", "dc3"]
		count = 3
		group "deploy" {
			task "zookeeper-deploy" {
				image = "mon.docker.registry:port/my-zookeeper"
				auth {
					username = "mylogin"
					password = "mypassword"
				}
				port_map {
					zk-1 = 2181
					zk-2 = 2888
					zk-3 = 3888
				}
				tty = true
				interactive = true
				hostname = "zookeeper"
			}
			resources {
				cpu = 90
				memory = 10
				network {
					mbits = 10
					port zk-1 {}
					port zk-2 {}
					port zk-3 {}
				} 
			}
		}
	}

#### Les conteneurs Tomcat

	job "tomcat" {
		datacenter = ["dc1", "dc3", "dc4"]
		count = 3
		group "deploy" {
			task "tomcat-deploy" {
				image = "mon.docker.registry:port/my-tomcat"
				auth {
					username = "mylogin"
					password = "mypassword"
				}
				port_map {
					tc-1 = 8080
					tc-2 = 8081
					tc-3 = 8090
				}
				tty = true
				interactive = true
				hostname = "tomcat"
			}
			resources {
				cpu = 90
				memory = 10
				network {
					mbits = 10
					port tc-1 {}
					port tc-2 {}
					port tc-3 {}
				} 
			}
		}
	}

### Le monitoring
J'ai déjà fais un tour du côté de *Consul* lors d'un précédent use-case...je vous invite à le lire pour avoir plus d'infos concernant son installation/configuration.  
En ce qui concerne la partie monitoring de conteneurs, il faut ajouter la section *service* à chaque *task*, voir ci-dessous : 

	service {
		tags = ["zookeeper"]
		port = "zk-1"
		check {
			type 		= tcp
			port 		= "zk-1"
			interval 	= "10s"
			timeout 	"2s"
		}
	}

Ou bien, créer un script *ad-hoc* et le spécifier dans le type avec la cammande pour l'exécuter.


## Conclusion
Malgré la relative jeunesse de **Nomad** et le fait que très peu de retours d'expérience soient encore disponible sur Internet, c'est un outils relativement complet et simple à prendre en main, bénéficiant de connexions assez bien pensées avec le reste de l'écosystème Hashicorp (notamment avec **Terraform**, **Vagrant** et **Consul**).  
De mon côté, après avoir effectué de nombreux tests (qui se sont révélés etre de gros echecs) avec **Kubernetes** et **Swarm**, j'ai adopté **Nomad** en production et je continue à travailler dessus pour du *fine tunning* afin que le projet de déploiement soit parfait...ou presque (il y a un serveur **Salt** qui tourne derrière...voila pourquoi je continue dessus).