+++
date = "2015-12-17T22:50:00"
draft = false
title = "Le Labo #8 | Ajout d'une fonction de Service discovery au cluster Serf"
+++

# Introduction
Dans de précédents documents, nous avions abordé **Serf**, un outil de clustering, ainsi que **Consul**, un outil de Service Discovery.  
Dans ce document, nous allons observer l'ajout des fonctionnalités de **Consul** à l'environnement dans lequel **Serf** est fonctionnel, dans un environnement hétérogène composé de trois serveurs sur lesquels sont installés deux systèmes d'exploitation différents : Windows et Linux.

# Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 3 serveurs : **mida-dockh1 - Ubuntu - (IP : 81.2.243.226)**, **mida-dockh2 - Ubuntu - (IP : 81.2.244.156)** et **mida-dockh3 - Windows - (IP : 81.2.245.130)**
- **Serf** installé sur chaque serveurs  
- **Consul** installé sur chaque serveurs

# Installation de Consul

- Sur **Linux** :  
- **Télécharger l'archive** : `wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_linux_amd64.zip`  
- **Décompresser l'archive** : `unzip consul_0.6.0_linux_amd64.zip -d /usr/local/bin/consul`  
- **Déclarer le chemin de Consul** : `echo "PATH=$PATH:/usr/local/bin/consul" >> .bashrc && exec bash`  
- **Vérifier l'installation** : `consul`

- Sur **Windows** :  
- **Télécharger l'archive** : https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_windows_amd64.zip
- Puis **Décompresser l'archive**

# Démarrer Consul
## Sur mida-dockh1
Nous allons démarrer le serveur **Consul** Serf sur **mida-dockh1** à l'aide des commandes suivantes :  

- **Générer la clé de cryptage** : `consul keygen`  
- **créer le dossier de configuration de Consul** : `mkdir consul`  
- **Démarrer le serveur Consul en mode bootstrap** : `consul agent -server -advertise=81.2.243.226 -bootstrap -data-dir=consul -dc=dc3 -encrypt=[clé]`

Le terminal affichera ce qui suit : 

	==> WARNING: Bootstrap mode enabled! Do not enable unless necessary
	==> Starting Consul agent...
	==> Starting Consul agent RPC...
	==> Consul agent running!
			Node name: 'mida-dockh1'
			Datacenter: 'dc3'
			Server: true (bootstrap: true)
			Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
			Cluster Addr: 81.2.243.226 (LAN: 8301, WAN: 8302)
			Gossip encrypt: true, RPC-TLS: false, TLS-Incoming: false
			Atlas: <disabled>

	==> Log data will now stream in as it occurs:
	2015/12/16 16:56:04 [INFO] serf: EventMemberJoin: mida-dockh1 81.2.243.226
	2015/12/16 16:56:04 [INFO] serf: EventMemberJoin: mida-dockh1.dc3 81.2.243.226
	2015/12/16 16:56:04 [INFO] raft: Node at 81.2.243.226:8300 [Follower] entering Follower state
	2015/12/16 16:56:04 [INFO] consul: adding LAN server mida-dockh1 (Addr: 81.2.243.226:8300) (DC: dc3)
	2015/12/16 16:56:04 [INFO] consul: adding WAN server mida-dockh1.dc3 (Addr: 81.2.243.226:8300) (DC: dc3)
	2015/12/16 16:56:04 [ERR] agent: failed to sync remote state: No cluster leader
	2015/12/16 16:56:06 [WARN] raft: Heartbeat timeout reached, starting election
	2015/12/16 16:56:06 [INFO] raft: Node at 81.2.243.226:8300 [Candidate] entering Candidate state
	2015/12/16 16:56:06 [INFO] raft: Election won. Tally: 1
	2015/12/16 16:56:06 [INFO] raft: Node at 81.2.243.226:8300 [Leader] entering Leader state
	2015/12/16 16:56:06 [INFO] consul: cluster leadership acquired
	2015/12/16 16:56:06 [INFO] consul: New leader elected: mida-dockh1

## Sur mida-dockh2
Nous allons démarrer le serveur **Consul** sur **mida-dockh2** à l'aide des commandes suivantes :  

- **créer le dossier de configuration de Consul** : `mkdir consul`  
- **Démarrer le serveur Consul afin qu'il joigne automatiquement le premier serveur** : `consul agent -data-dir=consul -dc=dc3 -advertise=81.2.244.156 -join=81.2.243.226 -encrypt=[clé] -server`

## Sur mida-dockh3
Nous allons démarrer le serveur **Consul** sur **mida-dockh3** à l'aide des commandes suivantes :  

- **créer le dossier de configuration de Consul** : `mkdir consul`  
- **Démarrer le serveur Consul afin qu'il joigne automatiquement le premier serveur** : `consul agent -data-dir=consul -dc=dc3 -advertise=81.2.245.130 -join=81.2.243.226 -encrypt=[clé]` 

## Sur mida-dockh1
Le terminal affichera ce qui suit à chaque fois qu'un nouveau serveur le rejoindra :

