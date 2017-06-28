+++
date = "2016-06-03T15:29:49+02:00"
draft = false
title = "Le Labo #12 1/2 | Docker Trusted Registry #1 - Le serveur LDAP"

+++

# Le serveur OPENLDAP


## Qu'est-ce qu'un serveur LDAP ?
A la base, LDAP (Lightweight Directory Access Protocol) est un protocole.  
Cependant, il a évolué pour représenter une norme pour les systèmes d'annuaires, incluant un modèle de données, un modèle de nommage, un modèle fonctionnel basé sur le protocole LDAP, un modèle de sécurité et un modèle de réplication

Si vous souhaitez tout savoir sur sur le protocole LDAP, je vous suggère la page Wikipedia correspondante.


## Comment l'installer ?
Pour installer un serveur LDAP, nous aurons besoin d'exécuter les commandes suivantes :  

**Sur Debian/Ubuntu**

`apt-get update -y`  
`apt-get install -y slapd ldap-utils`

Durant l'installation des paquets, les système demandera d'entrer le mot de passe *admin* du servive LDAP.

Le fichier de configuration à modifier se trouve dans */usr/share/slapd/slapd.conf* mais il faudra d'abord le copier dans */etc/ldap/slapd.conf*

**Sur CentOS**

`yum -y install openldap openldap-clients openldap-servers`

Une fois l’installation terminée, il faut créer le mot de passe de l’utilisateur qui va administrer notre serveur LDAP avec la commande `slappasswd`.

Il faut commencer par éditer le fichier suivant :  
*olcDatabase\=\{2\}bdb.ldif*  
se trouvant dans le dossier /etc/openldap/slapd.d/cn\=config 

Et trouver les lignes suivantes et éditez pour reflétez le nom de domaine que vous souhaitez.

	**olcSuffix:** dc=my-domain,dc=com
	**olcRootDN:** cn=Manager,dc=my-domain,dc=com
 
- **olcRootDN** est le DN de l’utilisateur qui va administrer OpenLDAP.

- Ajouter ensuite une ligne afin de configurer le mot de passe afin que le fichier ressemble à ce qui suit :

	olcSuffix: dc=ldap,dc=test
	olcRootDN: cn=ldapadmin,dc=ldap,dc=test
	olcRootPW: {SSHA}IzTO4Fr/u96D8+WVtWeT+25ddbgMnecx 

Maintenant, éditer le fichier suivant :  
*olcDatabase\=\{1\}monitor.ldif*  
se trouvant dans le même dossier et éditer la ligne olcAccess de cette manière:

	olcAccess: {0}to *  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=externa
	 l,cn=auth" read  by dn.base="cn=ldapadmin,dc=ldap,dc=test" read  by * none
 
Une fois ce fichier modifié, la configuration initiale de OpenLDAP est terminée, il faut redémarrer le service à l'aide de la commande suivante :

	service slapd start

	chkconfig slapd on (pour rendre son démarrage automatique dès allumage du serveur) 

Maintenant, initialiser l'annuaire LDAP avec les entrées nécessaire a son bon fonctionnement:

Créer un fichier base.ldif avec le contenu suivant:

	dn: dc=ldap,dc=test
	objectclass: dcObject
	objectclass: organization
	o: ldap
	dc: ldap
	 
	dn: cn=ldapadmin,dc=ldap,dc=test
	objectclass: organizationalRole
	cn: ldapadmin
 
Puis taper la commande suivante afin de pousser les changements dans le serveur LDAP.

	ldapadd -D cn=ldapadmin,dc=ldap,dc=test -W -f base.ldif
	Enter LDAP Password:
	adding new entry "dc=ldap,dc=test"
	 
	adding new entry "cn=ldapadmin,dc=ldap,dc=test"
 
L'installation **ldap-utils** permet de bénéficier d'une interface graphique ! **phpldapadmin**.


**Prochain article : L'installation et la configuration de Docker Trusted Registry**
