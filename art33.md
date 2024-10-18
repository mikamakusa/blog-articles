+++
date = "2024-10-18T15:29:49+02:00"
draft = false
title = "La théorie du Chaos"

+++

Le **Chaos Engineering** est une pratique qui consiste à tester la résilience d'un système informatique en introduisant intentionnellement des défaillances ou des perturbations pour observer comment le système réagit et récupère.<br>
L'objectif est de découvrir les faiblesses ou vulnérabilités avant leur apparitions dans des conditions réelles.<br>

L'idée principale derrière le **Chaos Engineering** est que les systèmes modernes, en particulier les infrastructures distribuées ainsi que les microservices, sont complexes et sujets à des défaillances imprévues. <br>
Il devient donc crucial de tester comment ils se comportent face à des interruptions, des pannes ou des comportements inattendus, afin de mieux comprendre leur résilience.

## Revue des forces en présence
### Gremlin
**Gremlin** est l'un des outils les plus populaires dans le domaine du **Chaos Engineering**. <br>
Il permet de créer des scénarios de perturbation sur différents types d’infrastructures (cloud, serveurs physiques, containers, etc.). <br>
**Gremlin** offre des expériences de chaos à différents niveaux tels que :<br>
- l'injection de latence réseau, <br>
- La simulation de pannes de disque,<br> 
- L'épuisement de ressources.<br>

### Chaos Monkey
**Chaos Monkey** fait partie de la suite **Simian Army** développée par **Netflix**. <br>
Cet outil arrête aléatoirement des instances de services en production pour tester la résilience et l'auto-récupération des applications cloud. <br>
L'objectif est de s'assurer que les services peuvent continuer à fonctionner même en cas de perte d'une ou plusieurs instances.

### Litmus Chaos
**LitmusChaos** est une plateforme open-source qui permet de réaliser des expériences de chaos sur des applications Kubernetes. <br>
En fournissant un cadre en vue d'exécuter des tests de chaos sur des microservices dans un environnement Kubernetes, il offre une grande flexibilité dans le but de personnaliser les scénarios de test.

### Chaos Toolkit
**Chaos Toolkit** est un outil open-source qui permet de créer et d'exécuter des scénarios de chaos de manière programmable. <br>
Il s'intègre relativement bien avec des environnements cloud ou containerisés et permet d’automatiser les tests afin d'évaluer la résilience d'une application.

### Pumba
**Pumba** est un outil spécialisé pour injecter du chaos dans des environnements Docker. <br>
En simulant des pannes réseau, des arrêts ou des redémarrages de containers, ou encore en introduisant des délais de latence, il est en mesure de perturber le fonctionnement des conteneurs.

### PowerfulSeal
Cet outil est conçu pour tester la résilience des applications déployées sur des cluster **Kubernetes** en perturbant directement les pods ou les nœuds du cluster. <br>
**PowerfulSeal** peut fonctionner en mode manuel pour tester des cas spécifiques ou en mode autonome pour injecter automatiquement des perturbations.

### ToxiProxy
**Toxiproxy** est un *proxy toxique* conçu pour simuler des défaillances réseau. <br>
Il peut introduire des latences, limiter la bande passante ou même couper la connexion entre des services, ce qui permet de tester comment les applications réagissent à des conditions réseau dégradées.

Parmi les fournisseurs de services Cloud, seuls **AWS**, **Azure** et **Oracle** proposent des services de **Chaos Engineering** via, respectivement : <br>
- **Fault Injection Simulator** au sein du **Resilience Hub** d'**AWS**,<br>
- **Chaos Studio**,<br>
- **Fault Injection**<br>

Ces services permettent de tester l'injection d'erreurs au sein de services **PaaS** tels que les bases de données relationnelles, les services à base de conteneurs ou d'instances Cloud ainsi qu'au niveau du réseau (latence ou simulations de pannes).

## Quels sont les principes fondamentaux ?

