+++
date = "2020-12-19T15:29:49+02:00"  
draft = "false"
title = "Le Labo #32 | Vault dans tout ses états"
+++

Il y a quelques mois, suite au passage de la certification **Hashicorp Terraform Associate** j'ai eu envie de préparer celle sur **Vault** pour plusieurs raisons :  
- L'aspect sécurité m'intéresse fortement,  
- Parce que je suis un grand fan des projets **Hashicorp** depuis ma découverte de **Terraform** il a près de cinq ans.

Cependant je n'avais pas pu me lancer dedans car je préparais également deux autres certifications (que j'ai obtenu par la suite).  
Depuis le temps, ca m'était sorti de l'esprit pour me concentrer sur la certification **AZ-500** (**Azure Security Engineer Associate**) et **Certified Kubernetes Security Specialist**...et récemment on m'a proposé de travailler sur un POC autour de **Vault** avec les pré-requis suivants :  
- Utilisation de PostgreSQL en tant que *Backend Storage*,  
- Cloud Hybride : On-Premises + Google Cloud Platform,  
- Authentification LDAP,  
- Auto-Unseal de Vault,  
- Observabilité (via **Prometheus**)  

Une fois la surprise passé, j'ai commencé par visionner une grande partie de la documentation disponible sur le site officiel, puis sur **Pluralsight** (qui propose deux séries de vidéos sur le sujet)...et je me suis lancé.

## Le besoin
L'idée derrière cet article est de pouvoir proposer un service de gestion, de chiffrage et de rotation de mot passe le plus simple à utiliser et qui puisse s'intégrer dans n'importe quel environnement hybride.  
Le fait de pouvoir le sécuriser un minimum est également un facteur important car les utilisateurs ou applicatifs auront besoin du service devront être des acteurs connnus et bien identifiés.  
C'est pour cela que de nombreux mécanismes seront mis en place, tels que **Auth Methods**, **Policies** et **Auto-Unseal**.  
Je vous rassure, ceci n'est pas un tutoriel mais plutôt un retour d'expérience après une plongée relativement profonde sur cette technologie.  
De plus, j'ai choisi de présenter cla via **Vagrant** (autant rester chez **Hashicorp**) en sachant que ce n'est absolument pas un outils à utiliser en production.

## L'infrastructure en local
J'ai encore une fois fais appel à mon expérience de quelques années sur **Vagrant** pour monter une petite infrastructure assez légère, celle-ci ne consommant, au final, que 4,5 Go de RAM sur les 16 disponible sur mon portable (et j'hésite encore à passer sur 32 Go).  
Les machines virtuelles montées son les suivantes :  
- *vault1*, *vault2* et *vault3* seront, evidemment, dédiées au cluster **Vault**,  
- *consul1*, *cpnsul2* et *consul3* seront exclusivement dédiées au cluster **Consul** pour l'aspect *Service Discovery*,  
- *pgstor* sur laquelle sera installée **PostgreSQL** (en version 9.6, bien que la version n'ait pas une grand incidence...du moment qu'elle est ultérieure à 9.5),  
- *ldap*...pas besoin de faire un dessin pour celle-la, un serveur **LDAP** ainsi que sa web UI seront installés dessus (j'avoue avoir un peu de mal avec la configuration et la création d'utilisateurs en ligne de commande),  
- *prom* qui s'occupera de l'observabilité.  

A ce stade, seul l'aspect **Auto Unseal** n'est pas géré localement...sauf si je fais appel à **Transit**.  

