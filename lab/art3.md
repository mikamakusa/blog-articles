+++
date = "2017-12-21T13:35:49+02:00"
draft = false
title = "Reverse Proxying"

+++

## Pourquoi un match sur les Reverse Proxy ?
En fait, j'avais envie de faire un petit comparatif entre les différentes méthodes de *Reverse Proxying* en faisant un tour d'horizon de ce qui est proposé sur internet...en restant dans l'Open Source.  
J'aurais aussi pu vous proposer les outils non Open Source, mais je reste dans l'optique : tant que c'est testable sans dépenser un centime, autant s'amuser avec.  
A date, je n'ai déployé que **Nginx** sur un projet entier (le dernier, en l'occurrence), mais ne vous en faites pas, je vais aborder les autres outils.

## Les outils
Parmi les outils de **Reverse Proxying**, nous avons :  
Ceux qui ne sont que des proxy:  
- [Sozu](https://www.sozu.io/)  
- [Traefik](https://traefik.io/)  
- [Varnish](https://www.varnish-cache.org/)  
- [Squid](http://www.squid-cache.org/)  
Les serveurs HTTP qui peuvent servir de **Reverse Proxy** : 
- [Nginx](https://nginx.org/)  
- [Apache](https://httpd.apache.org/)  
- [GateJs](https://www.gatejs.org/)  
- [Hiawatha](https://www.hiawatha-webserver.org/)  
- [Cherokee](http://cherokee-project.com/)  

## Comment les installer ?
Certains de ces outils sont d'une simplicité *quasi enfantine* à installer...et sont présent parmi les *packages repository* de vos distributions préférées :  
- Apache
- Nginx
- Cherokee
- Varnish
- Squid

D'autres nécessitent de *mettre les mains dans le camboui* du système en installant des dépendances et en exécutant quelques commandes additionnelles afin de désarchiver/compiler/installer l'outil désiré.  
L'objet det article n'est pas de vous expliquer comment les installer, je vous suggère très fortement d'aller faire un tour sur le site officiel des outils suivant:  
- Traefik  
- Sozu
- GateJs

## Quelles sont les principales key-features de chaque outils ?
Je ne reviendrais pas sur la feature principale des serveurs HTTP, je vais juste évoquer leurs fonctions additionnelles ainsi que celles proposées par **Sozu**, **Traefik**, **Varnish** et **Squid**.	