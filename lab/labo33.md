+++
date = "2019-01-15T15:29:49+02:00"
draft = "false"
title = "Le Labo #27 | Monitoring et Service Discovery"

+++

Il y a quelques années, quatre (pour être précis), j'avais travaillé sur un article sur le sujet **Service Discovery** sans pour autant aller vraiment très loin...En l'occurence, j'avais juste ajouté la fonctionnalité en question à un cluster **Serf**.  
Entre temps, **Serf** a été abandonné par **Hashicorp** au profit de **Nomad** (l'alternative plus ou moins crédible à **Kubernetes**, **Openshift** et **Kontena**, malgré sa relative jeunesse).  
Depuis, j'ai eu le temps de me pencher un peu plus sur le sujet.

Au sujet du **Monitoring**, je ne peux pas cacher ma préférence envers **ELK**, soit **ElasticSearch**/**Logstash**/**Kibana**...Mais en remplaçant **Logstash** par **Filebeat** et **Metricbeat**, on obtient un outil de monitoring vraiment complet permettant de gérer aussi bien les métriques que les logs sans que ce soit la galère à configurer.  
Toujours dans la même idée, nous avons **Greylog**, qui permet d'aller plus loin au niveau de la sécurité des utilisateurs et de l'alerting (là ou **ELK** propose un pack payant pour faire la même chose), mais l'inconvénient est que les dashboards de métriques sont très compliqués à créer.

Jusqu'à maintenant, je n'y avais pas pensé étant donné que je provisionnais mes serveurs (ainsi que le serveur de monitoring) à l'aide d'**Ansible**, mais je me suis dis qu'ajouter du **Service Discovery** pouvait simplifier encore plus le déploiement d'une serveur de ce type.

Cet article sera en deux parties, la première va aborder **Prometheus** et le service Discovery...en attendant de pouvoir vous présenter la même chose avec **ELK**.

## Le Service Discovery
Je ne vais pas vous faire un comparatif des différents outils de **Service Discovery** présent sur le marché, tels que :

- Etcd  
- Hazelcast/Zookeeper
- Consul
- Weave Net

D'autant plus que le dernier n'est pas qu'un simple outil de **Service Discovery**, mais un **Network Overlay**, et associé à **Weave Scope**, il offre des fonctionnalités relativement similaire à du monitoring (et c'est assez beau à regarder).

**Etcd**, je ne le connais que de nom, j'ai travaillé plusieurs mois avec le duo **Zookeeper/Hazelcast** pour ne pas avoir envie de m'aventurer sur ce terrain...et aussi parce que je n'ai pas envie de vous perdre.  
Il reste donc **Consul** qui est bien plus **User-Friendly** et dont l'installation/démarrage vous prendra moins de 10 minutes (montre en main).  
Pour le télécharger, c'est par [ici]()  
Une fois téléchargé et décompacté, on exécute la commande suivante (sans oublier de créer un dossier *data* pour **Consul**) : 

- sur le serveur :  
```
consul agent -bootstrap -advertise=<ip_server> -bind=<ip_server> -client=<ip_server> -data-dir=/opt/consul/data -server
```

- Sur les *workers* : 
```
consul agent -advertise=<ip_worker> -join=<ip_server> -config-dir=/opt/consul/conf/ -data-dir=/opt/consul/data
```

Si vous connaissez bien **Prometheus**, vous savez que cet outil de monitoring ne fonctionne pas de la même manière que **Centreon/Nagios/Check_MK** et autres. Prometheus *vient chercher* les métriques que les *exporters* mettent à disposition via un *serveur web*.  
Maintenant que vous savez comment fonctionne **Prometheus**, on va pouvoir télécharger les différents exporters...allez donc faire un tour [ici](https://prometheus.io/docs/instrumenting/exporters/).  
Une fois les *exporters* necessaires téléchargés et avant le démarrage de l'agent **Consul** sur les *workers*, il faut créer la configuration qui permettra au Service Discovery de faire remonter les informations relatives aux métriques de chaque serveur vers **Consul** (dans un premier temps) puis vers **Prometheus**.  
Parmi les *exporters*, nous avons **node_exporter** qui est vraiment la base et qui gère les métriques systèmes de base, la configuration **Consul** est très simple :  

```json
{
 "service": {
   "name": "node_exporter",
   "tags": ["monitor"],
   "port": 9100
 }
}
```

Par contre, si vous utilisez aussi d'autres exporters, tels que **mongo_exporter**, **elasticsearch_exporter**, ou celui pour **nginx** (faites attention, il y en a deux), il faudra modifier les champs *name* et *port* afin qu'il correspondent au nom de l'exporter ainsi qu'au port exposé par celui-ci (9001 pour *mongo* et 9108^pour *elasticsearch*).  

Il est temps de démarrer **Consul** sur chaque serveurs.  

## Le monitoring
Passons à **Prometheus**. Téléchargeable directement depuis le site officiel, la configuration est extremement simple et démarrer le serveur avec la bonne configuration est elle aussi très rapide.  
Techniquement démarrer le deux services (**Consul** et **Prometheus**) devrait prendre à peine 10 minutes, mais pour faire en sorte que tout fonctionne correctement, vous aurez besoin d'ouvrir le fichier de configuration de **Prometheus** et remplacer les lignes à partir de *scrape_configs* par ce qui suit (en sachant que le service **Consul** peut aussi être installé sur le même serveur que **Prometheus**:  

```
scrape_configs:
 - job_name: consul
   consul_sd_configs:
     - server: '<ip_consul>:8500'
   relabel_configs:
     - source_labels: [__meta_consul_tags]
       regex: .*,monitor,.*
       action: keep
     - source_labels: [__meta_consul_service]
       target_label: service
```

Une petite explication s'impose :  
- Nous avons créé un job de *scrapping* de métriques que l'on nomme *consul*  
- Celui-ci va intéroger le *service discovery*  
- Un premier filtre est défini : *monitor*, tout ce qui contient le tag en question dans **consul** sera affiché dans **Prometheus**  

Maintenant vous pouvez démarrer **Prometheus** avec la commande suivante : `/opt/prometheus/prometheus --config.file=prometheus.yml` puis rendez-vous sur l'**UI/targets** et ...tadaaaaa !!! Vous pouvez continuer en ajoutant un autre serveur et reproduisant le même processus :  

- Installation  et démarrage du *node_exporter*  
- Installation, configuration et démarrage de **Consul**  

Et vous verrez un nouveau serveur apparaître dans les **targets** de l'UI de **Prometheus**.  

Certes, l'interface de Prometheus n'est pas très *eye-candy*, mais vous pouvez utiliser un outils de graphing tel que **Grafana**.  

## Conclusion
Prometheus n'est pas fait que pour les plateformes de conteneurs, c'est un fait.  
Pour les plateformes de conteneurs, hors **Kubernetes**, nous avons **cadvisor** (qui fera peut-être l'objet d'un prochain article).  
Mais pour les serveurs classiques, il y a toujours des solutions dont celle que je viens de présenter...et il y en a forcément d'autres : à creuser.  

N'hésitez surtout à faire un tour du côté des exporters pour **Prometheus**...et pas seulement sur le site officiel (Celui pour Keycloak n'est pas référencé, par exemple).