1. **Définir un état stable** : Identifier ce que signifie le bon fonctionnement du système, souvent mesuré par des indicateurs clés de performance (KPI), comme le temps de réponse ou la disponibilité.
2. **Créer des hypothèses** : Émettre des hypothèses sur ce qui pourrait se passer en cas de défaillance (par exemple, "si ce service tombe en panne, l'expérience utilisateur ne sera pas affectée").
3. **Injecter des perturbations contrôlées** : Introduire de manière contrôlée des pannes (comme l'arrêt d'un serveur, la latence réseau accrue, la perte de données) pour tester les hypothèses.
4. **Observer et analyser** : Surveiller comment le système réagit à la perturbation et comparer les résultats aux attentes. Si le système ne se comporte pas comme prévu, cela révèle une faiblesse qui peut être corrigée.
5. **Automatisation et sécurité** : Les tests sont généralement automatisés, et l'objectif est de les exécuter de manière sécurisée en production ou dans un environnement de préproduction, sans affecter les utilisateurs finaux.

## Types de cibles
Les outils de **Chaos Engineering** visent différents types de cibles (infrastructure, services, réseau, etc.) en fonction de leur spécialisation. <br>
Voici un aperçu des types de cibles sur lesquels il est possible d’exécuter des expériences de chaos pour chacun des outils mentionnés :<br>

- **Machines Virtuelles** (VMs) : Gremlin, AWS FIS, Azure Chaos Studio, OCI Fault Injection, Chaos Toolkit.<br>
- **Containers** (Docker, Kubernetes) : Gremlin, LitmusChaos, Pumba, Chaos Toolkit, PowerfulSeal, AWS FIS, OCI Fault Injection.<br>
- **Réseau** (latence, pannes) : ToxiProxy, Gremlin, Chaos Toolkit, LitmusChaos, PowerfulSeal, AWS FIS, Azure Chaos Studio, Pumba.<br>
- **Services Cloud** : Gremlin, AWS FIS, Azure Chaos Studio, OCI Fault Injection.<br>
- **Serverless** : AWS FIS, Azure Chaos Studio, Gremlin, Chaos Toolkit (via intégration API).<br>
- **Bases de données/API** : ToxiProxy, Gremlin, AWS FIS, Chaos Toolkit.<br>


En fonction des environnements et des infrastructures sur lesquels il sera appliqué : chaque outil de **Chaos Engineering** a ses cibles spécifiques. <br>
Certains outils vont se concentrer beaucoup plus sur les containers et Kubernetes (Pumba, LitmusChaos, PowerfulSeal), tandis que d'autres permettent d'effectuer des tests sur une palette de services bien plus étendus.<br>
Et puis, il reste une troisième catégorie : les services managés (**sur étagère** comme disent certains) dont le principal défaut reste le fait d'être limité à une plateforme Cloud.

## Les fonctionnalités specifiques de chacun
**Gremlin** étant l'outil de **Chaos Engineering** le plus polyvalent du marché et, accessoirement, **Cloud Agnostique**, j'ai décidé de creuser un peu plus le sujet des fonctions spécifiques.<br>

- Les **FailureFlags**
Il s'agit d'une fonctionnalité totalement inédite à **Gremlin** et compatible uniquement avec **AWS**.
Utilisée pour tester la résilience des applications (aussi bien des **Lambda** que des assets déployés sur **Kubernetes** ou sur **ECS/Fargate**). Cette fonctionnalité permet aux équipes de **Chaos Engineering** d'injecter des défaillances directement dans le code des applications.<br><br>

- Les **Scenarios**
Tout comme les **FailureFlag**, il s'agit d'une fonctionnalité propre à **Gremlin** grâce à laquelle il est possible de créer un workflow comprennant une ou plusieurs expériences de défaillances/pannes séquentiellement ou parallèlement, ce qui est idéal pour reproduire des situations réelles.<br><br>

- Les **GameDays**
Idéal pour tester les synergies entre équipes, cette fonctionnalité aide à organiser des événements afin de tester la résilience des systèmes et d'analyser les résultats pour des actions concrètes.<br><br>

- Les **Processus de terminaison aléatoires**
Une fois n'est pas coutume, il s'agit d'une fonctionnalité propre à **Chaos Monkey**, la plateforme de **Netflix**, et est utilisée pour *tuer* des instances de manière aléatoire pour tester la capacité d'un système à résister à la perte de services ou de machines.<br><br>

- Les **Attaques Kubernetes Spécificiques**
Cette fois-ci, il s'agit d'une fonctionnalité propre à **LitmusChaos**. Elle permet de simuler des erreurs spécifiques à Kubernetes, comme des CrashLoopBackOff, ou des pannes de services sous forme de conteneurs.

