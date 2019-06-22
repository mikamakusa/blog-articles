+++
date = "2019-06-22T15:29:49+02:00"
draft = "false"
title = "Le Labo #29 | Thanos et Prometheus / Google Cloud Store"

+++

Il y a quelques mois, j'avais publié un article de présentation sur **Thanos**. Après autant de temps, j'ai décidé d'explorer un peu plus la technologie en l'associant avec un storage de type S3 sur Google Cloud.  
Et le fait qu'en début de semaine c'était le Google Cloud Summit n'a rien à voir là-dedans.

## Pre-requis
Obviously...parmi les prérequis, il y a :  
- un compte de service + un projet sur google cloud  
- Prometheus  
- Thanos  
- 3 VMs  

Sur deux d'entre-elles seront installé **Promtheus** et **Thanos**, ce dernier ne sera démarré qu'en mode **sidecar**. Sur la dernière, nous aurons **Thanos** en mode **store** et **query**.  
Je rappelle juste plusieurs points relatifs à **thanos**, tels que :  
- le **sidecar** se connecte à **prometheus** et expose un point de connexion pour **thanos** query,  
- le **store** gère tout l'aspect retention des métriques de prometheus et va les stocker soit localement (pas vraiment recommendé) ou sur un stockage de type S3,  
- le **query** expose une interface (quasiment identique à **prometheus**) et se connecte à un ou plusieurs sidecar (via le point de connexion exposé par ce dernier).  

Maintenant que tous ces point ont été éclaircis, je vais pouvoir continuer sur la configuration.

## Configuration
### Prometheus
La configuration de **prometheus** ne diffère pas de ce que vous avez...sauf un point qui n'est documenté nulle-part (à part [ici]('https://github.com/improbable-eng/thanos/blob/master/docs/getting-started.md')). Il y a trois options de configuration à ajouter :  
- dans le fichier de configuration, ajouter *external_labels* dans la section *globals* (voir ci-dessous):  
```yaml
global:
   scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
   evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
   # scrape_timeout is set to the global default (10s).
   external_labels:
     label: <label>
```
Peut importe le nombre de labels que vous allez ajouter, il faut qu'il y en ait au moins un pour que le *sidecar* ne remonte pas d'erreur au démarrage...mais ce n'est pas tout, il y a deux flags à ajouter au démarrage de **prometheus**.  
Et ils ne sont documentés nulle-part, ni dans la [documentation officielle]('https://prometheus.io/docs/introduction/overview/') ni dans l'aide (via `./prometheus --help`). Les deux flags à ajouter sont les suivants :  
- *--storage.tsdb.min-block-duration*  
- *--storage.tsdb.max-block-duration*  

Les deux sont à définir sur *2h* lors du démarrage : `./prometheus --config.file=prometheus.yml --storage.tsdb.min-block-duration=2h --storage.tsdb.max-block-duration=2h`

### sidecar
le *sidecar* a besoin des flags de base suivants pour démarrage sans remonter d'erreurs fatales :  
- prometheus.url  
- tsdb.path - même si vous ne précisez pas de chemin *tsdb* lors du démarrage de **prometheus**, ce dernier créera lui même un dossier tsdb...et c'est vers celui-ci qu'il faudra faire pointer ce flag,  
- objstore.config-file - doit pointer vers un fichier contenant les informations relative au conteneur S3 et à l'authentification,  
- http-address et grpc-address ne sont pas des flags réellement obligatoires, mais seront certainement utile lorsque les *sidecars* **thanos** sont sur des VMs différentes que le **thanos** *query**.  

Revenons sur le flag **objstore.config-file**, plusieurs provider sont gérés de manière stable...et forcément ce sont les 3 Cloud Provider *mainstream* : **Azure**, **Google** et **AWS**.  
Deux autres sont en cours de développement : **Openstack Swift** et **Tencent COS**.  
J'ai choisi Google en tant que stockage cloud, la configuration de celui est décrite ci-dessous :  
```yaml
type: GCS
config:
  bucket: <thanos>
  service_account: |-
    {
      "type": "service_account",
      "project_id": "project",
      "private_key_id": "abcdefghijklmnopqrstuvwxyz12345678906666",
      "private_key": "-----BEGIN PRIVATE KEY-----\...\n-----END PRIVATE KEY-----\n",
      "client_email": "project@thanos.iam.gserviceaccount.com",
      "client_id": "123456789012345678901",
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/thanos%40gitpods.iam.gserviceaccount.com"
    }
```
Dans *service_account*, il suffit juste de copier le contenu du fichier *credentials.json* récupéré depuis la console **Google Cloud** (IAM et Administration/Comptes de Service et **créer une clé**).  

Pour démarrer le **sidecar**, exécutons la commande suivante :  
`./thanos sidecar --prometheus.url="http://<prometheus_address>:9090" --tsdb.path="<path_to_tsdb_directory>" --objstore.config-file=<path_to_objectstore_config_file> --http-address="0.0.0.0:10903" --grpc-address="0.0.0.0:10904"`

### Store
Nous avons déjà fait le plus dur, **prometheus** est configuré et démarré avec les bonnes options, le **sidecar** lui aussi démarré et n'a remonté aucune erreur...On peut même réutiliser le fichier créé pour le flag *objstore.config* du *sidecar*.  
Et enfin démarrer **thanos** *store* à l'aide de la commande suivante :  
`./thanos store --objstore.config-file=<path_to_objectstore_config_file>`

### Query
Comme évoqué plus tôt dans cet article, **thanos** *query* va avoir besoin de se connecter à un ou plusieurs *sidecar*...deux dans notre cas à l'aide de la commande suivante :  
`./thanos query --http-address="0.0.0.0:10905" --grpc-address="0.0.0.0:10906" --store="10.10.0.3:10904" --store="10.10.0.50:10904"`  

Maintenant, à part vous connecter sur l'url de la VMs sur laquelle est installé **thanos** *query* et y découvrir l'interface (oui, je vous l'ai dis...similaire à **prometheus**), vous pouvez aussi utiliser **Grafana** pour afficher des graphes provenant des métriques remontées par les instances de **Prometheus** relayées par **Thanos**.
