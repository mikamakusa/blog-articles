+++
date = "2020-12-24T15:29:49+02:00"  
draft = "false"
title = "Le Labo #33 | Kubernetes invite Vault à danser"
+++

Je sais que le titre de cet article est un tantinet *putaclic* et j'assume complètement, néanmoins je continue à travailler sur deux technologies qui m'attirent énormément...il s'agit de **Kubernetes** et **Vault**.  
Dans mon article précédent, il s'agissait plus ou moins d'une découverte de **Vault** - bien qu'intégrer l'auto-unseal, l'observabilité et l'authentification LDAP à un cluster **Vault** ne soit pas, à proprement parler, un article de découverte...il permettait de poser quelques base sur le sujet (Notament autour de la création d'un cluster).  
Cette fois-ci, j'ai décidé d'intégrer **Vault** à **Kubernetes**...Mais attention, il ne s'agit pas là de le déployer dans l'orchestrateur de conteneur mais plutôt en tant que ressource externe, ce qui permettra d'explorer un peu plus certains objets relatifs à **Kubernetes**, tels que :  
- Les *ServiceAccounts*,  
- Les *endpoints*,
- Les *Roles* et *ClusterRoleBinding*,  
- Les *Annotations*,  
- Les *Init Containers* et les *Sidecar Containers*

A savoir que les *Annotations* pourront également servir pour du monitoring.  
Chose importante à savoir : **Rancher** propose un niveau d'abstraction au delà des *namespaces* via le principe des *projets*...Mais ce n'est pas l'objet de cet article.

A l'instar de **Kubernetes** est des points qui seront abordés ici, je vais également aborder d'autres points important du côté de **Vault**, à savoir :  
- Les *policies*,  
- La configuration de l'authentification du **Kubernetes**

## Le point de départ
Je vous invite à consulter [l'article précédent](http://www.ageekslab.com/lab/labo38/) autour de **Vault** et de la création d'un cluster.  
Pour ainsi dire, je n'ai presque rien modifié pour ce cluster là, si ce n'est l'absence de serveur **LDAP** pour l'authentification, le *master token* suffira amplement.  

### Consul - La configuration
Je préfère préciser qu'il y a trois serveurs **Consul** (même si un seul suffisait pour ce que j'en fait).

```json
{
  "datacenter": "dc_local",
  "data_dir": "consul_data",
  "log_file": "consul_log/consul.log",
  "log_level": "INFO",
  "acl": {
    "enabled": true,
    "default_policy": "deny",
    "enable_token_persistence": true
  }
}
```

J'ai effectivement ajouté les **ACL** afin de sécuriser un minimum mon cluster...j'ai également généré le token à l'aide de la commande : `consul acl bootstrap` (Le bon token, c'est *SecretID*).  

Les deux autres serveurs **Consul** disposent d'une configuration identique ainsi que le **MASTER_TOKEN** (généré précédemment) en tant que variable d'environnement.  

