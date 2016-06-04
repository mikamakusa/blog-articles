+++
date = "2016-05-28"
draft = false
title = "Le Labo #11 | Gitlab - Intégration Continue"
+++

# Qu'est ce que Gitlab ?  
Au départ, Gitlab est une simple plateforme de versionning, tout comme peut l'être BitBucket, Github ou bien Git.  
Il permet :  

- de gérer des dépôts Git ainsi que les utilisateurs et leurs droits d'accès aux dépôts  
- l'authentification peut utiliser deux facteurs et la connexion à un annuaire LDAP4  
- de gérer l'accès par branche à un dépôt4  
- d'effectuer des examens de code et renforcer la collaboration avec les demandes de fusion  
- que chaque projet puisse avoir un outil de ticket et un wiki.


# De l'intégration continue ?
C'est un ensemble de pratiques utilisées en génie logiciel consistant à vérifier, à chaque modification de code source, que le résultat des modifications ne produit pas de régression dans l'application développée.  
Le concept a pour la première fois été mentionné par Grady Booch et se réfère généralement à la pratique de l'extreme programming.  
Le principal but de cette pratique est de détecter les problèmes d'intégration au plus tôt lors du développement.  
De plus, elle permet d'automatiser l'exécution des suites de tests et de voir l'évolution du développement du logiciel.
(Merci à Wikipedia pour la définition)


# Comment faire de l'Intégration Continue avec Gitlab ?
Gitlab propose un outil extrêment simple, et accessoirement développé en Go, permettant de lié un depot sur Gitlab à une ou plusieurs machines.  
Cet outil, gitlab-ci, se présente sous la forme d'un *runner* déployable sur n'importe quelle machine ou sur un conteneur **Docker** :


# Comment installer le runner ?

- Debian  
`curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash`  
`apt-get install -y gitlab-ci-multi-runner`

- CentOS  
`curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash`  
`yum install -y gitlab-ci-multi-runner`

- Via un conteneur **Docker**  
`docker run -d --name gitlab-runner -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:latest`

## L'étape **Register**  
Une fois le *runner* installé, l'étape suivante avant toute tentative d'Intégration Continue via Gitlab est l'enregistrement.
Pour cela, gitlab-ci propose les deux commandes à utiliser selon l'utilisation que vous en ferez par la suite.

- Dans le cas d'un *job unique*, utilisez la commande : `gitlab-runner register`  
- Dans le cas de *multiple jobs*, utilisez la commande : `gitlab-ci-multi-runner register`  