## Préparer le cluster...
Au début, j'avais prévu d'utiliser la même technologie en ce qui concerne le **backend storage** et le **service discovery** et je me suis dit : *Trop facile, pour en apprendre plus sur le sujet, autant introduire un peu de complexité*.  
Par le passé, j'avais déjà eu l'occasion de publier des articles sur **Consul**, notamment sur du [*Service Discovery*](http://www.ageekslab.com/lab/labo33/)...et bien il s'agit du même principe.

Ci dessous le bout de *Vagrantfile* relatif aux VMs **Consul** :
```
  {:hostname => 'consul1', :ram => ram0, :box => box1, :provision => "shell"},
  {:hostname => 'consul2', :ram => ram0, :box => box1, :provision => "shell"},
  {:hostname => 'consul3', :ram => ram0, :box => box1, :provision => "shell"},
```

J'ai commencé par la configuration générale des trois serveurs en activant les **ACL** : 
```json
{
        "datacenter": "dc_local",
        "data_dir": "consul_data",
        "log_file": "consul_log/consul.log",
        "log_level": "INFO",
        "acl": {
                "enabled": true,
                "default_policy": "deny",
                "down_policy": "extend_cache",
                "enable_token_persistence": true,
                "enable_token_replication": true
        }
}
```

Vous pouvez également constater la policy **deny** par défaut des **ACL** afin de rendre **Consul** non accessible en lecture sans le token adéquat.

## Préparer le backend Storage
Comme expliqué précédemment, au lieu d'utiliser **consul** en tant que **Backend Storage**, je me suis orienté vers **PostgreSQL**.  
De la même manière que **Consul**, j'ai égalément déployé un serveur de base de données **PostgreSQL** à l'aide de **Vagrant** : 

```
{:hostname => 'pgstor', :ram => ram0, :box => box1, :provision => "shell"},
```

Bien que ce type de SGBD ne supporte pas la haute disponibilité, un cluster **Vault** peut y stocker ses secrets dans la table **vault_kv_store** (voir ci-dessous la requête SQL) : 
```
CREATE TABLE vault_kv_store (
  parent_path TEXT COLLATE "C" NOT NULL,
  path        TEXT COLLATE "C",
  key         TEXT COLLATE "C",
  value       BYTEA,
  CONSTRAINT pkey PRIMARY KEY (path, key)
);

CREATE INDEX parent_path_idx ON vault_kv_store (parent_path);

CREATE TABLE vault_ha_locks (
  ha_key                                      TEXT COLLATE "C" NOT NULL,
  ha_identity                                 TEXT COLLATE "C" NOT NULL,
  ha_value                                    TEXT COLLATE "C",
  valid_until                                 TIMESTAMP WITH TIME ZONE NOT NULL,
  CONSTRAINT ha_key PRIMARY KEY (ha_key)
);
```

Ne surtout pas oublier la configuration d'accès à distance de **PostgreSQL**, sinon vous risqueriez de vous arracher les cheveux.

## Déployer Prometheus
Cette partie est probablement la plus simple, car à part télécharger, compléter la configuration des *targets* et démarrer **Prometheus**, il n'y a rien à faire de particulier.

A ce stade, vous pouvez déjà démarrer **Vault** à l'aide de la commande suivante : `vault server -config=vault.json`, il est prêt à être utilisé.
Vous n'aurez pas besoin d'exécuter les commandes `vault operator init` et `vault operator unseal` (puisque un mécanisme d'**auto unseal** est présent dans la configuration).  
Cependant je vous invite à faire un tour sur l'UI afin d'initialiser le mécanisme d'auto unseal et de récuperer le master token.

## Déployer le serveur LDAP
N'ayant pas de serveur **LDAP** à porté de main, j'en ai déployé un toujours à l'aide de **Vagrant** : 

```
{:hostname => 'ldap', :ram => ram0, :box => box1, :provision => "shell"},
```

Je vais également vous épargner la configuration d'un serveur **LDAP** que vous pouvez retrouver [ici](https://kifarunix.com/install-and-configure-openldap-server-on-debian-9-stretch/) (par exemple).

## Déployer Vault
Toujours grâce à **Vagrant**, je déploie autant de VMs que pour **Consul** :

```
  {:hostname => 'vault1', :ram => ram0, :box => box1, :provision => "shell"},
  {:hostname => 'vault2', :ram => ram0, :box => box1, :provision => "shell"},
  {:hostname => 'vault3', :ram => ram0, :box => box1, :provision => "shell"},
```

Dans ce déploiement, je vais configurer plusieurs éléments :

- **Backend** : **Consul**, Il ne s'occupera pas du stockage mais de la partie clustering,
- **Storage** : **PostgreSQL**, ne s'occupera que du stockage,
- **Seal** : Dans le but de configurer l'auto unseal de **Vault**,
- **Listener** : En clair, il s'agit du port et de l'addresse sur laquelle **Vault** répondra aux requêtes,
- **Telemetry** : Il s'agit de la configuration de la métrologie.

Voici la configuration en détail :
```json
{
  "backend": {
    "consul": {
      "address": "172.28.128.29:8500",
      "path": "vault_data/"
    }
  },
  "listener": {
    "tcp":{
      "address": "172.28.128.24:8200",
      "tls_disable": 1,
      "telemetry": {
        "unauthenticated_metrics_access": true
      }
    }
  },
  "storage": {
    "postgresql": {
      "connection_url": "postgres://postgres:pw@172.28.128.27:5432/postgres?sslmode=disable",
      "ha_enabled": "true"
    }
  },
  "telemetry": {
    "prometheus_retention_time": "30s",
    "disable_hostname": true
  },
  "seal": {
    "gcpckms": {
      "credentials": "/vagrant/tf-project-1351-5deb2c7350dd.json",
      "project": "tf-project-1351",
      "region": "europe-west1",
      "key_ring": "tf-project-keyring",
      "crypto_key": "new-key-1"
    }
  },
  "ui": true
}
```

Comme vous pouvez le constater, j'ai également activé l'interface utilisateur...parfois c'est plus agréable pour les démonstrations techniques.
Avec la configuration ci-dessus, la commande `vault server -config=[config].json` démarre le serveur **Vault**.

## Configurer Vault
Notre serveur **Vault** est désormais actif et correctement configuré.

### Préparation de rôles applicatifs
Quelques définitions avant d'aller plus loin :  
- Une *policy* est définie par un bloc *json* indiquant les droits (*create*, *liste*, *delete*) sur l'arborescence de *secrets*,  
- Un *role* peut assumer une application et lier une ou plusieurs *policies*,  
- Un token permet de traiter des informations et aura une durée de vie limitée (qui peut être renouvelée lors d'une connexion).  

Pour activer un *rôle*, exécutons la commande suivante : `vault auth enable approle`.  

Dans le but d'utiliser le provider **Vault** avec **Terraform** (et donc de pouvoir sécuriser les mots de passe, clés, etc..), je vais commencer par éxecuter la commande `vault policy write <policy_name> <policy_file>.hcl`, avec *policy_file.hcl* contenant ce qui suite :  
```hcl
{
  path "secret" {
    capabilities = ["create", "list", "update", "read"]
  }
}
```

Ensuite créons le role *iac_policy* à l'aide de la commande `vault write auth/approle/role/iacrole secret_id_ttl=10m token_num_uses=10 token_ttl=10m token_max_ttl=30m secret_id_num_uses=10`.  
Après avoir récupéré l'id du role que nous venons de créer (`vault read auth/apprle/role/iacrole/role-id`), nous allons créer un secret pour ce rôle :  
```shell
vault write -f auth/approle/role/iacrole/secret-id
Key                   Value
---                   -----
secret_id             6529337b-b62c-0637-c968-42faf454d5ea
secret_id_accessor    b0a5a45c-e1f-cdff-5155-d74c2f18c01d
```

### Préparation de l'authentification via LDAP
Nous avons déjà un serveur **LDAP** prêt à l'utilisation, on va pouvoir passer à l'activation `vault auth enable ldap` et à sa configuration : 
```bash
vault write auth/ldap/config \
url="ldap://172.28.128.28:389" \
insecure_tls=true \
binddn="cn=admin,dc=test,dc=vault,dc=local" \
binpass="passw01" \
userdn="ou=users,dc=test,dc=vault,dc=local" \
userattr="uid" \
groupdn="ou=techs,dc=test,dc=vault,dc=local" \
groupfilter="(|(memberUid={{.Username}})(member={{.UserDN}})(uniqueMember={{.UserDN}}))"
```

Une fois la configuration du connecteur **LDAP** effectuée, il faut effectuer un mapping entres les groupes et les policies : `vault write auth/ldap/groups/techs policies=default,techs`.  
Ensuite, on crée les utilisateurs et les policies qui leur sont propres : `vault write auth/ldap/users/adam groups=techs policies=adam`  
Pour résumer, nous venons de créer un groupe *techs* sur lequel s'appliquent les policies *default* et *techs* ainsi qu'un utilisateur *adam* auquel s'applique la policie *adam* ainsi que celle de son groupe.  

Pour vérifier que tout fonctionne, autant essayer de se connecter :  
```shell
vault login -method=ldap username=adam
Password (will be hidden):
Successfully authenticated! The policies that are associated
with this token are listed below:

default, default, techs, adam
```

## Conclusion
Je n'ai pas abordé de nombreux type d'**Auth Methods** malgré le fait d'en avoir testé quelques-uns...parmis ceux-la, j'ai pu me faire la main sur **gcp** et **gcpckms** (respectivement création de ServiceAccounts/Token et création de Cryptographic Keys sur **Google Cloud Plaform**) qui sont deux méthodes assez puissante (surtout la première en ce qui concerne la sécurisation des échanges entre services).  
Cependant, la prise en main est assez complexe...mais se plonger dedans permet d'avoir une très bonne idée de comment configurer et sécuriser cette excellente solution de stockage, gestion et sécurisation/rotation de données critiques.