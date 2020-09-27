+++
date = "2020-09-30T22:23:00+02:00"
draft = false
title = "News from the Clouds"
+++

Review des nouveautés des fournisseurs de services Cloud **Azure**, **AWS** et **Google Cloud**.

# AWS
## AWS Transcribe - 15 Septembre 2020
Dorénavant ce service supporte la fonctionnalité **Automatic Language Identification**.  
**AWS Transcribe** est un service de reconnaissance vocale automatique qui a évolué depuis sa création avec la prise en charge de 31 langues dont 6 en temps réel.  
Un cas d'utilisation populaire de ce service est la transcription des appels clients permettant aux entreprises d'effectuer des analyses à l'aide de techniques de traitement de langage naturel.  
Grâce à cette nouvelle fonctionnalité, **AWS Transcribe** peut désormais identifier automatiquement la langue dominante dans un enregistrement audio et, par conséquent, aider les clients à créer des flux de travail de transcription plus efficaces en supprimant le marquage manuel.  
En seulement 30 secondes, **AWS Transcribe** peut générer efficacement des transcriptions dans la langue parlée sans perdre de temps et de ressources en étiquetage manuel. L'identification automatique de la langue dominante est disponible en mode de transcription par lots pour les 31 langues.

Plus d'infos [ici](https://aws.amazon.com/fr/blogs/aws/amazon-transcribe-now-supports-automatic-language-identification/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29).

## AWS EC2 T4g Instances - 15 Septembre 2020
Après avoir introduit les instances de type T3 il y a deux ans afin de proposer un moyen d'exécuter des charges de travail le plus rentable possible.  
**AWS** recommence avec, cette fois ci, les instances de type T4g, une nouvelle génération de type d'instance extensible à faible coût optimisée par AWS Graviton2, un processeur conçu sur mesure par AWS à l'aide de cœurs Arm Neoverse 64 bits.  
Avec ce type d'instances, vous pouvez profiter d'un avantage de performances allant jusqu'à 40% à un coût 20% inférieur par rapport aux instances T3, offrant le meilleur rapport prix / performances pour un plus large éventail de charges de travail.  
Disponible en 7 tailles (voir ci-dessous), ce type d'instance est prévu pour faire tourner des applications n'ayant pas besoin d'un CPU à pleine puissance tout le temps, tout en bénéficiant d'un modèle de facturation identique aux instances de type T3.  

| Name | vCPUs | Baseline Performance/vCPUs | CPU Credits Earned/Hour | Memory |
| ---- | ----- | -------------------------- | ----------------------- | ------ |
| t4g.nano | 2 | 5% | 6 | 0.5 GiB |
| t4g.micro | 2 | 10% | 12 | 1 GiB |
| t4g.small	| 2	| 20% | 24 | 2 GiB |
| t4g.medium | 2 | 20% | 24 | 4 GiB |
| t4g.large	| 2 | 30%	| 36 | 8 GiB |
| t4g.xlarge | 4 |40% | 96 |16 GiB |
| t4g.2xlarge | 8 | 40% | 192 | 32 GiB |
 
Plus d'infos [ici](https://aws.amazon.com/fr/blogs/aws/new-t4g-instances-burstable-performance-powered-by-aws-graviton2/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+AmazonWebServicesBlog+%28Amazon+Web+Services+Blog%29).

# Azure
## Azure Spring Cloud - 2 septembre 2020
Un nouveau Service Managé disponible dans la galaxy **Azure** : **Azure Spring Cloud**.  
Grâce à ce nouveau service, vous pouvez vous concentrer sur le code et la construction de votre Web Application sans avoir a vous creuser la tête sur la gestion de l'infrastructure nécessaire.  
Déployez simplement le package *.JAR* ou bien uniquement le code et **Azure Spring Cloud** câblera automatiquement vos applications avec le moteur d'exécution du service Spring. Une fois déployé, vous pouvez facilement surveiller les performances des applications, corriger les erreurs et améliorer rapidement les applications.

Plus d'infos [ici](https://azure.microsoft.com/fr-fr/blog/azure-spring-cloud-a-fully-managed-service-for-spring-boot-apps-is-now-generally-available/).

## Connectors for AWS in Azure Cost Management - 2 Septembre 2020
En 2019 avait été annoncé en *Preview* ce nouveau service permettant aux utilisateurs de surveiller leurs dépenses sur les plateformes **AWS** et **Azure** directement sur le portail **Azure**.  
Ce service est désormais disponible pour tous.

Plus d'infos [ici](https://azure.microsoft.com/fr-fr/blog/connector-for-aws-in-azure-cost-management-billing-is-now-generally-available/).

# Google Cloud
## Cloud Run for Anthos
La création de microservices sur Google Kubernetes Engine (GKE) offre une flexibilité maximale pour créer des applications, tout en bénéficiant de l'échelle et de l'ensemble d'outils que Google Cloud a à offrir. Mais une grande flexibilité s'accompagne d'une grande responsabilité. L'orchestration de microservices peut être difficile, nécessitant une implémentation, une personnalisation et une maintenance non triviales des systèmes de messagerie.  
Cloud Run for Anthos inclut désormais une fonctionnalité d'*Events* qui permet de créer facilement des systèmes événementiels sur Google Cloud. Désormais en version bêta, la fonctionnalité événementielle de Cloud Run for Anthos assume la responsabilité de la mise en œuvre et de la gestion de l'infrastructure d'événements, afin que leurs utilisateurs n'aient pas à s'en occuper.  

Plus d'infos [ici](https://cloud.google.com/blog/products/serverless/cloud-run-for-anthos-adds-events).

## Cloud Migration
Tout ce que vous pourriez avec besoin de savoir sans oser le demander, c'est par [ici](https://cloud.google.com/blog/products/cloud-migration/guide-to-all-google-cloud-migration-guides).

## gVisor
La sécurité est une priorité absolue pour Google Cloud.  
Partant de ce principe, ils ont créé certains des composants fondamentaux des conteneurs, comme les **cgroups**, et font partie des *early adopter* conteneurs pour leurs systèmes internes.  
L'amélioration de la sécurité de cette technologie a conduit au développement de gVisor, un bac à sable de sécurité des conteneurs, désormais open source et intégré dans plusieurs de leurs services Cloud.  
gVisor permet de protéger les utilisateurs à chaque découvert de vulnérabilité du noyau Linux.

Plus d'info [ici](https://cloud.google.com/blog/products/containers-kubernetes/how-gvisor-protects-google-cloud-services-from-cve-2020-14386).

