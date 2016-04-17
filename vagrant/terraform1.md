+++
date = "2015-12-15T18:21:49+02:00"
draft = false
title = "Terraform - présentation et Installation"

+++

# Qu'est-ce que Terraform ?
Terraform est un outil pour la construction, la modification et le versioning infrastructure.
Grâce aux fichiers de configuration, Terraform génère un plan d'exécution décrivant les tâches à effectuer pour atteindre l'état désiré, puis exécute le tout pour construire l'infrastructure décrite.

# Comment l'installer ?
Pour l'installer, il suffit de taper les commandes suivantes : 

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/terraform/0.6.8/terraform_0.6.8_linux_amd64.zip`
- **Décompresser l'archive** : `unzip terraform_0.6.8_linux_amd64.zip -d /usr/local/bin/terraform`
- **Déclarer le chemin de Terraform** : `echo "PATH=$PATH:/usr/local/bin/terraform" >> .bashrc && exec bash`
- **Vérifier l'installation** : `terraform`

# Les commandes de Terraform
**Terraform** Dispose des commandes suivantes : 

- **apply** : Démarre la construction ou la modification de l'infrastructure  
`terraform apply [OPTIONS] [dir-or-plan]`  
`-backup=[path]` | `-input=true` | `-no-color` | `-parallelism=n` | `-refresh=true` | `-state=[path]` | `-state-out=[path]` | `var [variables]` | `-var-file=` | `-target=[resource]`  

- **destroy** : Détruit l'infrastructure éxistante  
`terraform destroy [OPTIONS] [DIR]`  
`-force` | `-target`  

- **get** : Télécharge et installe les modules nécessaires  
`terraform get [OPTIONS] [DIR]`  
`-update`  

- **graph** : Crée un graphique visuel des ressources  
`terraform graph [OPTIONS] [DIR]`  
`-draw-cycles` | `-module-depth=n` | `-verbose`  
**terraform** peut générer des images à l'aide de la commande suivante : `terraform graph | dot -Tpng > [nom_du_fichier].png  

- **init** : Initialise la configuration Terraform depuis un module  
`terraform init [OPTIONS] source [DIR]`  
`-backend=` : Sépcifie le type de backend parmi **Atlas** (par défaut), **Consul**, **S3** ou **HTTP**.  
`-backend-config=` : Sépcifie une variable de configuration de backend.  

- **output** : Affiche la sortie d'un fichier  
`terraform output [OPTIONS] [NAME]`  
`-state=[PATH]` | `-module=[NOM_MODULE]`  

- **plan** : Génère et affiche un plan d'exécution  
`terraform plan [OPTIONS] [DIR]`  
`-backup=[path]` | `-destroy`  
`-detailed-exitcode` : Retourne le code de sortie : 0 = succès | 1 = Erreur | 2 = succès (si des modifications ont été effectuées)  
`-input=true` | `-module-depth=n` | `-no-color` | `-out=path` | `-parallelism=n` | `-refresh=true` | `-state=[chemin]` | `-target=[ressource]` | `-var` | `-var-file=`  

- **push** : charge la configuration Terraform vers **Atlas**  
`terraform push [OPTIONS] [PATH]`  
`-atlas-address=` | `-upload-modules=true` | `-no-color`  
`-overwrite=` : Défini une variable spécifique à uploader sur Atlas.  
`-token=` | `-var=` | `-var-file`  
`-vcs=true` : Terraform va détecter si un outil de version est utilisé.  

- **refresh** : Met à jour le fichier d'état local  
`terraform refresh [OPTIONS] [DIR]`  
`-backup=[path]` | `-no-color` | `-state=[path]` | `-state-out=[path]` | `-target=[ressource]` | `-var` | `-var-file=`  

- **remote** : Configure un stockage distant  
`terraform remote [SOUS-COMMANDE] [OPTIONS]`  
**config** : `-backend=`Atlas/Consul/S3/HTTP | `-backend-config=` | `-backup=[path]` | `-disable` | `-pull=true` | `-state=[path]`  
**pull**   
**push**  

- **show** : Inspect le plan/statut de Terraform  
`terraform show [OPTIONS] [PATH]`  
`-module-depth=n` | `-no-color`  

- **taint** : Marque une ressource pour recréation  
`terraform taint [OPTIONS] [NOM]`  
`-allow-missing` | `-backup=[path]` | `-module=[path]` | `-no-color` | `-state=[path]` | `-state-out=[path]`  

- **version** : Affiche la version de Terraform

# Providers et Provisioners
## Les Providers
Les **providers** sont les drivers (ou API) permettant d'intéragir avec les plateformes sur lesquelles les infrastructures seront construites.
**Terraform** est agnostique et peut utiliser de nombreuses plateformes.

## Les Provisioners
Lorsqu'une ressource est créée, un **provisioner** est exécuté pour l'initialiser afin d'ajouter des ressources à un gestionnaire d'inventaire, démarrer un outils de gestion de configuration, intégrer la ressource dans un cluster, etc...

# Modules et Plugins
## Les Modules
**Terraform** peut utiliser des paquets autonomes de configurations gérés comme un groupe et peuvent être utilisés pour créer des composants réutilisables.

## Les plugins
**Terraform** est basé sur une architecture de plugins regroupant aussi bien les **Providers** que les **Provisioners**.
Il est d'ailleurs possible d'en créer de nouveaux afin d'ajouter de nouvelles fonctionnalités.

Si vous souhaitez plus d'informations concernant **Terraform**, nous vous suggérons de consulter la documentation complète disponible sur le site : [https://terraform.io/](https://terraform.io/)

# Pour aller plus loin
Jusqu'ici, nous avons pu oberserver les technologies [Serf](http://www.ageekslab.com/vagrant/serf1/), [Vagrant](http://ageekslab.com/vagrant/vagrant2/), [Packer](http://ageekslab.com/vagrant/packer1/) et [Consul](http://www.ageekslab.com/vagrant/consul1/) de l'écosystème Hashicorp.  
Dans celui-ci, nous avons pu découvrir l'outil de création d'infrastructure **Terraform**.  
Dans un prochain document, nous étudierons **Vault**.
