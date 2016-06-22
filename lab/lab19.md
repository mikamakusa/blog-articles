+++
date = "2016-06-22T15:29:49+02:00"
draft = false
title = "Le Labo #16 - 3eme partie | Suite de l'initiation à Vagrant"

+++

# Suite de l'initiation à Vagrant
Pour ce nouveau Lab, j'ai souhaité me concentrer sur le provisioning des VM générées grâce à Vagrant.  
Ayant le choix des outils, j'avais fais le choix de commencer par **Ansible** puis je me suis concentré sur **Salt** (j'aurais peut être la chance de travailler avec cette technologie à l'avenir).

# Comment démarrer ?
## Le Vagrantfile
Ayant déjà créé et téléchargé ma première image lors de mes deux premiers Lab sur **Vagrant**, je vais juste le modifier afin d'y intégrer le nouveau type de provisioner.

	Vagrant.configure("2") do |config|
	    config.vm.provider "virtualbox" do |vb|
	        vb.memory = 512
	    end
	    config.vm.box = "ubuntu/trusty64"
	    config.ssh.forward_agent = true
	    
	    config.vm.synced_folder "salt/roots/", "/srv/salt/"
	    config.vm.define :master do |master_config|
	      master_config.vm.network "private_network", ip: "10.10.0.2"
	      master_config.vm.provision :salt do |salt|
	        salt.minion_config = "salt/minion.conf"
	        salt.run_highstate = true
	      end
	    end
	    config.vm.define :qa do |qa_config|
	      qa_config.vm.network "private_network", ip: "10.20.0.2"
	      qa_config.vm.provision :salt do |salt|
	        salt.minion_config = "salt/minion.conf"
	        salt.run_highstate = true
	      end
	    end
	    config.vm.define :dev do |dev_config|
	      dev_config.vm.network "private_network", ip: "10.20.0.2"
	      dev_config.vm.provision :salt do |salt|
	        salt.minion_config = "salt/minion.conf"
	        salt.run_highstate = true
	      end
	    end
	    config.vm.define :prod do |prod_config|
	      prod_config.vm.network "private_network", ip: "10.50.0.2"
	      prod_config.vm.provision :salt do |salt|
	        salt.minion_config = "salt/minion.conf"
	        salt.run_highstate = true
	      end
	    end
	end

## Le fichier minion.conf
Dans le fichier de configuration, j'indique quel est le serveur salt maître et où se trouvent les *states* et *pillars* :

	master: localhost
	file_client: local

	file_roots:
	  base:
	    - /srv/salt/states

	pillar_roots:
	  base:
	    - /srv/salt/pillars

## Le fichier top.sls
Ce fichier est similaire au playbook d'**Ansible** :

	base:
	  '*':
	    - base
	  'dev':
	    - go
	  'qa':
	    - jenkins
	  'prod':
	    - nginx
	    - mysql

## L'arborescence du dossier

	- vagrant-files
	|-minion.conf
	| |-roots
	| | |-pillars
	| | |-states
	| | | |-base
	| | | |-go
	| | | |-jenkins
	| | | |-mysql
	| | | |-nginx
	| | `-top.sls


# Et c'est parti...
Une fois les différent fichiers créés (*Vagrantfile*, *minion.conf*, *top.sls* ainsi que les définitions de package), je vais pouvoir lancer la création des VM à l'aide de **vagrant up**.  
Pendant que la génération des VM, il arrivera le moment du provisioning via **Salt** et c'est là que l'on verra si la logique de Lab est respectée.  
On devrait voir quelque chose comme cela pour chaque VM : 

	==> prod: Importing base box 'ubuntu/trusty64'...
	==> prod: Matching MAC address for NAT networking...
	==> prod: Checking if box 'ubuntu/trusty64' is up to date...
	==> prod: A newer version of the box 'ubuntu/trusty64' is available! You currently
	==> prod: have version '20160602.0.0'. The latest is version '20160620.0.0'. Run
	==> prod: `vagrant box update` to update.
	==> prod: Setting the name of the VM: vagrant-files_prod_1466606835038_60230
	==> prod: Clearing any previously set forwarded ports...
	==> prod: Fixed port collision for 22 => 2222. Now on port 2202.
	==> prod: Clearing any previously set network interfaces...
	==> prod: Preparing network interfaces based on configuration...
	    prod: Adapter 1: nat
	    prod: Adapter 2: hostonly
	==> prod: Forwarding ports...
	    prod: 22 (guest) => 2202 (host) (adapter 1)
	==> prod: Running 'pre-boot' VM customizations...
	==> prod: Booting VM...
	==> prod: Waiting for machine to boot. This may take a few minutes...
	    prod: SSH address: 127.0.0.1:2202
	    prod: SSH username: vagrant
	    prod: SSH auth method: private key
	    prod: Warning: Remote connection disconnect. Retrying...
	    prod: Warning: Remote connection disconnect. Retrying...
	    prod: 
	    prod: Vagrant insecure key detected. Vagrant will automatically replace
	    prod: this with a newly generated keypair for better security.
	    prod: 
	    prod: Inserting generated public key within guest...
	    prod: Removing insecure key from the guest if it's present...
	    prod: Key inserted! Disconnecting and reconnecting using new SSH key...
	==> prod: Machine booted and ready!
	==> prod: Checking for guest additions in VM...
	    prod: The guest additions on this VM do not match the installed version of
	    prod: VirtualBox! In most cases this is fine, but in rare cases it can
	    prod: prevent things such as shared folders from working properly. If you see
	    prod: shared folder errors, please make sure the guest additions within the
	    prod: virtual machine match the version of VirtualBox you have installed on
	    prod: your host and reload your VM.
	    prod: 
	    prod: Guest Additions Version: 4.3.36
	    prod: VirtualBox Version: 5.0
	==> prod: Configuring and enabling network interfaces...
	==> prod: Mounting shared folders...
	    prod: /vagrant => /home/mike/vagrant-files
	    prod: /srv/salt => /home/mike/vagrant-files/salt/roots
	==> prod: Running provisioner: salt...
	Copying salt minion config to vm.
	Checking if salt-minion is installed
	salt-minion was not found.
	Checking if salt-call is installed
	salt-call was not found.
	Bootstrapping Salt... (this may take a while)
	Salt successfully configured and installed!
	Calling state.highstate... (this may take a while)

Avec un *vagrant status**, je vais pouvoir vérifier que mes machines virtuelles sont bien démarrées : 

	Current machine states:
	master                    running (virtualbox)
	dev                       running (virtualbox)
	qa                        running (virtualbox)
	preprod                   running (virtualbox)
	prod                      running (virtualbox)

# Et quid des autres ?
Après **Ansible**, un tour du côté de **Salt** qui semble disposer d'une syntaxe aussi naturelle que le premier cité. Malgré le fait que le principe de base de **Salt** soit un fonctionnement selon le principe de *maître*/*minion*, il est possible de le faire fonctionner en mode *masterless*.