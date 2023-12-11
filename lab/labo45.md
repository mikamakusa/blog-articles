+++
date = "2023-12-11T22:23:00+02:00"
draft = false
title = "Data Management avec Apache NIFI"
+++ 

Une fois n'est pas coutume, au lieu d'aborder des sujets de type Cloud, Devops, retour d'expérience de conférences, de cybersécurité, etc...Cet article aura pour objet la **Data**. Non, je ne vais pas aborder le concept des **Datalake** mais plutôt le volet *software* et *outillage* du sujet.  

Chez un client, j'ai eu la possibilité de découvrir une technologie liée au domaine de la **Data**, à savoir **Apache NIFI**. Il s'avère que c'est un outil de gestion de flux de données permettant de les gérer et de les automatiser entre plusieurs systèmes à l'aide d'une interface graphique très épurée.  
Il faut savoir que ce projet n'a pas été initié par la fondation **Apache** (A l'origine de **Zookeeper**, que j'avais abordé il y a plusieurs années, ou de **Kafka**) mais par la **[NSA](https://fr.wikipedia.org/wiki/National_Security_Agency)** en 2006 qui a finalement libéré le projet en 2014 dans le cadre de son programme de transfer de technologie pour le confier à la **[fondation Apache](https://www.apache.org/)**.

## Le principe de NIFI
**Apache NIFI** est une solution d'automatisation de gestion de données, en gros : un passez plat.  
Selon les **Processors** que vous allez définir sur l'interface, il va chercher la donnée (un fichier, une requête API, le contenu d'un bucket S3, du CSV, du JSON, etc...) et va l'envoyer sur sa destination finale (RabiitMQ, Kafka, JMS, PubSub, AMQP, etc...).  
Le nombre de **¨Processors** est assez élevé et permet d'adresser de nombreux *use case* différents, afin de stocker et traiter localement ou sur les plateformes Cloud les plus connues.  
D'ailleurs, des *templates* sont disponibles un peu partout sur Internet, notamment [ici](https://cwiki.apache.org/confluence/display/nifi/example+dataflow+templates) (sur une page **Confluence** de la **Fondation Apache**) ou bien [ici](https://community.cloudera.com/t5/Community-Articles/A-Collection-of-NiFi-Examples/ta-p/244334) (sur la page **Community** de **Cloudera**).  

De nombreuses applications, ou **Flow** sont possible parmi les suivantes :  
- Conversion d'une entrée *CSV* en document *json* : 
![img](/images/nifi-1.png)  
- Ingestion de journaux de logs via *syslogs* pour stockage dans **HBase** :   
![img](/images/nifi-2.png)  
- Appel vers un webservice HTTP:  
![img](/images/nifi-3.png) 

Les exemples présentés ci-dessus ne sont pas exhaustifs, ils également éditables et importables dans votre instance **Apache NIFI**.

