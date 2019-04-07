+++
date = "2019-04-07T15:29:49+02:00"
draft = "false"
title = "Le Labo #28 | Smart Monitoring"

+++

L'idée de départ était de monter une infrastructure disposant de fonctions de **self-healing**, en gros à chaque incident recensé devait correspondre une notification ainsi qu'une réaction automatique d'un outil de CI/CD.  
A l'aide d'outils tels que **Prometheus**, **AlertManager**, **Slack** et **Jenkins**, j'ai pu travailler sur ce type d'infrastructure...Par contre, je ne pense pas que ce soit limité à **Slack** (notifications) et **Jenkins** (Job CI/CD), je vous laisse tester avec d'autres outils.

## Outils nécessaires
Les outils nécessaires de ce lab sont ceux avec lesquels j'ai pu présenter par le passé, ceux que je n'avais pas encore abordé auront leur partie plus développées :  
- Prometheus  
- Consul  
- NodeExporter  
- AlertManager  
- BlackboxExporter  
- Slack  
- Jenkins

Dans mon [article précédent](http://ageekslab.com/lab/labo33/), j'avais déjà abordé le sujet de la configuration du Service Discovery Consul pour Prometheus.  
Néanmoins d'autres options de Service Discovery sont possible, telles que **Azure**, **AWS** ou même **GCP** en renseignant plusieurs paramètres, tels que :  

Pour Azure :  
- tenant_id  
- subscription_id  
- client_id  
- client_secret

Pour AWS:  
- access_key  
- secret_key

Pour GCP:  
- project

Ce type de configuration interviendrait dans le cas ou il serait impossible techniquement d'utiliser **Consul** (car la communication entre les **subnet** n'est possible que dans un sens...ou absolument impossible).

## Blackbox Exporter
Le **Blackbox Exporter** permet de superviser et de générer des métriques de services externes sur les requêtes http, icmp, tcp, dns, etc...  
On peut laisser la configuration de l'outil inchangée avant de le démarrer puisque c'est dans la configuration de **Prometheus** que l'on va définir les services.  

## AlertManager  
Le fonctionnement de l'**AlertManager** est simple et ne fait que remplir sa fonction principale, à savoir envoyer des alertes sur divers outils (mail, chat) ou bien des réactions aux alertes...ce qui est l'objet de cet article.  
La configuration de l'**AlertManager** est en deux parties :  
- les alertes, un fichier au format *yml* dans lequel les alertes seront définies.  
- la configuration des hooks (mail, chat, etc...) dans laquelle les réactions face aux alertes seront définies.  

### Les alertes
Le principe de la définition des alertes est extrêmement simple : 3 keywords (**alert**, **expr**, **for**) mandatory...et des **labels** a placer pour catégoriser plus en profondeur les alertes.  
- **alert** : nom de l'alerte  
- **expr** : requête PromQL que vous pouvez définir et afficher des résultats de requêtes.  
- **for** : l'intervale de temps entre la détection de l'événement et le déclenchement de l'alerte.

Comme évoqué plus haut, on peut ajouter des **labels**, comme **severity** (pour définir le degré de criticité de l'alerte).  

Par exemple :  
```yaml
groups:
- name: Instances
  rules:
  - alert: service_down
    expr: up{job="job1"} == 0
    for: 5s
    labels:
      severity: "critical"
  - alert: CpuProbe
    expr: container_cpu_usage_seconds_total{cpu=~"cpu0*"} > 60000
    for: 30s
    labels:
      severity: "warning"
  - alert: TomcatProbeConnectionsActive
    expr: tomcat_connections_active_total{job="job1"} == 0
    for: 10m
    labels:
      severity: "critical"
  - alert: TomcatContextState
    expr: tomcat_context_state_started{job="job1",context="/job1"} == 0
    for: 10s
    labels:
      severity: critical
  - alert: EndpointAlert
    expr: probe_success{job="blackbox"} == 0
    for: 10s
    labels:
      severity: critical
```
Dans le cadre de cet article, on peut aussi ajouter *jenkins_job* qui link un job jenkins qui est démarré une fois une anomalie détectée.  
Par exemple :  
```yaml
groups:
- name: Instances
  rules:
  - alert: service_down
    expr: up == 0
    for: 5s
    labels:
      severity: "critical"
      service: database
      self_healing: ##true/false
      jenkins_job: {{ job_url }}/build
```

### Les hooks
La configuration des hooks nécessite un peu plus de recherche car elle se décompose en quatre parties : *global*, *route*, *inhibit* et *receivers*:  
- *global* : regroupe la configuration de base de hooks, telle que l'adresse mail de contact, les webhooks de base, etc...  
- *route* : regroupe les routes et les *receiver* qui seront utilisés dans la section éponyme.  
- *inhibit*  
- *receivers* : regroupe les webhooks.  

Par exemple :  
```yaml
global:
  slack_api_url: '{{ slack_webhook }}'
route:
  group_by: [alertname, datacenter, app]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: team-1
  routes:
  - receiver: 'database'
    group_wait: 10s
    match:
      service: database
      self_healing: ##true/false
      jenkins_job: {{ job_url }}/build
receivers:
  - name: 'team-1'
    slack_configs:
      - channel: c4d-test
        text: |-
          {{ range .Alerts }}
             *Alert:* {{ .Annotations.summary }} - `{{ .Labels.severity }}`
            *Description:* {{ .Annotations.description }}
            *Details:*
            {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
            {{ end }}
          {{ end }}
        fallback: '{{ template "slack.default.fallback" . }}'
        icon_emoji: '{{ template "slack.default.iconemoji" . }}'
        icon_url: '{{ template "slack.default.iconurl" . }}'
  - name: 'database'
    webhook_configs:
    - url: "{{ job_url }}/build"
```

## Intégration dans Prometheus
Maintenant que **BlackboxExporter** et **AlertManager** sont configurés, nous allons pouvoir passer à leur intégration dans **Prometheus**. Pour l'**AlertManager**, c'est extrêmement simple...en ce qui concerne **BlackboxExporter** : beaucoup moins, donc je vais vous déblayer le terrain pour vous aider le plus possible.  

### AlertManager
A ce niveau, il n'y a pas grand chose à faire à part ajouter les lignes suivantes :   
```yaml
rule_files:
 - "alert.rule"

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']
```

### BlackboxExporter
Dans le cas du BlackboxExporter, il faut préciser plusieurs choses dont :  
- le chemin de metriques (en général, c'est `/probe`),  
- le module (parmis ceux défini dans la configuration de l'outil),  
- les targets (les url) qui peuvent être hardcodées ou regroupées dans un fichier externe (yaml ou autre),
- les labels (ou `relabels`).

Par exemple :  
```yaml
- job_name: blackbox
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
      - http://{{ip_address}}:{{ port }}/{{context | if needed }}
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: {{ ip_address }}:9115
```

## Conclusion
Le **Smart Monitoring** est plus ou moins l'avenir du monitoring, que ce soit *on premise*, en environnement *cloud* (**kubernetes** ou non), mais j'ai toujours considéré que le monitoring sans les logs ne servait absolument à rien.  
Malgré la grande utilité/puissance de Prometheus/Grafana, l'outil est relativement incomplet car il manque justement les logs pour compléter les métriques...un des manque que peut combler **Graylog** ou la stack **ELK** (qui ne propose pas d'alerting - pour ELK - et qui est relativement difficile d'accès - pour Graylog).
Cependant, **Loki** propose d'ajouter la gestion des logs à Grafana...Ce qui ajouterai une seconde **Disney Princess** à la communauté (oui, je sais, c'est encore du troll gratuit.)
