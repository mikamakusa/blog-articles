+++
date = "2015-12-14T18:21:49+02:00"
draft = false
title = "Consul - présentation et Installation"

+++

# Qu'est-ce que Consul ?
**Consul** est un outil de découverte de service, distribué et hautement disponible. C'est également un datastore de type clé/valeur permettant le stockage d'élément de configuration, ainsi qu'un système de supervision de service.  
Tout comme **Serf**, **Consul** s'installe sous la forme d'un agent. Cet agent est chargé d'enregistrer les services, de répondre aux requêtes, de collecter des informations du cluster, etc...  
S’exécutant sur chaque noeud du cluster, il peut fonctionner en mode **client** ou **serveur**. Le cluster doit disposer d’au moins un agent serveur. Au delà, un mécanisme d’élection du leader du cluster permet d’attribuer ce rôle à un des agents serveur. En production, il est recommandé d’avoir de 3 à 5 agents serveur par cluster pour éviter toute perte de données en cas de panne d’un des serveurs.

# Comment l'installer ?
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_linux_amd64.zip`
- **Décompresser l'archive** : `unzip consul_0.6.0_linux_amd64.zip -d /usr/local/bin/consul`
- **Déclarer le chemin de Consul** : `echo "PATH=$PATH:/usr/local/bin/consul" >> .bashrc && exec bash`
- **Vérifier l'installation** : `consul`

# Les commandes de Consul
**Consul** Dispose des commandes suivantes, certaines de ces commandes disposent des options **http-addr** et **rpc-addr**, celles ci ne sont pas obligatoires  :

- **agent** : Démarre l'agent Consul
`consul agent [OPTIONS]`  
`-advertise` | `-advertise-wan` : Utilisé pour modifier l'adresse annoncée aux autres noeuds du cluster.  
`-atlas` | `-atlas-join` | `-atlas-token` | `-atlas-endpoint` : Uniquement dans le cas d'une intégration de l'outils **Atlas**.  
`-bootstrap` | `-bootstrap-expect`  
`-bind` : L'adresse que Consul va utiliser pour communiquer avec les autres agents.  
`-client` : l'adresse que Client liera à l'interface du client.  
`-config-file` : Pour définir un fichier de configuration à charger.  
`-config-dir` : Pour définir un dossier de configuration.  
`-data-dir` : Fourni un dossier dans lequel les agents stockent les statuts.
`-dc` : Contrôle le datacenter dans lequel se trouve l'agent.  
`-domain` : Par défaut, Consul répond aux requêtes DNS du domain ".consul".  
`-encrypt` : Spécifie la clé secrète à utiliser (à utiliser avec **serf keygen**).  
`-http-port` : Relatif au port de l'API.  
`-join` : Ajouter un nouvel agent.  
`-retry-join` | `-retry-interval` | `-retry-max` | `join-wan` | `retry-join-wan` | `-retry-interval-wan` | `retry-max-wan`  
`-log-level` : Le niveau de log à afficher parmi **trace**, **debug**, **info** (par défaut), **warn** et **err**.  
`-node` : Le nom du noeuds dans le cluster.  
`-pid-file`
`-protocol` : Le protocole Consul à utiliser.  
`-recursor`
`-server`
`-syslog` : Afin d'envoyer les logs vers une solution syslog définie.  
`-ui-dir` : Relatif à l'interface Web.

- **configtest** : Exécute un test de fichier configuration  
`consul configtest [OPTIONS]`

- **event** : Lance un nouvel événement  
`consul event [OPTIONS] [PAYLOAD]`  
`-datacenter` |
`-name` |
`-node` |
`-service` |
`-tag` |
`-token`

- **exec** : Exécute une commande sur un noeud Consult  
`consul exec [OPTIONS] [-|COMMAND]`  
`-datacenter` |
`-prefix` |
`-node` |
`-service` |
`-tag` |
`-wait` |
`-wait-repl` |
`-verbose` |
`-token`

- **force-leave** : Force un membre du cluster à quitter le cluster  
`consul force-leave [OPTIONS] [NODE]`

- **info** : Fourni des informations sur l'agent Consul
`consul info`

- **join** : Indique à un agent Consul de joindre le cluster  
`consul join [OPTIONS] [ADDRESS]`  
`-wan`

- **keygen** : Génère une nouvelle clé  
`consul keygen`

- **keyring** : Gère les clés de cryptage
`consul keyring [OPTIONS]`
`-install` : Installe une nouvelle clé de cryptage.  
`-use` : Change la clé de cryptage.  
`-remove` : Supprime une clé de cryptage.  
`-list` : Liste les clés utilisée par les membres du cluster.

- **leave** : Quitte le cluster  
`consul leave`

- **lock** : Exécute une commande de verrouillage  
`consul lock [OPTIONS] [PREFIX] [CHILD]`  
`-n` : Optionnel, limite les vérouillages (par défaut : 1).  
`-name`
`-token` : Token à utiliser.  
`-pass-stdin`
`-verbose`

- **maint** : Fourni un contrôle sur le mode maintenance node et service  
`consul maint [OPTIONS]`  
`-enable` | `-disable` | `-reason` | `-service` | `-token`

- **members** : Liste des membres du cluster  
`consul members [OPTIONS]`
`-detailed` : Affiche des informations supplémentaires par membres.  
`-status` : Si fourni, n'affiche que les membres en relation avec le status précisé.
`-wan`

- **monitor** : Affiche les logs de l'agent Consul  
`consul monitor [OPTIONS]`  
`-log-level` : Le niveau de log à afficher parmi **trace**, **debug**, **info** (par défaut), **warn** et **err**.

- **reload** : Recharge les fichiers de configuration  
`consul reload`  

- **rtt** : Estimation du temps de latence entre les noeuds  
`consul rtt [OPTIONS] [NODE]`  
`-wan`

- **version** : Affiche la version de Consul  

- **watch** : Recherche les modifications effectuées dans Consul  
`consul watch [OPTIONS] [CHILD...]`
`-datacenter` : Datacenter à interroger. Par défaut, celui de l'agent.  
`-token` : Token à visualiser.  
`-key` : Clé à visualiser.  
`-name` : nom de l'événement à visualiser.  
`-passingonly=` | `-prefix`  
`-service` : Service à observer.  
`-state` : Filtre sur **state**.  
`-tag` : Filtre sur service tag.  
`-type` : Nécessite l'une des clé suivante : **key**, **keyprefix**, **services**, **nodes**, **service**, **checks** ou **event**.

# Pour aller plus loin
Jusqu'ici, nous avons pu oberserver les technologies [Serf](http://www.ageekslab.com/vagrant/serf1/), [Vagrant](http://ageekslab.com/vagrant/vagrant2/) et [Packer](http://ageekslab.com/vagrant/packer1/) de l'écosystème Hashicorp.  
Dans celui-ci, nous avons pu découvrir l'outil de service discovery **Consul**.  
Dans un prochain document, nous étudierons Terraform.
