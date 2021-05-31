+++
date = "2021-05-27T15:29:49+02:00"  
draft = "false"
title = "Le Labo #35 | Une approche FinOps sur Google Cloud avec les Cloud Functions"
+++

Cela faisait longtemps que je n'avais pas publié d'article et cette fois-ci, j'ai décidé de m'atteler à un sujet que je n'avais encore jamais abordé : Le FinOps.  
J'imagine que la gestion des coûts est un sujet que nombreux d'entre vous ont approché de près ou de loin que ce soit personellement ou lors des différents projets sur lesquels vous avez pu évoluer au sein de vos clients.  
De mon côté, à part pour mon blog...je n'ai pas vraiment eu à m'en soucier, même si j'ai toujours eu à coeur de gérer des workloads comme si je devais les payer moi-même (donc en reduisant au maximum l'impact financier).  
Il s'avère que Google Cloud propose un outil appelé Cost Management affichant le suivi des dépends, la possibilité des créer des tableau de bord personnalisés, une gestion des ressources par projets, un pilotage des habilitations financières (à l'aide de BigQuery et de DataStudio). Cependant, ce ne sera pas le sujet de cet article.  
En fait, je vais plutôt aborder l'interaction entre quatre services différents :  
- PubSub,  
- Cloud Functions,  
- Cloud Scheduler,  
- Compute

Je ne m'attarderais pas énormément sur **Compute** à part pour décrire ce que j'ai déployé...j'ai prévu de développer la partie **Cloud Functions** pour deux raisons :  
- Ce sera la première fois que je vais m'aventurer sur du Nodejs (en version 10),
- l'autoscaler des **ManagedInstancesGroup** ne fonctionne pas comme je le voudrais, *scaling_schedule* ne doit pas être la seule politique de scaling (merci **Google Cloud**).  

## Compute Engine
Comme évoqué plus haut, seul les **ManagedInstanceGroup** seront utilisés ici afin ge gérer plus facilement un *bloc* de machines virtuelles...tout simplement.  
![img1](/images/igm.png)  
Pas de spécifications particulières pour le *template*, un OS Linux avec un type de machine très basique, à savoir *e2-small*.