### Pour mida-dockh2

	2015/12/16 17:30:13 [INFO] consul: adding LAN server mida-dockh2 (Addr: 81.2.244.156:8300) (DC: dc3)
	2015/12/16 17:30:13 [INFO] raft: added peer 81.2.244.156:8300, starting replication
	2015/12/16 17:30:37 [INFO] consul: member 'mida-dockh2' joined, marking health alive
	2015/12/16 17:30:54 [INFO] agent.rpc: Accepted client: 127.0.0.1:52732

### Pour mida-dockh3

	2015/12/16 17:30:37 [INFO] consul: member 'mida-dockh3' joined, marking health alive
	2015/12/16 17:30:54 [INFO] agent.rpc: Accepted client: 127.0.0.1:52732

En exécutant la commande `consul members`, nous allons pouvoir afficher les membres du cluster : 

	Node         Address            Status  Type    Build  Protocol  DC
	mida-dockh1  81.2.243.226:8301  alive   server  0.6.0  2         dc3
	mida-dockh2  81.2.244.156:8301  alive   server  0.6.0  2         dc3
	mida-dockh3  81.2.245.130:8301  alive   client  0.6.0  2         dc3

# Allons un peu plus loin...
Dans cette section, nous allons ajouter plusieurs éléments :

- L'interface web de Consul, téléchargeable ici : https://dl.bintray.com/mitchellh/consul/0.3.0\_web_ui.zip
- Un nouveau membre au cluster **Serf** **Consul** : **mida-dockh4 - IP 81.2.25.10**
- Un service web sur le nouveau membre.

## Opération à effectuer sur mida-dockh1
Actuellement, **Consul** est en mode bootstrap sur le serveur **mida-dockh1**, ce qui signifie qu'il est seul à pouvoir prendre des décisions sans avoir à consulter les autres serveurs.  
Afin de remédier à cela, nous allons relancer **consul** sans le mode **bootstrap** à l'aide de la commande suivante :  
`consul agent -server -advertise=81.2.243.226 -data-dir=consul -dc=dc3 -encrypt=[clé]`

## Opération à effectuer sur mida-dockh2 et mida-dockh3
Avant de lancer la commande permettant de rejoindre le cluster **Serf/Consul** sur **mida-dockh4**, nous allons d'abord relancer **mida-dockh2** et **mida-dockh3** en tant que *serveur* parmi les membres de **Consul** à l'aide des commandes suivantes :  

- Sur **mida-dockh2** : `consul agent -data-dir=consul -dc=dc3 -advertise=81.2.244.156 -join=81.2.243.226 -encrypt=[clé] -server`  
- Sur **mida-dockh3** : `consul agent -data-dir=consul -dc=dc3 -advertise=81.2.245.130 -join=81.2.243.226 -encrypt=[clé] -server`  

## Installation des éléments nécessaires sur mida-dockh4
Une fois les agents **Consul** relancés sur chaque serveur, nous allons effectuer les opérations suivantes :

- **création du dossier data-dir** : `mkdir consul`

- **Installation du serveur web** : `apt-get install nginx`

- **Installation de Serf et de Consul** :  
`wget https://releases.hashicorp.com/serf/0.6.4/serf_0.6.4_linux_amd64.zip`  
`wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_linux_amd64.zip`  
`apt-get install -y unzip`  
`unzip serf_0.6.4_linux_amd64.zip -d /usr/local/bin/serf && unzip consul_0.6.0_linux_amd64.zip -d /usr/local/bin/consul`  
`echo "PATH=$PATH:/usr/local/bin/serf" >> .bashrc && echo "PATH=$PATH:/usr/local/bin/consul" >> .bashrc && exec bash`  

- **installation de l'interface web de Consul** :  
`wget https://dl.bintray.com/mitchellh/consul/0.3.0_web_ui.zip`  
`mkdir /home/web_ui`
`unzip 0.3.0_web_ui.zip -d /home/web_ui`

- **Création des Services et des Checks** :  
Le serveur Nginx étant déjà installé, nous allons pouvoir créer un Service pour Consul à l'aide de la commande suivante :  
`mkdir /home/consul/services && vim /home/consul/service/web.json`

Un fichier de service basique contient les balises suivantes : 

	{
	    "service": {
	        . . .
	        "check": {
	            . . .
	        }
	    }
	}

Dans notre exemple, nous allons avoir besoin des suivantes : **name**, **port**, **tags**, **script** et **interval**, afin que le script ressemble à ce qui suit : 

	{
	    "service": {
	        "name": "web server",
	        "port": 80,
	        "tags": ["nginx", "demonstration"],
	        "check": {
	            "script": "curl localhost:80 > /dev/null 2>&1",
	            "interval": "10s"
	        }
	    }
	}

- **Démarrage de l'agent Consul et de l'interface web** :  
`consul agent -data-dir=consul -advertise=81.2.25.10 -ui-dir=/home/web_ui -join=81.2.243.226 -encyrpt=[clé] -config-dir=/home/consul/service`  

Pour vérifier que tout fonctionne bien, il faut se rendre sur http://[IP_Client]:8500/ui

# Conclusion
Nous vous avons présenté un exemple de Service Discovery à l'aide de **Consul**.  
**Consul** est un outil évolutif, il s'adapte à la situation et fonctionne aussi bien avec un réseau LAN que sur Internet, tel que nous l'avons démontré ici.
