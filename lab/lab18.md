+++
date = "2016-06-21T15:29:49+02:00"
draft = false
title = "Le Labo #16 - 2eme partie | Suite de l'initiation à Vagrant"

+++

# Suite de l'initiation à Vagrant
Pour ce nouveau Lab, j'ai souhaité me concentrer sur le provisioning des VM générées grâce à Vagrant.  
Ayant le choix des outils, j'ai préféré commencer par **Ansible** pour une raison extrêmement simple : C'est celui que je connais le mieux.  
Par contre, après ce lab j'irais faire un tour du côté de **Puppet** et de **Salt**.

# Comment démarrer ?
## Le Vagrantfile
Ayant déjà créé et téléchargé ma première image lors de mon premier Lab sur **Vagrant**.  
Par contre, je vais créer plusieurs environnement différents.  
L'idée est que la configuration soit progressive : partir de paramètres les plus communs pour arriver àux plus spécifique.
Dans ce but, mon *Vagrantfile* va devenir ainsi : 

	Vagrant.configure("2") do |config|
	    config.vm.provider "virtualbox" do |vb|
	        vb.memory = 512
	    end
	    config.vm.box = "ubuntu/trusty64"
	    
	    config.vm.define :master do |master_config|
	      master_config.vm.network "private_network", ip: "10.10.0.2"
	    end
	    config.vm.define :dev do |dev_config|
	      dev_config.vm.network "private_network", ip: "10.20.0.2"
	    end
	    config.vm.define :qa do |qa_config|
	      qa_config.vm.network "private_network", ip: "10.30.0.2"
	    end
	    config.vm.define :preprod do |preprod_config|
	      preprod_config.vm.network "private_network", ip: "10.40.0.2"
	    end
	    config.vm.define :prod do |prod_config|
	      prod_config.vm.network "private_network", ip: "10.50.0.2"
	    end
	    config.vm.provision :ansible do |ansible|
	      ansible.playbook = "/home/mike/vagrant-files/provisioning/playbook.yml"
	      ansible.groups = {
	        "master" => ["10.10.0.[1:254]"],
	        "dev" => ["10.20.0.[1:254]"],
	        "qa" => ["10.30.0.[1:254]"],
	        "preprod" => ["10.40.0.[1:254]"],
	        "prod" => ["10.50.0.[1:254]"],
	      }
	    end
	end

Comme vous pouvez le voir :

- le fichier d'inventaire d'**Ansible** est généré automatiquement et ressemblera à ceci : 

	[master]
	10.10.0.[1:254]

	[dev]
	10.20.0.[1:254]

	[qa]
	10.30.0.[1:254]

	[preprod]
	10.40.0.[1:254]

	[prod]
	10.50.0.[1:254]

En fait, chaque groupe est rattaché à un environnement et à un range d'adresses IP.
 
- le playbook est défini directement dans le *Vagrantfile* afin que le déploiement soit complètement automatique.

- J'ai repris une grande partie des paramètres utilisés lors du premier Lab consacré à Vagrant.

## Le playbook
Au niveau du playbook, je l'ai souhaité le plus simple possible.  
La logique de ce Lab a été expliquée plus tôt.  

	---
	- hosts: all
	  roles:
		- apache2 
		- mysql 
		- git 
		- consul 
		- docker 
	- hosts: dev
	  roles:
		- nodejs 
		- postgresql 
		- ruby 
		- go 
	- hosts: qa
	  roles:
	  - jenkins 

# Et c'est parti...
Une fois les différent fichiers créés (*Vagrantfile*, *playbook* et *roles*), je vais pouvoir lancer la création des VM à l'aide de **vagrant up**.  
Pendant que la génération des VM, il arrivera le moment du provisioning via **Ansible** et c'est là que l'on verra si la logique de Lab est respectée.  
On devrait voir quelque chose comme cela : 

Pour **master**

	==> master: Running provisioner: ansible...
	==> master: Vagrant has detected a host range pattern in the `groups` option.
	==> master: Vagrant doesn't fully check the validity of these parameters!
	==> master: 
	==> master: Please check https://docs.ansible.com/ansible/intro_inventory.html#hosts-and-groups
	==> master: for more information.
	    master: Running ansible-playbook...

	PLAY [all] *********************************************************************

	TASK [setup] *******************************************************************
	ok: [master]

	PLAY [dev] *********************************************************************
	skipping: no hosts matched

	PLAY [qa] **********************************************************************
	skipping: no hosts matched

	PLAY RECAP *********************************************************************
	master                     : ok=1    changed=0    unreachable=0    failed=0   

Pour **dev**

	==> dev: Running provisioner: ansible...
	==> dev: Vagrant has detected a host range pattern in the `groups` option.
	==> dev: Vagrant doesn't fully check the validity of these parameters!
	==> dev: 
	==> dev: Please check https://docs.ansible.com/ansible/intro_inventory.html#hosts-and-groups
	==> dev: for more information.
	    dev: Running ansible-playbook...

	PLAY [all] *********************************************************************

	TASK [setup] *******************************************************************
	ok: [dev]

	PLAY [dev] *********************************************************************

	TASK [setup] *******************************************************************
	ok: [dev]

	PLAY [qa] **********************************************************************
	skipping: no hosts matched

	PLAY RECAP *********************************************************************
	dev                        : ok=2    changed=0    unreachable=0    failed=0   

Pour **qa**

	==> qa: Running provisioner: ansible...
	==> qa: Vagrant has detected a host range pattern in the `groups` option.
	==> qa: Vagrant doesn't fully check the validity of these parameters!
	==> qa: 
	==> qa: Please check https://docs.ansible.com/ansible/intro_inventory.html#hosts-and-groups
	==> qa: for more information.
	    qa: Running ansible-playbook...

	PLAY [all] *********************************************************************

	TASK [setup] *******************************************************************
	ok: [qa]

	PLAY [dev] *********************************************************************
	skipping: no hosts matched

	PLAY [qa] **********************************************************************

	TASK [setup] *******************************************************************
	ok: [qa]

	PLAY RECAP *********************************************************************
	qa                         : ok=2    changed=0    unreachable=0    failed=0   

Avec un *vagrant status**, je vais pouvoir vérifier que mes machines virtuelles sont bien démarrées : 

	Current machine states:
	master                    running (virtualbox)
	dev                       running (virtualbox)
	qa                        running (virtualbox)
	preprod                   running (virtualbox)
	prod                      running (virtualbox)

# Ansible...c'est bien beau...mais, et les autres ?
**Ansible** c'est *ma zone de confort*, mais j'ai prévu de travailler d'autres outils de **Config Management**, dont **Puppet** et **Salt**.  
Comme je vous l'avais dis, ce sera pour les prochains Labs.