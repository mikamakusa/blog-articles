+++
date = "2015-08-27T15:37:49+02:00"
draft = false
title = "Docker - Installation"

+++

# 1 - Qu'est ce Docker ? #
Docker est un logiciel Open Source permettant de gérer des conteneurs.  
Contrairement à la virtualisation classique comprenant un système hôte sur lequel il est possible d'installer tout ce qui le serait sur un système classique, les conteneurs contiennent au maximum, que les applications/bibliothèques.  
Un container s’appuie sur le système d’exploitation de l’hôte pour fonctionner. Il s’agit de simuler un ensemble applicatif au sein de l’OS de l’hôte, cela de façon isolée au sein d’un container.  
Un container est léger, performant et peut être déployé rapidement, car il partage ses ressources avec le système d’exploitation de l’hôte physique : kernel, périphériques, processeur, RAM, etc.


# 2 - Comment installer Docker ? #
*Sur Linux*   
Docker étant disponible dans les dépots d'Ubuntu et de CentOS, la commande à taper pour l'installer est la suivante :  
<u>Debian/Ubuntu</u> : `apt-get install docker.io`  
<u>CentOS</u> (Jusqu'à la version 6.x) : `yum install docker.io`  

La méthode d'installation deDocker sur la version 7.x de CentOS est la suivante :  
`cat >/etc/yum.repos.d/docker.repo <<-EOF`  
`[dockerrepo]`  
`name=Docker Repository`  
`baseurl=https://yum.dockerproject.org/repo/main/centos/7`  
`enabled=1`  
`gpgcheck=1`  
`gpgkey=https://yum.dockerproject.org/gpg`  
`EOF`  

Puis :  
`yum update -y`  
`yum install -y docker-engine`

Une fois l'installation terminée, il faut démarrer le service à l'aide de la commande suivante : `service docker.io start`

Sur CentOS, si vous souhaitez démarrer Docker au démarrage, entrez la commande suivante : `chkconfig docker on`

Pour CentOS 7.x, la commande est la suivante : `systemctl enable docker.service`

Pour être sûr de disposer de la dernière version, vous pouvez aussi la télécharger : `wget -qO http://get.docker.com/ | sh`

*Sur Windows Server*  
Docker met à disposition un programme d’installation pour Windows qui installe le client Docker pour Windows, VirtualBox, Git for Windows (MSYS-git), Docker Machine et Kitematic. 

L'installeur est disponible ici : [https://www.docker.com/toolbox](https://www.docker.com/toolbox "https://www.docker.com/toolbox")

# 3 - Conclusion #
Dans cette série de tutoriels, nous allons aborder le fonctionnement de Docker et de ses outils Compose et Swarm.  
Dans ce document, nous avons pu observer comment installer Docker sur les sytèmes Linux et Windows.  
Dans le prochain document, nous nous concentrerons sur les commandes de base de Docker.
