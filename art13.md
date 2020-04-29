+++
date = "2020-04-28T22:23:00+02:00"
draft = false
title = "News from Google Cloud Platform"
+++

Du nouveau sur **Google Cloud Platform**, depuis 18h trois nouvelles sont tombées :

### Nouvelle région : **Las Vegas**  
Parmi les régions déjà disponible depuis longtemps, une nouvelle vient de faire son apparition aux **Etats-Unis**.  
Ce qui porte au nombre de quatre dans l'Ouest des **Etats-Unis** et au nombre de sept au niveau national.  
Une nouvelle région comprenant trois zones et donnant, evidemment, accès à tous les services disponible sur la plateforme.

### Optimize Dataproc costs using VM machine type
Rappelons tout d'abord ce qu'est **Dataproc** : C'est un service Cloud managé pour exécuter des cluster **Apache** **Spark**, **Presto** et **Hadoop**.  
Cette nouvelle fonctionnalité propose, dans le but d'optimiser les charges de travail, de choisir des types de machines spécifique parmi les suivants :  
- N1 : Offre le meilleur ratio prix/performance  
- E2 : Idéal pour les charges de travail petite à moyenne et nécessitant au moins 16 vCPUS  
- M2 : Pour les taches nécessitant un usage mémoire intensif  
- C2 : Idéal pour du *machine-learning*

### Explore Anthos
**Google Cloud** propose désormais un **Anthos Sample Deployment** afin d'explorer les fonctionnalités du service.  
Avec ce *sample*, vous pourrez étudier les *Core Components* du service, ce qui inclue : 
- Kubernetes certifié qui fournit une orchestration de niveau production de vos applications conteneurisées,  
- Outils de gestion de la configuration qui utilisent une approche GitOps moderne pour garder votre environnement de production synchronisé avec votre état souhaité, tel que stocké sous contrôle de version,  
- Un *Service Mesh* pour gérer, et sécuriser, les communications réseau internes *service-to-service*,  
- Un ensemble d'outils de tableau de bord intégrés, y compris la surveillance et l'alerte d'objectif de niveau de service (SLO), qui fournissent un aperçu riche des performances des applications
