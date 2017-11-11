+++
date = "2017-11-11T15:29:49+02:00"
draft = "false"
title = "Le Labo #25 | Déploiement automatisé d'une instance Jenkins"

+++

Dans l'univers **DevOps**, l’intégration continue (ou **CI** pour les intimes) s'est imposée en tant que pratique à part entière dans l'optique de construire un projet de manière la plus agile possible, en vérifiant le code à chaque étape et en détectant les éventuelles régressions.  

L'intérêt de cette pratique repose souvent sur la mise en place d'une brique logicielle permettant l'automatisation des tâches de compilation, les tests unitaires et fonctionnels, la validation produit ainsi que les tests de performance.  
Ceci afin de produire un ensemble de résultats consultables par tous sur une interface utilisateur.  
Pour réaliser cela, il faut à minima :
- Un outil de versionning de code,  
- Des développeurs à l'aise avec l'outil précédemment cité,  
- Un outil de **CI** correctement configuré.

Les outils de **CI** sont nombreux, on peut citer parmi les plus connus :
- **Jenkins** (qui sera utilisé dans cet article)  
- **Bamboo** (de la suite Atlassian)  
- **TeamCity**  
- **CruiseControl**  
- **Travis**
- **CircleCI**

## Problématique.
L'idée générale étant la suivante :  
- Automatiser l'installation  
- La configuration  
- Et la sécurisation d'un outil de **CI**
Le tout à l'aide d'un outil de gestion de configuration.

Maintenant que l'idée générale de l'article est définie, je vais pouvoir en expliquer le déroulement.  

## Ce dont nous aurons besoin.
Pour tout cela, nous aurons besoin d’un serveur linux sur lequel sera installé *vim* (pour l’édition de fichiers de configuration) et *curl* (pour la récupération du binaire d’installation de *Saltstack*). Le reste sera installé par la suite à l’aide de **SaltStack**.
Je vais procéder ainsi afin d’éviter d'avoir à monter un serveur spécial, le service **Saltstack** sera en mode **masterless**[voir ici](https://docs.saltstack.com/en/latest/topics/tutorials/quickstart.html).  

Par contre, si vous souhaitez installer **Saltstack** en mode **master/minion**, n'hésitez surtout pas à suivre la documentation officiel [ici](https://docs.saltstack.com/en/latest/topics/installation/index.html#quick-install)

## Comment configurer tout cela ?
En fait, l'installation du service **Salt** en mode **masterless** est d'une simplicité désarmante grâce à la commande suivante :  
<code>curl -L https://bootstrap.saltstack.com -o bootstrap_salt.sh</code>.  

