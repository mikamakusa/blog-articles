+++
date = "2015-09-08T15:00:49+02:00"
draft = false
title = "Docker - Gestion du réseau avec Weave"

+++

# 1 - Qu'est ce que Weave ? #
Weave est un logiciel composé de trois fonctions optimisées pour la visualisation et la communication des application créées à l'aide de Docker.  
Grâce à des outils et protocoles connus, weave fourni une topologie réseau réseau permettant aux applications conteneurisées de communiquer entre elles de manière rapide et efficace.

# 2 - Avant d'aller plus loin... #
Avant d'installer Weave, vous devrez installer un serveur doté des caractéristiques suivantes :  
 
- Kernel Linux 3.8 (au minimum)
- Docker 1.3.1 (au minimum)

# 3 - Comment l'installer ? #
Weave ne fait pas partie des dépôts des distributions, pour l'installer il faudra taper les commandes suivantes :
  
- **Weave Net** et **Weave Run** s'installent à l'aide des commandes suivantes :
`curl -L git.io/weave -o /usr/local/bin/weave`  
`chmod a+x /usr/local/bin/weave`

- **Weave Scope** s'installe à l'aide des commandes suivantes
`wget -O scope git.io/scope && cp scope /usr/local/bin/`  
`chmod a+x /usr/local/bin/scope`

# 4 - Quels sont les outils composants Weave ?
Weave se présente sous trois outils différents : Run, Net et Scope.

- **Weave Scope**  
Grâce à la commande `Scope launch`, Scope détecte automatiquement tous les conteneurs déployés sur le serveur et génère une carte 
De plus, il est possible de mettre plusieurs serveurs en commun en déployant Scope aussi sur ces derniers, grâce à la commande suivante :  
`scope launch [nom du service]:[n° du port] [serveur1]:[n° du port] [serveur2]:[n° du port]`  
**important : le port doit être identique pour chaque occurence**  

- **Weave Run** et **Weave Net**  
Weave Run inclue des fonctions de **Service Discovery**, **routage**, **load balancing** et de **gestion d'adresse IP**.  
Weave Net permet de connecter les conteneurs entre eux. **Weave Run** et **Weave Net** sont livrés avec les commandes suivantes : 

>- <u>Gestion du Proxy</u> :  
`weave launch-proxy` : initialisation de Weave Proxy

>- <u>Gestion du DNS</u> :  
`weave dns-add` : Permet d'ajouter l'adresse IP d'un serveur de noms  
`weave dns-remove` : Permet d'enleweave pspsver l'adresse IP d'un serveur de noms

>- <u>Gestion du routeur Weave</u> :  
`weave launch` : initialisation de Weave Router  
**Exemple** : A l'aide de la commande suivante, Weave gère l'ensemble du réseau 10.2.0.0/16 et alloue une address IP du réseau 10.2.1.0/24 si rien n'est précisé lors de la création du conteneur.
`weave launch --ipalloc-range 10.2.0.0/16 --ipalloc-default-subnet 10.2.1.0/24`  
A l'aide de la commande suivante, il est possible d'éviter la reconnaissance automatique des conteneurs :
`weave launch --no-discovery --ipalloc-range 10.2.0.0/16`  
Il est possible de mettre en place une solution de **Service Discovery** à l'aide des commandes suivantes : 
<code>weave launch && weave launch-dns && weave launch-proxy && \`weave proxy-env\`</code>

>- <u>Gestion d'adresses IP automatique</u> :  
`weave run` : Permet de créer les conteneurs  
**Exemple** : `weave run 10.0.0.1/24 --name S1C1 -i -t ubuntu`  
`weave attach` : Permet d'attacher un conteneur à un réseau de manière dynamique  
**Exemple** : `weave attach net:10.2.2.0/24 $S1C1`  
`weave detach` : Permet de détacher un conteneur d'un réseau de manière dynamique  
**Exemple** : `weave detach net:10.2.2.0/24 $S1C1`  
`weave expose` : Permet à un hôte d'accéder au réseau et aux applications d'un conteneur  
`weave hide` : Permet de cacher le réseau et les applications d'un conteneur à son hôte 

>- <u>Gestion dynamique de topologie réseau</u> :  
`weave connect` : Permet d'ajouter un nouvel hôte au réseau weave existant  
**Exemple** : `weave connect [nom de l'hôte à connecter]`  
`weave forget` : Permet d'oublier l'hôte ciblé  
**Exemple** : `weave forget [nom de l'hôte à déconnecter] ` 
**Attention** : Pour un contrôle complet de la topologie réseau, il est possible de désactiver l'option "discovery" (voire la commande weave launch).

>- <u>Gestion de l'outil Weave</u> :  
`weave setup` : Démarre le conteneur Weave  
`weave version` : Affiche les version de Weave Script, Router et Proxy  
`weave env`	: Affiche les variables d'environnement de Weave  
`weave config` : Affiche la configuration de Weave (donne un résultat similaire à weave env)  
`weave status` : Affiche l'état de Weave  
La commande `weave status connections` permet d'afficher les connexions entre hôtes différents si cela à été configuré lors du lancement de Weave.  
`weave ps` : Affiche les conteneurs lancés à l'aide de weave ainsi que les adresses IP assignées aux conteneurs  
`weave stop` : Arrête Weave et Weave Proxy  
`weave stop-router` : Arrête seulement Weave  
`weave stop-proxy` : Arrête seulement Weave Proxy  
`weave reset` : redémarre Weave  
`weave rmpeer` : Supprime <peerid> associé à Weave

# 5 - Comment travailler avec Weave ? #
Partons du principe que nous avons deux serveurs sur lesquels Weave et Docker sont installé : S1 et S2.  
Nous tapons la commande suivant sur S1 : `weave launch`  
Puis sur S2 : `weave launch S1`  
Les deux hôtes sont maintenant connectés entre eux.

Ensuite, nous allons pouvoir lancer quatres conteneurs, deux sur chaque hôtes, qui seront dans différents sous-réseaux :
 
- Sur **S1** :  
`weave run 10.0.0.1/24 --name S1C1 -i -t ubuntu`  
`weave run --name S1C2 -i -t ubuntu`

- Sur **S2** :  
`weave run 10.0.0.2/24 --name S2C1 -i -t ubuntu`  
`weave run --name S2C2 -i -t ubuntu`

Dans cet exemple là, **S1c1** et **S2c1** peuvent communiquer car ils sont dans le même réseau.  
Par contre, **S1c1** et **S1c2** ne sont pas dans le même réseau et ne peuvent donc pas communiquer.

# 5 - Pour aller plus loin... #
Dans un prochain tutoriel, nous mettrons en pratique les fonctionnalités de Docker et de Weave.