## Cloud Scheduler
Pour faire extremement simple, il s'agit d'un cron amélioré avec des *cibles* sur lesquelles envoyer des signaux parmi **HTTP**, **PubSub** et **AppEngine**.
Chaque type de cible est a configurer de la manière la plus fine possible.
Par exemple, dans le cas de **PubSub**, le **Message Body** sera inclus dans le message stocké dans le service ciblé...et utilisé en bout de chaine (par la **Cloud Function**, par exemple).
Dans ce cas précis, le Cloud Scheduler sera programmé pour déclencher les Cloud Function uniquement lors des jours de la semaine, soit l'instruction suivante :  
- **0 7 * * 1-5** : La Cloud Function se déclenche à 07h00 du Lundi au Vendredi (Pour augmenter le nombre d'instance),  
- **0 21 * * 1-5** : La Cloud Function se déclenche à 21h du Lundi au Vendredi (Pour diminuer le nombre d'instance).  
Par conséquent, le nombre d'instance actif durant le week end est identique a celui programmé entre 21h00 et 07h00.


## Pubsub
Le fameux service de publication de messages de **Google Cloud**.  
Il s'avère qu'il n'est pas possible de lier directement **Cloud Functions** et **Cloud Scheduler**, ce dernier se connecte à un *Topic* **PubSub** qui est consommé par La **Cloud Function**.


## Cloud Function
La partie la plus fun de l'article et celle qui fut le plus chronophage.  
Bien que jai déjà travaillé sur une **Cloud Function** pour un [article précédent](http://www.ageekslab.com/lab/labo37/), celle-ci est différente car en **Nodejs** (oui, j'aime me faire du mal parfois).  
J'ai commencé par chercher des exemples ou des bouts de codes qui pouvaient me mettre sur la voie, mais je me trouvais face à des blogs évoquant deux fonctions différentes et principalement orientés *Instances* et non *InstanceGroup*, voir ci-dessous : 
```javascript
var http = require('http');
var Compute = require('@google-cloud/compute');
var compute = Compute();
exports.startInstance = function startInstance(req, res) {
    var zone = compute.zone('instance zone here');
    var vm = zone.vm('instance name here');
    vm.start(function(err, operation, apiResponse) {
        console.log('instance start successfully');
    });
    res.status(200).send('Success start instance');
};
```

J'ai donc continué à cherché et, en désespoir de cause, je suis allé voir l'API Compute Engine sur le [site officiel](https://cloud.google.com/compute/docs/apis) à la rubrique **ComputeEngine/v1/regionInstanceGroupManagers**.  
Et voici le resultat (à l'aide du *framework* **Cloud Function**, de la référence de code de l'API et de nombreux tests) :  
```javascript
const {google} = require('googleapis');
const {Buffer} = require('safe-buffer');
const compute = google.compute('v1');

exports.resizeRegionInstanceGroup = (event, context, callback) => {
    authorize(function(authClient) {
        // Conversion du message provenant de PubSub en base64 exploitable par Cloud Functions
        try {
            const payload = _validatePayload(
                JSON.parse(Buffer.from(event.data, 'base64').toString())
            );
        // Création de la requête
            var request = {
                region: payload.region,
                instanceGroupManager: payload.name,
                size: payload.size,
                project: payload.project,
                auth: authClient,
            }
        // Exécution de la requête *regionInstanceGroupManagers.resize* avec les données précédentes
            compute.regionInstanceGroupManagers.resize(request).then(data => {
                const operation = data[0];
                const apiResponse = data[1];
                console.log(apiResponse);
                return operation.promise();
            }).then(() => {
                const message = 'Successfully resize to ' + payload.size + ' the instance group named ' + payload.name;
                console.log(message);
                callback(null, message);
            }).catch(err => {
                console.log(err);
                callback(err);
            });
        } catch (err) {
            console.log(err);
            callback(err);
        }
    })
};

function authorize(callback) {
    google.auth.getClient({
        scopes: ['https://www.googleapis.com/auth/cloud-platform']
    }).then(client => {
        callback(client);
    }).catch(err => {
        console.error('authentication failed: ', err);
    });
}

function _validatePayload(payload) {
    if (!payload.region) {
        throw new Error(`Attribute 'region' missing from payload`);
    } else if (!payload.name) {
        throw new Error(`Attribute 'name' missing from payload`);
    } else if (!payload.size) {
        throw new Error(`Attribute 'size' missing from payload`);
    }
    return payload;
}
```

N'étant pas du tout développeur **NodeJs**, il s'avère que le code ci-dessus n'est pas parfait...Je m'en suis moi même rendu compte (d'autant plus qu'il génère des erreurs bien que, au final, la Cloud Function fait ce qu'on lui demande).

Partant de ce constant, j'ai pris la liberté de convertir ce script **NodeJs** en **Python** (que je maîtrise beaucoup plus) pour obtenir certes le même résultat...mais exempt d'erreurs (et qui me permettrait, à l'avenir, de faire évoluer la **Cloud Function** plus facilement).  
Voici le script final ci-dessous :  
```python
from googleapiclient import discovery
from oauth2client.client import GoogleCredentials
import datetime, logging, base64, json
from string import Template


def python_resize_region_instance_group(event, context):
    # Répération du Payload de Pubsub
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    # Conversion en objet JSON pour exploitation
    message = json.loads(pubsub_message)
    # Connexion au service Compute Engine
    credentials = GoogleCredentials.get_application_default()
    service = discovery.build('compute', 'v1', credentials=credentials)
    try:
        current_time = datetime.datetime.utcnow()
        log_message = Template('Cloud Function was triggered on $time')
        logging.info(log_message.safe_substitute(time=current_time))
        # Démarrage de l'action "resize" du Group d'Instance Régional
        resize_action = service.regionInstanceGroupManagers().resize(project=message['project'],
                                                     region=message['region'],
                                                     instanceGroupManager=message['name'],
                                                     size=message['size'])
        response = resize_action.execute
        return response
    except Exception as error:
        log_message = Template('$error').substitute(error=error)
        logging.error(log_message)

```

## Finalement
L'idée de départ était de pouvoir organiser la scalabilité des machines à l'intérieur d'un groupe d'instance en fonction de l'heure de la journée.  
Par exemple :  
- à 21h, réduire le nombre de machines à 1,  
- à 7h du matin, augmenter le nombre de machines à 3 ou plus.  

Le tout dans l'optique de réduire l'impact du workload sur la facturation.  
![img2](/images/full.png)