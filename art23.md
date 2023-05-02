+++
date = "2023-05-02T22:23:00+02:00"
draft = false
title = "Un petit tour de Packer..."
+++

Cela faisait très longtemps que je n'avais pas publié d'article sur **Packer**, et pour cause...Je n'y avais plus touché depuis 2019, bien que mon [dernier article sur le sujet date de 2016](http://www.ageekslab.com/lab/lab23/).  
Loin de moi l'idée de vous proposer un article de type **lab**, au contraire ce sera plutôt un retour d'expérience de l'outil.

## Packer ?
Distribué par **Hashicorp**, au même titre que **Terraform**, **Vault**, **Vagrant** ou **Consul**, **Packer** fait partie des technologies qui sont devenues des standards en ce qui concerne les outils orientés **DevOps** relatif à l'infrastructure et à la création de machines virtuelles à l'aide d'un fichier de configuration.  
Au départ en *json*, le langage de configuration des images créées à l'aide de **Packer** a évolué vers le **HCL2** (certainement afin de gagner en homogénéité par rapport à **Terraform** ou **Consul**), bien que le *json* soit toujours compatible.  

## Entre les deux, mon coeur balance
Je ne vais par revenir sur tous les mots-clés importants d'une configuration **Packer**, à savoir :  
- **communicator** (SSH ou winrm),  
- **builders**,  
- **provisioners**,  
- **post-processors**

Chacune de ces mots-clés 