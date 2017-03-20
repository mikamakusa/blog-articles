+++
date = "2017-03-20T15:29:49+02:00"
draft = "false"
title = "Le Labo #22 | Kubernetes - Joies et bonheurs"

+++

## Avant de commencer
Pas de jaloux...bientôt, je me lancerais sur un article autour de Docker Swarm.  
Après un article, il y a quelques mois sur Nomad, je me suis dis qu'il fallait que je fasse un nouveau tour d'horizon sur Kubernetes pour plusieurs raisons : 

- Il me semble que la technologie est bien plus mature que Nomad et Swarm
- Il est compatible avec de nombreux Network Overlay
- Contrairement à ce que l'on peut penser...il est extremement simple a configurer
- Le déploiement et la maintenance d'un cluster de conteneurs est quasi automatisée


## Ce qu'il nous faut
Dans ce labo, je vais avoir besoin de plusieurs serveurs : 

- Docker Private Registry (au bureau je suis sur Nexus, mais ca fonctionne avec n'importe lequel...pour peu qu'il y ait de lauthentification par mot de passe)
- Plusieurs serveurs Kubernetes, 1 maître et plusieurs minions
- Docker
- Weave Net

## Installation de Kubernetes
L'installation de l'outil en elle même est extrêmement simple, et se compose de 4 commandes parmi les suivantes : 

	cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
	       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	EOF
	yum install -y docker kubelet kubeadm kubectl kubernetes-cni
	systemctl enable docker && systemctl start docker
	systemctl enable kubelet && systemctl start kubelet

Par contre, ce que je ne peux que vous suggérer, c'est de désinstaller **firewalld** et de le remplacer par iptables : 

	systemct stop firewall; systemctl disable firewalld; yum remove -y firewall; yum install -y iptable*; systemctl enable iptables; systemctl start iptables

Je vous suggère aussi de vérifier toutes les règles d'iptables et de supprimer celles qui commencent par *reject* (normalement, vous devrier en voir deux - une pour **INPUT** et la seconde pour **FORWARD**), cela évitera que vos nodes se fassent refoulent par le master au moment du **kubeadm join**.

Une fois ceci effectué, on peut passer à l'étape suivante...

## Initialisation du cluster
Encore une fois, étape hautement...compliquée : une seule commande suffit sur le master et sur les nodes.

	`kubeadm init --api-advertise-addresses <IP>`

Une fois le contrôle récupéré sur le serveur maître, vous pouvez copier la commande `kubeadm join --token=<token> <IP du master>` sur chaque node.

## Installation du Network Overlay
L'ajout d'un **Network Overlay** est tout aussi simple, chaque composant étant un add-on, une simple commande permet de l'installer et le système de **Kubernetes** effectuera un scalling automatique en fonction du nombre de Node qui est enregistré.
Dans notre cas, pour installer **Weave Net**, il faut juste exécuter la commande suivante : `kubectl apply -f https://git.io/weave-kube`

## Troubleshooting
Avant d'aller plus loin, il est nécessaire de vérifier que chaque composant est bien déployé : 

A l'aide de la commande suivante : `kubectl get po -n kube-system` vous allez pouvoir obtenir les informations necessaire.

	NAMESPACE     NAME                                      READY     STATUS    RESTARTS   AGE       IP                NODE
	kube-system   po/dummy-2088944543-js4fg                 1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/etcd-kubernetes-0                      1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/kube-apiserver-kubernetes-0            1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/kube-controller-manager-kubernetes-0   1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/kube-discovery-1769846148-wxd60        1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/kube-dns-2924299975-66p06              4/4       Running   0          1h        10.40.0.1         kubernetes-0
	kube-system   po/kube-proxy-2jzmm                       1/1       Running   0          55m       188.226.140.115   kubernetes-0
	kube-system   po/kube-proxy-2vsmn                       1/1       Running   0          55m       188.226.140.112   kubernetes-2
	kube-system   po/kube-proxy-t2z0r                       1/1       Running   0          55m       188.226.140.113   kubernetes-1
	kube-system   po/kube-scheduler-kubernetes-0            1/1       Running   0          1h        188.226.140.115   kubernetes-0
	kube-system   po/weave-net-0lrg1                        2/2       Running   0          56m       188.226.140.113   kubernetes-1
	kube-system   po/weave-net-qrl14                        2/2       Running   0          56m       188.226.140.115   kubernetes-0
	kube-system   po/weave-net-xpdvj                        2/2       Running   0          56m       188.226.140.112   kubernetes-2

