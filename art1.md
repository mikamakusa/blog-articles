+++
date = "2016-06-29T15:29:49+02:00"
draft = false
title = "Hugo...en prod (sur Ubuntu 16.04)"

+++

## Petite introduction
Pourquoi refaire un article concernant mon blog ?  
J'avais juste envie de trouver une autre voie que celle prise initialement...et me passer d'un conteneur **Docker** pas vraiment sécurisé.  
Donc, j'ai cherché un moyen de démoniser **Hugo** mais ca ne semblait pas possible.  
Donc, j'ai continué à chercher et je suis tombé sur une voie détournée : **Hugo** derrière **Nginx** et c'est cela que je vais présenter dans cet article.


## De quoi ais-je besoin ?
Pour parvenir à mes fins, je vais avoir besoin des éléments suivants : 

- **Hugo** (captain obvious...),
- **Nginx**
- **Git**

De plus, je souhaitais automatiser la création du serveur qui allait héberger le tout...J'ai donc utilisé **Packer** et **Terraform** (oui, j'avoue, je suis une feignasse).


## C'est parti !!!
### Les packages
Je ne pense pas que je vais avoir besoin d'expliquer la création d'une image et son déploiement à l'aide de **Packer** puis **Terraform**, donc je vais passer directement aux packages nécessaires parmi les suivants : 

- unzip 
- nginx 
- python-pip 
- golang 
- hugo 
- git 
- wget 
- curl 
- bison 
- make 
- binutils 
- gcc 
- build-essential 
- python-software-properties

### Le blog
Je vous suggère d'aller faire un tour sur le site officiel d'**Hugo** pour la documentation officielle. Toujours-est-il que, de mon côté, je vais générer les dossiers et le fichier de configuration de mon blog à l'aide de la commande `hugo new site <name> <folder>`.  

Ensuite, je vais télécharger les thèmes disponibles :  
`git clone --recursive https://github.com/spf13/hugoThemes <folder>/themes`

Enfin, il ne me reste plus qu'a créer le dépot **git** contenant les dossiers et fichiers du blog :  
`cd <folder>`  
`git init`  
`git config --global user.name "<name>"`  
`git config --global user.email "<email>"`  
`git add .`  
`git commit -m 'initial commit'`  
`git push origin master`  

Avant d'effectuer les action *add*, *commit* et *push*, je vous suggère très fortement de modifier le fichier **config.toml**...Mais vous pouvez toujours le faire après.

Une fois les précédentes opérations effectuées, on exécute les commandes suivantes :  
`cd ~/<folder>/ && hugo --destination=<folder1>`  
`rm -rf /var/www/html/*.*`  
`cp -R ~/<folder>/<folder1>/. /var/www/html`


### La suite ?
Un outil d'intégration continue aurait parfaitement sa place dans tout cela. Cela permettrait d'automatiser les trois dernières commandes à chaque évenements **git**.  
Si votre outil de versionning est **gitlab**, je ne peux que vous suggérer d'utiliser **gitlab-ci** directement sur votre serveur.  
Dans d'autres cas, n'hésitez pas à explorer d'autres voies comme **Jenkins** ou **Travis** (personnelement, je reste sur le premier).
