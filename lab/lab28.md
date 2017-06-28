+++
date = "2017-04-24T15:29:49+02:00"
draft = "false"
title = "Le Labo #23 | Vagrant, Kubernetes, Docker et Salt"

+++

Lors de mon précédent article, je faisais un tour d'horizon de **Kubernetes**, comment l'installer, créer un cluster rapidement et déployer un **Service** et deux **Pods** *de base* en faisant en sorte que la résolution de noms fonctionne.  
Au bureau...je me suis littéralement arraché les cheveux jusqu'a finir par comprendre (tout en travaillant sur d'autres projets).

L'idée de cet article est de :

	- Déployer des VM à l'aide de **Vagrant**
	- Créer un cluster des ces VM grâce à **Kubernetes**
	- Configurer une connexion à un **Docker Private Registry**
	- Déployer des conteneurs **Docker** sur ce cluster
	- Déployer des applications dans les conteneurs à l'aide de **Salt**

# 1ere étape: Vagrant
Je pourrais aussi déployer les VM à l'aide de Terraform, ca n'aurait aucune incidence sur le résultat final...ce serait même plus simple.  
En déployant les VM à l'aide de Vagrant, j'introduis un potentiel point de blocage au niveau des interfaces réseau.  
Il s'avère que la version de Vagrant que j'utilise crée automatiquement une interface réseau avec la même adresse IP pour toutes les VM (enp0s3 avec 10.0.2.15)...quelque chose de totalement inutile pour **Kubernetes**.

Le script de déploiement embarque un provisionning très simple consistant uniquement à suivre les recommandations du site officiel de Kubernetes, à savoir : 

	- Ajouter le repo officiel Kubernetes au système
	- Désactiver SE-LINUX
	- Installer docker, kubelet, kubeadm, kubectl et kubernetes-cni (le pack de driver network-overlay)

Mais, il ne faut surtout pas oublier un détail très important : **Kubernetes** n'aime PAS firewalld.  
Donc, si comme moi, vous continuez avec **CentOS 7**, désinstallez **firewalld** et installez **iptabes**, les commandes suivantes seront vos amies : 

	- systemctl stop firewalld; systemctl disable firewalld; yum remove -y firewall
	- yum install -y iptable.services; systemctl enable iptables; systemctl start iptables

Je ne peux que vous suggérer la commande suivante qui vous évitera de vous demander **Pourquoi kubeadm join ne fonctionne pas vers kube0 ?** :

	- iptables -L --line-numbers 
	- Supprimez toutes règles avec *reject* : iptables -D <line-number>

Une fois votre lab de test **up & running**, il est temps de créer le cluster avec **Kubernetes**

# 2ème étape : Kubernetes
Partons du principe que **Kubernetes** à été installé automatiquement lors de la génération du lab de test.  
Je vous suggère de bien vérifier les points suivants avant de continuer :

	- Vos variables d'environnement (surtout NO_PROXY et no_proxy) doivent contenir les IP de kube0, node1 et node2, le proxy (si vous en avez un) et le docker registry,
	- Votre fichier /etc/hosts : doit aussi contenir les adresses IP et les noms des VM kubernetes.

Maintenant que tout semble prêt, lancez la commande `kubeadm init` sans oublier le flag `--apiserver-advertise-address` et l'adresse IP liée à la seconde interface réseau...et patientez le temps de récupérer la main sur le serveur ainsi que le token qui permettra à **node1** et **node2** de rejoindre le cluster.  
Pour joindre le noeuds à kube0, exécutez juste la commande que donne **kubernetes** sur **kube0** en fin d'initialisation de cluster.  
Ensuite exécutez la commande suivante : `kubectl get po,ds,deploy,svc,ds --all-namespaces -o wide`  