## Comment le déployer ?
Plusieurs méthodes de déploiement sont disponibles :  
- **Standalone AiO** : La configuration par défaut de NIFI embarque une instance **Zookeeper** interne,  
- **Standalone Cluster** : Deux options de configuration permettent de décentraliser la prise en charge du cluster par une instance **Zookeeper** externe.  
- **Standalone AiO multi-instances** : à l'instar de la configuration **Standalone AiO**, il est possible d'opérer de l'équilibrage de charge au sein même d'un cluster **Zookeeper** directement intégré dans **Apache NIFI**.  
- **Standalone Cluster multi-instances** : De la même manière que la configuration précedente...à l'aide de plusieurs instances **Zookeeper** externe.
Mais ce n'est pas tout, la **Fondation Apache** (ou du moins, les développeurs) ont également pensé aux fans de conteneurs et, par extension de Kubernetes, puisqu'il est également possible de déployer l'application à l'aide de [**docker-compose**]() et de **Kubernetes** (via le gestionnaire de package **Helm** et l'Operator disponible [ici](https://konpyutaika.github.io/nifikop/)).  

## Points d'attentions
- **Apache NIFI** est **extrêmement** gourmand en ressources et d'espace disque, comptez au minimum 1Go pour l'archive. Le service démarre sans problème sur un serveur *asthmatique* mais le démarrage sera très chronophage. Pour petit exemple, sur une machine virtuelle dotée de **1vCPU** et **1Go RAM**, le démarrage de **NIFI** dure plusieurs minutes.  
- A l'instar de nombreux projets portés par la **Fondation Apache**, i y a quelques pré-requis. Le premier et le plus important est **Java**, de mon côté j'ai utilisé **openJDK** en version 11.  
- La configuration est certes bien expliquée sur le site officiel, chaque option est détaillée, que ce soit le clustering (**Zookeeper**), le chiffrement (j'aborderais le sujet plus tard), les notifications, la configuration de proxy...Tout y est et facile d'accès. Si jamais vous ne trouvez pas votre bonheur, une simple recherche sur Internet vous fournira la réponse que vous demandiez.  
- La **Fondation Apache** a également mis à disposition un **NIFI Toolkit** répondant à de nombreux usages : **Flow Analyzer**, **Node Manager**, le **Cli** (dans le but de faciliter le démarrage de **NIFI**), **Zookeeper Migrator**, etc...  

## Securité et Chiffrement
Lorsque je fais référence à la sécurité et au chiffrement, j'englobe les certificats TLS, la gestion des utilisateurs ainsi que les magasins de clés de type **KMS**, ce qui nous envoie à la section **nifi.security** du fichier **nifi.properties**.  

### Certificats TLS
Dans **Apache**, **Nginx**, **Cherokee** et d'autres services à la configuration *relativement* simple, la déclaration de certificats TLS ne requiert l'utilisation de *magie noire* ou autre outils/software un peu exotique...cependant, certains service nécessitant **Java** pour fonctionner ont besoin d'un **Keystore**.  
La création d'un tel artefact oblige parfois à des acrobaties à l'aide des commandes suivantes :  
```bash
# Création du keystore
keytool -genkey -alias mydomain -keyalg RSA -keystore KeyStore.jks -keysize 2048
# Génerer un CSR base dans le nouveau keystore
keytool -certreq -alias mydomain -keystore KeyStore.jks -file mydomain.csr
# Importez les certificats root et intermédiares dans votre keystore
keytool -import -trustcacerts -alias root -file root.crt -keystore KeyStore.jks
keytool -import -trustcacerts -alias intermediate -file intermediate.crt -keystore KeyStore.jks
# Téléchargez et importez votre nouveau certificat
keytool -import -trustcacerts -alias mydomain -file mydomain.crt -keystore KeyStore.jks
```

De son côté, **NIFI** à l'aide du NIFI Toolkit propose une méthode bien plus simple : `tls-toolkit.sh`, une seule commande, au lieu de cinq, pour générer un keystore de manière bien plus fiable.  
```bash
bin/tls-toolkit.sh standalone -n 'nifi[01-10].subdomain[1-4].domain(2)' -C 'CN=username,OU=NIFI' --subjectAlternativeNames 'nifi[21-30].other[2-5].example.com(2)'
```  
Je vous invite à parcourir la documentation au sujet des différents scénarios autour de *tls-toolkit* (Standalone ou Client/Serveur).

### Magasins de clés de type KMS
Vaste sujet que celui-ci, entre le **NIFI Toolkit** proposant la commande `encrypt-config.sh` et les options de configuration de chiffrement de mots de passes à l'aide de **Hashicorp Vault**, **AWS KMS**, **Google Cloud KMS** ainsi que **Azure Key Vault**, rien ne manque dans le but de sécuriser le mieux possible votre instance **NIFI** (et je n'ai pas abordé le durcissement de l'OS dans le cas d'une machine virtuelle, qui ne sera pas abordé dans cet article).  

En ce qui concerne les services de gestion/génération de clés de sécurité, chacun à sa propre section dans le fichier de configuration **nifi.properties** :  
- **Hashicorp Vault** : *vault*,  
- **AWS KMS** : *aws.kms*,  
- **Azure Key Vault** : *azure.keyvault*,  
- **Google Cloud KMS** : *google.kms*  

Je ne viens d'aborder que les sections, bien évidemment les options de configuration sont bien plus complètes avec l'url du service, les crédentials pour que **NIFI** pour s'y athentifier, le chemin de la clé ou son id et, dans le cas de **Hashicorp Vault**, les types de magasins (ssl keystore/truststore, key/value), les suites de chiffrement activées, les différents protocoles, etc...Des options de configuration extrêmement complètes.

La commande `encrypt-config.sh` du **NIFI Toolkit** pousse relativement loin le concept de sécurisation en chiffrant la configuration du chiffrement directement dans le fichier **nifi.properties**.  
Qui a pensé : **Deux sécurités valent mieux qu'une seule** ?

### Gestion des utilisateurs
A ce niveau, les options disponible sont relativement nombreuses : **Single User**, **Lightweight Directory Access Protocol**, **Kerberos**, **OpenID Connect**, **SAML**, **Apache Knox** ainsi que **JSON Web Tokens**, avec **Single User** par défaut.  
Les identifiants par défaut sont dans le fichier *login-identity-providers.xml*...**non chiffrés**, cependant la commande suivante permet de les modifier :  
```bash
./bin/nifi.sh set-single-user-credentials <username> <password>
``` 

La configuration **Lightweight Directory Access Protocol** (ou LDAP) requiert un peu plus d'effort, surtout dans le fichier *login-identity-providers.xml*, après avoir modifié l'option *nifi.security.user.login.identity.provider*.  

#### le fichier *login-identity-providers.xml* spécial LDAP
```xml
<provider>
    <identifier>ldap-provider</identifier>
    <class>org.apache.nifi.ldap.LdapProvider</class>
    <property name="Authentication Strategy">START_TLS</property>

    <property name="Manager DN"></property>
    <property name="Manager Password"></property>

    <property name="TLS - Keystore"></property>
    <property name="TLS - Keystore Password"></property>
    <property name="TLS - Keystore Type"></property>
    <property name="TLS - Truststore"></property>
    <property name="TLS - Truststore Password"></property>
    <property name="TLS - Truststore Type"></property>
    <property name="TLS - Client Auth"></property>
    <property name="TLS - Protocol"></property>
    <property name="TLS - Shutdown Gracefully"></property>

    <property name="Referral Strategy">FOLLOW</property>
    <property name="Connect Timeout">10 secs</property>
    <property name="Read Timeout">10 secs</property>

    <property name="Url"></property>
    <property name="User Search Base"></property>
    <property name="User Search Filter"></property>

    <property name="Identity Strategy">USE_DN</property>
    <property name="Authentication Expiration">12 hours</property>
</provider>
```

La documentation officielle est très complète et je vous encourage vivement à y faire un tour pour connaître toutes les options de configuration disponible et ainsi sécuriser le mieux possible votre instance **NIFI**.

## Conclusion
Les alternatives vraiment gratuites et **OpenSource** manquent, parmi les Cloud Providers, il n'y a pas de service intégrant toutes les fonctionnalités de **NIFI** mais une combinaison de plusieurs :  
- **AWS** : Data Pipeline, Glue et SWS (Simple Workflow Service),  
- **Azure** : Data Factory, Data Catalog et Logic Apps,  
- **Google Cloud** : DataPrep et Cloud Composer,  
- **IBM Cloud** : DataStage et Watson Knowledge Catalog,  

De plus, malgré l'apparente simplicité de l'interface et la relative simplicité à mettre en place des *workflow* de gestion de flux de données, le nombre de **Processors** et de **Controllers** (que je n'ai même pas abordé dans cet article) est très élevé, dont certains en pleine phase de dépréciation, toujours documentés mais sans date de disparition annoncée, peut réellement devenir un frein à l'utilisation de cette solution.  
Et je n'évoque même pas les multiples versions d'un même **Processor**...*au cas où, ca pourrait servir*.