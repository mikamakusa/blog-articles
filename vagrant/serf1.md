+++
date = "2015-12-12T22:50:00"
draft = false
title = "Serf - Présentation et Installation"
+++

# Qu'est-ce que Serf ?
**Serf** est un outil de création de cluster comprenant également des fonctionnalités de haute disponibilité et de détection de pannes.  
**Serf** Utilise le protocole de **gossip**, celui-ci permet la communication entre machine inspiré par la forme de bavardage dans les réseaux sociaux, tels que Twitter ou Facebook, car cela rend la résolution de de problème plus rapide et plus efficace.

# Comment l'installer ?
Pour l’installer, il suffit de taper les commandes suivantes :  

- **Télécharger l’archive** : `wget https://releases.hashicorp.com/serf/0.6.4/serf_0.6.4_linux_amd64.zip`  
- **Décompresser l’archive** : `unzip serf_0.6.4_linux_amd64.zip -d /usr/local/bin/serf`  
- **Déclarer le chemin de Serf** : `echo "PATH=$PATH:/usr/local/bin/serf" >> .bashrc && exec bash`  
- **Vérifier l’installation** : `serf`

# Les commandes de Serf
**Serf** Dispose des commandes suivantes, chacune de ces commandes disposent des options **rpc-addr** et **rpc-auth**, celles ci ne sont pas obligatoires  : 

- **agent** : Démarre l'agent Serf
`-bind` : L'adresse sur laquelle Serf va utiliser pour communiquer avec les autres agents.  
`-iface` : Peut être utilisé à la place de **-bind**, si l'interface est connue et non l'adresse.  
`-advertise` : Utilisé pour modifier l'adresse annoncée aux autres noeuds du cluster.   
`-config-file` : Pour définir un fichier de configuration à charger.  
`-config-dir` : Pour définir un dossier de configuration.  
`-discover` : Utilisé pour la découverte automatique de noeuds.  
`-encrypt` : Spécifie la clé secrète à utiliser (à utiliser avec **serf keygen**).  
`-keyring-file` : Spéficie un fichier de gestion de clés.  
`-event-handler` : Ajouter un gestion d'événements pour Serf.  
`-join` : Ajouter un nouvel agent.  
`-replay` : relance les anciens évènements.  
`-log-level` : Le niveau de log à afficher parmi **trace**, **debug**, **info** (par défaut), **warn** et **err**.  
`-node` : Le nom du noeuds dans le cluster.  
`-profile` : Par défaut configuré pour démarrer sur le **LAN**, mais peut être défini sur **WAN** ou sur **local**.  
`-protocol` : Le protocole Serf à utiliser.  
`-retry-join`  
`-retry-interval`  
`-retry-max`  
`-snapshot` : Défini un dossier pour stocker les snapshot.  
`-rejoin` : Si fourni avec **-snapshot**, serf ignorera une première séparation d'agent du cluster et tentera d'y retourner au démarrage.  
`-tag` : Afin d'associer une nouvelle valeur à l'agent.  
`-tags-file` : Défini un fichier pour conserver les données **tag**.  
`-syslog` : Afin d'envoyer les logs vers une solution syslog définie.

- **event** : Envoie un évènement personalisé via le cluster Serf  
`serf event [OPTIONS] [NODE] [PAYLOAD]`  
`-coalesce=` : Définit si oui ou non cet événement peut être fusionné par Serf (true/false).  

- **force-leave** : Force un membre du cluster à entrer en statut "left"  
`serf force-leave [OPTIONS] [NODE]`  

- **info** : Fourni des informations de deboggage pour les opérateurs  
`serf info [OPTIONS]`  
`-format` : Contrôle le format de sortie (Text/json). "Text" par défaut.  

- **join** : Indique à l'agent Serf de rejoindre le cluster  
`serf join [OPTIONS] address ...`  
`-replay` : Si fourni, relance les anciens évènements.  

- **keygen** : Génère une nouvelle clé de cryptage  
`serf keygen`  

- **keys** : Gère le fichier des clé de cryptage  
`serf keys [OPTIONS] `  
`-install` : Installe une nouvelle clé de cryptage.  
`-use` : Change la clé de cryptage.  
`-remove` : Supprime une clé de cryptage.  
`-list` : Liste les clés utilisée par les membres du cluster.

- **leave** : Quitte le cluster Serf et l'éteint  
`serf leve`  

- **members** : Liste les membres du cluster Serf
`serf members [OPTIONS]`  
`-detailed` : Affiche des informations supplémentaires par membres.  
`-format` : Contrôle le format de sortie (Text/json). "Text" par défaut.  
`-name` : Si fourni, n'affiche que les membres en rapport avec celui donné.  
`-status` : Si fourni, n'affiche que les membres en relation avec le status précisé.  
`-tag key=` : Si fourni, affichera seulement le node relatif au tag spécifié.

- **monitor** : Affiche les logs d'un agent Serf  
`serf monitor [OPTIONS]`  
`-log-level` : Le niveau de log à afficher parmi **trace**, **debug**, **info** (par défaut), **warn** et **err**.

- **query** : Envoie une requête au cluster Serf  
`serf query [OPTIONS] name [PAYLOAD]`  
`-format` : Controle le format de sortie (Text/json). "Text" par défaut.  
`-no-ack` : Si fourni, la requete de demandera pas d'accusè de réception. 
`-node [node name]` : Si fourni, affichera seulement le nom des nodes.  
`-tag key=` : Si fourni, affichera seulement le node relatif au tag spécifié.  
`-timeout=` : Si fourni, se substitue au délai par défaut

- **reachability** : Test de réseau  
`serf reachability [OPTIONS]`  

- **rtt** : Estimation du temps de latence entre les noeuds  
`serf rtt [OPTIONS] [NODE]`  

- **tags** : Modifie le tag d'un agent Serf  
`-serf tags [OPTIONS]`  
`-set` : Crée ou met à jour un tag pour un membre.  
`-delete` : Supprime un tag existant.  

- **version** : Affiche la version de Serf

# Pour aller plus loin
Jusqu'ici, nous avons pu oberserver les technologies [Vagrant](http://ageekslab.com/vagrant/vagrant2/) et [Packer](http://ageekslab.com/vagrant/packer1/) de l'écosystème Hashicorp.  
Dans celui-ci, nous avons pu découvrir l'outil de gestion et de création de cluster **Serf**.  
Dans un prochain document, nous étudierons Consul.
