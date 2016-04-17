+++
date = "2015-12-16T22:50:00"
draft = false
title = "Le Labo #7 | Création d'un cluster à l'aide de Serf"
+++

# Introduction
Dans un précédent document, nous avions abordé **Serf**, un outil décentralisé de Service Discovery et d'orchestration.  
Dans ce document, nous allons observer le fonctionnement de **Serf** dans un environnement hétérogène composé de trois serveurs sur lesquels sont installés deux systèmes d'exploitation différents : Windows et Linux.

# Avant d'aller plus loin...
Nous allons avoir besoin des éléments suivants : 

- 3 serveurs : **mida-dockh1 - Ubuntu - (IP : 81.2.243.226)**, **mida-dockh2 - Ubuntu - (IP : 81.2.244.156)** et **mida-dockh3 - Windows - (IP : 81.2.245.130)**
- **Serf** installé sur chaque serveurs

# Installation de Serf

- Sur **Linux** :  
**Télécharger l’archive** : `wget https://releases.hashicorp.com/serf/0.6.4/serf_0.6.4_linux_amd64.zip`  
**Installer Unzip** : `apt-get install -y unzip`  
**Décompresser l’archive** : `unzip serf_0.6.4_linux_amd64.zip -d /usr/local/bin/serf`  
**Déclarer le chemin de Serf** : `echo "PATH=$PATH:/usr/local/bin/serf" >> .bashrc && exec bash`  
**Vérifier l’installation** : `serf`

- Sur **Windows** : 
**Télécharger l’archive** : `https://releases.hashicorp.com/serf/0.6.4/serf_0.6.4_windows_amd64.zip`  
Puis **Décompresser l'archive**

# Créer le cluster Serf
## Démarrage de Serf sur mida-dockh1
Nous allons démarrer le premier noeud Serf sur **mida-dockh1** à l'aide de la commande suivante :  
`serf agent -node=**mida-dockh1** -bind=81.2.243.226:7496 -profile=wan &`

Le terminal affichera ce qui suit : 

	==> Starting Serf agent...
	==> Starting Serf agent RPC...
	==> Serf agent running!
	    Node name: '**mida-dockh1**'
	    Bind addr: '81.2.243.226:7496'
	     RPC addr: '127.0.0.1:7373'
	    Encrypted: false
	     Snapshot: false
	      Profile: wan

	==> Log data will now stream in as it occurs:

	    2014/01/18 21:57:57 [INFO] Serf agent starting
	    2014/01/18 21:57:57 [WARN] Binding to public address without encryption!
	    2014/01/18 21:57:57 [INFO] serf: EventMemberJoin: **mida-dockh1** 81.2.243.226
	    2014/01/18 21:57:58 [INFO] agent: Received event: member-join

## Démarrage de Serf sur mida-dockh2
Nous allons démarrer le second noeud Serf sur **mida-dockh2** à l'aide de la commande suivante :  
`serf agent -node=**mida-dockh2** -bind=81.2.244.156:7496 -rpc-addr=127.0.0.1:7373 -profile=wan &`  
puis `serf join 81.2.243.226:7496`

## Démarrage de Serf sur mida-dockh3
Nous allons démarrer le second noeud Serf sur **mida-dockh2** à l'aide de la commande suivante :  
`serf agent -node=**mida-dockh3** -bind=81.2.245.130:7496 -rpc-addr=127.0.0.1:7373 -profile=wan &`  
puis `serf join 81.2.243.226:7496`

Le terminal de **mida-dockh1** va afficher ce qui suit : 

	==> Log data will now stream in as it occurs:

	    2014/01/18 21:57:57 [INFO] Serf agent starting
	    2014/01/18 21:57:57 [WARN] Binding to public address without encryption!
	    2014/01/18 21:57:57 [INFO] serf: EventMemberJoin: **mida-dockh1** 81.2.243.226
	    2014/01/18 21:57:58 [INFO] agent: Received event: member-join
	    2014/01/18 21:59:00 [INFO] serf: EventMemberJoin: **mida-dockh2** 81.2.244.156
	    2014/01/18 21:59:02 [INFO] agent: Received event: member-join
	    2014/01/18 22:01:00 [INFO] serf: EventMemberJoin: **mida-dockh3** 81.2.245.130
	    2014/01/18 22:01:02 [INFO] agent: Received event: member-join

# Création d'événements pour le Cluster Serf
Depuis n'importe noeuf **Serf** il est possible de générer un événement simple tel que le suivant : 
`serf event hello`  
Il sera visible dans les logs de chaque noeud **serf** de la manière suivante :  

	2015/12/15 16:23:35 [INFO] agent: Received event: user-event: hello

De plus, il est possible de créer des gestionnaires d'événements personnalisés à l'aide des commandes suivantes à intégrer dans un fichier : 

- **SERF_EVENT** : Type de l'événement
- **SERF\_SELF_NAME** : Nom du noeud exécutant l'événement
- **SERF\_SELF_ROLE** : Rôle du noeud exécutant l'événement
- **SERF\_TAG_${TAG}** : Défini pour chaque balise d'agent
- **SERF\_USER_EVENT** : Nom de l'événement "user"
- **SERF\_USER_LTIME**
- **SERF\_QUERY_NAME** : Nom de la requête
- **SERF\_QUERY_LTIME**

Nous allons utiliser le script suivant : 

	#!/bin/bash
	if [ "${SERF_USER_EVENT}" = "mem" ]; then
	   serf event memresponse "$(awk '/MemTotal/ {printf( "%.2f\n", $2 / 1024 ) }'              /proc/meminfo) MB from $(wget -qO- http://ipecho.net/plain ; echo) at $(date)"
	fi

A enregistrer dans /usr/src/[nom_fichier].sh sur **mida-dockh1**.  
Puis sur **mida-dockh2** et **mida-dockh3**, créer le fichier suivant dans le même dossier, avec le contenu suivant : 

	#!/bin/bash
	if [ "${SERF_USER_EVENT}" = "memresponse" ]; then
	    cat >> mem.txt
	    echo "\n" >> mem.txt
	fi

Rendez les scripts exécutables à l'aide de la commande `chmod +x` puis relancez la création du cluster à l'aide des commandes suivantes :  

- Sur **mida-dockh1** :  
`serf agent -log-level=debug -event-handler=./[nom_fichier].sh -node=**mida-dockh1** -bind=81.2.243.226:7496 -profile=wan &`

- Sur **mida-dockh2** :  
`serf agent -log-level=debug -event-handler=./[nom_fichier].sh -node=**mida-dockh2** -bind=81.2.244.156:7496 -rpc-addr=127.0.0.1:7373 -profile=wan &`  
`serf join 81.2.243.226:7496`  
`serf event mem`

- Sur **mida-dockh3** :  
`serf agent -log-level=debug -event-handler=./[nom_fichier].sh -node=**mida-dockh3** -bind=81.2.245.130:7496 -rpc-addr=127.0.0.1:7373 -profile=wan &`  
`serf join 81.2.243.226:7496`  
`serf event mem`

Et enfin, vous pouvez aussi visualiser les membres du cluster à l'aide de la commande **serf members** sur **mida-dockh1**, le terminal affichera ce qui suit : 

	**mida-dockh1**		81.2.243.226:7496	alive
	**mida-dockh2**		81.2.244.156:7496	alive
	**mida-dockh3**		81.2.245.130:7496	alive

# Conclusion
Nous avons présenté un cas simple de création de cluster avec gestion des événements.
Serf est aussi extrêmement personnalisable et peut être adapté pour être une solution à un large éventail de problèmes.
