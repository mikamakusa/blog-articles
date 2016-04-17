+++
date = "2015-09-22T13:35:49+02:00"
draft = false
title = "Le Labo #1 | Déploiement automatique de paquets dans un conteneur"

+++

# 1 - Introduction #
Dans de précédents documents, nous avons abordé les thématiques [Ansible](http://localhost:1313/ansible/) et [Docker](http://localhost:1313/docker/) en détail, en exposant chaque éléments indispensable à leur utilisation.  
Dans ce document, nous allons déployer des conteneurs à l'aide de Ansible.

# 2 - Avant d'aller plus loin #
Durant ce cas pratique, nous aurons à disposition les éléments suivants deux serveurs linux de distributions différentes organisées ainsi :  
- Un serveur Ansible  
- Un serveur Docker, 1 conteneur

# 3 - Déroulement du cas pratique #
## Création de la clé SSH
Dans notre cas, il faudra copier la clé publique sur le serveur docker ainsi que sur le (ou les) conteneur(s) à l'aide des commandes suivantes :  
`ssh-keygen` : Pour créer la clé publique  
`ssh-copy-id -i /root/.ssh/[nom du fichier].pub [utilisateur]@[alias du serveur]` : Pour copier la clé publique sur le serveur Docker

## Déployer le conteneur
Nous allons déployer un conteneur dans lequel il n'y aura que SSH d'installé, prenons par exemple **rastasheep/ubuntu-sshd** et lançons la commande suivante :  
`docker run -d -P -p 80:50000 --name [Nom du conteneur] rastasheep/ubuntu-sshd:14.04 -v /root/.ssh/[nom du ficher].pub`  
Pour connaître le port ssh utilisé par ce conteneur, tapez la commande suivante :  
`docker port [nom du conteneur] 22`

## le fichier d'inventaire d'ansible
Ce fichier sert d'inventaire des ressources sur lesquelles Ansible peut déployer les paquets logiciels spécifiés dans les playbooks.  
Le conteneur étant créé, nous connaissons aussi le port d'écoute de SSH, nous allons pouvoir remplir le fichier "hosts" :
  
	[Alias du serveur Docker]  
	@ IP

	[Alias du conteneur]  
	@ IP du serveur Docker ansible_ssh_port=[n° du port -> résultat de la commande "docker port [nom du conteneur] 22"]  
	@ IP du serveur Docker ansible_ssh_user=[nom de l'utilisateur, dans le cas du conteneur utilisé, il s'agit de root]  
	@ IP du serveur Docker ansible_ssh_user=[mot de passe de l'utilisateur, dans le cas du conteneur utilisé, il s'agit aussi de root]

## Test de communication
il faut tester la communication entre Ansible et le serveur Docker à l'aide de la commande suivante :  
`ansible [alias du serveur docker] -m ping -u [nom d'utilisateur] --ask-pass -c paramiko`  
Si tout est ok :
  
	[@ IP] | success >> {  
	"changed": false,  
	"ping": "pong"

Tapez ensuite la commande suivante pour tester la communication avec le conteneur :   
`ansible [alias du conteneur] -m ping -u [nom d'utilisateur] --ask-pass -c paramiko`

Si tout est ok :
  
	[@ IP] | success >> {  
	"changed": false,  
	"ping": "pong"

## Le playbook
Maintenant que nous sommes certain que la communication entre le serveur Ansible et le conteneur est possible, nous allons pouvoir déterminer quels paquets vont être installés à l'aide d'Ansible.

Lorsque nous avons abordé Ansible, nous avions énumérés les éléments contenus dans un playbook :  

- hosts  
- tasks  
- vars  
- handlers  
- remote_users

Ce playbook étant un exemple, il ne se composera que de "hosts", "tasks" et "handlers".

	---  
	- hosts: [alias du conteneur]  
		tasks:  
			- name: Install Nginx  
			apt: pkg=nginx state=installed update_cache=true  
  
		handlers:  
			- name : start nginx  
			service: name=nginx state=started  

## déploiement
Une fois le playbook créé, lancez la commande suivante pour déployer le contenu du playbook :  
`ansible-playbook [nom du fichier playbook].yml`

# 4 - Conclusion #
Au tout début de ce cas pratique, nous avions déployé un conteneur Docker en précisant que le port 80 du conteneur devait être mappé sur le port 50000.  
Afin de vérifier que nginx à bien été déployer, il faudra lancer la commande suivante dans le terminal :  
`curl http://localhost:50000`  
Si le déploiement a été un échec, nous devrions voir  le message d'erreur suivant en sortie standard : 

	curl: (7) Failed connect to localhost:50000; Connection refused