### Vault - La configuration
Tout comme pour le serveur **Consul**, il y en a également trois (un seul suffisait en mode dev, je suis tout à fait d'accord...mais je ne fais jamais rien comme les autres).  
```json
{
  "backend": {
    "consul": {
      "address": "<CONSUL_IP>:8500",
      "path": "vault_data/",
      "scheme": "http",
      "service": "vault",
      "token": "<MASTER_TOKEN>"
    }
  },
  "listener": {
    "tcp":{
      "address": "<VAULT_IP>:8200",
      "tls_disable": 1,
      "telemetry": {
	      "unauthenticated_metrics_access": true
      }
    }
  },
  "storage": {
	  "postgresql": {
		  "connection_url": "postgres://<user>:<password>@<POSTGRESQL_IP>:5432/<DATABASE>?sslmode=disable",
		  "ha_enabled": "true"
	  }
  },
  "telemetry": {
	  "prometheus_retention_time": "30s",
	  "disable_hostname": true
  },
  "seal": {
	  "gcpckms": {
		  "credentials": "<GCP_SERVICEACCOUNT>",
		  "project": "<PROJECT_ID>",
		  "region": "<REGION>",
		  "key_ring": "<KEYRING>",
		  "crypto_key": "<KEY>"
	  }
  },
  "ui": true,
  "api_addr": "http://<VAULT_IP>:8200",
  "cluster_addr": "http://<VAULT_IP>:8201"
}
```

### Prometheus et PostgreSQL
De ce côté là, rien de particulier ni d'exotique a révéler...tout est déjà expliqué sur la documentation officielle de **Vault**.  

## Kubernetes
J'aurais pu monter un lab de plusieurs serveurs pour **Kubernetes**...j'ai fini par opter pour **Minikube**.  
Mais l'objectif reste le même : pouvoir utiliser **ETCD** en tant que **Backend Storage** pour **Vault**.  
Toujours est-il que le résultat de cet article est un **Quick Win** (mais cela reste une victoire...loin du **Quick and Dirty**).  

Pour démarrer **Minikube** : `minikube start --driver=<DRIVER>`, comme *driver*, j'ai opté pour **virtualbox**.  
Le dashboard (pour une démo prochaine, par exemple) : `minikube dashboard` et `kubectl proxy --address='0.0.0.0' --disable-filter=true` (qui permet d'afficher le dashboard sur un autre ordinateur).  

### Le workflow

Le principe est relativement simple et schematisé ci-dessous :  
![img1](/images/kubevault.png){ width=50% }  

- 1 : Un utilisateur effectue une requête de déploiement,  
- 2 : L'API de **Kubernetes** détecte une création de pod dans le manifeste et le place dans la queue de déploiement,  
- 3 : Un webhook est détecté dans le manifeste, l'API de **Kubernetes** contacte le *webhook*,  
- 4 : Le webhook retourne les données à l'API de **Kubernetes** qui démarre un *init container*,  
- 5 : L'*init container* effectue une requête à **Vault** pour récupérer les secrets demandés,  
- 6 : Le *SideCar* **Vault** de la requête de deploiement s'authentifie sur **Vault** pour ensuite injecter les secrets dans l'*App Container*.  

Ca c'était le côté schematisé...cependant, le chemin pour arriver à ce résultat est relativement long.

### Intégration de Vault
Avant toutes choses, il faut connaître un peu **Helm**.  
**Helm** est un gestionnaire de *package* destiné à faciliter les déploiements sur **Kubernetes** via l'utilisation de manifestes prêts à l'emploi pour lesquels on peut également surcharger les variables.  
Dans ce cas, je n'ai besoin de **Helm** que pour déployer le **Vault Agent** dans **Kubernetes** à l'aide de la commande suivante : `helm install vault hashicorp/vault --set injector.externalVaultAddress=http://<HELM_ADDRESS>:8200`, ne surtout pas oublier de remplacer l'adresse de votre serveur **Vault** par celle de votre reverse proxy si vous en avez un.  
Il faut savoir que ce **Vault Agent** fonctionne à la création et à l'update des *pods* en vérifiant si l'*annotation* `vault.hashicorp.com/agent-inject: true` est présente...et va modifier les *pod* en fonction des autres *annotations* relative à **Vault**.  

Une fois l'installation du **Vault Agent** effectuée, il ne faut surtout pas oublier de configurer l'authentification via **Kubernetes** dans **Vault**, celle-ci se déroule en plusieurs étapes : 

- Activation dans **Vault** : `vault auth enable kubernetes`,  
- Création du *ServiceAccount* dans **Kubernetes** (vous pouvez même le nommer comme vous le souhaitez),  
- A l'aide de *Role* et de *ClusterRoleBinding*, définir les **Role Based Access Controls** pour le *ServiceAccount* qui vient juste d'être créé,  
- Configurer l'*endpoint* dans **Vault** avec le secret rattaché au *ServiceAccount*, ainsi que le *token* et le *certificat*...à l'aide de la commande suivante :  
```shell
vault write auth/kubernetes/config token_reviewer=<token> kubernetes_host=https://<KUBERNETES_IP>:443 kubernetes_ca_cert=<certificat>
```
- Déployer dans **Kubernetes** un *service* nommé *external-vault* ainsi que le *endpoint* correspondant (intégrant l'adresse du serveur **Vault**).

### Les policies
Comme évoqué au début de l'article, faisons un tour dans les policies de **Vault**, je vous rassure il n'y a rien de très compliqué ici.  
Une *policy* se compose de plusieurs élémements :  
- Le chemin, il s'agit simplement du chemin d'accès aux secrets ciblés,  
- Les actions possibles : **READ**, **WRITE**, **DELETE**, **UPDATE**, **LIST**  

Par conséquent, une *policy* est représentée ainsi :  
```json
path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/restricted" {
  capabilities = ["create"]
  allowed_parameters = {
    "foo" = []
    "bar" = ["zip", "zap"]
  }
}
```

Dans le cas qui nous intéresse ici, nous allons en une créer pour accéder à un secret préalablement créé à l'aide de la commande suivante : `vault kv put secrets/devapp/config dbuser='user1' dbpass='passw''`.  
Pour pouvoir y accéder dans un pod déployé sur **Kubernetes**, il faudra définir la *policy* suivante :  
```json
path "secrets/data/devapp/config" {
  capabilities = ["read"]
}
```
De plus, afin de garantir le lien entre le *ServiceAccount* de **Kubernetes** et la *policy* définie dans **Vault**, il faut également créer un *authentication role* dans **Vault** :  
```shell
vault write auth/kubernetes/role/<NAME> \
        bound_service_account_names=<SERVICEACCOUNT_NAME> \
        bound_service_account_namespaces=<NAMESPACE> \
        policies=<POLICY_NAME> \
        ttl=24h
```

### Les Annotations
A quoi servent les annotations ?  
Dans le cas de **Rancher** (par exemple), elles servent à définir un niveau d'abstraction plus élevé que les *namespaces* : les *projets* et permettent de regrouper des *namespaces*.  
En terme plus général, il s'agit de rattacher des des métadonnées relatives à des librairies (**Prometheus**, **Vault**, **Grafana**, etc...) à des objets tels que les **Deployments**, les **Pods** et autres.

Dans le cas ici présent, les *annotations* sont préfixées en *hashicorp.vault.com/* et on peut retrouver :  
- agent-inject : défini si l'injection est explicitiment active ou non pour ce pod,  
- agent-configmap : nom du *configmap* où le *Vault Agent* peut trouver sa configuration ou son template,  
- secret-volume-path,  
- agent-limits-cpu,  
- agent-revoke-grace.

Ici nous n'aurons besoin que de *agent-inject*, *role* et *agent-inject-secret-credentials.txt* pour effectuer notre test, mais tout dépend ce que vous voudrez faire par la suite.  
De plus **agent-inject** ne modifie un déploiement que s'il contient un ensemble d'annotations spécifique...mais **Kubernetes** permet aussi de patcher un déploiement existant.  

Voici ce que l'on peut ajouter pour patcher un déploiement existant :  
```yaml
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "<ROLE>"
        vault.hashicorp.com/agent-inject-secret-credentials.txt: "<SECRET_PATH>"
```

Une fois déployer, pour vérifier que l'intégration s'est bien passé, je vous invite à exécuter la commande suivante : `kubectl exec <POD> --container <APP> -- cat /vault/secrets/credentials.txt`
Si le résultat de la commande affiche bien les secrets que vous avez enregistré dans **Vault**, alors c'est gagné.

## Conclusion
L'intégration de **Vault** dans **Kubernetes** permet, entre autre, de se passer du *secret engine* de **Kubernetes** pour gérer les secrets de manière beaucoup plus décentralisée et sécurisée...et il y a tellement d'application possible que le cas présenté ici ne fait qu'effleurer la surface de ce qui est possible.
Parmis les *Backend Secret* disponible pour **Vault**, il y a également **ETCD**, qui fait partie intégrante de **Kubernetes** et qui fera peut-être l'objet d'un futur article.  
Ce n'est pas tout, il est également possible de déployer un cluster **Vault** dans **Kubernetes** à l'aide des charts **Helm**