Pour ce qui est de l'installation de *Jenkins* et de *Nginx*, je peux vous suggérer d'utiliser les formules Salt qui sont dédiées à ces services, et disponibles ici :  
- [formule Jenkins officielle](https://github.com/saltstack-formulas/jenkins-formula)    
- [formule Nginx Officielle](https://github.com/saltstack-formulas/nginx-formula)

Cependant, par soucis de maîtriser la totalité du processus d’installation, j’ai préféré écrire moi même mes *states* sans utiliser les formules décrites un peu plus haut et, pour pouvoir les utiliser, j’ai configuré mon **salt minion** de cette manière :

```
# Salt n’ira pas chercher autre part que localement les formules/states/pillar à utiliser
file-client: local
# Lui indiquer où exactement se trouvent les states
file_roots:
  base:
	- /srv/state
# Si vous créez des pillar (ce qui est souvent recommandé), lui indiquer aussi où ils sont
pillar_roots:
  base:
	- /srv/pillar
# Le log level...au cas où vous souhaiteriez vérifier que tout se passe bien (all = très bavard)
log_level: all
```

Ce fichier appelé *minion* est dans le dossier **/etc/salt**, n'oubliez surtout pas de redémarrer **SaltStack** afin que toutes modifications effectuée dans ce fichier soient bien prises en compte.

### La structure du dossier /srv
```
- **/srv**
  - **state**
	- top.sls
	- **jenkins**
  	- **config**
    	- **template**
      	- config.xml.jinja
      	- init.groovy.jinja
    	- init.sls
  	- **install**
    	- **template**
      	- jenkins.jinja
    	- init.sls
  	- **securise**
    	- **template**
      	- jenkins.conf.jinja
    	- init.sls
  	- **slave**
    	- init.sls
  	- init.sls
  - **pillar**
	- top.sls
	- **jenkins**
  	- init.sls
```

### Le pillar
Le pillar est une liste de clés et de valeurs qui seront injectées dans les states et les templates jinja utilisés durant l'exécution de la commande `salt-call state.apply jenkins`

Voici un aperçu du pillar utilisé :
```
jenkins:
  master:
	args: -Djenkins.install.runSetupWizard=false
	url: jenkins-ci.org
	config_folder: /etc/default/jenkins
	pkgs: jenkins
	source_folder: /etc/apt/sources.list.d/
	deb_apt_source: jenkins-ci.list
	folder: /var/lib/jenkins
	platform: debian
	user: jenkins
	group: jenkins
	home: /var/lib/jenkins
  java:
	pkg: openjdk-8-jre-headless
	args: -Djava.awt.headless=true
	executable: /usr/bin/java
  nginx_conf:
	http_port: 80
	https_port: 443
	ssl_cert: /etc/nginx/cert
	access_log: /var/log/nginx/jenkins.access.log
	config_file: jenkins.conf
	domain: test.local
  docker:
	install:
  	deb_apt_source: docker.list
  	url: https://download.docker.com/linux/debian/gpg
  	os: debian
  	release: stretch
  	type: stable
  	key: 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
  slaves:
	slave1:
    	name: slave1
    	description: test
    	remotefs: /home/
    	executors: 2
    	mode: NORMAL
    	address: 172.28.128.6
    	port: 22
    	credentialid: 1
    	javapath: /usr/bin/java
```

### Les states
Je vais créer les quatres *states* suivants :
- install (l'installation de jenkins et des pré requis)
- config (la configuration de jenkins incluant la création d'un premier job)
- securise (dans lequel sera décrit l'installation de nginx/openssl et la configuration reverse-Proxy)
- slave (la création d'un node)

#### Le state Install
```
{% set jenkins = salt['pillar.get']('jenkins:master') %}
{% set java = salt['pillar.get']('jenkins:java') %}
{% set fqdn = salt['grains.get']('fqdn') %}
{% if "jenkins-master" in fqdn %}
Jenkins_Group_and_User:
  group.present:
	- name: {{ jenkins.group }}
	- gid: 7648
  user.present:
	- fullname: {{ jenkins.user }}
	- home: {{ jenkins.home }}
	- shell: /bin/bash
	- gid: {{ jenkins.group }}
	- remove_groups: False
JDK8_install:
  pkg.installed:
	- pkgs:
  	- {{ java.pkg }}
jenkins_install:
  pkgrepo.managed:
	- humanname: Jenkins Package Repository
	- file: {{ jenkins.source_folder }}{{ jenkins.deb_apt_source }}
	- name: deb http://pkg.{{ jenkins.url }}/{{ jenkins.platform }}-stable binary/
	- key_url: https://{{ jenkins.url }}/{{ jenkins.platform }}/jenkins-ci.org.key
	- require-in:
  	- pkg: {{ jenkins.pkgs }}  
  pkg.installed:
	- name: {{ jenkins.pkgs }}
  service.dead:
	- name: {{ jenkins.pkgs }}
Jenkins_Config_Modify:
  file.managed:
	- name: {{ jenkins.config_folder }}
	- source: salt://jenkins/install/template/jenkins.jinja
	- template: jinja
	- ignore_if_missing: True
  service.running:
	- name: {{ jenkins.pkgs }}
	- enable: True
{% endif %}
```
L'action *jenkins_config_modify* consiste à remplacer le fichier *jenkins* du dossier */etc/default/jenkins* par celui qui se trouve dans le dossier template et qui contiendra, au final, les valeurs définies dans le pillar.

#### Le state Config
```
{% set jenkins = salt['pillar.get']('jenkins:master') %}
{% set fqdn = salt['grains.get']('fqdn') %}
{% if "jenkins-master" in fqdn %}
Jenkins_Folder:
  file.directory:
	- name: {{ jenkins.folder }}/jobs/seeds/
	- user: {{ jenkins.user }}
	- group: {{ jenkins.group }}
	- dir_mode: 755
	- file_mode: 644
	- recurse:
  	- user
  	- group
  	- mode
	- makedirs: True
Jenkins_Stop:
  service.dead:
	- name: jenkins
Jenkins_xml_config_file_for_Seed_Project:
  file.managed:
	- name: /{{ jenkins.folder }}/jobs/seeds/config.xml
	- source: salt://jenkins/config/template/config.xml.jinja
	- template: jinja
	- user: root
	- group: root
	- mode: 644
Jenkins_goovy_file_plugins_auto_install:
  file.managed:
	- name: /{{ jenkins.folder }}/init.groovy
	- source: salt://jenkins/config/template/init.groovy.jinja
	- template: jinja
	- user: root
	- group: root
	- mode: 644
Jenkins_restart:
  service.running:
	- name: jenkins
{% endif %}
```
A l'aide de ce state, je crée un premier job et je démarre l'installation des plugins.

#### Le state Securise
```
{% set jenkins = salt['pillar.get']('jenkins:master') %}
{% set java = salt['pillar.get']('jenkins:java') %}
{% set nginx = salt['pillar.get']('jenkins:nginx_conf') %}
{% set fqdn = salt['grains.get']('fqdn')%}
{% if "jenkins_master" in fqdn %}
Web_Server_Install:
  pkg.installed:
	- pkgs:
  	- nginx
  	- openssl
SSL_Cert_Generation:
  cmd.run:
	- name: openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout {{ nginx.ssl_cert }}.key -out {{ nginx.ssl_cert }}.crt -subj "/C=FR/ST=Paris/L=Paris/O=Jenkins/OU=Jenkins/CN={{ nginx.domain }}"
  service.dead:
	- name: nginx
Nginx_Reverse_Proxy_Configure:
  file.managed:
	- name: /etc/nginx/sites-available/{{ nginx.config_file }}
	- source: salt://jenkins/securise/templates/jenkins.conf.jinja
	- template: jinja
	- user: root
	- group: root
Nginx_Activate_New_Site:
  file.symlink:
	- name: /etc/nginx/sites-enabled/{{ nginx.config_file }}
	- target: /etc/nginx/sites-available/{{ nginx.config_file }}
	- makedirs: False
  service.running:
	- name: nginx
	- enable: True
Stop_Jenkins:
  service.dead:
	- name: {{ jenkins.pkgs }}
Jenkins_config_for_Reverse_Proxy:
  file.replace:
	- name: {{ jenkins.config_folder }}
	- pattern: 'JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT'
	- repl: 'JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --httpListenAddress=127.0.0.1 -ajp13Port=$AJP_PORT'
	- ignore_if_missing: True
  service.running:
	- name: {{ jenkins.pkgs }}
{% endif %}
```

Dans ce state, j'install nginx et openssl, je configure le reverse proxy nginx et dans le fichier de configuration de jenkins.

#### Le state Slave
```
{% set jenkins = salt['pillar.get']('jenkins') %}
{% set master = jenkins.master %}
{% set slave = jenkins.slaves %}
{% set docker = jenkins.docker %}
{% set fqdn = salt['grains.get']('fqdn') %}
{% if 'jenkins-master' in salt['grains.get']('fqdn') %}
{% for info, arg in salt['pillar.get']('jenkins:slaves', {}).iteritems() %}
  Jenkins_add_slave_{{ arg.name }}_Directory:
	file.directory:
  	- name: {{ master.folder }}/nodes/{{ arg.name }}
  	- user: {{ master.user }}
  	- group: {{ master.group }}
  	- dir_mode: 755
  	- file_mode: 644
  	- makedirs: True
  Jenkins_add_slave_{{ arg.name }}_infos:
	file.managed:
  	- name: {{ master.folder }}/nodes/{{ arg.name }}/config.xml
  	- source: salt://jenkins/slave/files/config.xml
  	- template: jinja
  	- user: {{ master.user }}
  	- group: {{ master.group }}
  	- mode: 644
{% else %}
{% else %}
Add_Docker_Group:
  group.present:
	- name: docker
Jenkins_Group_and_User:
  group.present:
	- name: {{ master.group }}
	- gid: 7648
  user.present:
	- fullname: {{ master.user }}
	- home: {{ master.home }}
	- shell: /bin/bash
	- groups:
  	- {{ master.group }}
  	- docker
	- remove_groups: False
Docker_Prerequisites:
  pkg.installed:
	- pkgs:
  	- apt-transport-https
  	- ca-certificates
  	- curl
  	- gnupg2
  pkgrepo.managed:
	- humanname: Docker Package Repository
	- file: {{ master.source_folder }}{{ docker.install.deb_apt_source }}
	- name: deb http://pkg.{{ docker.install.url }}/{{ docker.install.os }} {{ docker.install.release }} {{ docker.install.type }}
	- keyid: {{ docker.install.key }}
	- enabled: True
	- refresh_db: True
Docker_install:
  pkg.installed:
	- name: docker-ce
  service.running:
	- name: docker
	- enable: True
{% endfor %}
{% endif %}
```

#### Le state racine
Il reste un state que je n'ai pas encore expliqué, il s'agit de celui qui se trouve à la racine du dossier **/srv/state/jenkins**, ce petit fichier pourrait tout à fait rester vide, il suffirait juste de préciser dans le fichier **/srv/state/top.sls** quelles sont les actions à appliquer sur tel ou tel serveur, mais dans mon cas, je voulais appliquer le state **slave** aussi bien sur le jenkins maître que sur les nodes en fonction de la clé *fqdn* des *grains* de **Saltstack**.  
C'est pour cela que mon state racine *jenkins* contient ce qui suit :
```
{% set fqdn = salt['grains.get']('fqdn') %}
{% if "jenkins-master" in salt ['grains.get']('fqdn') %}
include:
  - jenkins.install
  - jenkins.config
  - jenkins.securise
  - jenkins.addtions
  - jenkins.slave
{% else %}
include:
  - jenkins.slave
{% endif %}
```

## Ensuite ?
Maintenant que les installations de **Jenkins**, **Nginx** et **OpenSSL** sont faites, que le **Reverse Proxy** est actif et que tout est fonctionnel.il reste quelques tâches à effectuer :
- Créer un (ou plusieurs) utilisateurs
- Installer les plugins dont vous aurez besoin dans l'immédiat
- Créer un node/esclave
- Créer un job à exécuter

Puisque l'objectif de cet article est d'automatiser l'installation, la configuration et la sécurisation de **Jenkins**, nous allons donc faire appel à son API.

## L'API de Jenkins
L'API de Jenkins est utilisable à l'aide de script groovy ou de fichiers XML. Selon ce que vous utiliserez, l'effet sera différent.  

### Les scripts groovy
Voici quelques exemples d'utilisation de scripts groovy :

**La création d'un utilisateur**

```
	realm.createAccount("{{ user }}","{{ pass }}")
	instance.setAuthorizationStrategy(strategy)
	instance.setSecurityRealm(realm)
	instance.save()
```


**L'installation de plugins**
```
	plugins = ["active-directory",
           	"ansicolor",
           	"ant",
           	"authentication-tokens"]
	def pm = instance.getPluginManager()
	def uc = instance.getUpdateCenter()
	def installed = false
	plugins.each {
    	if (!pm.getPlugin(it)) {
        	def plugin = uc.getPlugin(it)
        	if (plugin) {
            	println("Installing " + it)
            	plugin.deploy()
            	installed = true
        	}
    	}
	}
	instance.save()
```

Si vous souhaitez d'autres exemples de scripts groovy utilisable avec jenkins, je vous suggère de vous rendre [ici](https://wiki.jenkins.io/display/JENKINS/Jenkins+Script+Console)

### Les fichiers XML
Toujours dans le cadre des interactions avec l'API de jenkins, les fichiers XML permettent d'effectuer diverses actions telles que :

- La création de nodes
```
<?xml version='1.0' encoding='UTF-8'?>
<slave>
  <name>{{ info }}</name>
  <description>{{ description }}</description>
  <remoteFS>{{ remotefs }}</remoteFS>
  <numExecutors>{{ executors }}</numExecutors>
  <mode>{{ mode }}</mode>
  <retentionStrategy class="hudson.slaves.RetentionStrategy$Always"/>
  <launcher class="hudson.plugins.sshslaves.SSHLauncher" plugin="ssh-slaves@1.21">
	<host>{{ address }}</host>
	<port>{{ port }}</port>
	<credentialsId>{{ credentialsid }}</credentialsId>
	<javaPath>{{ javapath }}</javaPath>
	<maxNumRetries>0</maxNumRetries>
	<retryWaitTime>0</retryWaitTime>
	<sshHostKeyVerificationStrategy class="hudson.plugins.sshslaves.verifiers.KnownHostsFileKeyVerificationStrategy"/>
  </launcher>
  <label></label>
  <nodeProperties/>
</slave>
```

- La création de jobs
```
<?xml version='1.0' encoding='UTF-8'?>
<project>
  <actions/>
  <description>{{ description }}</description>
  <logRotator>
	<daysToKeep>7</daysToKeep>
	<numToKeep>-1</numToKeep>
	<artifactDaysToKeep>-1</artifactDaysToKeep>
	<artifactNumToKeep>-1</artifactNumToKeep>
  </logRotator>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers class="vector"/>
  <concurrentBuild>false</concurrentBuild>
  <builders>
	<hudson.tasks.Shell>
  	<command>{{ command }}</command>
	</hudson.tasks.Shell>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
```

Et pour que **Jenkins** prenne en compte les fichiers XML et les scripts groovy que vous allez créer, rien de plus simple :
Les jobs dans le dossier <home de Jenkins>\jobs\<nom du job>
Les nodes dans le dossier <home de Jenkins>\slaves\<nom du node>
Les scripts groovy dans le dossier <home de jenkins>

Selon le type de fichier utilisé, l'utilisation de l'API de Jenkins nécessite un redémarrage du service, nous vous suggérons tout de même de suivre les logs de Jenkins afin de vérifier que les plugins s'installent correctement.  
N'hésitez surtout pas à vous servir de la commande suivante une fois que vous avez redémarré **Jenkins** : `tail -f /var/log/jenkins`  
Si vous apercevez (par exemple) les lignes suivantes, l'installation des plugins est en cours :  
```
INFO: Starting the installation of <plugin name> on behalf of <user>
<date> <heure> hudson.model.UpdateCenter$UpdateCenterConfiguration download
INFO: Downloading <plugin name>
<date> <heure> hudson.model.UpdateCenter$DownloadJob run
INFO: Starting the installation of <plugin name> on behalf of <user>
<date> <heure> hudson.model.UpdateCenter$UpdateCenterConfiguration download
```

## Une dernière chose...
**Saltstack** propose un module pour **Jenkins**, vous pouvez l'utiliser en ligne de commande ou bien l'intégrer directement dans un *state* à l'aide du mot clé *module.run*, comme l'exemple ci-dessous relatif à l'installation de modules :  

```
Jenkins_modules_install:
  module.run:
	jenkins.plugin_installed:
  	- name: |
    	- active-directory
    	- ansicolor
    	- ant
    	- authentication-tokens
```  

Toutes les informations sur ce dernier sont disponibles dans la [documentation officielle](https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.jenkins.html#salt.modules.jenkins.build_job)

## Conclusion  
Sans avoir à vous salir les mains dans les entrailles de **Jenkins** et **SaltStack**, vous avez à présent, un serveur d’intégration continue fonctionnel.
Vous souhaitez aller encore plus loin ? Vous pouvez toujours étudier *Groovy* et taper vos propres scripts.