Vous verrez peut-être ce qui suit : 

	NAMESPACE     NAME                                      READY     STATUS    RESTARTS   AGE       IP                NODE
	kube-system   po/dummy-2088944543-js4fg                 1/1       Running   0          1h        188.226.140.115   kube0
	kube-system   po/etcd-kubernetes-0                      1/1       Running   0          1h        188.226.140.115   kube0
	kube-system   po/kube-apiserver-kubernetes-0            1/1       Running   0          1h        188.226.140.115   kube0
	kube-system   po/kube-controller-manager-kubernetes-0   1/1       Running   0          1h        188.226.140.115   kube0
	kube-system   po/kube-discovery-1769846148-wxd60        1/1       Running   0          1h        188.226.140.115   kube0
	kube-system   po/kube-dns-2924299975-66p06              4/4       Running   0          1h        10.40.0.1         kube0
	kube-system   po/kube-proxy-2jzmm                       1/1       Running   0          55m       188.226.140.115   kube0
	kube-system   po/kube-proxy-2vsmn                       1/1       Running   0          55m       188.226.140.112   kube2
	kube-system   po/kube-proxy-t2z0r                       1/1       Running   0          55m       188.226.140.113   kube1
	kube-system   po/kube-scheduler-kubernetes-0            1/1       Running   0          1h        188.226.140.115   kube0

A ce stade là, votre cluster **Kubernetes** est censé être pleinement opérationnel...passons à la configuration de **Docker**.

# 3ème partie : Docker
## Configuration
Il faut savoir deux choses : 

	- **Kubernetes** n'installe pas une version de **Docker** totalement à jour. Sur le site officiel, la 1.13 est disponible alors que **Kubernetes** install la 1.12. Cependant rien de grave, elles fonctionne de manière identique.
	- La configuration de **Docker** est différentes de ce que l'on peut trouver habituellement. Il y a deux fichier à modifier pour que toute la configuration soit vraiment prise en compte : /etc/sysconfig/docker et /lib/systemd/system/docker.service

**/etc/sysconfig/docker** contiendra tout ce qui est en relation avec les proxy (https, http et no_proxy) ainsi que les *insecure_registry*...mais après un démarrage du démon et un `docker info`, il est possible que tout ne soit pas affiché, dans ce cas là, ajoutez les mêmes informations dans **/lib/systemd/system/docker.service**.  

Il faudra aussi se connecter au **Docker Private Registry** avec `docker login <server> -u <username> -p <password>`

**Important** : Ceci est à faire sur toutes les VM.

## Les dockerfiles
Au bureau, j'ai créé plusieurs Dockerfiles, un contenant uniquement le minion Salt et trois autres plus spécifiques utilisant le minion Salt comme base.  
A côté de celui-là, j'ai aussi le master Salt.

# 4ème partie : Le déploiement
Maintenant, il convient de savoir quel type d'environnement nous allons avoir besoin : 

	- Plusieurs Pods, plusieurs Services 
	- Plusieurs Pods, un seul Service et Résolution de Noms via le Service
	- Plusieurs Deployments, plusieurs Services - pas d'insolation via Namespace
	- Plusieurs Deployments, plusieurs Services - Isolation via Namespace et Resolution de Noms (avec Réplication)

Les possiilités sont multiples mais seule la dernière m'intéresse (pour la dernière partie relative à Salt), pour mettre en oeuvre tout cela, je vais avoir besoin des composants suivants :

	- Namespace
	- Secret
	- Service
	- Deployment

Pourquoi créer un nouveau **Namespace** ? 

	- Pour bénéficier de la résolution de noms a ce niveau 
	- Pour éviter de perdre tout le déploiement si le namespace par défaut venait à tomber
	- Parce la résolution de noms n'est pas possible autrement si l'on utilise les Deployments au lieu de simple Pods

## Le Namespace

```yaml
	apiVersion: v1
	kind: Namespace
	metadata:
	  name: <namespace_name>
```

Ensuite nous allons créer un **Secret** qui fera le lien entre le fichier $HOME/.docker/config.cfg et **kubernetes** et permettra d'utiliser les images stocker sur notre Private Registry : 

## Le Secret

