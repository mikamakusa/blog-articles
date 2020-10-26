+++
date = "2020-10-26T22:23:00+02:00"
draft = false
title = "News from the Clouds #2"
+++

## Azure
Je n'avais pas encore abordé cette information dans le précedent article sur les nouveautés du Cloud de **Microsoft**.  
Mais voici une information intéressante sur les **[Confidential Containers Nodes](https://techcommunity.microsoft.com/t5/microsoft-security-and/confidential-containers-nodes-now-supported-on-azure-kubernetes/ba-p/1726992)**, des workers **Kubernetes** du même type que les **Confidential Instances** sur **Google Cloud**, disponible depuis le début du mois d'Octobre 2020 en Preview.  

Un article de [Forbes](https://www.forbes.fr/technologie/microsoft-365-les-causes-de-la-coupure-massive/) est revenu sur les effets de la panne massive des services **Azure** et **Microsoft 365** survenue en début du mois d'Octobre.  

Encore une nouveauté chez **Microsoft**, mais cette fois-ci, elle est à mettre au crédit de **.NET** et de **Azure**, il s'agit des **[Connection Pool Limits](https://devblogs.microsoft.com/azure-sdk/net-framework-connection-pool-limits/)** (pour **.NET**) et de la release d'un nouveau SDK pour la plateforme Cloud maison.  

Les nouveautés sont extrêmement nombreuses ce mois-ci chez **Azure** : 
- Le service [Live Video Analytics](https://azure.microsoft.com/fr-fr/blog/azure-introduces-new-capabilities-for-live-video-analytics/) en preview,  
- Business Analytics et Congitive Search proposent désormais [Knowledge Mining](https://azure.microsoft.com/fr-fr/blog/deliver-ai-powered-application-search-with-azure-cognitive-search-and-ba-insight/),  
- [Azure Advisor Score](https://azure.microsoft.com/fr-fr/blog/optimize-your-azure-workloads-with-azure-advisor-score/) dans le but d'optimiser les charges de travail,  
- Les [Private Endpoint](https://azure.github.io/AppService/2020/10/06/private-endpoint-app-service-ga.html) pour le service **Web App**.  
- [Watchlist](https://techcommunity.microsoft.com/t5/azure-sentinel/what-s-new-watchlist-is-now-in-public-preview/ba-p/1765887), annoncé en Preview, est une fonctionnalité de **Azure Sentinel** permettant la collection de données externe en correlation avec des événements de votre environnement.  


## AWS
Et l'on reparle de **AWS** et de **Outposts**, cette fois-ci autour du service **S3**. L'idée étant de pouvoir utiliser ce dernier comme si il faisait partie d'une région classique.  

Plus d'infos par [ici](https://www.zdnet.com/article/aws-makes-s3-on-outposts-available-for-use-just-like-in-the-cloud/)  

Encore une nouveauté chez **AWS**, il s'agit de la release de **[Timestream](https://aws.amazon.com/fr/blogs/aws/store-and-access-time-series-data-at-any-scale-with-amazon-timestream-now-generally-available/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29)**, un service **Serverless**, scalable et rapide de *TSDB* (Time Series Database) qui facilite la gestion de bout en bout des métriques.  
De plus ce dernier pourra être utilisé conjointement avec **S3** pour la sauvegarde des données du *memory store*.  

Le service **S3** de **AWS** améliore un peu plus l'aspect sécurité en ajoutant trois [nouvelles fonctionnalités axées sur la sécurité et les contrôles d'accès](https://aws.amazon.com/fr/blogs/aws/amazon-s3-update-three-new-security-access-control-features/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29) 

**AWS** propose désormais :  
- [**Redis** en version 6 pour son service **ElastiCache**](https://aws.amazon.com/fr/blogs/aws/new-redis-6-compatibility-for-amazon-elasticache/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29).  
- [Serverless Application Model](https://aws.amazon.com/fr/serverless/sam/?ref=https://play.google.com/store/apps/details?id=io.sundeep.android&hl=en), une framework de build d'applications Serverless.  
- [SNS FIFO](http://feedproxy.google.com/~r/AmazonWebServicesBlog/~3/DtUO6Dghcd0/) introduisant la distribution des messages dans l'ordre d'arrivée.  


## Google Cloud Platform
Tout comme **Azure**, **Google Cloud** propose également une fonctionnalité de **[Confidential Nodes](https://cloud.google.com/blog/products/identity-security/confidential-gke-nodes-now-available)** à destination de **Kubernetes** (bien que ce soit en preview pour le moment).  

La plateform Cloud de chez **Alphabet** propose également des [nouvelles fonctionnalités pour les **Cloud Functions**](https://cloud.google.com/blog/products/serverless/a-roundup-of-cloud-functions-enhancements), ainsi que la création de [**Workspace**](https://cloud.google.com/blog/products/workspace/introducing-google-workspace) ... Qui est presque un *non-événement* puisque le service ne fait que rassembler les outils tels que **Mail**, **Calendar**, **Drive**, **Docs**, **Sheets**, **Slides**, **Meet**, etc...  

Parmi les fonctionnalités centrées sur **CloudRun** annoncées il y a quelques semaines, plusieurs avaient retenu l'attention, dont la suivante : [le server side HTTP streaming](https://cloud.google.com/blog/products/serverless/cloud-run-now-supports-http-grpc-server-streaming).  
Le but est de pouvoir fournir des temps de réponse plus rapide aux applications fonctionnant avec **CloudRun**. 

**Google** aime les développeurs et le fait savoir en [lançant une plateforme de formation](https://developers.google.com/learn) plus ou moins inspirée de **Microsoft Learn** (pour ne citer que celle-ci).


## DigitalOcean
Dernière release en date chez **DigitalOcean** : [**App Platform**](https://www.digitalocean.com/press/releases/digitalocean-launches-app-platform/)  
Encore un pas de plus vers les mastodontes du Cloud.


## Hashicorp

Du nouveau chez **Hashicorp** avec l'annonce de [11 nouveaux *Providers*](https://www.hashicorp.com/blog/announcing-11-verified-providers-for-terraform) pour **Terraform**.  
Il s'agit de **Splunk**, **Jfrog Artifactory**, **Sematext**, **Cloudsmith**, **Onelogin**, **Amixr**, **NetApp for Google Cloud**, **PhoenixNAP**, **Rancher**, **StrongDM** et **Transloadit**.  
[**Boundary**](https://www.boundaryproject.io/), le nouvel outils nous venant de chez **Hashicorp** permet tout simplement d'accéder à n'importe quel type de ressource, à l'aide de credentials stockées de manière sécurisée dans **Vault**.  
[**Waypoint**](https://www.waypointproject.io/), fait partie de ces outils, celui-ci nous permet de déployer des applications à l'aide d'un workflow, [ils en ont même profité pour expliquer pourquoi ils l'ont développé](https://www.hashicorp.com/blog/why-we-built-hashicorp-boundary).  
Après **Terraform** et **Vault**, voici la nouvelle certification : [Hashicorp Consul Associate](https://www.hashicorp.com/blog/announcing-consul-cloud-engineering-certification-for-network-automation).  

## Autres News / Acquisitions / Tooling

Et l'on parle également de l'effort de Cisco dans le *game* du Cloud/DevOps et de la sécurité autour de **Kubernetes** en acquerrant **Portshift**.  

Plus d'infos [ici](https://techcrunch.com/2020/10/01/cisco-acquires-portshift-to-raise-its-game-in-devops-and-kubernetes-security/?guce_referrer=YW5kcm9pZC1hcHA6Ly9jb20uZ29vZ2xlLmFuZHJvaWQuZ29vZ2xlcXVpY2tzZWFyY2hib3gv&guce_referrer_sig=AQAAAE81fYW2iVejMzc7iBKldtyT-tnI_-xd7XfkHeIVVrh2zPLz_SIhCA4Jqh0Xc2vt2zox1ETfgMLhnwtd5tth6E3Su3xfSf7LaAfatTpAZ4w4dTts4rV7bwCAhSlK9E45DF_PIlscdUSZNc21RO1wMbH9VEcfwI6CmtGg11rCv8Mf&guccounter=2)  
Le site de [PortShift](https://www.portshift.io/)  

[L'acquisition de **Kasten** par **Veeam**](https://www.google.com/search?q=google+traduction&oq=google+tr&aqs=chrome.1.69i57j69i59j35i39j0l3j69i60l2.3655j0j1&sourceid=chrome&ie=UTF-8) dans l'optique d'un partenariat solide autour de la gestion de données dans le Cloud.   
 
[**Okteto**](https://okteto.com/) est un service PaaS à destination des développeurs dans le but de les aider à développer des applications **Cloud Native**.  
[**Split**](https://split.io/) est également un service PaaS de **Feature Delivery** permettant, d'un côté, d'effectuer de l'ingestion de données provenant de **Google Analytics** vers des services tels que **Jira**, **Datadog** ou **New Relic**.  
Et un nouveau challenger dans la galaxie des outils de CI/CD entre en scène : [**Dispatch**](https://d2iq.com/products/dispatch) - Il fait partie de la pateforme **Kubernetes** de **D2IQ**. Le Leitmotiv derrière ce nouvel outils est : *Cloud Native applications should be built with cloud native CI/CD*  
Nouvelle annonce de **Docker** et la compatibilité entre [**Compose** et **AWS ECS** et **Microsoft ACI**](https://www.infoq.com/news/2020/10/docker-announces-compose-ecs-aci/) (Elastic Container Service / Azure Containers Instances).  
Le nouveau *Buzz Word* (le Chaos Testing) voit l'arrivée d'un petit nouveau : [Kraken](https://www.openshift.com/blog/introduction-to-kraken-a-chaos-tool-for-openshift/kubernetes)  
Un nouvel outil de monitoring d'API voit le jour : [Checkly](https://www.checklyhq.com/?ref=https://play.google.com/store/apps/details?id=io.sundeep.android&hl=en)  