Dans notre cas, nous avons eu *kube-dns* en statut *ContainerCreation*...*Kube-dns* étant l'un des composant essentiels à toute installation de **Kubernetes**...il faut obligatoirement qu'il soit *up & running*.
Dans ce cas là, exécuter la commande suivante permet de contourner le problème : 
`kubectl -n kube-system get ds -l 'component=kube-proxy' -o json | jq '.items[0].spec.template.spec.containers[0].command |= .+ ["--proxy-mode=userspace"]' | kubectl apply -f - && kubectl -n kube-system delete pods -l 'component=kube-proxy'`

## Configuration de Docker
Le *Docker* livré avec *Kubernetes* est installé de manière assez...hybride : 

- le fichier docker dans /etc/sysconfig gère aussi bien les variables d'environement Proxy/No Proxy ainsi que les Insecure-registry
- Le fichier docker.service dans /lib/systemd/system gère tout le service docker.

En gros, lorsque vous ferez `docker info` après le démarrage de *docker* et qu'il vous manque des données...insérées dans un des deux fichiers, il faudra éditer l'autre pour ajouter ces mêmes infos.

Une fois **Docker** configuré, connectez-vous au *docker registry* avec `docker login <serveur>`

Vous n'aurez pas à vous préoccuper de la configuration de **Docker** par la suite, tout est géré par **Kubernetes**.

## Kubernetes et les Docker Private Registry
Afin d'éviter que **Kubernetes** n'aille piocher sur les registres *exotiques*, tels que **docker hub**, **Red Hat** ou bien **Google**, il faut créer une clé secrète qui fera le lien entre les descriptions de composants que nous allons lui fournir et notre propre registre, pour cela il faut exécuter la commande suivante : 

`kubectl create secret odcker-registry <name> --docker-server=<server> --docker-username=<login> --docker-password=<password> --docker-email=<email>`

Ne surtout pas oublier le flag **-docker-server**, autrement kubernetes continuera a chercher sur les autres registry et non sur le notre.

## Le déploiement
Effectivement...après avoir passé *un peu* de temps à configurer **Kubernetes**, il va falloir passer à l'application pratique...Certes, il existe sur Internet de nombreuses applications pré-configurées (ce qui peut s'avérer pratique). Mais le but de cet article est justement de comprendre comment **kubernetes** fonctionne.
Et non, de chercher à comprendre comment d'autres applications démarrées à l'aide de **kubernetes** se comportent.

### Description des composants : 

- Pods : Ce sont les conteneurs, et peuvent être créés de plusieurs manières différentes
- Deployment : Permettent de gérer le déploiement automatique des pods en cas de disparition de celui-ci
- Service : Gère l'exposition (et l'accessibilité) d'un pod/deployment au niveau du cluster
- Replication Controler : Gère la scallabilité du pod

### Définition des composants à déployer
Voici ce que nous allons utiliser : 

- Un service
- Deux Pods

	apiVersion: v1
	kind: Service
	metadata:
	  name: default-subdomain
	spec:
	  selector:
	    name: busybox
	  clusterIP: None
	  ports:
	    - name: foo # Actually, no port is needed.
	      port: 1234 
	      targetPort: 1234
	---
	apiVersion: v1
	kind: Pod
	metadata:
	  name: busybox1
	  labels:
	    name: busybox
	spec:
	  hostname: busybox-1
	  subdomain: default-subdomain
	  containers:
	  - image: busybox
	    command:
	      - sleep
	      - "3600"
	    name: busybox
	---
	apiVersion: v1
	kind: Pod
	metadata:
	  name: busybox2
	  labels:
	    name: busybox
	spec:
	  hostname: busybox-2
	  subdomain: default-subdomain
	  containers:
	  - image: busybox
	    command:
	      - sleep
	      - "3600"
	    name: busybox

On peut voir les pods *up & running* dans **kubernetes** : 

	NAMESPACE     NAME                                      READY     STATUS    RESTARTS   AGE       IP                NODE
	default       po/busybox                                1/1       Running   0          48m       10.32.0.3         kubernetes-1
	default       po/busybox1                               1/1       Running   0          49m       10.32.0.2         kubernetes-1
	default       po/busybox2                               1/1       Running   0          49m       10.36.0.1         kubernetes-2

Pour vérifier qu'ils sont bien joignables par chacun, rien de plus simple : 

	kubectl exec -ti busybox1 -- ping busybox-2.default-subdomain.default.svc.cluster.local
	PING busybox-2.default-subdomain.default.svc.cluster.local (10.36.0.1): 56 data bytes
	64 bytes from 10.36.0.1: seq=0 ttl=64 time=5.298 ms
	64 bytes from 10.36.0.1: seq=1 ttl=64 time=1.555 ms
	64 bytes from 10.36.0.1: seq=2 ttl=64 time=0.894 ms

A partir de là, on peut tout à fait voir ce que l'on créer comme infrastructure avec chaque pod communiquant avec les autre grâce à la résolution de noms via kube-dns.