Ces deux commandes permettent de configurer le *runner* et de le mettre en relation avec l'instance Gitlab. Elles sont intéractive et vont poser les questions suivantes (nous verrons par la suivante comment éviter qu'elles le soient en intégrant les paramètres nécessaire à sa configuration directement lors de l'appel de la commande) :

	    Please enter the gitlab-ci coordinator URL (e.g. http://gitlab-ci.org:3000/ )
	    *https / http://url_gitlab/ci*
	    Please enter the gitlab-ci token for this runner
	    *token fourni par gitlab*
	    Please enter the gitlab-ci description for this runner
	    *nom du runner*
	    INFO[0034] fcf5c619 Registering runner... succeeded
	    Please enter the executor: shell, docker, docker-ssh, ssh?
	    *nom de l'executor - nous verrons cela juste après*

### Le token
Dans l'optique de l'intégration continue souhaitée par Gitlab, l'instance Gitlab vous fournira un token afin d'effectuer la liaison entre le runner et l'instance en elle même.  
Ce token est disponible dans **Admin Area/Runners**

### L'executor
Le *runner* Gitlab-ci peut être configuré pour fonctionner avec de nombreux types d'*executor* parmi les suivants :  

- **shell**  : Permet de provisionner directement la machine sur laquelle le *runner* s'exécute,  
- **docker** : Permet de provisionner des conteneurs docker sur la machine sur laquelle le *runner* s'exécute,  
- **docker-ssh** : Permet de provisionner des conteneurs docker sur la machine sur laquelle le *runner* s'exécute, si le conteneur est aussi accessible en ssh,  
- **ssh** : Permet de provisionner directement une machine accssible en ssh.  

Pour les deux derniers, l'opération **register** demandera les informations suivantes : **user**, **password**, **port**, **host** ainsi que **identity_file** (la clé publique de SSH)

### Le projet
Une fois le runner enregistré sur l'instance Gitlab, il faut pouvoir démarrer un *build* de tel ou tel projet sur le serveur associé au *runner*.  
C'est pour cela qu'il faut associer le runner au projet en question.  
Dans **Admin Area/Runners**, les runners sont listés et l'on peut y voir leur état, s'ils sont *shared* ou *specific*.  
Pour modifier leur état, cliquer sur **Edit** et leur associer un projet un fois arrivé sur la page suivante.  
Selon toutes les documentations disponibles, peu d'administrateurs font le choix d'associer un projet/dépot à un *runner*...mais, selon ma propre expérience, c'est **obligatoire**, sans quoi le build ne démarre pas.


# Comment démarrer un build ?
Une fois toutes les étapes précédentes effectuées, nous allons pouvoir nous concentrer sur les *builds* et sur le principal composant : le fichier **gitlab-ci.yml**.  
Je vous invite à consulter la documentation disponible ici : 

	http://docs.gitlab.com/ce/ci/yaml/README.html

Une fois le fichier **gitlab-ci.yml** uploadé dans le projet sur l'instance gitlab, le build va démarrer automatiquement.

## Builder un microservice
Il est possible d'effectuer un build de microservice (un conteneur **Docker**) de deux manière différentes : 

- **En intégrant un appel à un image Docker construite localement à l'aide d'un Dockerfile** : 

###### Le dockerfile exemple

		FROM ubuntu

		RUN apt-get update -y
		RUN apt-get install apache2 make build-essential -y
		RUN apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql curl wget git -y
		RUN apt-get install php5 libapache2-mod-php5 php5-mcrypt php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-sqlite php5-tidy php5-xmlrpc php5-xsl -y

		RUN apt-get install ruby1.9.1-dev libsqlite3-dev -y
		RUN gem install mailcatcher

		RUN export DEBIAN_FRONTEND=noninteractive
		RUN curl -sS https://getcomposer.org/installer | php

		EXPOSE 22 80
		CMD ["/usr/sbin/apache2ctl -D FOREGROUND"]

Pour construire l'image **Docker** :  
`docker build -t php56 .`


###### Le fichier gitlab-ci.yml exemple
```yaml
image: php56

before_script:
    - service mysql start
    - echo "create database laravel" | mysql -u root
    - mysql -u root laravel < "sql/laravel.sql"
    - echo "SET PASSWORD FOR 'root'@'localhost' = PASSWORD('root');" | mysql -u root
    - php /composer.phar install --no-progress
job1:
    script:
        - php artisan migrate
        - vendor/bin/phpunit tests
```


- **En Intégrant l'installation des packages necessaires dans le fichier gitab-ci.yml** :

###### Le fichier gitlab-ci.yml exemple
```yaml
	before_script:  
	  - apt-get install wget

	stages:  
	  - test  
	  - deploy  
	  - build

	deploy:2.1:
	  image: debian:latest  
	  script:  
		  - apt-get install apache2 make build-essential -y  
		  - apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql curl wget git -y  
		  - apt-get install php5 libapache2-mod-php5 php5-mcrypt php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-sqlite php5-tidy php5-xmlrpc php5-xsl -y

	deploy:2.2:  
	  image: ubuntu:latest  
	  script:  
	  - apt-get install apache2 make build-essential -y  
	  - apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql curl wget git -y  
	  - apt-get install php5 libapache2-mod-php5 php5-mcrypt php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-sqlite php5-tidy php5-xmlrpc php5-xsl -y
```

De cette manière, on peut passer plusieurs étapes intérmédiaires telles que la création du dockerfile et la construction de l'image.


# Avant de terminer...
Il existe de nombreux applications différentes comme, par exemple, le déploiement de toute une architecture de microservices ou de serveurs virtuels...un peu à la manière de Terraform.  
Même si Terraform permet d'effectuer bien plus que la création de serveurs.
Le prochain article sera consacré à **Taïga et à son intégration avec Gitlab**.