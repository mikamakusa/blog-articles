+++
date = "2015-08-11T11:37:49+02:00"
draft = false
title = "Ansible - Installation et configuration"

+++

## 1 - Qu'est-ce que Ansible ? ##
Ansible est une plate-forme logicielle libre pour la configuration et la gestion des ordinateurs qui combine le déploiement de logiciels et services, l'exécution de tâches ad-hoc, et la gestion de configuration.  
De plus, Ansible ne nécessite pas d'installation de client sur les serveurs ciblés, la communication entre le serveur et les noeuds est rendue possible par connexion SSH sécurisée.

## 2 - Comment l'installer ? ##
Il y a trois manières d'installer Ansible sur votre système :
  
- Depuis les sources  
- Via les dépôts  
- à l'aide de l'outil "pip"

### 2.1 - Installation depuis les sources ###
Pour commencer, il faut télécharger l'archive sur Git à l'aide de la commande suivante :      
`git clone git://github.com/ansible/ansible.git --recursive`

Rendez-vous ensuite dans le dossier :  
`cd ./ansible`

tapez ensuite la commande suivante :   
`source ./hacking/env-setup`  
(ajoutez l'option "-q" afin de supprimer les avertissements/erreurs qui pourraient survenir).

L'outil "pip" est nécessaire pour pouvoir installer les outils suivants :  
`pip install paramiko PyYAML Jinja2 httplib2 six`

### 2.2. - Installation via les dépôts
Vous trouverez ci-dessous les commandes permettant d'installer Ansible à l'aide des dépôts :  
*Debian/Ubuntu* :  
`apt-get install software-properties-common -y`   
`add-apt-repository ppa:ansible/ansible`  
`apt-get update -y`  
`apt-get install ansible -y`

*CentOS* :  
Pour CentOS 6.4 (Il se peut que les versions suivantes de CentOS ne nécessitent pas l'ajout d'un nouveau dépôt)  
`rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm`  
`yum install ansible -y`

### 2.3 - Installation à l'aide de "pip"
Ansible peut être installé via "pip", l'outil de gestion de paquets Python.  
Selon la distribution utilisée, la commande pour installer "pip" est différente.  
La commande pour installer Ansible via "pip" est la suivante :   
`pip install -U ansible`

## 3 - Comment configurer le serveur ?
Une fois Ansible installé, il y a certaines tâches à effectuer.  
La première et plus importante consistant à générer la clé publique qui permettra la communication entre le serveur et les hôtes :   
`ssh-keygen`

Ensuite, il faudra provisionner le fichier "/etc/ansible/hosts" en spécifiant uniquement les adresses IP ou le nom du serveur, voir ci-dessous :  
`[nom du groupe]`  
`@ IP du serveur`

Certaines variables permettent de préciser le type de connexion, le nom d'utilisateur, le port, le mot de passe à utiliser, etc...  
Voici les variables possible :  
*Connexion à l'hôte :*  
`ansible_connection` : type de connexion à l'hôte (local, smart, ssh ou paramiko)

*Connexion SSH* :  
`ansible_ssh_host` : Nom de l'hôte.  
`ansible_ssh_port` : Numéro du port (si différent de 22).  
`ansible_ssh_user` : Nom de l'utilisateur.  
`ansible_ssh_pass` : Password à utiliser.  
`ansible_ssh_private_key_file` : fichier de clé privée à utiliser.  

*Escalade de privilèges* :  
`ansible_become` : Equivalent de ansible_sudo.  
`ansible_become_method` : Permet de définir la méthode d'escalade de privilège.  
`ansible_become_user` : Permet de définir l'utilisateur à impersonnifer.  
`ansible_become_pass` : Permet de définir le mot de passe élevant les privilèges utilisateur.

*Paramètres d'environnement d'hôte distant* :   
`ansible_shell_type` : Type de shell distant.  
`ansible_python_interpreter` : L'interpréteur python de l'hôte distant.  
`ansible\_\*\_interpreter` : fonctionne pour n'importe quel interpréteur, Ruby par exemple.

il est possible de définir un fichier d'hôte bien spécifique autre que celui :  
`/etc/ansible/hosts`  
A l'aide de la commande suivante :   
`export ANSIBLE_HOSTS=[/chemin/vers/fichier/hosts]`

## 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder l'installation d'Ansible et sa configuration, les playbooks ainsi que les rôles.  
Dans ce document, nous avons pu observer comment installer Ansible.  
Dans le prochain document, nous nous concentrerons sur [les notions de base des playbooks](http://www.ageekslab.com/ansible/ansible2/).
