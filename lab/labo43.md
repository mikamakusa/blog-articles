+++
date = "2021-09-02T15:29:49+02:00"
draft = "false"
title = "Le Labo #37 | Un petit tour du côté du framework Operator"
+++

# Framework Operator et déploiement de Thanos et Prometheus sur Kubernetes

Il y a environ deux ans, j'avais publié un article autour de Thanos et d'un stockage sur Google Cloud Storage relativement simple car il n'était pas du tout question de Prometheus Operator et de sa configuration ni de Kubernetes. Cette fois-ci, je vais aller plus loin en abordant les thématiques citées précédemment (à savoir Prometheus Operator et Kubernetes), mais également les exporters et comment les connecter à notre Prometheus, le tout dans un environnement multi-custer...sans oublier les certificats TLS.

## Les ingrédients de la recette
Nous allons avoir besoin de plusieurs éléments :  
- Deux clusters Kubernetes, vous pouvez faire appel à un service managé, à une solution pré-packagée Vagrant ou les monter vous-même,  
- Helm - la toute dernière version, ce qui vous évitera des sueurs froides à cause de Tiller,  
- et...c'est tout.

Je ne vais pas détailler le montage de cluster Kubernetes car, selon la solution que vous choisirez, de nombreux tutoriels sont disponibles sur Internet (et s'il s'agit de Service Managé Cloud, il suffit de se laisser guider pour l'interface).

Je vais également partir du principe que Thanos Querier est déjà déployé sur un des deux cluster (idéalement celui sur lequel vous n'avez pas prévu de déployer Prometheus Operator).

### Prometheus
Je ne pense pas avoir besoin de présenter Prometheus, bien que tout le monde (ou presque) le connaisse, voici une petite présentation : Prometheus permet de collecter les métriques exposées par les applications et remontées par des exporters auxquels il se connecte. Par définition, ce dernier ne propose pas de stockage réellement persistant car de très courte durée. Les métriques, au format time series sont requêtable aussi bien via l'interface utilisateur de Prometheus qu'avec celle de Grafana.

### Helm
Maintenant que Prometheus et Operator Framework ont été présentés, il faut les déployer...et c'est à ce moment qu'Helm intervient car de nombreux charts proposent le déploiement de Prometheus, mais peu incluent Prometheus Operator ainsi qu'un sidecar Thanos qui nous sera utile pour la suite des opérations.  
Concernant l'architecture de l'outil, il n'y aucune différence fondamentale malgré un apport de quelques objets spécifique à Kubernetes (les Custom Resource Definition ou CRD sur lesquels je vais revenir).

### Le framework Operator
Il s'agit d'un toolkit spécifique à Kubernetes permetant de gérer plus simplement les services déployés. S'appuyant sur des concepts de base de ressources et de contrôleurs et incluant des mécanismes spécifiques à une application bien définie, un Operator permet de déployer et de manager un service sans avoir a connaître à fond sa configuration et son cycle de vie.

#### Les Custom Resource Definition ou CRD
Introduit depuis la version 1.7, les Custom Resource Definition sont un moyen d'étendre l'API de Kubernetes pour des cas bien particulier qui ne seraient, à la base, pas directement couvert par le coeur de Kubernetes. Bien que similaire aux ressources par défaut et définissable de la même manière (un manifeste au format YAML), l'API va traiter les custom resource definition comme toutes les autres ressources.  

Parmi les CRD qui seront déployés :  
- ServiceMonitor : La sonde qui pointera sur les Services afin d'y collecter les métriques,  
- AlertManager : Gèrera les alertes sur les expression définies en fonction de seuils,  
- PodMonitor : La sonde qui pointe sur les Pods afin de collecter les métriques,  
- Prometheus : l'Instance qui agrège les métriques,  
- Thanos Sidecar : Se connecte à Prometheus et offre un point d'accès à Thanos Querier.

#### Les objets Monitor
J'ai évoqué les CRD ServiceMonitor ainsi que PodMonitor un peu plus haut sans pour autant expliquer réellement ce que ça pouvait être. Il s'agit d'un niveau d'abstraction relatif à l'export de métriques et à la configuration des endpoints pour Prometheus.  
Ainsi il ne sera plus nécessaire de modifier la configuration de Prometheus (comme on pourrait le faire avec le même outils déployé sur un serveur sans Service Discovery) et, par conséquent, de gagner énormément de temps lors du déploiement et d'éviter de potentielles erreurs de configuration, ce qui n'est pas négligeable.  

De plus leur utilisation dépend de la configuration de l'application à déployer.  
Par exemple, une instance Wordpress déploie les Pods à l'aide de l'objet Deployment, et expose via Service et Ingress...et nécessite PodMonitor ou ServiceMonitor.  
En revanche, Kafka en mode StatefulSef expose directement un port du Worker pour être sollicité directement...et nécessite, par conséquent, un PodMonitor.  

Pour en revenir à Prometheus, ce dernier peut être configuré pour collecter les métriques exposées par les deux types d’objets Monitor dont le descripteur de déploiement. 
Ce dernier est à ajouter dans le fichier values.yaml du chart  de cette manière :  
- pour le chart stable/prometheus-operator :  
```yaml
prometheus:
  [...]
  prometheusSpec:
    serviceMonitorSelector:
      prometheus.io/monitor: "true"
```  
- pour le chart bitnami/kube-prometheus :  
```yaml
prometheus:
  [...]
  serviceMonitorSelector:
  	prometheus.io/monitor: "true"
```  

Il nous reste à définir comment récupérer dynamiquement les Monitor pourvu du label défini plus haut. La configuration peut varier d'un chart à l'autre, mais le point commun reste le serviceMonitor, dans le cas du chart stable/prometheus-operator on ajoutera le label de cette façon :  
```yaml
metrics:
  enabled: true
  [...]
  serviceMonitor:
    enabled: true
    additionalLabels:
      prometheus.io/monitor: "true"
```  

Par contre, c'est un peu différent sur le chart de Bitnami :  
```yaml
serviceMonitor:
    https: true
    interval: ""
    metricRelabelings:
      prometheus.io/monitor: "true"
    [...]
```

Une fois l’application déployée, Prometheus collecte automatiquement ses métriques.

## Un peu de plongée
Maintenant que les différents ingrédients ont été présentés , il est temps de plonger dans le vif du sujet qu'est le déploiement du Sidecar Thanos, de Prometheus et des certificats TLS qui leur seront associés.

## Les certificats TLS
Au départ, il ne s'agit que d'un bonus destiné à complexifier un peu l'exercice.
Ce n'est pas la création de toute la chaîne de certificat qui peut être la plus challengeante, mais quel certificat ajouter a quel niveau de chaque chart.

### Création de la chaîne de certification  
Pour créer toute la chaîne de certification, nous allons avoir besoin :  
- d'un certificat TLS,  
- d'une autorité de certification, dans le cas de certificats auto-signés. Cependant, le principe reste le même si vous utilisez Let's Encrypt,  
- de certificats type intermédiaire (optionnels),
-  de certificats finaux
Pour cela, il faudra exécuter les commandes suivantes :  
- Certifcats TLS
```bash
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout thanos_tls.key -out thanos_tls.crt -config req.conf
```
- Autorité de certification
```bash
openssl req -x509 -sha256 -newkey rsa:2048 -keyout thanos_ca.key -out thanos_ca.crt -days 356 -nodes -config req.conf
```
- Certificats finaux
```bash
openssl req -new -newkey rsa:2048 -keyout thanos_server.key -out thanos_server.csr -nodes -config req.conf
openssl x509 -req -sha256 -days 365 -in thanos_server.csr -CA thanos_ca.crt -CAkey thanos_ca.key -set_serial 01 -out thanos_server.crt
```

Si votre but n'est pas forcément de chiffrer tous les échanges entre les composants de Thanos, la création de certificats TLS avec Openssl sera amplement suffisante. Par contre, il s'avère qu'entre le Query et le StoreGateway, l'absence de Subject Alternative Name dans votre chaîne de certificat empêchera la connexion entre les deux.
Après quelques recherches, j'ai découvert une alternative à openssl :  
- [cfssl](https://github.com/cloudflare/cfssl) - Présentation de l'outil,  
- [Un tutoriel](https://rob-blackbourn.medium.com/how-to-use-cfssl-to-create-self-signed-certificates-d55f76ba5781).  

Grâce à ce dernier, j'ai pu contourner le blocage rencontré avec la chaîne de certificats généré via Openssl.

### Déploiement des secrets sur Kubernetes
Une fois les certificats disponibles, il est nécessaire de les rendre disponible pour Kubernetes.
Dans notre cas, il y a au moins deux clusters, donc il faudra changer de contexte pour déployer les secrets dans chacun d'eux.
```bash
kubectl config set-context [CONTEXT]
kubectl create secret generic [GENERIC_K8S_SECRET_NAME] --from-file=[CA-CERT]=thanos_ca.crt --from-file=[TLS-CERT]=thanos_client.crt --from-file=[TLS-KEY]=thanos_client.key
kubectl create secret tls [TLS_K8S_SECRET_NAME] --key=thanos_client.key --cert=thanos_client.crt
```

## Déploiement de Prometheus Operator
Maintenant que les secrets sont exploitables par les futurs déploiements, nous pouvons passer à la seconde étape...c'est à dire le déploiement de Prometheus Operator. Cette étape se compose de 3 phases :  
- Ajout du dépôt et récupération du chart,  
- Edition du chart,  
- Déploiement.

### Ajout du dépot et récupération du chart
Cette étape est certainement la plus simple des trois car trois commandes suffisent :  
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm pull bitnami/kube-prometheus -d .
tar xzf kube-prometheus-6.1.4.tgz && cd kube-prometheus
```

### Edition du chart
Plus tôt dans cet article, j'ai abordé le sujet de Prometheus Operator et des objets Monitor, cependant ce ne seront pas les seuls éléments sur lesquels il faudra se concentrer dans le fichier values.yaml livré avec le chart précédemment récupéré. Ce dernier contient plusieurs sections relativement importantes :  
- operator,  
- prometheus,  
- altermanager,
- exporters,
- node-exporters

La première section est relative à Prometheus Operator et à sa configuration, nous pouvons déjà remarquer que ce dernier est activé par défaut, qu'un serviceAccount sera créé, que le service sera de type ClusterIP avec port 8080, que le ServiceMonitor est également activé par défaut et est déployé avec livenessProbe et readinessProbe.  
```yaml
operator:
  enabled: true
  image:
    registry: docker.io
    repository: bitnami/prometheus-operator
    tag: 0.48.1-debian-10-r14
    pullPolicy: IfNotPresent
	[...]
  serviceAccount:
    create: true
    name:
  [...]
  service:
    type: ClusterIP
    port: 8080
    [...]
    externalTrafficPolicy: Cluster
	[...]
  serviceMonitor:
    enabled: true
    interval: ""
    metricRelabelings: []
    relabelings: []
  [...]
  livenessProbe:
    enabled: true
    path: /metrics
    [...]
  readinessProbe:
    enabled: true
    path: /metrics
    [...]
  [...]
```

La seconde section gère le déploiement de l'instance Prometheus là où la première section prenait en charge l'automatisation de sa configuration et gestion. Bien que la majorité des options soient commune aux deux sections, nous avons le sidecar Thanos, dans celle qui nous intéresse actuellement, désactivé par défaut et le type de service à remplacer par LoadBalancer (afin de faciliter la communication entre deux cluster Kubernetes différents)...à vous de définir le type d'équilibrage de charge entre Internal (pour plus de sécurité) ou External.  
```yaml
  thanos:
    create: true
    image:
      registry: docker.io
      repository: bitnami/thanos
      tag: 0.21.1-scratch-r0
      pullPolicy: IfNotPresent
      pullSecrets: []
    [...]
    service:
      type: LoadBalancer
      port: 10901
      [...]
    ingress:
      enabled: true
      certManager: false
      annotations: {}
      hosts:
        - name: thanos.prometheus.local
          path: /
	  tls: 
	  - hosts:
	  	  - thanos.prometheus.local
	  	secret_name: [SECRET_TLS]
	  [...]
```

La troisième section concerne l'alertmanager et propose de gérer toutes les alertes que remonteront les exporters connectés à Prometheus, les sections suivantes concernent les exporters.

### Déploiement
Vous avez ajouté le dépot helm et édité le chart afin qu'il réponde le mieux à vos attente ?  
Vous pouvez maintenant le déployer à l'aide de la commande suivante : `helm install [RELEASE_NAME] bitnami/kube-prometheus -f ./values.yml`  

Bien que je recommande la première, les options `prometheus.serviceMonitorSelector` et `prometheus.podMonitorSelector` sont à ajouter soit dans le fichier values.yaml soit lors du déploiement, et permettront de configurer automatiquement la cible du serviceMonitor de Prometheus sans avoir à modifier manuellement sa configuration.

## La collecte des métriques externe
Configurer et déployer Prometheus et son sidecar Thanos c'est très bien...mais en l'état, si les métriques des pods applicatifs ne remontent pas vers le premier : cela ne sert pas à grand chose. C'est pour cela que chaque fois que vous aurez à déployer un chart, je vous encourage à chercher la section (ou sous-section) metrics. C'est celle-ci qui fera tout le travail à la place de Prometheus.  
Voici un aperçu de Metrics:  
```yaml
metrics:
  enabled: true
  image:
    registry: docker.io
    repository: bitnami/apache-exporter
    tag: 0.7.0-debian-9-r50
    pullPolicy: IfNotPresent
  serviceMonitor:
    namespace: "[POD_NAMESPACE]"
    additionalLabels:
      prometheus.io/monitor: "true"
```
Ceci n'est qu'un exemple, il s'agit d'un apache-exporter. Il crée un objet de type ServiceMonitor dans le namespace dans lequel sera déployé le pod applicatif.

## Conclusion
Cela faisait relativement longtemps que je n'avais pas proposé d'article sur Kubernetes, de plus je n'avais jamais abordé le Framework Operator et comment en déployer un à l'aide de Helm. 
Avec autant d'informations sur chacun des concepts, vous ne devriez plus avoir de difficultés à installer et configurer un Prometheus Operator accompagné de son Sidecar Thanos.
