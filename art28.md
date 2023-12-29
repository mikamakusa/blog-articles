+++
date = "2023-12-29T22:23:00+02:00"
draft = false
title = "Des nouvelles du Cloud...pour la fin d'année"
+++ 

L'actualité du monde de l'IT a été relativement chargée en ce mois de Décembre, je vous propose de revenir sur quelques unes des annonces les plus impactantes.

# Quelques doses de cybersécurité
Le Webzine [**Security**](https://www.securitymagazine.com/gdpr-policy?url=https%3A%2F%2Fwww.securitymagazine.com%2Farticles%2F100195-new-york-airport-adds-credential-authentication-scanners) nous rapporte que l'**administration américaine des transports** (la **TSA**) a annoncé que les aéroports internationals d'**Albany** et de **Syracuse** (Tous deux dans l'état de New York) ont implémentés une technologie de validation des identités à l'aide de la reconnaissance faciale. Détail très important à retenir : les photos ne seront stockées nulle part et ne seront utilisée que pour effectuer une correspondance entre la photo de la pièce d'identité et la personne qui la présente.  
Un article assez étonnant paru en début de mois sur le Webzine [**Cyber Security Hub**](https://www.cshub.com/security-strategy/news/us-federal-agencies-fail-to-meet-cyber-security-incident-response-requirements) relatif au fait que les agences fédérales américaines ne parviennent pas à répondre aux exigences de sécurité relatives aux réponses aux incidents de cybersécurité.  
Selon un rapport gouvernemental, une vingtaine d'agences gouvernementales n'auraient pas réussi à mettre en oeuvre des capacités de réponses aux incidents avant la deadline stipulée par un décret présidentiel datant de 2021, le fameux [décret 14028](https://www.federalregister.gov/documents/2021/05/17/2021-10460/improving-the-nations-cybersecurity.

Toujours sur le Webzine [**Cyber Security Hub**](https://www.cshub.com/threat-defense/articles/how-to-build-an-effective-cyber-attack-response-plan), un article détaillant point par point les étapes de construction d'un cycle de vie d'une politique de réponse aux incidents.  

Ainsi qu'une faille de sécurité chez Xfinity qui a impacté trente cinq millions de clients, c'est par [ici](https://www.cshub.com/attacks/articles/iotw-xfinity-data-breach-impacts-35-million-customers). 

Une reflexion sur pourquoi les entreprises investissent de plus en plus sur la sécurité dans le Cloud, par **Olivia Powell**. Une étude portant sur plus de sept cent professionnels de la cybersecurité, entre Avril et Mai 2023, avec une simple question : Comment penser-vous que les investissement en terme de cybersécurité de votre entreprise vont évoluer au cours des douze prochains mois. [A voir ici](https://www.cshub.com/cloud/articles/investment-in-cloud-security).  

Un article de blog de l'agence de cybersecurité américaine (la **NCSC**) relatif à la migration de systèmes vers du chiffrement *post-quantique* suite à un [article pubié en 2020](https://www.ncsc.gov.uk/whitepaper/next-steps-preparing-for-post-quantum-cryptography) sur le même blog et traitant du même sujet. Le tout au sujet de la dangerosité des machines quantiques pour les systèmes de chiffrement actuels. [A lire ici](https://www.ncsc.gov.uk/blog-post/migrating-to-post-quantum-cryptography-pqc)

Un article un peu plus technique, pour les administrateurs et ingénieurs système, sur les erreurs de configuration les plus répandues, impactantes en matière de cybersécurité et comment les résoudre [à voir ici](https://www.cshub.com/threat-defense/articles/10-cyber-security-misconfigurations-you-should-fix-right-now).  

L'Intelligence Artificielle, un domaine assez vaste dans lequel on peut surtout rencontrer actuellement des modèles d'IA génératives...pour lesquelles l'agence américaine de cybsécurité (la **NCSC**) a publié un recueil de bonnes pratiques à destination des développeurs au sujet du design, du développement, du déploiement et de la maintenance de leurs bébé. [C'est à lire ici](https://www.ncsc.gov.uk/blog-post/introducing-guidelines-secure-ai-system-development).

# Un zeste de Google Cloud
[**Vertex AI**](https://cloud.google.com/vertex-ai?hl=fr), l'outil de création d'Intelligence Artificielle Générative, a connu quelques upgrade ce mois-ci dont [Allowlist](https://cloud.google.com/generative-ai-app-builder/docs/data-source-access-control), [l'importation de documents](https://cloud.google.com/generative-ai-app-builder/docs/create-data-store-es#bring-parsed-document), le support de certains modèles tels que **Falcon-Instruct**, **Latent Consistent**, **SDXL-Turbo** ainsi que le support de nouveau Notebooks, tels que **Mistral**.  
Au niveau matériel, les [processeurs d'inférence de 5eme génération](https://cloud.google.com/tpu/docs/v5e-inference?hl=fr) sont désormais supportés.  
De plus, la dépréciation de Python 3.7 pour python 3.10 ainsi qu'un upgrade vers JupyterLab 3.6.6 a été effectué.

Du nouveau pour **Stackdriver** également, aussi bien au niveau **Logging** que **Monitoring**. L'upgrade de **OpsAgent** vers la version 2.44.0 introduit le support de nouvelles version de systèmes d'exploitation telles que **Ubuntu 23.10** et **Debian 12**.  
Effectuer des requêtes afin d'obtenir les erreurs de groupes spécifiques dans **Logs Explorer** et dans **Logs Analytics** est dorénavant possible.  

**ApigeeX** et **Apigee Hybrid** ont tout les deux été mis-à-jour (*1-11-0-apigee-8* et *v1.11.1*), afin de corriger quelques bugs et failles de sécurité...ainsi que l'**Advanced API Security** (principalement au niveau du *proxy security score*), l'**Integrated Portal**.  
Ainsi **Apigee** supporte les fonctions [**data residency**](https://cloud.google.com/apigee/docs/api-platform/get-started/drz-concepts), [**Forward Proxying**](https://cloud.google.com/apigee/docs/api-platform/fundamentals/environments-overview#forward-proxying) ainsi que les [**customer managed encryption keys**](https://cloud.google.com/apigee/docs/api-platform/get-started/cmek-concepts).  
De son côté, **Apigee Advanced API Security** charge bien plus rapidement le *security score* dans l'interface de gestion grâce à un cache amélioré côté serveur.

Au niveau sécurité, le service **Security Command Center** propose désormais une fonctionnalité permettant de créer ses propres détecteurs pour l'**Event Threat Detection**, [voir ici](https://cloud.google.com/security-command-center/docs/custom-modules-etd-overview).  
**Chronicle** a reçu un update pour sa [liste de *parsers* par défaut](https://cloud.google.com/chronicle/docs/ingestion/parser-list/supported-default-parsers) ainsi que l'intégration de [**Duet AI**](https://cloud.google.com/chronicle/docs/secops/duet-ai-chronicle#nl-to-udm) afin de rechercher des événements en langage naturel.  

Et comme tous les mois, les updates de sécurité dans le but de combler les failles CVE relatives aux différentes version de **GKE**


# Après l'AWS Re:Invent 
Quelques annonces assez intéressantes depuis la *grand messe* annuelle d'AWS.  
Tout d'abord, une nouvelle fonctionnalité de **Route 53 Application Recovery Controller** appelée [**Zonal Autoshift**](https://aws.amazon.com/fr/blogs/aws/zonal-autoshift-automatically-shift-your-traffic-away-from-availability-zones-when-we-detect-potential-issues/).  
Pour rappel **Route 53 Application Recovery Controller** vise à maintenir la haute disponibilité des applications *multi région* et/ou *multi zone de disponibilité* en déplaçant le trafic entre plusieurs environnements. Il sera possible de pousser le concept de disponibilité bien plus profondement avec **Zonal Autoshift** en redirigeant le trafic hors d'une zone de disponibilité et le rétablir...de manière complètement automatique.  
D'autant plus que **Route 53 Resolver** propose dorénavant la fonction [**DNS over HTTPS**](https://aws.amazon.com/fr/blogs/aws/dns-over-https-is-now-available-in-amazon-route-53-resolver/).  
Pour rappel, **Route 53 Resolver** permet de résoudre les requêtes DNS dans les environnements **Cloud Hybride**, ainsi les services **On-Premise** peuvent accéder à des ressources disponible sur **AWS**. **DNS over HTTPS** permet, comme son nom l'indique, de prendre en charge *HTTP* et *HTTPS* de chiffrer les données échangées pour la résolution de requêtes DNS.

Toujours au niveau résilience et haute disponibilité à l'aide du concept de **chaos engineering**, **AWS** propose **Fault Injection Service** dans le but de démontrer la fiabilité et la résilience des applications déployées en mode **multi région** et **multi zone de disponibilité**.  

Une nouvelle région est disponible : **AWS Canada West**, se situant à Calgary. En plus des soixante dix services déjà présents sur la plateforme Cloud, les services suivant seront également disponible au lancement : **AWS Artifact**, **AWS Cloud Control API**, **AWS Directory Service**, **EC2 Image Builder** ainsi que **AWS Transit Gateway**.

# Ainsi qu'une pincée de Microsoft Azure
Tout d'abord, plusieurs nouveautés sur des services déjà existants :  
- Le chiffrement à l'aide de clés de sécurité client (**customer managed keys**) enfin disponible pour **Azure Health Data Service**,  
- Le support de 5000 noeuds pour les clusters **AKS** *standard*,  
- Le support de la version 16 de PostgreSQL par le service **Azure Database for PostgreSQL**,  
- La possibilité d'effectuer des audits de connexion sur le service **Azure Cache for Redis**,  
- La possibilité de construire des applications de type **event driven** avec **Azure Functions** et **Azure SQL Database** à l'aide des fonctionnalités **Azure SQL Trigger**,  
- Effectuer des *snapshots* sur **Azure App Configuration** devient possible,  
- Les machines virtuelles supportent **RedHat Entreprise Linux** en version 8.9,  
- **Azure Stream Analytics** est dorénavant disponible dans sept nouvelles régions.  

**Microsoft** a également effectué une mise-à-jour sur son service de **Web Application Firewall** afin de combler une faille de sécurité relative à la vulnérabilité notée *CVE-2023-50164*, [plus d'information ici](https://azure.microsoft.com/en-us/updates/general-availability-security-update-for-application-gateway-waf-cve202350164/).  

Depuis le  20 Décembre en *Preview Publique*, afin de faciliter les efforts de migration des entreprises et des particuliers et, entre autre, pour faciliter l'adoption du Cloud **Azure**, [il est possible de tester gratuitement le service **SQL Managed Instance**](https://azure.microsoft.com/en-us/updates/public-preview-free-sql-managed-instance/).  

Une nouveauté, également en *Public Preview*, mais concernant le monde médical : la possibilité de stocker et gérer des données d'imageries médicale à l'aide du service **Azure Data Lake Storage** via l'intégration du service [DICOM](https://www.dicomstandard.org/), [plus d'information par ici](https://azure.microsoft.com/en-us/updates/store-and-manage-medical-imaging-data-with-azure-data-lake-storage-preview/).

La fin prochaine des **Pod Security Policy**, via leur dépréciation dans la version 1.25 du service **AKS**, a été annoncée, ainsi que celle des **Object Anchors** et des **Spatial Anchors** d'ici Mai et Novembre 2024.  

# Verser le tout dans un conteneur
Très peu d'annonces du côté de **Docker**, uniquement l'arrivée de Docker Desktop en version 4.26 intégrant **Rosetta**, **PHP init**, **Build Views**, **Admin Enhancements** et **Desktop Image for Microsoft Dev Box**.  
**Rosetta** fixe quelques bugs, notamment ceux liés à **Nodejs**, à **PHP**, au **programmes dépendants de chroot** ainsi qu'à **Sonoma**.  
**docker init**, cette fameuse commande simplifiant la création d'un environnement de développement de conteneur et d'image **Docker** demande également, lors de son exécution, pour quel type de projet il est solicité...si biensûr il ne le détecte pas lui même. Si j'évoqué **PHP**, c'est parce qu'il depuis début Décembre, c'est également supporté.  
**Build Views** est une fonctionnalité proposant des informations les plus détaillées possible sur la raison d'un echec de build d'une image...visuellement (*alors que les logs de la commande docker build sont très explicites...*).  

[Pour plus d'information, c'est par ici](https://www.docker.com/blog/docker-desktop-4-26/).

La toute dernière version de **Kubernetes** est arrivée, il s'agit de la 1.29.  
Comme un cadeau de Noël, ou presque, elle apporte plusieurs cadeaux, tels que :  
- [**Contextual Logging**](https://www.kubernetes.dev/blog/2023/12/20/contextual-logging/),  
- [**Load Balancer IP Mode for Services**](https://kubernetes.io/blog/2023/12/18/kubernetes-1-29-feature-loadbalancer-ip-mode-alpha/),  
- [**Single Pod Access Mode for PersistentVolumes**](https://kubernetes.io/blog/2023/12/18/read-write-once-pod-access-mode-ga/),  
- [**CSI Storage Resizing**](https://kubernetes.io/blog/2023/12/15/csi-node-expand-secret-support-ga/),  
- Dorénavant il faudra utiliser des composants additionnels afin d'intégrer un cluster Kubernetes à un Cloud Provider, [voir ici](https://kubernetes.io/blog/2023/12/14/cloud-provider-integration-changes/).