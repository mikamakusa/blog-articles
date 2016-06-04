+++
date = "2016-06-04T15:29:49+02:00"
draft = false
title = "Le Labo #12 2/2 | Docker Trusted Registry"

+++

# Docker Trusted Registry


## De quoi avons nous besoin ?
Afin de faire fonctionner un Docker Registry aux côté d'un serveur LDAP, les éléments suivants seront nécessaires :

- Un serveur LDAP (biensûr),  
- l'**UCP** (Universal Control Plane),  
- Le **Docker Trusted Registry**

L'installation et la configuration du serveur LDAP a été décrite dans l'article précédent. Dans celui-ci, nous nous concentrerons uniquement sur l'installation et la configuration de l'**UCP** et du **Docker Trusted Registry**.


## The Docker Registry
### Comment l'installer ?  
Nous allons partir du principe que Docker, dans sa toute dernière version, est déjà installé sur le serveur.

Tout d'abord, lancez les commandes suivantes : 

`wget https://packages.docker.com/caas/ucp-1.1.0_dtr-2.0.0.tar.gz`  
`docker load < ucp-1.1.0_dtr-2.0.0.tar.gz`

Une fois cette étape effectuée, lancez la commande suivante : 

`docker run docker/trusted-registry:1.4.3 install | bash`

Celle-ci déploie 8 conteneurs, en exécutant la commande `docker ps`, vous pourrez visionner le type de conteneurs déployés et, si vous le souhaitez inspecté les spécifications de chaque conteneur avec la commande `docker inspect <conteneur_id>`.
Certains conteneurs exposent des volumes utiles (dont le conteneur Nginx qui expose les dossiers contenant le certificat SSL et la configuration de l'UCP.)

Cet **UCP** est d'ailleur accessible à partir de l'URL suivante : **http://<Server.IP.address>**  


## Comment configurer chaque options ?
La page **Settings** regoupe toutes les options : 

- **General** : Remplissez les champs **domain name** et **HTTP Proxy** / **HTTPS Proxy**.

- **Security** : Renseignez seulement la clé SSL et la clé privée. Nécessaire pour, par la suite, se connecter au registry avec la commande *docker login fqdn.registry*

- **Storage** : Vous pouvez configurer le type de stockage parmi *local filesystem*, *S3*, *Azure* and *OpenStack Swift* et uploader, au besoin, un fichier décrivant avec précision la configuration du stockage.

- **License** : Uploadez le fichier de licence.

- **Auth** : Vous avez le choix entre plusieurs options parmi les suivantes :

	- **None**
	- **Managed** : Rassemble les comptes LDAP après synchronisation. Mais il est possible de créer des comptes directement via l'UCP,
	- **LDAP** : Les champs requis pour rendre cette option complètement fonctionnelle sont les suivants : 
		- **Admin Password** : Renseigner un mot de passer pour protéger l'UCP,  
		- **LDAP Server URL** : L'adresse du serveur LDAP *ldap://adresse.IP.du.serveur*,  
		- **Search User DN** : le *distinguished name* de l'administrateur du serveur LDAP,  
		- **Search User Password** : Le mot de passe de l'administrateur du serveur LDAP,  
		- **LDAP Sync Interval** : L'intervale de synchronisation,  
		- **User Base DN** : Le *distinguished name* de la base utilisateurs.  
		Par exemple : *ou=Users,dc=mydom,dc=com*  
		- **User Login Attribute** : *cn*  
		- **User Search Filter** : Dépend du type de serveur LDAP  
		Pour Linux LDAP server : *(objectClass=posixAccount)*  
		Pour Active Directory : *(objectClass=inetOrgPerson)*  
		- **Admin LDAP DN** : Le *distinguished name* du domaine LDAP,  
		- **Admin Group Member Attribute** : Dépend du type de serveur LDAP  
		Pour Linux LDAP server : *memberUid*  
		Pour Active Directory : *memberOf* 

		Il est possible de vérifier que la configuration est valide en testant une connexion d'un  utilisateur.  
		Si tout est ok, le message suivant s'affichera : 

		{
		  "accountInfo": {
		    "isAdmin": false,
		    "isAnonymous": false,
		    "isReadOnly": false,
		    "isReadWrite": false,
		    "name": "user2"
		   }
	    }

	- **Garbage Collection** : Nécessaire si l'on veut que rendre possible la suppression de chaque images et tags.


## Dépots et Organisations

**Pour les utilisateurs gérés par DTR**  
- Créer un dépot sans créer une organisation au préalable l'assigne à l'administrateur.

- Il est possible de créer un nombre ilimité d'organisations.

- Il est possible de créer plus d'une équipe par organisations.

- Il est possible d'assigner un utilisateur d'une organisation à une équipe d'une autre organisation.

- Les dépots **doivent** être assignés à une organisation/équipe et ne peuvent pas être utilisés en dehors de cette organisation.

- La gestion des droits est effectuée via trois options parmi les suivantes : 

	- **Admin** : 
		view and browse, pull, push, modify/delete tags, edit description, make public/private

	- **Read/Write** : view and browse, push, pull, modify/delete tags

	- **Read-Only** : view and browse, pull

**Pour les utilisateurs synchronisés depuis le serveur LDAP**

Côté **Administrateur** : 
	
- Il est possible de créer des équipes et de sélectionner l'option LDAP afin de rendre les comptes utilisateurs disponible pour cette équipe.

- Au lieu de la gestion des droits proposée par le **DTR**, c'est la gestion des droits du serveur OPENldap qui prend le relai.

Côté **Utilisateurs** : 

- Il est possible de créer un nombre de dépot ilimité **Public** ou **Privé**. Les dépots **Public** sont visible et utilisable par tous au contraire des dépots **Privé**.


## Connexion et utilisation du Docker Registry

- Connexion : `docker login fqdn.docker.registry`

- Upload d'une image docker :  
`docker tag <image> fqdn.docker.registry/username/dépot/`  
`docker push fqdn.docker.registry/username/dépot/`

- récupération d'une image docker :  
`docker pull <image> fqdn.docker.registry/username/dépot/`