## Gremlin, les FailureFlags et les Lambda
Pour le moment limité à **AWS** et quatre langages (à savoir **Nodejs**, **Python**, **Java** et **Go**), c'est la fonctionnalité de **Gremlin** dont le ticket d'entrée est peut-être d'un niveau plus élevé que les autres.<br>
Non seulement elle nécessite une bonne connaissance d'un langage en particulier mais également le fonctionnement des **Lambda** sur **AWS** et comment en développer.<br>

### Un exemple de code
```python
import os
import logging
import time
from failureflags import FailureFlag, defaultBehavior
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

logger = logging.getLogger()
logger.setLevel(logging.INFO)
patch_all()

def customBehavior(ff, experiments):
    logger.debug(experiments)
    return defaultBehavior(ff, experiments)

def lambda_handler(event, context):
    start = time.time()

    # Change 2: add a FailureFlag to your code
    FailureFlag("http-ingress", {}, debug=True, behavior=customBehavior).invoke()

    end = time.time()
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': {
            'processingTime': f"{start - end}",
            'isActive': active,
            'isImpacted': impacted
        }
    }
```

Bien que ce ne soit pas vraiment explicite, mais cet exemple de code aura besoin de plusieurs modules Python dont *setuptools* en plus de *failureflags* et d'*aws_xray_sdk*.<br>

### Création de l'archive
L'étape écriture du code est derrière nous...mais avant de la déployer, il y a quelques étapes à suivre au préalable : <br>
- Le fichier *requirements.txt* dans lequel seront inscrit les modules python nécessaire à la **Lambda**,<br>
- Installer en local les modules python, à l'aide de la commande `pip install -r requirements.txt -t .`,<br>
- Télécharger le fichier *config.yaml* dans lequel on peut trouver **team_id**, **team_certificate** et **team_private_key**, sans oublier de décommenter **tags** pour y ajouter des tags personnalisés,<br>
- Créer une archive *.zip* du contenant le code, les fichiers *config.yaml* et *requirements.txt ainsi que les modules Python.

### Déployer la Lambda
Déployer la **Lambda** sur la console **AWS** est relativement simple et peu chronophage, cependant il y a tout de même certains détails importants : <br>
- Vous pouvez déploer la **Lamba** en chargeant l'archive *.zip* depuis le disque local ou depuis un stockage **S3**,<br>
- Vous aurez besoin d'un **Layer** spécial **Gremlin** au format *arn:aws:lambda:\<region\>:\<version\>layer:gremlin-lambda-\<architecture\>*, par exemple : *arn:aws:lambda:us-east-2:044815399860:layer:gremlin-lambda-x86_64:13* (plus de détails [ici](https://www.gremlin.com/docs/failure-flags-lambda))
- Vous aurez également besoin de définir les variables d'environnement : <br>
  - **FAILURE_FLAGS_ENABLED** à **true** ou **1**, <br>
  - **GREMLIN_LAMBDA_ENABLED** à **true** ou **1**,<br>
  - **GREMLIN_CONFIG_FILE** à **/var/task/config.yaml**<br>

### Exploiter les FailureFlags
Vous pouvez invoquer la Lambda et tester différents scénarios en passant différents paramètres d'événements.
Par exemple : 
```bash
# Introduire une latence de 2000 milisecondes
aws lambda invoke --function-name myLambda --payload '{ "latency": 2000 }' output.txt
# introduire entre 2 000 et 2 200 millisecondes de latence là où il existe une probabilité uniforme pseudo-aléatoire de tout retard entre 2 000 et 2 200.
aws lambda invoke --function-name myLambda --payload '{"latency": {"ms": 2000,"jitter": 200}}' output.txt
```

## Conclusion
Le but ultime du **Chaos Engineering** est de rendre les systèmes plus robustes, en s'assurant qu'ils continuent à fonctionner correctement même en cas de défaillances imprévues, et d'améliorer la confiance dans leur résilience en production.<br>
L'offre d'outils spécialisés est vaste mais peu d'outils sortent vraiment du lot...tel que **Gremlin**, le plus polyvalent et Cloud agnostique d'entre eux et permet d'adresser de nombreux cas et usages.