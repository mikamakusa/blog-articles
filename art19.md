+++
date = "2020-11-24T22:23:00+02:00"
draft = false
title = "November News from the Clouds"
+++

## AWS
Ce mois-ci, nous avons assisté à plusieurs événements :  
Les lancements de deux nouvelles régions :  
- [Zurich](https://aws.amazon.com/fr/blogs/aws/in-the-works-new-aws-region-in-zurich-switzerland/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29),  
- [Hyberabad](https://aws.amazon.com/fr/blogs/aws/in-the-works-aws-region-in-hyderabad-india/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29).

Et de nouveaux services :  
- [**Nitro Enclaves**](https://aws.amazon.com/fr/about-aws/whats-new/2020/10/announcing-general-availability-of-aws-nitro-enclaves/) (qui est l'équivalent des **Confidential VMs** ou des **Confidential Instances** disponibles respectivement sur **Google Cloud** et **Azure**) et les [certificats SSL](https://aws.amazon.com/fr/about-aws/whats-new/2020/10/announcing-aws-certificate-manager-for-nitro-enclaves/) qui leur seront associés via **Amazon Certificate Manager**.  
- [**OpsCenter**](https://aws.amazon.com/fr/blogs/aws/a-new-integration-for-cloudwatch-alarms-and-opscenter/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29) : une intégration de **CloudWatch** dans **AWS System Manager** qui nous permettra d'effectuer des aggrégations d'événements, d'alertes et d'incidents dans un seul et même dashboard.  
- [AWS Gateway Load Balancer](https://aws.amazon.com/fr/blogs/aws/introducing-aws-gateway-load-balancer-easy-deployment-scalability-and-high-availability-for-partner-appliances/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29), un service qui facilite et économise le déploiement, la mise à l'échelle et la gestion de la disponibilité des appareils virtuels tiers tels que les pare-feu, les systèmes de détection et de prévention des intrusions et les systèmes d'inspection approfondie des paquets dans le cloud.  
- [AWS Glue DataBrew](https://aws.amazon.com/fr/blogs/aws/announcing-aws-glue-databrew-a-visual-data-preparation-tool-that-helps-you-clean-and-normalize-data-faster/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29), un outil visuel de préparation de données développé dans le but de *normaliser et nettoyer* les données provenant de **Redshift**, **RDS**, un datastore **JDBC** ou bien des données indexées par **AWS Glue Data Catalog** (un service similaire à **Cloud Dataprep** de **Google Cloud**),  
- [AWS Network Firewall](https://aws.amazon.com/fr/blogs/aws/aws-network-firewall-new-managed-firewall-service-in-vpc/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29), un service de firewalling dédié au VPC complémentaire aux **Network Security Group**.  
- [S3 Storage Lens](https://aws.amazon.com/fr/blogs/aws/s3-storage-lens/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29), un outils d'analyse et optimisation de votre espace de stockage **S3** développé grâce aux 14 ans d'expérience acquise par **AWS** dans l'exploitation du service.

## Azure
Alors qu'en Juin 2020, **Azure** avait annoncé de nouvelles offres CPU/Memory optimized basées sur les processeurs **Cascade Lake** d'**Intel**, le fournisseur de services Cloud n'avait plus communiqué sur le sujet.  
Le 4 Novembre, ils ont annoncé [l'extension de leur certification **SAP-HANA**](https://azure.microsoft.com/fr-fr/blog/reduce-costs-with-new-memoryoptimized-azure-virtual-machines-certified-for-sap-hana/) afin de permettre l'exécution de charges sur des machines virtuelles à base des processeurs cités précédemment.  
De [nouvelles fonctionnalités DNS spécifique à Azure Firewall](https://azure.microsoft.com/fr-fr/blog/new-enhanced-dns-features-in-azure-firewall-now-generally-available/) sont disponible, dont **Custom DNS**, **DNS Proxy** et **FQDN Filtering**.  
Alors que la fonctionnalité de **Long Term Retention Backup** n'était disponible que pour **Azure SQL Server**, **Azure** a élargi son champs d'application à [**PostgreSQL**](https://azure.microsoft.com/fr-fr/blog/azure-backup-for-azure-postgresql-long-term-retention-in-preview/).  

## Google Cloud Platform
[Trois nouvelles fonctionnalités](https://cloud.google.com/blog/products/networking/cloud-cdn-gets-improved-useability-features) sont disponibles en Preview pour CloudCDN, il s'agit de :  
- **Cache Modes**,  
- **Cache TTL**,  
- **Custom Response Headers**

Lancement d'un nouveau service : [Document AI Platform](https://cloud.google.com/blog/products/ai-machine-learning/google-cloud-announces-document-ai-platform) une interface de traitement de document unifiée pensée et développée pour améliorer l'efficacité d'extractions de données via l'utilisation de l'intelligence artificelle et du Machine Learning.

Google Cloud annonce également :  
- [le support de PostgreSQL 13 pour CloudSQL](https://cloud.google.com/blog/products/databases/cloud-sql-database-service-adds-postgresql-13),  
- [Database Migration Service](https://cloud.google.com/blog/products/databases/database-migration-service-now-available-for-cloud-sql-and-more) et propose [une série de bonnes pratiques](https://cloud.google.com/blog/products/databases/tips-for-migrating-across-compatible-database-engines)  
- La nouvelle certification Google Cloud : [Professional Machine Learning Engineer](https://cloud.google.com/blog/products/ai-machine-learning/google-cloud-launches-machine-learning-engineer-certification)  

## Hashicorp
La prochaine version de **Terraform** (la 0.14), actuellement en preview, proposera de nouvelles fonctionnalités :  
- [un rendu de *diff* plus concis](https://www.hashicorp.com/blog/terraform-0-14-adds-a-new-concise-diff-format-to-terraform-plans) lors du *terraform plan*,  
- les [sensitive values en output](https://www.hashicorp.com/blog/terraform-0-14-adds-the-ability-to-redact-sensitive-values-in-console-output),  
- Le [Dependency Lock file](https://www.hashicorp.com/blog/terraform-0-14-introduces-a-dependency-lock-file-for-providers), généré automatiquement au lancement d'un *init*, cette fonctionnalité s'applique principalement aux providers customisés. 

Les versions [0.16](https://www.hashicorp.com/blog/announcing-hashicorp-sentinel-0-16) de **Sentinel** et [1.6](https://www.hashicorp.com/blog/vault-1-6) de **Vault** sont également annoncées.  

Ainsi qu'une mise-à-jour de l'UI de Nomad introduisant de nouvelles fonctionnalités :  
- La première se nomme [Topology Visualization](https://www.hashicorp.com/blog/see-your-entire-cluster-at-once-with-nomad-s-topology-visualization) et permet l'observation du cluster,  
- La seconde quant a elle, disponible uniquement pour l'offre **Entreprise**, est appelée [**Dynamic Application Sizing**](https://www.hashicorp.com/blog/hashicorp-nomad-dynamic-application-sizing) et propose d'effectuer l'optimisation intelligente de la consommation de ressource,  

## Docker
Docker a enfin annoncé [l'intégration de Compose CLI et d'Azure Containers Instances](https://www.docker.com/blog/compose-cli-aci-integration-now-available/), dans les cartons depuis l'annonce du partenariat entre Docker et Microsoft (en Mai).  
Suite à l'événement **One More Thing** d'**Apple** annonçant la future disponibilité du processeur maison M1 sur les **Mac Mini**, **Macbook Air** et **Macbook Pro 13"**. [Docker a décidé de l'intégrer dans sa roadmap de développement](https://www.docker.com/blog/apple-silicon-m1-chips-and-docker/) afin que tout soit compatible nativement (et non en passant par **Rosetta 2** comme c'est le cas actuellement).  
Dernière annonce de la part de **Docker** : [Docker Compose for AWS](https://www.docker.com/blog/docker-compose-for-amazon-ecs-now-available/) est dorénavant disponible et permet de déployer des services sur **ECS**. Pour cela, il suffit d'avoir installé la toute dernière version de **Docker Desktop Community** (version 2.5.0.1 ou plus récente).

## Cybersecurity
Pour commencer, un petit article sur [les différentes stratégies possible de prévention des fraudes](https://www.darkreading.com/vulnerabilities---threats/fraud-prevention-strategies-to-prepare-for-the-future/a/d-id/1339172?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple).  
le **Project Zero** de Google a détecté une nouvelle [faille 0-Day sur Windows au sujet d'une vulnérabilité Buffer Overflow](https://www.schneier.com/blog/archives/2020/11/new-windows-zero-day.html) dans le driver Cryptographique du noyau, [ainsi que deux nouveaux *exploits* sur Chrome](https://www.welivesecurity.com/2020/11/03/google-squashes-two-more-chrome-bugs-active-attacks/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+eset%2Fblog+%28ESET+Blog%3A+We+Live+Security%29).

L'acquisition prochaine de **PAS Global** par **Hexagon** a [été annoncée ici](https://www.darkreading.com/iot/hexagon-announces-deal-to-acquire-pas-global/d/d-id/1339378?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple)

Actuellement, personne n'est épargné par les cyber-attaques, [Capcom en a également subie une](https://www.welivesecurity.com/2020/11/05/major-gaming-company-capcom-hit-cyberattack/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+eset%2Fblog+%28ESET+Blog%3A+We+Live+Security%29).  

Dans la continuité du Google Project Zero et des détections de failles de sécurité, Apple a mis à disposition plusieurs patch dans le but [corriger trois failles 0-Day sur chacun de leur OS](https://www.darkreading.com/vulnerabilities---threats/apple-patches-24-vulnerabilities-across-product-lines/d/d-id/1339399?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple).  
Deux d'entre elles peuvent permettre à un attaquant d'exécuter du code arbitraire sur le système dépourvu du patch, la dernière permettrait à une application malveillante de divulguer le contenu du kernel.  
Suivant l'exemple de **Google** et d'**Apple**, **Microsoft** a également [corrigé de nombreuses failles de sécurité (112 pour être exact), dont certaines se sont révélées critiques](https://www.darkreading.com/threat-intelligence/microsoft-patches-windows-kernel-flaw-under-active-attack/d/d-id/1339415?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple).  

Voici un article intéressant de **Alan Bavosa** sur [les différentes méthodes d'attaques possible dans le but de bypasser une fonctionnalité MFA](https://www.darkreading.com/vulnerabilities---threats/how-hackers-blend-attack-methods-to-bypass-mfa/a/d-id/1339370?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple).

Les dernière élections américaines, [bien qu'extrêment sécurisées](https://www.darkreading.com/edge/theedge/we-secured-the-election-now-how-do-we-secure-trust-in-results/b/d-id/1339433?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple), risquent de faire des dégats au sein de l'agence **CISA**. En effet, le [directeur de l'agence s'attendait à se faire licencier](https://www.darkreading.com/risk/cisa-director-expects-to-be-fired-following-secure-election/d/d-id/1339442?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple) d'ici la fin du mandat du président Trump.  
[Ce qui est arrivé quelque jours plus tard](https://www.darkreading.com/threat-intelligence/trump-fires-cisa-director-chris-krebs/d/d-id/1339476?_mc=rss_x_drr_edt_aud_dr_x_x-rss-simple).  

## Tooling
Nous venant de la **Sandbox** de la **CNCF**, voici [Metal³](https://metal3.io/?ref=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Dio.sundeep.android&hl=en_IN&gl=US) un outils de provisionning *Bare Metal* pour **Kubernetes** à surveiller.  
Cette fois-ci, il s'agit de **Security & Compliance Automation** avec [Divvy Cloud](https://divvycloud.com/), supportant de nombreux environnements tels que **AWS**, **Azure** et **GCP** (pour ne citer qu'eux) et proposant de l'intégration avec **Slack** ou **Pageduty**, **DivvyCloud** gère aussi bien l'aspect **Security as Code** (incluant IAM, Threat Protection et Risk Assessment) que l'automatisation des actions visant à renforcer la sécurité dans le Cloud.  
[DeployHQ](https://www.deployhq.com/) est un nouvel outil de déploiement de package...certes, encore un...mais celui-ci gère également le build, le provisionning des serveurs sur lequels votre code va compiler et tourner, ainsi que le déploiement.  
