+++
date = "2016-06-13T15:29:49+02:00"
draft = false
title = "Le Labo #15 - 1ère partie | Industrialisation d'infrastructure Cloud sur AWS"

+++

# Industrialisation d'infrastructure Cloud sur AWS - 1ère partie


## Petite introduction
Je continue mon tour d'horizon des outils existants d'industrialisation/orchestration d'infrastructure Cloud.  
Après avoir fais un tour du côté de **DigitalOcean** à l'aide de **Terraform**, Je continue l'aventure du côté d'**Amazon**.  
Bien que cette plateforme propose un outils similaire à **Visual Cloud**, proposé par **ArubaCloud**, et nommé **Cloud Formation**, ce n'est pas celui là que je vais utiliser.  
Non pas que je n'ai pas trouvé comment l'utiliser...après tout, la création d'un fichier JSON décrivant chaque aspects des serveurs à créer est assez simple...mais je préfère laisser cela à **Quicklabs** et à d'autres organismes de formation.  
De mon côté, je vais plutôt utiliser **Packer** et **Ansible** pour créer les images et **Terraform** pour déployer les instances correspondantes sur **Amazon**.


## Avant d'aller plus loin
Je vais partir du principe que **Terraform**, **Packer** et **Ansible** sont déjà installé et qu'un compte chez AWS ouvert (avec le token qui va bien déjà téléchargé).

Pour cet article, je vais déployer les serveurs suivants, tous basés sur **Ubuntu Server** : 

- Un LDAP, 
- Un Docker registry,
- Une base de données MySQL,
- Un serveur Web Apache

Il est possible d'ajouter un serveur **Jenkins** afin d'automatiser la vérification et la création de l'image sur la plateforme cloud d'Amazon...mais ce sera peut être pour plus tard.

Pour que ce soit plus lisible (parce qu'expliquer le travail effectué sur chaque outils peut être long), j'ai fais le choix de partager cet article en plusieurs.

## Du côté de Packer
Dans un précédent article, j'avais effectué une rapide présentation de **Packer**... De plus, de nombreux sites en expliquent le fonctionnement.  
Donc, si vous cherchez plus d'information, Google est votre ami.  

Un fichier de build nécessite, au moins, le keyword suivant : **builders**  
Cependant, dans le cas qui m'intéresse actuellement, je vais aussi utiliser les suivants : 

- **variables**
- **provisioners**

### Du côté des "variables"
Il est tout à fait possible de ne pas inclure directement les variables dans le template **Packer** pour faire passer ces mêmes variables lors de la commande *build*, voir ci-dessous : 

`packer build -var "aws_access_key=<aws_access_key> -var "aws_secret_key=<aws_secret_key>"`

Voici les **variables** que je vais utiliser : 

	"variables": {
	    "ami_name": "<ami_name>",
	    "aws_access_key": "<aws_access_key>",
	    "aws_secret_key": "<aws_secret_key>",
	    "ansible_playbooks_path": "<ansible_playbooks_path>",
	    "aws_region": "<aws_region>",
	    "ami_instance_type": <ami_instance_type>"",
	    "ami_description": "<ami_description>"
	  },

Après test, il est tout à fait possible de démarrer un build packer sans inclure une partie **variable** dans le template si, dans la partie **builders**, on ajoute *aws_access_key* ainsi que *aws_secret_key*.

### Du côté des "builders"
Pour tout ce qui est de la plateforme AWS, nous avons le choix entre *amazon-ebs*, *amazon-instance*, *amazon-chroot*.  
Le premier démarre une instance avec une image AMI puis, après l'avoir provisionné à l'aide d'Ansible ou de n'importe quel autre outil de gestion de configuration, crée une nouvelle image.  
Le second effectue les mêmes opérations mais upload l'image sur un compte S3.  
Le dernier semble être réservé aux utilisateurs plus avancés.  

Voici la section **builders** de mon template : 

	"builders": [{
	  "type": "amazon-ebs",
	  "access_key": "<aws_access_key>",
	  "secret_key": "<aws_secret_key>",
	  "region": "eu-central-1",
	  "source_ami": "ami-87564feb",
	  "instance_type": "t2.micro",
	  "ssh_username": "ubuntu",
	  "ami_name": "ubuntu-packer-{{timestamp}}",
	  "communicator": "ssh",
	  "ssh_keypair_name": "<keyname",
	  "ssh_private_key_file": "<chemin/vers/keyfile.pem>"
	  "tags": {
	    "OS_Version": "Ubuntu",
	    "Release": "Latest"
	  }
	}]

### Du côté des "provisioners"
Comme expliqué au début de cet article, je vais utiliser **Ansible** pour provisionner les serveurs, mais d'autres sont prévus, tels que **Puppet** ou **Chef**.  

Voici la section **Provisioners** : 

	"provisioners": [
		{
			"type": "shell",
			"inline": [
				"sudo apt-get update -y",
				"sudo apt-get install -y git software-properties-common",
				"sudo add-apt-repository ppa:ansible/ansible -y",
				"sudo apt-get update -y",
				"sudo apt-get install ansible -y"
			]
		},
		{
		  "type": "ansible-local",
		  "command": "ansible-playbook",
		  "playbook_file": "local.yml"
		}
	]

Tout comme avec **Terraform**, il est possible de *templatiser* plusieurs images à l'aide d'un seul fichier **Packer** et donc, de définir un playbook pour chaque.

