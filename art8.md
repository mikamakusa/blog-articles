+++
date = "2019-08-24T15:29:49+02:00"
draft = "false"
title = "Comparatif de solutions de monitoring"

+++

Les solutions de monitoring les plus souvent citées sont très souvent Opensource et gratuites. Nous avons à faire à **Prometheus/Grafana/node_exporter/AlertManager** d'un côté et à **Telegraf/InfluxDB/Chronograf/Kapacitor** qui fonctionnent de manière totalement différentes. Le premier fonctionne en **PULL** (les métriques sont tirées par **Prometheus** depuis les serveurs) et le second fonctionne en **PUSH** (les métriques sont envoyées par les serveurs monitorés).  
Dans les deux cas, les serveurs à monitorer devront être configurés pour qu'ils sachent exactement quels données envoyer...Et où les envoyer.  
Ceci dit, on oublie souvent un troisième acteur en pleine transformation : **ElasticStack** composé de :  
- Elasticsearch : Stockage des données  
- Kibana : Affichage des données  
- Metribeat : Récupération et envoi des données    
**Metricbeat** fonctionne de la même manière que **Telegraf**, c'est à dire qu'il envoie les données au middleware qui va s'assurer du stockage (en l'occurence : **ElasticSearch**).

### La collecte
Inconsciement j'ai déjà abordé la partie collecte des données dans l'introduction de cet article...Donc autant passer à la suite : La base de données.

### La base de données
**InfluxDB** est une vraie base de données (au sens strict du terme), tout comme **ElasticSearch**, toutes deux dispose d'une API consultable.  
**Prometheus** de son côté n'est pas vraiment une base de données même si il est possible de stocker les données et de les restituer par la suite...mais la rétention des données est limitée (beaucoup moins avec **Thanos**).  
**InfluxDB** supporte plusieurs types de data : int64, float64, bool, string data. Prometheus ne supporte que les float64 (tout comme **ElasticSearch**).

### Alerting
Selon l'outil utilisé, la gestion des alertes est plus ou moins simple.  
Les méthodes sont multiples et relativement simple à mettre en oeuvre :  
- Avec **AlertManager/Prometheus**, la configuration des alertes est relativement simple car basées sur **PromQL** (le langage de requête de **Prometheus**). Un simple fichier à remplir et à renseigner dans la configuration de **Prometheus**...et c'est parti.  
- Avec **Kapacitor**, deux méthodes s'offrent à vous...et il semblerait que la méthode graphique soit la plus simple des deux (via **Chronograf**),  
- Avec **ElasticStack**, tout se fait de manière graphique, via **Kibana**.

### Le rendu graphique
Après avoir joué pendant plusieurs jours avec chacune de solutions, **Kibana** et **Grafana** ont une longueur d'avance sur **Chronograf**...et ceci totalement au delà de l'aspect **Eye Candy** de chaque interface...Je dirais surtout qu'elles sont beaucoup plus **User Friendly**.  
**Grafana** est l'outil le plus simple à configurer :  
- datasource à configurer en 2/3 clics,  
- dashboard préconfigurés disponible sur le site officiel  
**Kibana** Propose des *snippets* préconfigurés uniquement si **metricbeat** est configuré pour activer l'option.  
Pendant ce temps : **Chronograf** est moins intuitif à configurer et les rendus sont moins clairs.  

### Conclusion
Chaque solution est comparable, tout dépend de votre choix pour votre structure.
Il faut savoir plusieurs choses :  
- InfluxDB, ElasticSearch et  Prometheus peuvent tourner parallèlement et peuvent aussi interagir l'une avec l'autre,  
- Il existe un exporter **ElasticSearch** et **InfluxDB** pour **Prometheus**, donc ce dernier peut afficher des métriques des services précités,  
- J'ai trouvé **Prometheus** et **ElasticStack** extraordinairement simple à configurer (mention spéciale pour les **beats** - metribeats, journalbeats, filebeats),  
- Les rendus graphiques avec **Grafana** sont meilleurs et les templates téléchargeables sont plus complets avec **Prometheus** qu'avec **InfluxDB**,  
- La configuration des alertes est très intuitive sur **Kibana** et extrement simple avec **AlertManager**.  

En gros, Je recommande **ElasticStack** et **Prometheus**...Sauf si vous préférez **Datadog** (dans ce cas, je m'incline).
