+++
date = "2017-06-26T15:29:49+02:00"
draft = "false"
title = "Le Labo #24 | Une image Docker...sans Dockerfile"

+++

Quoi de plus basique pour un utilisateur de Docker que de créer une image ?  
En général, on utilise un Dockerfile pour cela, mais il est tout à fait possible d'explorer d'autres voies :

- docker commit : créer l'image docker à partir d'un conteneur pré-existant et personnalisé
- Salt + dockerng : créer une image docker à l'aide de Saltstack

C'est cette dernière voie que nous allons expliquer dans cet article.  

# Le principe
Le principe général du module **docker** pour **saltstack** est l'utilisation de l'API de **Docker** à l'aide de la couche **Salt** qui, elle, gère tout ce qui est pillarisation, templates, runners, modules, etc...  
Selon ce mode de fonctionnement, il devient même possible de générer des Dockerfile (ce qui ne sera pas le but) ou de créer des images propres et prêtes à l'emploi.  
Et ceci de la même manière qu'il est possible de configurer toute son infrastructure à l'aide de **Salt**, de gérer un cluster **Nomad** ou **Kubernetes**.  

Ne surtout pas oublier un détail : la version de **Saltstack** que vous allez utiliser.
Seule la toute dernière version de **Saltstack** propose les commandes adéquates à la création d'images Docker (soit la 2016.11.5)

# Entrons dans le vif du sujet
**Saltstack** est composé de : 

- Modules: il ne sont utilisables qu'en ligne de commande et disposent souvent de possibilités plus étendues que les States. Dans le cas de Docker, la différence est assez flagrante.  
En effet, la majorité des commandes de base de docker est supportée (parfois en double) dans le module dockerng
- States: il seront utilisés pour créer l'image.  
- Pillar: Les pillars permettent de remplir de façon dynamique les states.

# Let's play
Maintenant que le décor est posé, passons à la création de l'image.  
Il faut savoir que ceci ne fonctionne qu'avec la toute dernière itération de **SaltStack**, soit la *2016.11.5*, même si l'on installe la toute dernière version du plugin Docker avec une autre version de **SaltStack**, vous ne pourrez pas en bénéficier.  
Par contre, il faudra faire attention avec cette version...il se peut que des states qui fonctionnent avec les versions plus anciennes génèrent des erreurs de déploiement (notamment pour les archives tar).

## De quoi aurons nous besoin dans notre image ?
Dans l'image que nous allons créer à l'aide de **SaltStack**, nous aurons : 

- JDK : Evidemment, des "inconvénients" légaux vont nous empêcher d'utiliser la version d'Oracle de JDK, donc nous utiliserons **OpenJDK**...Ce sera la version 8
- Tomcat : En version 8...l'objectif sera, biensûr, de déployer une webapp java.
- Hazelcast
- Zookeeper

Potentiellement, nous pouvons même créer 3 images différentes avec tout ce dont nous avons besoin.

## Le pillar
Des pillars ?  
Biensûr, après tout il est tout à fait possible de provisionner les states **Salt** avec des infos différentes selon les versions de logiciels que nous voulons intégrer dans les images...tout comme pour les dossiers d'installation de chaque soft.

```
base:
  base_dir: <base_dir>
  type: tar.gz
  user: <user>
  group: <group>
  options: xzf
app:
  apps:
    java:
      url: <url>
      archive: <archive>
      md5: <md5>
      directory: <directory>
    logstash:
      url: <url>
      archive: <archive>
      md5: <md5>
      directory: <directory>
    zookeeper:
      url: <url>
      archive: <archive>
      md5: <md5>
      directory: <directory>
    hazelcast:
      url: <url>
      archive: <archive>
      md5: <md5>
      directory: <directory>
    tomcat:
      url: <url>
      archive: <archive>
      md5: <md5>
      directory: <directory>
```

Nous pourrions tout à fait détailler beaucoup plus le pillar regroupant les infos des composants à installer, surtout si il y a besoin de certains détails...mais ce ne sera pas le cas ici. De plus, afin d'éviter tout risque de plantage, chaque section du pillar doit comporter les mêmes keywords.  

