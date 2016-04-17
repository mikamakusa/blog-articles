+++
date = "2015-12-04T18:21:49+02:00"
draft = false
title = "Le Labo #5 | Mon Blog, avec Docker, Go et Hugo"

+++

# Pourquoi ?
Ca ne paraît pas comme ça, mais ce blog est en fait un conteneur Docker.

Wordpress me semblait beaucoup trop, je n'avais pas besoin d'une usine à gaz pour un simple blog, il me fallait quelque chose de plus simple et réactif.  
Pour cela, j'ai trouvé Hugo.

Hugo est tout simple, une commande pour créer le site, une autre pour démarrer le serveur, pas besoin d'installer Apache, Nginx ou cherokee...pas besoin non plus d'une base de données...juste des pages à ajouter dans un dossier et elles s'affichent dans le navigateur.  
C'est du "markdown", ça ressemble à du wiki en un chouya plus beau.

Par contre, lorsque l'on quitte la session SSH, le serveur Hugo s'arrête...donc il fallait une pirouette.  
La pirouette ? Docker...mais pas n'importe comment : Un dockerfile personnalisé.

Je vais juste expliquer les commandes que j'ai eu à lancer pour avoir le tout fonctionnel. Si vous souhaitez savoir comment créer un dockerfile, je vous suggère de vous rendre sur le [document relatif à ce sujet](http://www.ageekslab.com/docker/docker3/).

# Comment faire ?
## Déployer le conteneur
Tout d'abord, je tiens à remercier ceux qui maintiennent l'image Docker "rastasheep/ubuntu-sshd:14.04", elle m'est très utile parce qu'elle propose SSH directement intégré et ça me permet de travailler dessus plus facilement.

- Démarrer le conteneur : `docker run -itdP -p 80:80 ---name [nom du conteneur] rastasheep/ubuntu-sshd:14.04`

- Se connecter au conteneur après avoir lancé la commande `docker ps` pour obtenir le port de l'hôte en rapport avec le port 22 du conteneur : `ssh localhost -p [n° port]`

## Installer Golang
- Installer Golang et les outils nécessaires : `apt-get update -y` et `apt-get install -y curl git mercurial make binutils bison gcc build-essential python-software-properties golang vim`

- Insérer les lignes suivantes dans le fichier /root/.bashrc : 
		
		export GOROOT=/usr/lib/go
		export GOBIN=/usr/bin/g
		export GOPATH=/root/gocode
		export PATH=$PATH:$GOPATH/bin

- Tapez les commandes suivantes : `mkdir /root/gocode` et `exec bash`

## Installer Hugo
Pour installer Hugo, il faut lancer les commandes suivantes : 

- Télécharger Hugo : `wget https://github.com/spf13/hugo/releases/download/v0.15/hugo_0.15_linux_amd64.tar.gz`
- Décompresser l'archive : `tar xzf hugo_0.15_linux_amd64.tar.gz`
- Renommer le dossier et copier dans /usr/local/bin : `cd hugo_0.15_linux_amd64 && mv hugo_0.15_linux_amd64 hugo && mv hugo /usr/local/bin/`

## Déployer le site
Je vais passer sur la customisation, après tout ce n'est pas le plus intéressant et vous trouverez de nombreux tutoriels et exemples sur le site de Hugo.

Lorsque le site est prêt a être déployé, retournez sur l'hôte et tapez la commande suivante : 
`docker exec -d [nom du conteneur] hugo server -w -t [theme] --baseUrl=http://[URL] --port=80 --appendPort=false --bind=0.0.0.0`
