+++
date = "2015-12-08T18:21:49+02:00"
draft = false
title = "Packer - Présentation et Installation"

+++

# Qu'est que Packer ?
Packer est Open-Source.
C'est un outil de création de création d'images ou de conteneurs identiques à partir d'un fichier de configuration unique.  
Il est pricnipalement utilisé pour automatiser la création d'images disposant d'un système d'exploitation et de logiciels particuliers en encourageant l'utilisation de framework tels que Puppet, Chef ou Ansible et peut être utilisé sur différentes plateforme, telles que Vagrant ou Docker.

# Comment l'installer ?
Pour l'installer, il suffit de taper les commandes suivantes :  

- **Télécharger l'archive** : `wget https://releases.hashicorp.com/packer/0.8.6/packer_0.8.6_linux_amd64.zip`  
- **Décompresser l'archive** : `unzip packer_0.8.6_linux_amd64.zip -d /usr/local/bin/packer`  
- **Déclarer le chemin de Packer** : `echo "PATH=$PATH:/usr/local/bin/packer" >> .bashrc && exec bash `
- **Vérifier l'installation** : `packer`  

# Les commandes de Packer  
**Packer** est un outil en ligne de commande.  
Toute intéraction avec Packer devra être effectuée à l'aide de l'outil éponyme et des commandes suivantes : 

- **build** : consruit une image depuis un template
Les options de cette commande sont les suivantes :  
`-color=false` : désactiver la sortie standard colorisée (activé par défaut)  
`-debug` : Désactive la parallélisation des tâches et active le mode "debug"  
`-except=xxx,xxx,xxx` : Lance tous les builds à l'exception de ceux mentionnés. Par défaut, le nom des builds équivaut au driver utilisé sauf si l'attribut "name" est spécifié.  
<code>-force</code> : force un build a démarrer lorsqu'un artefact d'un précédent build l'en empêche.  
`-only:xxx` : Effectue le contraire de la commande `except`.  

- **fix** : Trouve les éléments incompatibles/obsolètes d'un template et les mets à jour.  
- **inspect** : inspecte les composants d'un template  
- **validate** : Vérifie qu'un template est valide  
La commande **validate** ne comporte qu'une seule option :
`-syntax-only` : Seule la syntaxe du template est vérifiée, la configuration n'est pas validée. 

# La création de templates
Les templates sont des fichiers JSON, ils sont portables, statiques et ont l'avantage de pouvoir être facilement modifiables.
Ils sont composés des clés suivantes, certaines sont optionnelles : 

- **builders** : C'est un "tableau" composé d'un ou plusieurs objets qui définissent le builder (ou driver) qui sera utilisé pour créer les images de machines pour ce template, ainsi que pour les configurer.  
Une définition de "driver" correspond à un template. C'est un objet JSON qui requiert une clé "type". 

- **description** (optionnel) : Décrit les fonctionnalités du template.
- **min_packer_version** (optionnel) : Indique la version minimale nécessaire de Packer. 
- **post-processors** (optionnel) : Indique les différentes étapes nécessaires en "post-processing" afin que le template soit définitivement prêt.  
Il ya trois moyens de définir une action "post-processing" dans un template :  
<u>Simple</u> : Il s'agit d'un simple mot définissant l'action.  
<u>Détaillée</u> : Il s'agit d'un objet JSON composé d'un clé "type" et de plusieurs autres clés.  
<u>Séquence</u> : il s'agit d'un "tableau" expliquant les actions à effectuer d'un ordre prédéfini.  

- **provisioners** (optionnel) : C'est un "tableau" d'un ou plusieurs objets qui définit les "provisioners" qui seront utilisés pour installer et configurer le logiciel pour les machines créées par chacun des "drivers".  
Une définition de "provisioner" est un objet JSON qui doit contenir une clé "type", elle explique le nom du "provisioner" à utiliser.
Par exemple, le "provisioner" **shell** requiert une clé "script" indiquant le chemin d'un script à exécuter par la machine créée.  
Selon le type de driver utilisé, il est possible de définir des actions spécifiques à effectuer à l'aide des clé "only" ou "except".
Certains "provisioners" redémarrent la machine, il est possible de définir un temps de pause dans le script à l'aide de la clé "pause_before".  
- **variables** (optionnel) : C'est un "tableau" composé de clés et valeurs associées définissant des variables du template.

# Pour aller plus loin avec Packer
Dans ce document, nous avons pu observer comment fonctionne cet outils à l'aide des commandes de base et des différentes clés que peuvent composer un template.  
Si vous souhaitez plus d'informations sur Packer, rendez-vous sur le [site officiel](https://www.packer.io/intro).  
Dans le prochain document, nous aborderons l'outil de clustering lié à l'écosystème Hashicorp nommé **Serf**.
