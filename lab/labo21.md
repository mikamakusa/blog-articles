+++
date = "2016-06-14T15:29:49+02:00"
draft = false
title = "Le Labo #15 - 2nde partie | Industrialisation d'infrastructure Cloud sur AWS"

+++

# Industrialisation d'infrastructure Cloud sur AWS - 2nde partie

## Petite introduction
Je continue mon tour d'horizon des outils existants d'industrialisation/orchestration d'infrastructure Cloud.  
Après avoir fais un tour du côté de **DigitalOcean** à l'aide de **Terraform**, Je continue l'aventure du côté d'**Amazon**.  
Après avoir expliqué la création du *template* pour **Packer**, c'est au tour des *playbook* d'**Ansible**.

## Avant d'aller plus loin
Dans l'article précédent, j'avais évoqué le déploiement de serveurs **LDAP**, **Web**, **MySQL** et **Docker Registry**.  
Comme je vais déployer quatre images, je vais avoir besoin de quatre playbook différents.

## Du côté d'Ansible - Les playbooks

### LDAP

	---
	- hosts: localhost
	  tasks:
		- name: 1. Deploy LDAP Server
		  sudo: true
	      apt: pkg=slapd state=installed update_cache=true
	    - name: 2. Deploy LDAP Server
	      sudo: true
	      apt: pkg=ldap-utils state=installed update_cache=true

### Docker Registry

	---
	- hosts: localhost
	  tasks:
	    - name: Download UCP
	      command: wget https://packages.docker.com/caas/ucp-1.1.0_dtr-2.0.0.tar.gz
	    - name: Load UCP in docker
	      command: docker load < ucp-1.1.0_dtr-2.0.0.tar.gz
	    - name: Deploy registry
	      command: docker run docker/trusted-registry:1.4.3 install | bash

### MySQL

	---
	- hosts: localhost
	  vars:
        mysql_root_password: <password>
      tasks:
        - name: 1. Install MySQL Server
          sudo: yes
          apt: name=mysql-server update_cache=yes cache_valid_time=3600 state=present
        - name: 2. Install Python MySQL
          sudo: yes
          apt: name=python-mysqldb update_cache=yes cache_valid_time=3600 state=present
        - name: Start the MySQL service
	      sudo: yes
	      service: 
	        name: mysql 
	        state: started
	        enabled: true
	    - name: update mysql root password for all root accounts
	      sudo: yes
	      mysql_user: 
	        name: root 
	        host: "{{ item }}" 
	        password: "{{ mysql_root_password }}"
	        login_user: root
	        login_password: "{{ mysql_root_password }}"
	        check_implicit_admin: yes
	        priv: "*.*:ALL,GRANT"
	      with_items:
	        - 127.0.0.1
	        - ::1
	        - localhost 

### Web

	---
	- hosts: localhost
	  sudo: yes
	  tasks:
	    - name: install apache2
	      apt: name=apache2 update_cache=yes state=latest
	    - name: enabled mod_rewrite
          apache2_module: name=rewrite state=present
          notify:
            - restart apache2
      handlers:
        - name: restart apache2
          service: name=apache2 state=restarted


## Du côté de Packer
Retournons du côté de **Packer**.  
Maintenant que les playbooks pour chaque images ont été créés, je vais pouvoir démarrer la création des images.  
Avant cela, à l'aide de la commande **packer inspect**, je vais pouvoir vérifier qu'il n'y a aucune erreur : 

	Variables:
	  <No variables>
	Builders:
	  amazon-ebs
	Provisioners:
	  ansible-local
	Note: If your build names contain user variables or template
	functions such as 'timestamp', these are processed at build time,
	and therefore only show in their raw form here.

Ensuite, à l'aide de la commande **packer build**, la création des images démarre.  
Si la création des images est un succès, on peut obtenir quelque chose comme ce qui suit en fin de création: 

	==> amazon-ebs: Stopping the source instance
	==> amazon-ebs: Waiting for the instance to stop
	==> amazon-ebs: Creating the AMI: pack_ladp_1465981424
	    amazon-ebs: AMI: ami-c733daa8
	==> amazon-ebs: Waiting for AMI to become ready
	==> amazon-ebs: Adding tags to AMI (ami-c733daa8)
	    amazon-ebs: adding tag: "OS_Version": "Ubuntu"
	    amazon-ebs: adding tag: "Release": "Latest"
	==> amazon-ebs: Tagging snapshot: snap-2eca0024
	==> amazon-ebs: Terminating the source AWS instance...
	==> amazon-ebs: Cleaning up any extra volumes...
	==> amazon-ebs: Deleting temporary security groups...
	Build 'amazon-ebs' finished.

	==> Build finished. The artifacts of successful builds are:
	--> amazon-ebs: AMIs were created