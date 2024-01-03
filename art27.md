+++
date = "2024-01-03T22:23:00+02:00"
draft = false
title = "Plus ou moins inutile donc forcément indispensable : OnLogs"
+++ 

J'ai eu l'information assez récemment provenant de l'application mobile **DevOps Tools** avec laquelle j'effectue une grande partie de ma veille technologique. La notification est arrivée il y a quelques jours mais avec l'actualité de fin d'année relativement chargée, dont les nouveautés les plus fraiches des Cloud Provider, du monde des conteneurs et de la cybersécurité, je n'ai pas pris le temps de vous le présenter.  

Pour celles et ceux qui connaissent **Docker** et son CLI, visionner les logs est un jeu d'enfants...Après tout, avec la commande `docker logs <container_name>`, on peut afficher les logs sans trop de difficultés.  

Mais pour ceux qui ne sont pas familiarisé avec le CLI et cette fameuses commande très pratique, une équipe de développeurs a pensé a eux, il s'agit de [**devforth.io**](https://devforth.io/). Ces derniers ont mis a disposition de la communauté : [**OnLogs**](https://github.com/devforth).  

# De quoi s'agit-il ?
Il s'avère que c'est un client léger de visualisation de logs de conteneurs Docker...une application web développée en **Golang** ainsi qu'avec le **framework svelte.js** dans le but de bénéficier du maximum de performance tout en conservant une taille d'image et de consommation mémoire très réduite.  
Peut être démarré de deux manières différentes :  
- La commande `docker run`,  
- **docker-compose**  

# les modes de démarrage
## docker-compose
Un exemple sans **Traefik** :  
```yaml
  onlogs:
    image: devforth/onlogs
    restart: always
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /var/lib/docker/containers:/var/lib/docker/containers # if you want to delete duplicating logs from docker
     - /etc/hostname:/etc/hostname
     - onlogs-volume:/leveldb

volumes:
  onlogs-volume:
```

Dans cet exemple, **onlogs** va démarrer avec les paramètres suivants :  
- Login : admin,  
- Pas de mot de passe,  
- Port : 2874,  
- **Onlogs** en mode serveur.  

Un autre exemple en mode **agent** : 
```yaml
  onlogs:
    image: devforth/onlogs
    restart: always
    environment:
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=<any password>
      - PORT=8798
      - AGENT=true
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /var/lib/docker/containers:/var/lib/docker/containers # if you want to delete duplicating logs from docker
     - /etc/hostname:/etc/hostname
     - onlogs-volume:/leveldb

volumes:
  onlogs-volume:
```

Le principe du mode **agent** est de démarrer un service **onLogs** qui sera requêté par un serveur principal, un peu à la manière de la stack **Elastic** et de ses **Beats** qui transmettent les logs à **ElasticSearch** pour un affichage dans **Kibana**.

## docker run
Même exemple avec `docker run` :  
```bash
docker run --restart always -e ADMIN_USERNAME=admin -e PASSWORD=<any password> -e PORT=8798 -e AGENT=true \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /var/lib/docker/containers:/var/lib/docker/containers \
    -v /etc/hostname:/etc/hostname \
    -v onlogs-volume:/leveldb
```

![img](/images/onlogs.png)  

# Conclusion
Comme je l'ai déjà évoqué plus tôt : Peux ceux qui sont familiers de Docker, cet outil sera complètement inutile...Pour les autres, je ne me prononce pas, ils sont peut-être déjà outillés autrement.  
A voir le chemin pris par l'équipe de développement.