
+++

date = "2018-05-11T10:05:00+02:00"
draft = false
title = "La gestion des API avec Gravitee"

+++

## Les APIs vous dites ?
Ah...les micro services, toute cette hype récente autour des API proposées par des micro services qui communiquent entre eux lors de développement de gros projets, qu'ils soient internes ou externes.  
A l'origine, les projets étaient monolithique et plus le nombre de ligne de code grossissait, plus ça pouvait devenir compliqué à débugger lorsqu'un *glitch* apparaissait lors d'un déploiement de l'application en question (et souvent...ça partait directement en production à l'époque).  
D'autant plus que plusieurs inconvénients avaient la fâcheuse tendance à survenir durant le cycle de vie du projet :

- *Evolutivité* : Plus les lignes de codes s'accumulent, plus le code devient lourd et difficile à optimiser et à modifier
- *Fiabilité*
- *Scalabilité* : En cas de besoin d'augmentation de charge sur l'application, il faut l'avoir prévu dès le départ pour éviter tout comportement *exotique*.

Pour éviter des problèmes de ce style, l'idée fut d'adapter chaque composants d'un projet monolithique en plusieurs mini projets qui communiqueront ensemble par des *messages* (des appels de services REST/JSON).

## Oui...mais...comment les gérer ?
Effectivement...très bonne question.  
Quand il n'y a que deux micro services, leur gestion est extrêmement simple...mais lorsqu'il y en a, par exemple, une vingtaine ?  
Ca peut devenir un vrai bordel...Mais pas de panique, il existe des **API Managers** disponible sur internet parmi lesquels : **Gravitee**.  
Pourquoi celui-ci par rapport à d'autres, tels que :  

- **Kong**
- **WSO2**
- **Tyk**
- **Umbrella**
- **Apiman**
- **ApiAxl**

Et je ne mentionne pas les **API Managers** proposés par les Cloud Providers, dont celui de **Microsoft Azure** (surtout lorsque l'on cherche à éviter tout effet de *vendor locking*).

## Gravitee ?
Mais pourquoi celui-là (je suis certain que l'un de vous se pose la question) ?  
Pour plusieurs raisons : 

- Le projet est français, venant du *chnord* (C'est ma femme qui être contente)
- Il sert aussi bien d'*API manager* et d'*API gateway*
- Il est extensible, via des plugins déjà disponible ou à développer soi-même (en **java**)
- Dispose d'une architecture facilement scalable
- Et propose des fonctionnalités natives d'authentification et de stockage (que j'aborderais plus tard)

### Comment l'installer ?
L'installation **Gravitee** nécessite **Java** (*Obviously*), un datastore ([MongoDB](https://www.mongodb.org/downloads#production) ou [ElasticSearch](https://www.elastic.co/downloads/elasticsearch), par exemple) ainsi que les composants **Gateway** et **Manager** (Les deux composants s'installent via la même archive) : 

```
curl -L https://download.gravitee.io/graviteeio-apim/distributions/graviteeio-full-{VERSION}.zip -o gravitee-standalone-distribution-{VERSION}.zip
unzip gravitee-standalone-distribution-{VERSION}.zip
./bin/gravitee -d -p=/var/run/gio.pid
```

### Comment fonctionne-t-il ?
Parfois un schema est plus explicite qu'un long discours...Donc en voici un : 

![enter image description here](https://docs.gravitee.io/images/architecture/graviteeio-global-architecture.png)

Les grands composants de **Gravitee** sont resumés dans le schema précédent :

- **Gateway** reçoit les requêtes des consommateurs d'API, utilisateurs lambda ou autres micro services
- **Manager** comprend :
**Portal** est l'interface utilisateur  
**Management-API** est l'API de Gravitee (car oui, même Gravitee a la sienne)

Maintenant que l'**API Manager** est démarré, il est temps de référencer nos API et ainsi observer comment elles communiquent.  
**Gravitee** fait appel à deux concepts bien distincts : **API Publisher** et **API Consumer**

#### API Publisher
Ce concept se décompose en cinq étapes, exécutables aussi bien via le **Portal** que via l'API de **Gravitee**, va nous permettre de créer une entrée API à gérer dans l'**API Gateway** : 

##### Créer l'API
Ici on ne créé pas l'API à proprement parler, elle est censée exister et être accessible facilement.  

```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     -d '{"name":"{API_NAME}","version":"1","description":"Gravitee.io Echo API Proxy","contextPath":"/myfirstapi","endpoint":"https://api.gravitee.io/echo"}' \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/apis
```

##### Créer le *plan*  
Le *plan* fait référence uniquement au trafic généré par les requêtes...car le nombre de requêtes est également monitoré sur l'UI de **Gravitee**:  
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     -d '{"name":"My Plan","description":"Unlimited access plan","validation":"auto","characteristics":[],"paths":{"/":[]},"security":"api_key"}' \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/apis/{API-ID}/plans
```

##### Publier le *plan*
Une fois créé, il faut le publier afin qu'il soit visible pour tous:  
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/apis/{API-ID}/plans/|plan-id|/_publish
```

##### Déployer l'API
L'avant dernière étape, avant son activation, permet de déployer l'API sur les instances active de l'**API Gateway**:  
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -X POST \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/apis/{API-ID}/deploy
```

##### Démarrer l'API
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -X POST \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/apis/{API-ID}?action=START
```

#### API Consumer
Ce concept en revanche se compose des trois étapes suivantes :  
- Créer l'application 
- Souscrire à l'API
- Tester l'API

##### Créer l'application
Gravitee propose différentes manières de sécuriser/consommer les APIs via les *plans* disponibles ou via les [policies](https://docs.gravitee.io/apim_policies_apikey.html).  
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -H "Content-Type:application/json;charset=UTF-8" \
     -X POST \
     -d '{"name":"My First Application","type":"Web","description":"Web client for the Gravitee.io Echo API"}' \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/applications
```

##### Souscrire à l'API
En retournant dans la liste des APIs publiées sur l'UI de **Gravitee**, on peut sélectionner l'API souhaitée, y souscrire, choisir le *plan* et l'applicaton qui va la consommer...ou via l'API de **Gravitee**:  
```
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" \
     -X POST \
     http://{MANAGEMENT_API_SERVER_DOMAIN}/management/applications/{APPLICATION-ID}/subscriptions/?plan={PLAN-ID}
```

##### Tester l'API
Cette étape n'est pas vraiment obligatoire, mais il est toujours bon de tester la consommation d'une API par l'application qui en a besoin...au cas où:  
```
curl -H "X-Gravitee-Api-Key: {API_KEY}" \
     http://{GATEWAY_SERVER_DOMAIN}/{API_NAME}
```

### On va un peu plus loin ?
L'idée de *kubernetiser* la chose me tente énormément, d'autant plus que **Gravitee** est disponible en image **Docker** (rendu disponible par l'équipe à l'origine du projet).  
Je vais travailler sur un *Lab* pour un prochain article...Mais n'étant pas développeur, il va falloir que je trouve des solutions pour les API (Autre que du *Hello World* bien évidemment).