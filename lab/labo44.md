+++
date = "2023-04-27T15:29:49+02:00"
draft = "false"
title = "Le Labo #38 | Un peu d'intimité pour Terraform"
+++

Au départ, l'idée derrière **Terraform** était de provisionner des infrastructures Cloud en ayant un accès à la registry officielle de **Hashicorp**. Puis ils ont changé leur fusil d'épaule en se disant que, peut-être, certains avaient besoin de plus de flexibilité dans l'utilisation de leur outil préféré et souhaitaient se passer de la registry officielle et provisionner une ressource avec le provider sur leur disque dur.  

Mais et pour ceux qui évoluaient en environnement complètement fermé et sécurisé ?  
Et bien pas grand chose jusqu'à la version **0.13.2**.  
Cependant, depuis la sortie de cette version, rien n'a réellement changé puisque, malgré le support de *provider network mirror* et de *provider filesystem mirror*, il y avait très peu de documentation sur le sujet...et quand c'était documenté, cela restait assez incomplet.  

Mon dernier article date d'Avril 2022...il était temps que je trouve l'inspiration et ce sujet me semblait assez pertinent au point de vous proposer un article.

## Terraform Provider Mirror
Au dela du fait que que pour un *provider* **Terraform** en local, la configuration est extrêmement simple puisqu'il suffit de renseigner le chemin exact dans le bloc *terraform/required_provider* (voir ci-dessous), **Hashicorp** a implémenté depuis la version *0.13* de **Terraform** la commande **terraform proviers mirror**.  
```hcl
terraform {
  required_providers {
    vsphere = {
      source = "<PATH>/terraform.d/<namespace>/<name>"
      version = "<version>"
    }
  }
}
```

