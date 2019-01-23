+++

date = "2019-01-18T10:05:00+02:00"
draft = false
title = "Thanos...sans l'Infinity Glove"

+++

Désolé pour la petite référence à *Marvel* mais c'est assez rare de croiser des outils de monitoring ayant le nom d'un Bad Guy de la maison d'édition en question (surtout depuis que c'est devenu une Disney Princess - C'est pour le Troll, je sais, c'était gratuit).
Pour en revenir au sujet, **Thanos** est sorti il y a quelques mois (en Septembre, pour être exact)...l'outil est très jeune mais déjà très prometteur.

## Le but de Thanos
Non, son but n'est pas d'éliminer la moitié de l'univers d'un simple claquement de doigts...Je m'égare.  
En fait, le principe de **Thanos** est de fédérer des serveurs **Prometheus**, que ce soit en environnement **Kubernetes** ou **Legacy** (c'est à dire sur des VMs classiques) via plusieurs composants dont un connecteur **Prometheus**, tout en fournissant des features supplémentaire telles que du stockage S3 ou block Storage (Swift, Azure Blob Storage, etc...), une rétention plus élevée proposant le même mécanisme de compactage des données que **Prometheus**, quelque soit le type de stockage utilisé.  

## Les commandes de Thanos
**Thanos** propose plusieurs commandes que je vais développer plus tard : 

- **sidecar** : En fait, c'est le connecteur **Prometheus**,  
- **store** : Il s'agit du **Store Gateway** et qui est un moyen de faire comprendre à **Prometheus** de stocker moins localement et de diminuer sa retention...puisque c'est **Thanos** qui va gérer cet aspect,  
- **query** : Il s'agit d'une surcouche de requêtage propre à **Thanos** pour évaluer les requêtes PromQL contre l'ensemble des instances de **Prometheus** qui seront connectées à **Thanos**,  
- **rule** : Grâce à cette commande, **Thanos** va gérer l'évaluation des **rules** et **alerts** provenant de chaque requêteur (**query**),  
- **compact** : Cette commande effectue le même travail que **Prometheus** avec les données les plus anciennes dans le but de gagner en efficacité au niveau du requêtage,  

Les commandes ci-dessus sont les 5 principales de **Thanos** et celles sur lesquelles je vais développer dans la suite de l'article. Il en reste trois qui ne sont pas fondamentales dans le fonctionnement de **Thanos** et qui ne concernent que les *buckets* de stockage de données.

## Sidecar
En présentant le **sidecar** comme un simple connecteur **Prometheus**, j'ai certainement été en dessous de sa vraie fonction.  
En effet, certaines de ses options permettent de créer un cluster compatible avec le protocole **Gossip** (utilisé entre autre par **Consul**), exposer une API de stockage et aussi de se connecter à une instance **Prometheus**.

### Comment démarrer un sidecar Thanos ?
Sur chaque instance **Prometheus**, exécutez la commande suivante :  

Avec Backups :  
```
thanos sidecar \
--tsdb.path="/var/prometheus" \
--prometheus.url=http://localhost:9090 \
--objstore.config.file=config.yml
```

Avec StoreAPI :  
```
thanos sidecar \
--tsdb.path="/var/prometheus" \
--prometheus.url=http://localhost:9090 \
--objstore.config-file=config.yml \
--http-address="0.0.0.0:10999" \
--grpc-address="0.0.0.0:10998" 
```

Le contenu du fichier de configuration du sidecar dépendra évidemment du fournisseur de stockage cloud que l'on souhaite utiliser...Dans mon cas, ce sera **Azure** (Pourquoi ? Parce que...) et mon *config.yml* contiendra les informations suivantes :  
```yaml
type: AZURE
config:
  storage_account: ""
  storage_account_key: ""
  container: ""
```

## Query
Comme expliqué précédemment, **Thanos** propose une surcouche au système de requêtage **PromQL** de **Prometheus** permettant de requêter simultanément toutes les instances de **Prometheus** connectées via le **sidecar**.  
Mais le **Query Layer** ne se limite pas à ça, il est stateless, horizontalement scalable, peut être déployé à l'infini (ou presque) et propose également une interface utilisateur.  
Une fois connecté aux **sidecars** (ou au cluster de sidecar), il detecte automatiquement quelle instance **Prometheus** doit être contactée pour telle ou telle requête **PromQL**.  

### Comment démarrer le Query Layer ?
Tout dépend comment a été démarré le **sidecar**, si vous avez démarré un cluster, ce sera certainement plus simple car vous n'aurez qu'un seul *store* à définir lors du démarrage :  
```
thanos query \
--grpc-address="0.0.0.0:10997" \
--http-address="0.0.0.0:19990" \
--store="x.x.x.x:10998"
```

## Store
J'avoue ne pas vraiment avoir besoin de réexpliquer à quoi sert le **Store Gateway**.  
Plus haut dans cet article, j'avais expliqué que le **Store Gateway** ou **store** est un moyen de faire comprendre à **Prometheus** de diminuer sa rétention en local puisque **Thanos** était censé gérer toute la rétention et le stockage des données sur des buckets S3 ou sur du block Storage...En fait, il effectue ce type d'opération grâce à l'implémentation de l'API de stockage utilisée par le **sidecar**.

### Comment démarrer le Store Gateway ?
En exécutant la commande suivante :  
```
thanos store \
--data-dir="./data" \
--objstore.config-file="config.yml"
```  

Je recommande d'utiliser le même fichier de configuration que celui utilisé pour le démarrage des **sidecar**.

## Compact
Comme expliqué plus haut dans cet article, le **compactor** effectue le même travail que **Prometheus** avec les données les plus anciennes dans le but de gagner en efficacité au niveau du requêtage, il scanne le contenu de l'espace de stockage et effectue une opération de compactage lorsque c'est nécessaire.

### Comment démarrer le Compactor ?
En exécutant la commande suivante :  
```
thanos compact \
--data-dir="/var/thanos/compact" \
--objstore.config-file="config.yaml" \
--http-address="0.0.0.0:19191"
```

# Rule
Le **Ruler** évalue les règles d'enregistrement et d'alerte de Prometheus par rapport à des nœuds de requête aléatoires de son cluster. Les résultats de la règle sont réécrits sur le disque au format de stockage Prometheus 2.0. Les nœuds de règle participent simultanément au cluster eux-mêmes en tant que nœuds de magasin source et téléchargent leurs blocs TSDB générés dans un magasin d'objets.

Les données de chaque nœud de règle peuvent être étiquetées pour satisfaire le schéma d'étiquetage des grappes. Les paires à haute disponibilité peuvent être exécutées en parallèle et doivent être distinguées par l'étiquette de réplica désignée, tout comme les serveurs Prometheus habituels.

### Comment démarrer le Ruler ?
En exécutant la commande suivante : 
```
thanos rule \
--data-dir="/path/to/data" \
--eval-interval="30s" \
--rule-file=/path/to/rules/*.rules.yaml" \
--alert.query-url="http://0.0.0.0:9090" \
--alertmanagers.url="alert.thanos.io" \
--cluster.peers="thanos-cluster.example.org" \
--objstore.config-file="config.yml"
```

## En bref
**Thanos** bien que très jeune est également très prometteur, l'idée d'en faire un **Prometheus** bénéficiant de fonctionnalités de stockage Cloud intégrant également du compactage et de fédération d'instances **Prometheus** est séduisante...D'autant plus que le système de requêtage ne change pas...ce qui signifie que l'on peut aussi s'en servir comme datasource dans **Grafana**.  

Prochaine *Disney Princess* à intégrer les outils DevOps ? Hulk, peut-être ? 