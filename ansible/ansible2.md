+++
date = "2015-08-12T17:06:49+02:00"
draft = false
title = "Ansible - Les playbooks, notions de base"

+++

## 1 - Qu'est-ce que Ansible ?
Ansible est une plate-forme logicielle libre pour la configuration et la gestion des ordinateurs quicombine le déploiement de logiciels et services, l'exécution de tâches ad-hoc, et la gestion de configuration.  
De plus, Ansible ne nécessite pas d'installation de client sur les serveurs ciblés, la communication entre le serveur et les noeuds est rendue possible par connexion SSH sécurisée.  
Dans le précédent document, nous avions abordé uniquement l'installation et la configuration d'Ansible, dans celui-ci nous alons aborder les notions de bases d'un playbook.

## 2 - Qu'est-ce qu'un playbook ?
Un playbook est un scénario décrivant les actions qui seront réalisées par les serveurs, c'est à dire les paquets à installer, les fichiers à créer et les services à démarrer ou arrêter, etc...

## 3 - Comment les créer ?
Un playbook est composé des éléments suivants :   
`- hosts`  
fait référence au fichier d'inventaire des hôtes que nous avons évoqué dans le tutoriel précédent.  
*Exemple* :  

	hosts: all

`- tasks` 	
fait référence aux tâches qui seront exécutées sur les hôtes.  
*Exemple* : 
 
	tasks:  
		- name: ensure apache is at the latest version  
			yum:  
			pkg: httpd  
			state: latest

`- vars` (optionnel)  
fait référence à certaines variables tels que les ports d'écoute ou le nombre de connexion concurrentes maximum.  
*Exemple* : 

	vars:  
		http_port: 80

`- remote_user` (optionnel)  
fait référence à l'utilisateur distant disposant des privilèges les plus élevés.  
*Exemple* : 

	remote_user: root

`- handlers`  
fait référence aux actions annexes (redémarrage de service, par exemple).  
*Exemple* : 
 
	handlers:  
		- name: restart apache  
			service:  
			name: httpd  
			state: restarted

## 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder [l'installation d'Ansible et sa configuration](http://www.ageekslab.com/ansible/ansible1/), les playbooks ainsi que les rôles.  
Dans ce document, nous avons pu observer la création d'un playbook à l'aide des notions les plus basiques.  
Dans le prochain document, nous nous concentrerons sur [les notions les plus avancées relatives aux playbook](http://www.ageekslab.com/ansible/ansible3/).
