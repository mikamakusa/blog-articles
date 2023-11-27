+++
date = "2023-11-27T22:23:00+02:00"
draft = false
title = "Des nouvelles du Cloud..."
+++ 

Cela faisait près de trois ans que je n'avais publié un article sur les nouveautés toutes fraiche provenant des Cloud Providers, je me suis dit que ce mois-ci, cela pouvait être une bonne occasion de réittérer l'expérience car l'actualité a été relativement riche et mouvementé de ce côté là.

## Commençons par Azure.
Suite à l'évolution des technologies et des normes réglementaires, Microsoft a pris la decision de supprimer la prise en charge du chiffrement TLS dans les version 1.0 et 1.1. La version 1.2 étant bien plus rapide et sécurisée que les anciennes et prenant également en charge les algorithmes cryptographiques et les suites de chiffrements les plus récentes et modernes.  
Evidemment, cela affectera les comptes de stockages utilisant les versions bientôt obsolètes et afin d'éviter toutes interruptions de connexion, il sera obligatoire de migrer vers la version la plus récente.

Concernant le service **Machine Learning**, la mise à jour disponible depuis le 15 Novembre apporte son lot de nouveautés :  
- **Model Catalog** : Permet à l'utilisateur de découvrir, personnaliser et de rendre opérationel des grands modèles sans avoir à gérer l'infrastructure ou les dépendances logicielles,  
- **Managed Feature Store** : Permet d'expérimenter et de livrer des modèles plus rapidement, tout en les rendant plus fiable afin de réduire les coûts opérationnels,  
- **Model Training With Serverless Compute** : Comme son nom l'indique, permet de se concentrer sur la création de modèles de Machine Learning sans se soucier d'avoir une infrastructure à maintenir,  
- **Batch Endpoints** : Dans le but de tester plusieurs versions de pipelines avec un seul endpoint,  
- **Prompt Flow** : Dans le domaine du **Machine Learning**, un *Prompt* (une invite) est un extrait de texte en langage naturel. Le but de **Prompt Flow** est d'aider à créer rapidement un *wonrkflow* qui se connectera à différentes sources de données afin de créer des applications intelligentes.

**Azure Kubernetes Service** a également reçu sa mise-à-jour mensuelle.  
Outre la version 1.28 désormais disponible, les *Confidential Containers* sont également de la partie ainsi que la possibilité d'effectuer des opérations de *lift and shift* dans un environnement confidentiel, tout comme du chiffrement *in-memory* basé sur une clé dédiée par groupe de conteneurs.
Ce n'est pas tout car la collecte de données *syslog* depuis les noeuds **AKS** à l'aide d'**Azure Monitor Container Insights** ainsi que le service **Azure Backup** dédié sont désormais estampillée **Generally Available**.

Pendant qu'ils annoncaient la fin du support de **.NET 6** à l'horizon du 12 Novembre 2024 et celle de **.NET 7** pour le 14 Mai 2024, le support de **.NET 8** était rendu disponible sur le service **Azure Static Web Apps**.

## Du côté d'AWS
Le service **EC2** a été lre premier a recevoir quelques mise-à-jour.  
La première fut en lien avec le service de **Machine Learning** et concerne la possibilité de réserver de la capacité GPU pour améliorer les performances des workloads. En facilitant l'accès à des instances EC2 dotées de processeurs graphiques spécialisés.

La seconde mise-à-jour concerne **Aurora** (*RDBMS*) et **Redshift** (*Datawarehouse*) avec l'intégration d'**Aurora Zero ETL** avec **Redshift**. Le principe d'un **ETL** est d'effectuer des synchronisations massives d'informations d'une source de données vers une autre.  
Le but d'**Aurora Zero ETL** est d'offrir la possibilité d'exécuter des analyses et du **Machine Learning** en temps réel, sans avoir besoin de passer d'un service à l'autre. Une explication plus détaillée est disponible [ici](https://aws.amazon.com/fr/blogs/aws/amazon-aurora-mysql-zero-etl-integration-with-amazon-redshift-is-now-generally-available/) 

Troisième service à recevoir de nouvelles fonctionnalités : **Bedrock**. C'est un service d'IA générative.  
Déjà présent dans le *Marketplace* **AWS**, **Cohere Command** est modèle d'IA générative à base d'invites.  
Désormais **Cohere Command Light** et **Cohere English and Mutilingual models** seront disponible directement dans **AWS Bedrock**.  
**Cohere Comand**, un modèle de génération de texte, et **Cohere Embed** est une famille de modèles formés pour générer des intégration à partir de documents textuels et se présente sous plusieurs formes : Anglais ou multilingue.  
Plus d'informations [ici](https://aws.amazon.com/fr/blogs/aws/amazon-bedrock-now-provides-access-to-cohere-command-light-and-cohere-embed-english-and-multilingual-models/)  

