+++
date = "2020-04-10T15:29:49+02:00"
draft = false
title = "La certification Azure Solution Architect Expert"

+++

## Intro
Dans le cadre de mon travail sur les trois Cloud Providers les mieux implantés sur le marché, j'ai pu passer une première certification : AWS Solution Architect Associate.  
Et puis j'ai réalisé qu'il me serait aussi très utile de passer la certification correspondante chez Microsoft, afin de valoriser mon expérience d'un an et demi à travailler avec ce Cloud Provider lors d'une mission précédente.
Cependant je me suis aperçu qu'ils ne proposaient pas de version dite *Associate* à moins de ne passer qu'un seul des deux examens...Mais n'en passer qu'un seul ne donne pas la certification.  
Ils sont malin chez Microsoft.

Le but de cet article est de partager mon expérience et le maximum de conseils possible afin de se préparer au mieux aux épreuves.  

Votre méthode sera probablement différente, du moment que ça fonctionne pour vous c'est le principal.

## Les MOOCs
Cette liste est non exhaustive, je suis certain que vous pourrez en trouver de votre côté mais voici ceux que j'ai pu utiliser :

| Nom du site | Pour | Contre | Astuces | Prix |
| ----------- | ---- | ------ | ------- | ---- |
| Cloud Guru | Les deux examens (AZ-300 et AZ-301) sont extrêment bien couvert avec de nombreuse démo dans les vidéos | Les quizz sont absent | La période d'essai gratuite | Gratuit ou 29€/mois |
| Whizlabs | Les cours vidéos (du même style que Cloud Guru) | Cher...très cher |  | 36 € (ou moins, selon les promotions) |
| | les exams blancs avec rapport d'erreur *TRES* détaillé | mais on a le choix entre prendre les exams blanc ou les cours vidéos | | |
| Microsoft | Gratuit | Que du texte, aucune vidéo | | Gratuit |
| | | Pas assez détaillé | | |

J'avoue avoir combiné les deux premières solutions.

## Les labs
Je ne peux pas affirmer qu'il y ait des labs...autre que sur https://docs.microsoft.com/fr-fr/learn/ sur le site de Microsoft.  
Mais vous pouvez aussi ouvrir un compte sur Azure, bénéficier de la période d'essai gratuite et créer vos labs dessus.

## Le contenu couvert
Le contenu couvert par les cours est extrêmement vaste :  
- Design d'architecture.  
- Network Security.  
- Identity management (le principe de *Least Priviledges* est abordé avec les différents types de roles).  
- Cost management.  
- Migration (aussi bien *On-Premise* vers *Azure* que *AWS* vers *Azure*).  
- Application Design (Couvrant aussi bien les problématiques relative à Function App ou à Kubernetes).  
- Workflows (Messaging, Notifications).  
- Availability et SLA (Les fameux *Availability Sets* et *Scale Sets*)

Et je suis certain que j'oublie des thématiques.

Pour chaque examen, le barème suivant est associé :

| Examen | Compétences | Barème |
| ------ | ----------- | ------ |
| AZ-300 | Déployer et configurer l'infrastructure | ~30% |
|        | Workloads et Sécurité | ~25% |
|        | Créer et Déployer des applications | ~10% |
|        | Mettre en oeuvre une authentification et des données sécurisées | ~10% |
|        | Mettre au point pour le cloud et le stockage Azure | ~25% |
| AZ-301 | Déterminer les exigences de la charge de travail | ¬15% |
|        | Concevoir l'identité et la sécurité | ~25% |
|        | Concevoir une solution de plateforme de données | ~20% |
|        | Concevoir une stratégie de continuité des activités | ~20% |
|        | Concevoir le déploiement, la migration et l'intégration | ~15% |
|        | Concevoir une stratégie d'infrastructure | ~20% |

## Avant les examens
### Renseignement
Un collègue qui l'a déjà passé ? Un ami qui s'y est frotté ?  
Se renseigner autour de soi est toujours utile pour savoir à qui on s'attaque.

### Révisions et entrainements
Soyez méthodique et ne faites pas comme moi : ne commencez pas par AZ-301 pour finir par AZ-300.  
Le premier est BEAUUUUCOUP plus technique que le second...mais ca donne un avantage de couvrir de nombreux thème de manière plus approfondie.
Prévoir une petite heure par jour de cours vidéo ou un examen par soir (ou deux si vous êtes vraiment motivés).  
Pour les exams, ne vous découragez surtout pas si, au début, vous plafonnez entre 50 et 70. A un moment, certains thèmes deviendront complètement naturels pour vous et les réponses vous paraîtront d'une évidente simplicité (malgré la complexité de certain cas).

Les questions/cas pratiques proposés sont assez variés mais sont tous des QCM à choix unique ou multiples.  
Par exemple (AZ-300):
```
A company is developing an ecommerce web application. One of the modules of the application will be built using a messaging solution architecture. The modules will have the following features A Workflow run for several items published on the web application. The Workflow would be built using Azure Logic Apps. The item data would be stored in Azure BLOB storage. Which of the following would you additionally incorporate for the module?"
A. Azure Event Grid
B. Azure Event Hub
C. Azure HDInsight
D. Azure Service Bus
```
L'exemple parfait de question pour laquelle la réponse est dans la description du cas : `One of the modules of the application will be built using a messaging solution architecture`. Le service de Messaging est *Service Bus*.  
- *Event Grid* sert à gérer les événements.  
- *Event Hub* est un service d'ingestion de données.  
- *HD Insight* est un service d'analyse. 

Autre exemple (AZ-301):
```
A company is planning on deploying a stateless based application based on microservices using the Azure Service Fabric service. You need to design the infrastructure that would be required in the Azure Service Fabric service. Which of the following should you consider? Choose 2 answers from the applications given below  
A. The number of node types in the cluster  
B. The properties for each node type  
C. The network connectivity  
D. The service tier
```
Dans ce cas, il faut connaître *Azure Service Fabric* et ses options de *capacity planning* :  

- le nombre de noeud avec lequel demarrer  
- Le sizing de chaque noeud
- Reliability et Durability

Vu les réponses proposées, les deux bonnes réponses sont **A** et **B**

### Réservation
Une fois que vous vous sentez vraiment prêt : réservez vos sessions.  
- ici pour AZ-300 : https://docs.microsoft.com/fr-fr/learn/certifications/exams/az-300.  
- là pour AZ-301: https://docs.microsoft.com/fr-fr/learn/certifications/exams/az-301.  

## Pendant les examens
Il faut savoir que vous avez plus de 2h : 150 minutes exactement sur chacun.  
Le score minimum est de 70 % (700 sur 1000), je vous suggère d'avoir au moins 75 à 80% de réussite sur vos derniers exams blancs (afin d'engranger le plus de confiance possible, en sachant que sur **Whizlabs** la note minimum est de **80**).  
Si vous hésitez sur une question, *notez-la* afin de revenir dessus en fin d'examen (si vous avez le temps) pour ne pas rester bloqué.

### Le final
C'est marqué *Pass* ? Vous avez le droit à la *danse de la victoire*.  
Vous êtes surveillé tout le long du test et ne sortez pas votre téléphone tant que la fenêtre d'examen n'est pas fermée (en gros, ne faites pas comme moi...je venais de terminer).

## Conclusion

- Restez sérieux dans vos révisions mais prevoyez quand même des périodes calme,
- Trouvez la méthode adéquate selon votre profil d'étudiant (si vous êtes plus audio que visuel ou l'inverse),
- Pas de stress...si vous êtes bien préparés, tout se passera bien,
- Don't stop learning...surtout dans notre domaine
