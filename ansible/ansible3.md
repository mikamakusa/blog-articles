+++
date = "2015-08-24T10:10:49+02:00"
draft = false
title = "Ansible - Les playbooks, notions avancées"

+++

## 1 - Qu'est-ce que Ansible ?
Ansible est une plate-forme logicielle libre pour la configuration et la gestion des ordinateurs quicombine le déploiement de logiciels et services, l'exécution de tâches ad-hoc, et la gestion de configuration.  
De plus, Ansible ne nécessite pas d'installation de client sur les serveurs ciblés, la communication entre le serveur et les noeuds est rendue possible par connexion SSH sécurisée.  
Dans les précédents documents, nous avions abordé uniquement l'installation et la configuration d'Ansible ainsi que les notions de bases d'un playbook.  
Dans celui-ci, nous aborderons plus profondement les playbooks.

## 2 - Qu'appelle-t-on "notions avancées" ?
Les notions avancées regroupent les instructions permettant de préciser le fonctionnement des playbooks, telles que les conditions et les boucles.  
De plus, un playbook peut être organisé en rôles.

### 2.1 - Les conditions
Dans certains cas, les valeurs de variables peuvent dépendre d'autres variables, comme le type de distribution lors de l'installation d'un paquet en particulier.  
Pour cela, il faut utiliser la déclaration "when".  
*Exemple :*
  
	tasks:  
		- name: "shutdown Debian flavored systems"  
			command: /sbin/shutdown -t now  
			when: ansible_os_family == "Debian"

Il est aussi possible de grouper les conditions à l'aide de parenthèses, voir l'exemple ci-dessous :
  
	tasks:  
		- name: "shutdown CentOS 6 and 7 systems"  
		command: /sbin/shutdown -t now  
			when: ansible_distribution == "CentOS" and  
			(ansible_distribution_major_version == "6" or   ansible_distribution_major_version == "7")

### 2.2 - Les boucles
Dans certains cas, vous aurez besoin d'effectuer plusieurs tâches en une seule, telles que créer plusieurs utilisateurs ou installer plusieurs paquets, jusqu'à atteindre un résultat bien précis.

- Les boucles standard peuvent être décrite ainsi, en utilisant l'option "with_items" :

		- name: add several users  
			user: name={{ item }} state=present groups=wheel  
			with_items:  
			- testuser1  
			- testuser2

- Les boucles imbriquées utilisent l'option "with_nested" : 

		- name: give users access to multiple databases  
			mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo
			with_nested:  
			- [ 'alice', 'bob' ]  
			- [ 'clientdb', 'employeedb', 'providerdb' ]

- Les boucles sur dictionnaires :
En terminologie Python, un dictionnaire est un ensemble défini de variables possedant plusieurs valeurs, voir l'exemple ci-dessous :
   
		users:  
			alice:  
				name: Alice Appleworth  
				telephone: 123-456-7890  
			bob:  
				name: Bob Bananarama  
				telephone: 987-654-3210

- Les boucles de ce type utilisent l'option "with_dict" :
  
		tasks:  
			- name: Print phone records  
				debug: msg="User {{ item.key }} is {{ item.value.name }} ({{ item.value.telephone }})"  
				with_dict: "{{users}}"

- Les boucles sur "fileblogs" représentent tous fichiers dans un dossier correspondant à un pattern, et utilisent l'option "with_fileblogs".

## 3 - Qu'est-ce qu'un rôle ?
Il est tout à fait possible de mettre en place un playbook prennant en charge l'installation d'environnement totalement différents. Mais il s'avère parfois plus utile de regrouper les tâches listées danns un playbook dans des rôles bien précis.  
La création d'un rôle s'effectue de la même manière qu'un playbook en utilisant les mêmes balise, tout en créant un dossier pour chaque rôles avec l'arborescence suivante :  
- **files** : Contient les fichiers transférable sur les hôtes sur lesquels le rôle est déployés.  
- **handlers** : fait référence aux actions annexes (redémarrage de service, par exemple).  
- **meta** : Contient les fichiers relatif aux dépendances de rôles.  
- **templates** : Contient tous les fichiers qui utilisent des variables pour remplacer des informations lors de la création dans ce répertoire.  
- **tasks** : fait référence aux tâches qui seront exécutées sur les hôtes.  
- **vars** : LEs variables pour les rôles peuvent être spécifiés dans ce répertoire et utilisés dans vos fichiers de configuration.

Une fois les fichiers de rôles créés, le playbook devra ressembler à ce qui suit :
  
	- hosts: [alias ou @IP des hôtes]  
		roles:  
		- [nom du rôle #1]  
		- [nom du rôle #2]  
		- etc

## 4 - Conclusion
Dans cette série de tutoriels, nous allons aborder l'installation d'Ansible, sa configuration, les playbooks ainsi que les rôles.  
Dans ce document, nous avons pu observer les notions plus avancées relatives aux Playbook ainsi que les rôles.  
Dans les prochains documents relatif à Ansible, nous aborderons des exemples d'utilisation.
	