## Le state
Une fois le pillar défini, passons au state.  
C'est grace à lui que les images pourront être créées, elles seront au nombre de 4 : 

- Image *Base* : Elle contiendra *java* et *Logstash*,
- Image *Zookeeper* : avec l'image *Base* + *zookeeper*,
- Image *Hazelcast* : tout comme la précédente + *hazelcast*,
- Image *Tomcat* : *tomcat* + *Base*

**Image Base**
```
{% set app = pillar['get']('app') %}
{% set init = pillar['get']('base') %}

{{ init.group }}_group_creation:
  group.present:
    - name: {{ init.group }}

{{ init.user }}_user_creation:
  user.present:
    - name: {{ init.user }}
    - shell: /bin/bash
    - home: /home/{{ init.user }}
    - groups:
      - {{ init.group }}

actions_on_packages:
  pkg.installed:
    - pkgs:
      - wget
      - unzip
      - initscripts
      - sudo
      - crontabs
      - cronie
      - cronie-anacron
  pkg.removed:
    - name: iptables.service

{{ init.base_dir }}_creation:
  file.directory:
    - user: {{ init.user }}
    - group: {{ init.group }}
    - dir_mode: 755
    - file_mode: 644
    - makedirs: True

Java_installation:
  archive.extracted:
    - name: {{ init.base_dir }}
    - source: {{ app.apps.java.url }}/{{ app.apps.java.archive }}.{{ app.apps.base.type }}
    - source_hash: md5={{ app.apps.java.md5 }}
    {% if salt['grains.get']('saltversion' == '2016.11.4')%}
    - options: {{ ininit.options }}
    {% else %}
    - tar_options: {{ ininit.options }}
    {% endif %}
  file.symlink:
    - name: {{ init.base_dir}}/{{ app.apps.java.directory }}
    - target: {{ init.base_dir }}/<java_directory>
    - makedirs: True
    - force: True
    - user: {{ init.user }}
    - group: {{ init.group }}

Logstash_installation:
  archive.extracted:
    - name: {{ init.base_dir }}
    - source: {{ info.logstash.url }}/{{ app.apps.logstash.archive }}.{{ init.type }}
    - source_hash: md5={{ app.apps.logstash.md5 }}
    {% if salt['grains.get']('saltversion' == '2016.11.4')%}
    - options: {{ init.options }}
    {% else %}
    - tar_options: {{ init.options }}
    {% endif %}
  file.symlink:
    - name: {{ init.base_dir}}/{{ app.apps.logstash.directory }}
    - target: {{ init.base_dir }}/<logstash_directory>
    - makedirs: True
    - force: True
    - user: {{ init.user }}
    - group: {{ init.group }} 
```

Maintenant que le state permettant de créer l'image Docker est prêt, passons à la création en elle même. Ceci est faisable selon deux methodes différentes :

- La ligne de commande Salt : `salt "target" dockerng.sls_build <image_name> base=<image_base> mods=<sls_file>`

## Que fait la commande sls_build ?
La commande sls_build (qui, je le rappelle, n'est disponible qu'à partir de la version 2016.11.4 de **SaltStack**) crée un conteneur, exécute toute les étapes du fichier spécifié au niveau du keyword *mods* et commit l'image. Ce qui fait que l'image en question n'a qu'un seul *layer* au lieu d'une cinquantaine (comme ce que l'on peut trouver, parfois, sur *docker hub*)...et potentiellement de créer des images plus propre.  
Ici, je n'ai expliqué que la création d'une seule image Docker (l'image de base), mais vous pourrez sans aucune doute comprendre comment faire pour les autres images.  
Pour rappel, la commande **dockerng.sls_build** demande de préciser quelle image de base vous souhaitez utiliser pour la création de l'image...En fait, du moment que l'image en question contient python, vous pourrez utiliser **Saltstack** dessus et y installer ce que vous voudrez.