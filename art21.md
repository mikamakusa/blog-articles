+++
date = "2021-06-07T22:23:00+02:00"
draft = false
title = "En route vers la certification **Certified Kubernetes Security Specialist** - 1ere partie"
+++

Au lieu d'un seul article global sur la préparation d'une certification, comme j'ai pu vous le proposer par le passé, je vais initier une série d'articles décrivant chaque aspect pour lesquels vous serez censés être intérrogés (si vous souhaitez la passer).  
Les sujets ci-dessous sont au programme :  
- Cluster Setup,  
- Cluster Hardening,  
- System Hardening,  
- Microservices Vulnerabilities,  
- Supply Chain Security,  
- Monitoring, Logging and Runtime Security.  

Ce premier article se focalisera sur le premier aspect, à savoir **Cluster Setup**, et traitera de plusieurs points tels que :  
- Les **Network Security Policies**,  
- **CIS Benchmark** afin de vérifier le niveau de securité de configuration de chaque composant de **Kubernetes**,  
- La sécurisation des **Ingress Controllers**,  
- La protection meta-données des **Nodes**,  
- L'usage de l'UI de **Kubernetes** et sa securisation,  
- La vérification des binaires avant déploiement.

## Network Security Policies
Que sont les **Network Security Policies** ?  
Il s'agit d'instructions destinées à spécifier comment un Pod peut communiquer avec des *entitées* du réseau selon une combinaisons d'identifiants :  
- Les autres pods (à l'exception de lui même),  
- Les Namespaces avec lesquelles il peut communiquer,  
- Les blocs d'IP. Attention, dans ce cas, un **Pod** peut accéder à n'importe quel autre Pod situé sur le même **Node**  

Lorsque l'on défini une **Security Policy** relatif à un **Pod** ou à un **Namespace**, on fait appel à un **Selector** pour spécifier quel trafic est permis *depuis* ou *vers* le **Pod**/**Namespace** en rapport avec le **selector**.  
**A SAVOIR** :  
- **Par défaut**, les **Pods** acceptent du trafic de n'importe quelle source.  
- Les **NetworkPolicies** s'additionnent et n'entrent pas en conflit entre eux.

Un seul pré-requis : un plugin *Network*

### Quelques exemples : 
#### Limiter le trafic vers une application
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: bookstore
      role: api
  ingress:
  - from:
      - podSelector:
          matchLabels:
            app: bookstore
```

#### Refuser le trafic vers l'extérieur du cluster Kubernetes
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: foo-deny-external-egress
spec:
  podSelector:
    matchLabels:
      app: foo
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  - to:
    - namespaceSelector: {}
```

#### Refuser tout trafic non whitelisté vers un namespace
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  ingress: []
```

## CIS Benchmarks
De quoi s'agit-il ?
Il s'agit de directives relatives à l'audit et à la sécurité de cluster Kubernetes.  
Par défaut, un système peut être vulnérable à de nombreux types d'attaques et il est important de pouvoir le sécuriser un maximum.  
Par exemple, dans le cas d'un serveur sur lequel des ports USB sont disponible et que leur utilisation n'a pas été prévue, ceux-ci doivent être désactivés afin de prévenir tout types d'attaques par ce vecteur.  
Autre exemple, quels utilisateurs ont accès au système et peuvent se connecter en tant que *root* ?  
Si ceux-ci effectuent des changement impactant le fonctionnement des services, il pourrait être impossible de connaître l'auteur des modifications.  
C'est pour cela que les **best pratices** recommandent de désactiver le compte **root** et de se connecter avec son propre compte puis d'élever ses propres privilèges (à l'aide de **sudo**).  
Autres exemples, seuls les services et/ou *filesystems* utile au fonctionnement du server devront être activés...tout comme seuls les ports **vraiment** nécessaire doivent être ouverts et donc configurer le **firewall** de manière la plus fine possible.  

Le [site CIS](https://www.cisecurity.org/) propose de nombreux *benchmark* pour :  
- Les Systèmes d'Exploitation **Linux**, **Windows**, **OSX** ainsi que mobile **iOS** et **Android**,  
- Les plateformes Cloud **AWS**, **Azure**, **Google**,  
- Les matériels réseau provenant de **Check Point**, **Cisco**, **Juniper** et **Palo Alto**,  
- ainsi que des middlewares tels que **Tomcat**, **Docker** ou encore **Kubernetes**.

### CIS Benchmark appliqué à Kubernetes : kube-bench
**Kube-bench** est une application développée en GO, s'appuyant un maximum sur les recommandations de sécurité de [CIS](https://www.cisecurity.org/), avec une configuration très évolutive grâce à l'utilisation de fichier en *yaml* et pouvant être utilisé de plusieurs manières différentes :  
- En l'installation depuis un conteneur,  
- En l'installation grâce au binaire téléchargé,  
- En compilant les sources,  
- En l'exécutant depuis un conteneur, que ce soit un conteneur isolé ou dans un cluster **Kubernetes** ou **Openshift**.

**Kube-bench** exécute les contrôles défini dans un fichier *yaml* nommé **control.yaml** et peut s'appliquer tout type de nodes de type master ou worker (peu importe la version).
```yaml
- id: 1.2
  text: Scheduler
  checks:
    - id: 1.2.1
      text: "Ensure that the --profiling argument is set to false (Scored)"
      audit: "ps -ef | grep kube-scheduler | grep -v grep"
      tests:
        bin_op: or
        test_items:
          - flag: "--profiling"
            set: true
          - flag: "--some-other-flag"
            set: false
      remediation: "Edit the /etc/kubernetes/config file on the master node and
        set the KUBE_ALLOW_PRIV parameter to '--allow-privileged=false'"
      scored: true
```

Une fois les tests effectués par **kube-bench**, les résultats sont enregistrés dans un fichier et chaque *contrôle* indique un état en fonction des quatre suivants :  
- **PASS**, Le contrôle a été effectué avec succès.  
- **FAIL**, le contrôle est un echec, mais la *remediation* décrit comment corriger la configuration afin que, lors du prochain contrôle, celui soit OK.  
- **WARN**, le test nécessite une attention particulier, vérifier la $remediation* pour plus d'information. Il ne s'agit pas forcément d'un rapport d'erreur.  
- **INFO**

## L'usage de l'UI de **Kubernetes** et sa sécurisation
Il s'agit ici de savoir comment *installer* le Dashboard de **Kubernetes** et de le sécuriser le mieux possible.   
Le dashboard Kubernetes se déploie  à l'aide de la commande suivante : `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml` et pour y accéder : `kubectl proxy --address 0.0.0.0`.    
Il faut savoir que :  
- Kubernetes supporte quatre modes d'authentification,  
- Ceux-ci sont gérés par l'API de Kubernetes,  
- Le Dashboard ne sert que de proxy et fait office de "passe-plat" vers Kubernetes pour toutes les informations relatives à l'authentification.  

### L'authentification
Comme évoqué plus tôt, Kubernetes supporte quatre modes d'authentification parmi les suivants :  
- **Bearer Token**,  
- **Username/Password**,  
- **Kubeconfig**,  
- **Authorization Header** (supporté depuis le version 1.6 et dispose de la priorité la plus élevée).

#### Les Bearer Tokens
Pour les configurer le plus efficacement possible, il est nécessaire d'être extrêmement familier avec les concepts de **Service Account**, **Role**, **Cluster Role**, et les permissions qui peuvent leur être attribuées.

##### ServiceAccount  
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```
##### ClusterRoleBinding
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```
Une fois les deux *objets* précédement créés, on peut obtenir le **Bearer Token** grâce à la commande suivante : `kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
`

#### Username/Password
Désactivé par défaut, car l'API de Kubernetes a besoin d'une configuration au niveau des attributs et non au niveau des rôles.

#### Kubeconfig
Avec ce mode d'authentification, seules les options spécifiées par le flag `--authentication-mode` sont prise en charge dans le fichier sus-nommé. Dans le cas contraire, une erreur se affichée dans le dashboard.

#### Authorization Header
Pour ce type de méthode d'authentification, le **Bearer Token** sera nécessaire car vous en aurez besoin lors de chaque requête vers le Dashboard.

## La vérification des binaires pré-installation
Il s'agit ni plus ni moins que de vérifier la valeur du *hash* des binaires relatifs à Kubernetes et de les comparer avec ceux que vous télécharger sur votre serveur...au cas où votre action se ferait interceptée par un hacker.  
Même si le dit hacker modifie un fichier de l'archive, il faut savoir que toute modification d'une archive modifie également la valeur de son *hash*.  
Pour cela, une fois le téléchargement terminé, vous pouvez exécuter la commande suivante : `shasum -a 512 kubernetes.tar.gz` et comparer la valeur obtenue avec ce qui est affichée sur la page de téléchargement.

Par exemple, en téléchargeant [kubeadm](https://dl.k8s.io/v1.21.0/bin/linux/amd64/kubeadm) en version 1.21.0 dont la valeur de *hash* est **7bdaf0d58f0d286538376bc40b50d7e3ab60a3fe7a0709194f53f1605129550f**, je vais obtenir la même valeur de *hash* une fois l'archive téléchargée (cete fois-ci avec la commande `shasum -a 256 kubeadm`)  

## La sécurisation des **Ingress Controllers**
Tout d'abord : qu'est-ce qu'un **Ingress Controller** ?  
Il s'agit d'un *objet* **Kubernetes** qui gère l'accès externe aux services dans un cluster, généralement du trafic HTTP et peut également fournir des fonctionnalités de type équilibrage de charge.  
Les sujets traités dans cette partie seront :  
- La création de certificats TLS,  
- La création de **secrets** (intégrant les certificats TLS) dans **Kubernetes**,
- La configuration d'**Ingress Controllers** intégrant les **secrets**.

### Création d'un certificat TLS
Partant du principe que vous avez déjà créé votre **Ingress Controller** et qu'il est accessible en **HTTP**, la prochaine étape est de sécuriser son contenu à l'aide de HTTPS...et par conséquent de créer un certificat TLS auto-signé : `openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes`

### Création du secret
Une fois la paire clé/certificat créé, et à moins d'avoir déjà déployé **cert manager** dans **Kubernetes**, il faut créer un secret afin de pouvoir intégrer, par la suite, le certificat TLS à l'**Ingress controller** : `kubectl create secret tls-secure-ingress --cert=cert.pem --key=key.pem`

### Configuration de l'Ingress Controller
Le certificat TLS et le secret créés, la dernière étape est d'intégrer le secret dans l'**Ingress Controller**.  
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
      - test-secure-ingress.com
    secretName: tls-secure-ingress
  rules:
  - host: test-secure-ingress.com
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service: 
            name: service1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service: 
            name: service2
            port:
              number: 80
```

## La protection meta-données des Nodes
Les **Nodes Metadatas**...De quoi s'agit-il ?  
Toutes les machines virtuelles (nœuds) hébergées dans le cloud devraient accéder au **Endpoints** des **Nodes Metadata** pour diverses raisons. Cependant, permettre l'accès à toutes les ressources n'est pas recommandé. Pour améliorer la sécurité à ce niveau, il faut appliquer des **Network Policies**, de sorte que seules les ressources désignées au sein des cluster **Kubernetes** pourraient avoir la capacité de contacter les **Endpoints** des **Nodes Metadata**.  

Toutes les informations dont vous pourriez avoir besoin sur le sujet se trouvent ici :  
- [Google Cloud Platform](https://cloud.google.com/kubernetes-engine/docs/how-to/protecting-cluster-metadata?hl=fr),  
- [AWS](https://docs.aws.amazon.com/eks/latest/userguide/best-practices-security.html),  
- [Azure](https://docs.microsoft.com/fr-fr/azure/aks/concepts-clusters-workloads)

### Network Policies
Le principe des **Network Policies** est de gérer la communication **Pod-to-Pod**, **Pod-to-Service** et **External-to-Service**, de la même manière qu'un Firewall. L'identification des ressources impactées restant à la charge des administrateurs via des *labels*, des *namespaces* ou des *adresses IP*.  
La documentation officielle donne, par exemple ceci :  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

Ici, ce qui nous intéresse c'est principalement la possibilité de bloquer l'accès aux **Metadata Endpoint** à l'aide des **Network Policies** à un ou plusieurs **Pods**, voir ci-dessous:  
#### Deny trafic to metadata
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
          - <POD_IP_ADDRESS>
```

#### Allow trafic to metadata
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-allow
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
          - <POD_IP_ADDRESS>
```

A bientôt pour le prochain article, cette fois-ci il traitera de toutes les problématiques relatives au **cluster hardening**, à savoir : 
- Les restrictions d'accès à l'API de **Kubernetes**,  
- L'usage des **Role Based Access Control** dans le but de minimiser l'exposition,  
- Le *fine tuning* des **ServiceAccounts**,  
- La fréquence des mise à jour de **Kubernetes**.