La commande **terraform providers mirror** est documentée [ici](https://developer.hashicorp.com/terraform/cli/commands/providers/mirror).  
Par contre, ce que la documentation ne révèle pas est qu'il est obligatoire d'exécuter la commande au même endroit qu'un fichier contenant le bloc *terraform/required_provider* (au risque de ne rien récupérer).  
Par exemple :  
```hcl
terraform {
  required_providers {
    redfish = {
      source = "lexfrei/redfish"
      version = "0.0.2"
    }
  }
}
```  
Ainsi la commande `sudo terraform providers mirror -plaform=linux_amd64 /opt` téléchargera les trois fichiers suivants dans l'arborescence `/opt/registry.terraform.io/lexfrei/redfish` :  
- 0.0.2.json  
- index.json  
- terraform-provider-redfish_0.0.2_linux_amd64.zip

## Network ou Filesystem ?
Maintenant que les fichiers sont sur le disque dur...que faire avec ?  
Depuis de nombreuses version de **Terraform**, il est possible de définir des options de configuration aussi bien en variables d'environnement qu'à l'intérieur d'un fichier.  
Ce fichier, dénommé **CLI configuration File**, va servir à définir quel type de *mirror registry* je veux utiliser...et il y en a deux types différents :  
**filesystem**  
```txt
provider_installation {
  filesystem_mirror {
    path    = "<path>/<to>/<providers>"
  }
}
```

**network**  
```txt
provider_installation {
  network_mirror {
    url = "https://<domain>"
  }
}
```

En ce qui concerne **filesystem_mirror**, il n'y a pas de configuration particulière au niveau du chemin...seule la présence des fichiers est suffisante. En revanche, **network_mirror** requiert un peu plus de temps...et c'est de ce sujet que la suite de l'article traitera.  
Il s'avère que **Artifactory** propose la fonctionnalité **Terraform Registry** autant pour les *Providers* que pour les *modules*, mais avez-vous déjà lu un article dans lequel j'optais pour la solution la plus simple parmi un éventail ?  
Dans l'optique de coller le plus possible à un cas concret rencontré chez un client, j'ai choisi **Nginx** et **Nexus OSS**.  
Bien que **Nexus OSS** gère très bien tout type de *repository* (maven, npm, apt, bower, pypi, docker, etc...) il n'offre pas la possibilité d'exposer un private registry **Terraform**.

### Nginx
Configurer **Nginx** en tant que *Reverse Proxy* est presque trivial, configurer **Nginx** pour servir du *json* est une tâche que je n'avais encore jamais abordé et qui a failli me donner encore plus de cheveux blancs.  
Une fois les providers téléchargés sur le même serveur que **Nginx**, sans oublier de générer les certificats TLS adequats à l'aide des commandes suivantes :  

##### Le fichier de configuration OpenSSL
```text
[req]
default_bits  = 2048
distinguished_name = req_distinguished_name
req_extensions = req_ext
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
countryName = FR
stateOrProvinceName = Loire Atlantique
localityName = Nantes
organizationName = Self-signed certificate
commonName = <IP>
[req_ext]
subjectAltName = @alt_names
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
IP.1 = 127.0.0.1
DNS.1 = localhost
```

```bash
## Génération du certificat racine
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 365 -out rootCA.pem -config san.cnf
openssl req -new -sha256 -nodes -out server.csr -newkey rsa:4096 -keyout server.key -config san.cnf
## Génération du certificat
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateseial -out server.crt -days 365 -sha256 -config san.cnf
openssl req -x509 -new -nodes -days 365 -keyout server.key -out server.crt -config san.cnf
## Validation du certificat
sudo cp server.crt /usr/local/share/ca-certificates/server.crt
sudo update-ca-certificates --fresh
```

#### La configuration de Nginx
La configuration **Nginx** est relativement simple...mais il ne faut surtout pas oublier de mentionner :  
- **ssl_certificate** et **ssl_certificate_key** ou bien **Terraform** n'ira pas chercher ce que vous lui demanderez même si vous ne vous êtes pas trompé dans la localisation,  
- **server_name**, sinon vous risqueriez d'enchainer les erreurs dans les logs,  
- **root** afin de préciser avec **exactitude** l'emplacement du dossier racine qui sera recherché par **Terraform** dans **required_providers**,  
- **default_type application/json** car les fichiers recherchés par terraform son principalement du json lors de l'*init*,  
- **index index.json** car **nginx**, dans toute sa gentillesse, sert par défaut **index.html** si rien d'autre n'est précisé.  

```
server {
    listen 443 ssl http2;
    server_name <domain>;
    ssl_certificate /<path>/<to>/<certificate_file>.crt;
    ssl_certificate_key /<path>/<to>/<certificate_key>.key;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:!DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_protocols TLSv1.2;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
    root /<path>/<to>/<providers>/;

    location / {
            default_type application/json;
            autoindex on;
            autoindex_format json;
            autoindex_exact_size on;
            index index.json;
            proxy_set_header Accept-Encoding "";
            proxy_set_header X-Forwarded-For $remote_addr;
            tcp_nodelay on;
            sub_filter_types application/json;
            proxy_cache_use_stale error timeout;
    }
}
```

#### Nexus avec un proxy Nginx
En fait, très peu de difficulté à configurer Nexus en tant que **Provider Registry** car le *repository* devra avoir **absolument** la même arborescence que celle qui a été créée à l'aide de la commande `terraform providers mirror` : 

```
registry.terraform.io
	|-hashicorp
	|	|- vsphere
	|	|	|- index.json
	|	|	|- 2.3.1.json
	|	|	|- terraform-provider-vsphere_2.3.1_linux_amd64.zip
```

Donc on commence par créer un repository **raw(hosted)** nommé **registry.terraform.io** pour y uploader les même fichiers en précisant le champs **Directory**.

Concernant la configuration de Nginx ? Très peu de modifications à effectuer :  
- **root** (dans **server**) est a supprimer,
- Ajouter une ligne **proxy_pass** dans **location** est y indiquer l'URL de **Nexus** : proxy_pass http://<url_nexus>:<port>/repository  

Et...c'est tout...

#### La configuration de Terraform
J'ai déjà évoqué plus tôt la configuration de **Terraform** avec *provider_installation/network_mirror*, l'url à indiquer est, bien évidemment, celle de votre serveur nginx, ce qui donne (dans mon cas):  
```
provider_installation {
        network_mirror {
                url = "https://localhost/"
        }
}
```

Au niveau de la resource à générer, j'ai pu essayer avec **vsphere**. Le provider est, biensûr, disponible sur la registry officielle...mais je ne suis pas censé y avoir accès.
```hcl
terraform {
  required_providers {
    vsphere = {
      source = "local-registry.terraform.io/hashicorp/vsphere"
      version = "2.3.1"
    }
  }
}
```

Et lors du **terraform init**, ça donne ce qui suit : 
```bash
Initializing the backend...

Initializing provider plugins...
- Finding local-registry.terraform.io/hashicorp/vsphere versions matching "2.3.1"...
- Installing local-registry.terraform.io/hashicorp/vsphere v2.3.1...
- Installed local-registry.terraform.io/hashicorp/vsphere v2.3.1 (verified checksum)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Incomplete lock file information for providers
│ 
│ Due to your customized provider installation methods, Terraform was forced to calculate lock file checksums locally for the following providers:
│   - local-registry.terraform.io/hashicorp/vsphere
│ 
│ The current .terraform.lock.hcl file only includes checksums for linux_amd64, so Terraform running on another platform will fail to install these providers.
│ 
│ To calculate additional checksums for another platform, run:
│   terraform providers lock -platform=linux_amd64
│ (where linux_amd64 is the platform to generate)
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## le mot de la fin ?
Que dire à part que c'est extrêmement mal documenté ?  
J'ai croisé quelque blogs sur les internets expliquant comment monter un **Provider Registry** sur des espaces de stockage de *block storage* (**S3** ou **Google Cloud Storage**), parfois en ayant recours à la compilation des sources (en Go) ou en créant un provider pour l'occasion.  
A force de tester pendant plusieurs jours et, plus ou moins, de m'arracher les cheveux sur le sujet...j'ai fini par tenter l'approche la plus simple mais la plus mal documentée.

Etrange...