D'autres services ont également reçu une mise-à-jour, **Amazon FSx for NetApp ONTAP** a également été impacté. **FlexGroup Volume Management** est désormais disponible et permet de gérer la totalité du cycle de vie des volumes *FlexGroup* à l'aide de la *Console*, de **FSx Cli** ainsi que du SDK.  
De plus, le support des *Shared VPC* ainsi que des systèmes de fichiers proposant des performances bien plus intéressantes.

## Et chez Google Cloud ?
Desormais, il devient possible d'éditer et de créer des profiles de sécurité personnalisés dans **Apigee** qui affecte le **Security Score**. Cette fonctionnalité est en **Public Preview**.  
Mais ce n'est pas tout, de nombreux failles CVE ont été comblées sur **Apigee Hybrid** ainsi que la gestion des charts Helm et **API Security Actions**.  

Toujours en **Public Preview** et sur le service **BigQuery**, une nouvelle vue appelée **INFORMATION_SCHEMA** affiche les metadonnées de l'utilisation du storage avec **TABLE_STORAGE_USAGE_TIMELINE** ET **TABLE_STORAGE_USAGE_TIMELINE_BY_ORGANIZATION**.  
De plus, le service propose également l'intégration du **Machine Learning** et des **Large Language Models** dans le but de faciliter la facturation de ces types d'offres.  

Le service de sécurité **Chronicle** a également été mis-à-jour. **Chronicle Curated Detections** a été amélioré à l'aide de nouveau contenu pour **Google Cloud Threats** afin d'aider à identifier les comportements potentiellement malveillant sur les RBAC du service **GKE** (**Kubernetes**).  
Suite à un update des statuts des règles basées sur **YARA-L** (une évolution des règles **YARA**, très utilisées en cybersécurité, au niveau recherche et détection de malwares), Google a ajouté *Limited* et *Paused* aux statuts existant, à savoir *Enabled* et *Disabled*.  
A l'origine, **Chronicle** est un **SIEM** mais en tant que **SOAR**, il progresse petit à petit.  
La dernière mise-à-jour apporte la prise en charge dynamique des environnements (lorsqu'un *playbook* est censé s'exécuter sur plusieurs).  

Introduit il y à quelques mois dans la galaxie de services de Google, **Duet AI** est un outil collaboratif optimisé par l'IA et disponible pour Google Workspace. Ce service fait également des merveilles sur Google Cloud, il est dorénavant possible de synthétiser les entrées de journaux de logs grâce à cette technologie, il est également intégré à **Cloud Shell Editor** (l'éditeur de code intégré au Shell), d'effectuer de la completion de code dans **Colab Enterprise** (**Machine Learning**).  

Anthos a reçu une mise-à-jour pour ses version **bare metal**, **VMware**, **Azure** et **AWS** et quelques failles de sécurité CVE ont été découvertes et patchées...exceptée la dernière (CVE-2023-5717).

## Cybersecurity
Sur ce sujet, il y a toujours du mouvement, par exemple un group de hacker Russes a réussi à obtenir l'accès des comptes mail d'environ 632000 employés des départements de la défense et de la justice, [à voir ici](https://www.cshub.com/attacks/articles/iotw-us-federal-agencies-hit-with-moveit-cyber-attack).  

Une reflexion sur pourquoi les entreprises investissent de plus en plus sur la sécurité dans le Cloud, par **Olivia Powell**. Une étude portant sur plus de sept cent professionnels de la cybersecurité, entre Avril et Mai 2023, avec une simple question : Comment penser-vous que les investissement en terme de cybersécurité de votre entreprise vont évoluer au cours des douze prochains mois. [A voir ici](https://www.cshub.com/cloud/articles/investment-in-cloud-security).  

Un article de blog de l'agence de cybersecurité américaine (la **NCSC**) relatif à la migration de systèmes vers du chiffrement *post-quantique* suite à un [article pubié en 2020](https://www.ncsc.gov.uk/whitepaper/next-steps-preparing-for-post-quantum-cryptography) sur le même blog et traitant du même sujet. Le tout au sujet de la dangerosité des machines quantiques pour les systèmes de chiffrement actuels. [A lire ici](https://www.ncsc.gov.uk/blog-post/migrating-to-post-quantum-cryptography-pqc)

Un article un peu plus technique, pour les administrateurs et ingénieurs système, sur les erreurs de configuration les plus répandues, impactantes en matière de cybersécurité et comment les résoudre [à voir ici](https://www.cshub.com/threat-defense/articles/10-cyber-security-misconfigurations-you-should-fix-right-now).  

L'Intelligence Artificielle, un domaine assez vaste dans lequel on peut surtout rencontrer actuellement des modèles d'IA génératives...pour lesquelles l'agence américaine de cybsécurité (la **NCSC**) a publié un recueil de bonnes pratiques à destination des développeurs au sujet du design, du développement, du déploiement et de la maintenance de leurs bébé. [C'est à lire ici](https://www.ncsc.gov.uk/blog-post/introducing-guidelines-secure-ai-system-development).