```yaml
	apiVersion: v1
	kind: Secret
	metadata:
	  name: <secret_name>
	  namespace: <namespace_name>
	data:
	  username: <docker_username_base64>
	  password: <docker_password_base64>
	  server: <docker_server_base64>
	  .dockercfg: <docker_cfg_base_64>
	type: kubernetes.io/dockercfg
```

Pour éviter de vous tromper, je vous suggère quand même d'exécuter la commande ci-dessous : 
`kubectl create secret docker-registry --docker-server=<docker_server> --docker-username=<docker_username> --docker-password=<docker_password> --docker-email=<docker_email>` et ensuite exportez le secret créé en yaml (ou en json.) pour obtenir le *.dockercfg*.

Dans cette optique, plus vous allez créer de **Deployments**, plus vous devrez créer de **Services** qui leur seront associés.  

## Deployment et Service - Master

```yaml
	apiVersion: v1
	kind: Service
	metadata:
	  labels: 
	    name: salt-master
	  name: salt-master
	  namespace: <namespace_name>
	spec:
	  clusterIP: None
	  ports:
	  - port: 4505
	    targetPort: 4505
	  - port: 4506
	    targetPort: 4506
	  selector:
	    service: salt-master
	---
	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: salt-master
	  labels:
	    name: salt-master
	  namespace: <namespace_name>
	spec:
	  replicas: 1
	  template:
	    metadata:
	      name: salt-master
	      namespace: <namespace_name>
	      labels:
	        name: salt-master
	    spec:
	      containers:
	      - image: master
	        name: salt-master
	        volumeMounts:
	        - mountPath: /salt/conf
	          name: salt-conf
	        - mountPath: /salt/states
	          name: states
	        - mountPath: /salt/pillar
	          name: pillars
	        - mountPath: /salt/delivery
	          name: delivery
	        - mountPath: /salt/formulas
	          name: formulas
	        command: ["salt-master"]
	        args: ["--config-dir=/salt/conf"]
	        ports:
	        - containerPort: 4505
	        - containerPort: 4506
	        tty: true
	      restartPolicy: Always
	      imagePullSecrets:
	        - name: <secret_name>
	      volumes:
	      - name: salt-conf
	        hostpath:
	          path: $HOME/salt/conf
	      - name: states
	        hostpath:
	          path: $HOME/salt/states
	      - name: pillars
	        hostpath:
	          path: $HOME/salt/pillar
	      - name: delivery
	        hostpath:
	          path: $HOME/salt/delivery
	      - name: formulas
	        hostpath:
	          path: $HOME/salt/formulas
```
## Deployment et Service - Minion

```yaml
	apiVersion: v1
	kind: Service
	metadata:
	  labels: 
	    name: salt-minion
	  name: salt-minion
	  namespace: <namespace_name>
	spec:
	  clusterIP: None
	  selector:
	    service: salt-minion
	---
	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: salt-minion
	  labels:
	    name: salt-minion
	  namespace: <namespace_name>
	spec:
	  replicas: 1
	  template:
	    metadata:
	      name: salt-minion
	      namespace: <namespace_name>
	      labels:
	        name: salt-minion
	    spec:
	      containers:
	      - image: minion
	        name: salt-minion
	        command: ["salt-minion"]
	        tty: true
	      restartPolicy: Always
	      imagePullSecrets:
	        - name: <secret_name>
```

# 5ème partie : Salt et le provisionning automatique
Partons du principe que vous n'avez pas oublié les pré-requis des images **Docker** qui sont :

	- Salt-minion
	- Salt-master
	- Pas d'iptables (pour éviter que la communication entre conteneurs soit bloquée)

A ce stade là, si votre déploiement s'est bien déroulé, vous devriez pouvoir voir les demandes de clés en attente sur le Salt-Master et les accepter dans le même temps...et donc ne pas avoir de problèmes particuliers au niveau du provisionning via Salt.  
Ceci dit, à ce niveau, je vous laisse la main...mais j'ai fais quelques tests de mon côté avec le setup suivant : 

	- Un zookeeper
	- Un hazelcast
	- 4 conteneurs avec des applications métiers

Et tout fonctionne correctement avec les tests d'intégration qui vont bien.
