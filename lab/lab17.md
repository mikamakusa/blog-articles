+++
date = "2016-06-20T15:29:49+02:00"
draft = false
title = "Le Labo #16 - 1ere partie | Initiation à Vagrant"

+++

# Initiation à Vagrant
Pour ce nouveau **Labo**, je vais vous proposer une petite initiation à **Vagrant**.  
**Vagrant** est un outil que j'avais survolé lorsque j'avais publié la liste des outils de l'écosystème **Hashicorp** et, depuis, je n'avais rien publié à ce sujet...  
Mais, tout comme **Terraform** et **Packer**, je vais réparer cet oubli.  
Cet article est une simple initiation durant laquelle je vais juste déployer trois conteneurs vierge.

# Comment démarrer ?
Avant de me lancer dans la création d'un *Vagrantfile*, je dois commencer par exécuter la commande suivante :  
`vagrant init <image>`  
Dans mon cas, ce sera `vagrant init ubuntu/trusty64`  
Ensuite, je complète le *Vagrantfile* avec les informations nécessaires au déploiement de trois conteneurs.

## Le Vagrantfile
Mon *Vagrantfile* va ressembler à ce qui suit : 

	Vagrant.configure("2") do |config|
		config.vm.provider "virtualbox" do |vb|
			vb.memory = 512
		end
		config.vm.box = "ubuntu/trusty64"
		config.vm.define :master do |master_config|
			master_config.vm.host_name = "master"
			master_config.vm.network "private_network", ip: "10.10.0.2"
		end
		config.vm.define :slave1 do |slave1_config|
			slave1_config.vm.host_name = "slave1"
			slave1_config.vm.network "private_network", ip: "10.10.0.3"
		end
		config.vm.define :slave2 do |slave2_config|
			slave2_config.vm.host_name = "slave2"
			slave2_config.vm.network "private_network", ip: "10.10.0.4"
		end
	end

Comme vous pouvez le voir : 

- je n'utilise qu'une seule image, mais il est tout à faire possible d'en utiliser plusieurs...Dans ce cas là, il faut déclarer chaque image.
- Je nomme chaque conteneur, ca sera utile pour les prochains labs (au niveau du provisionning, puisque Ansible, Puppet ou bien Salt sont supportés)
- J'octroie une adresse IP pour chaque conteneur pour la même raison que les *hostname*.

Une fois le *Vagrantfile* créé, exécutons la commande `vagrant up` pour déployer les machines virtuelles et observer ce qui suit (durant le déroulement du déploiement) : 

	==> slave2: Importing base box 'ubuntu/trusty64'...
	==> slave2: Matching MAC address for NAT networking...
	==> slave2: Checking if box 'ubuntu/trusty64' is up to date...
	==> slave2: Setting the name of the VM: vagrant-files_slave2_1466453357699_21003
	==> slave2: Clearing any previously set forwarded ports...
	==> slave2: Fixed port collision for 22 => 2222. Now on port 2201.
	==> slave2: Clearing any previously set network interfaces...
	==> slave2: Preparing network interfaces based on configuration...
	    slave2: Adapter 1: nat
	    slave2: Adapter 2: hostonly
	==> slave2: Forwarding ports...
	    slave2: 22 (guest) => 2201 (host) (adapter 1)
	==> slave2: Running 'pre-boot' VM customizations...
	==> slave2: Booting VM...
	==> slave2: Waiting for machine to boot. This may take a few minutes...
	    slave2: SSH address: 127.0.0.1:2201
	    slave2: SSH username: vagrant
	    slave2: SSH auth method: private key
	    slave2: Warning: Remote connection disconnect. Retrying...
	    slave2: Warning: Remote connection disconnect. Retrying...
	    slave2: 
	    slave2: Vagrant insecure key detected. Vagrant will automatically replace
	    slave2: this with a newly generated keypair for better security.
	    slave2: 
	    slave2: Inserting generated public key within guest...
	    slave2: Removing insecure key from the guest if it's present...
	    slave2: Key inserted! Disconnecting and reconnecting using new SSH key...
	==> slave2: Machine booted and ready!
	==> slave2: Checking for guest additions in VM...
	    slave2: The guest additions on this VM do not match the installed version of
	    slave2: VirtualBox! In most cases this is fine, but in rare cases it can
	    slave2: prevent things such as shared folders from working properly. If you see
	    slave2: shared folder errors, please make sure the guest additions within the
	    slave2: virtual machine match the version of VirtualBox you have installed on
	    slave2: your host and reload your VM.
	    slave2: 
	    slave2: Guest Additions Version: 4.3.36
	    slave2: VirtualBox Version: 5.0
	==> slave2: Setting hostname...
	==> slave2: Configuring and enabling network interfaces...
	==> slave2: Mounting shared folders...
	    slave2: /vagrant => /home/mike/vagrant-files

Avec un `vagrant status` je vais pouvoir vérifier que mes machines virtuelles sont bien démarrées : 

	Current machine states:

	master                    running (virtualbox)
	slave1                    running (virtualbox)
	slave2                    running (virtualbox)

# Et le provisionning ?
Au niveau du provisionning, je vais étudier plusieurs options parmi **Ansible**, **Puppet** et **Salt** (que l'on m'a présenté dernièrement).  
Et celà, ce sera pour de prochains articles.