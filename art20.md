+++
date = "2021-06-04T22:23:00+02:00"
draft = false
title = "Review de la conférence Paris Containers Day 2021"
+++

Le **Paris Containers Day** est un événement organisé par **Wescale** et **Publicis Sapient** (Anciennement **Xebia**) depuis 2016 et orienté Conteneurs et Orchestration.  
Malgré le contexte sanitaire mondial, et bien qu'ils l'aient, à l'origine, lancé en physique sur Paris. L'événement a finalement pris la forme virtuelle, à l'instar d'autres conférences telle que l'**AWS Re:Invent**.  
De plus, contrairement aux précédentes éditions qui se déroulaient sur une seule journée, celle-ci prend place sur deux après-midi.

Que retenir de cet ensemble de conférences ?  
Peu d'annonces, ce n'est vraiment pas le but recherché par **Wescale/Publicis Sapient** et par les différents intervenants. Par contre, de nombreux retour d'expériences sur des retours d’expériences autour de l’écosystème des conteneurs et de leurs orchestrateurs.

---

## Détection et Réaction aux menaces dans Kubernetes avec Falco
Présentée par **Thomas Labarussias**, ***Site Reliability Engineer*** chez **Qonto**, Contributeur pour **Falco** et créateur de **FalcoSidekick**.  
Cette session a pour but d'expliquer le fonctionnement de Falco, un projet relatif à la sécurité des runtimes et détecteur de menaces au sein de Kubernetes, et de présenter également FalcoSidekick et son fonctionnement.  
A savoir : Falco est un agent de détection des comportements *exotiques* au runtime créé par sysdig, il capture en temp réel les journaux, aussi bien ceux du kernel, des conteneurs ou du kubernetes.  
A l'aide d'un moteur de règles relativement simple (il s'agit d'instruction au format yaml) et paramétrable, Falco déclenche des alertes en fonction du contenu des journaux capturés...De plus, il est livré avec plus d'une centaine de règles prédéfinies.  
L'architecture par défaut de Falco ne prévoyant que cinq type de sorties différentes (stdout, file, gRPC, shell et http), FalcoSidekick permet d'étendre tout cela en ajoutant également une interface utilisateur.

N'hésitez pas à faire un tour sur le [site officiel](https://falco.org/)

---

## CloudRun: Un produit managé de Google Cloud compatiable Knative
Présentée par **Guillaume Blaquière** - ***Cloud Architect*** chez **Sfeir**.  
Le principe de CloudRun : Une solution Serverless pour l'hebergement de conteneurs contrairement à Cloud Functions ou à App Engine qui sont plutôt orientés code source. Ce qui permet une portabilité plus importante.  
Parmi les fonctionnalités de CloudRun, on retrouve une scalabilité automatique s'appuyant sur le nombre de requête que le conteneur doit traiter ainsi qu'un mode managé basé sur Knative et la possibilité de bénéficier de la puissance de Anthos sur les clusters Cloudrun.  
Il présente également des démonstrations de Knative sur AWS/EKS, GCP/GKE, sur GCP/HKE + le plugin CloudRun et, finalement, un CloudRun Managé.

Plus d'infos sur :
- [CloudRun](https://cloud.google.com/run)
- [Knative](https://knative.dev/)    Le Paris Container
- [Anthos](https://cloud.google.com/anthos)

---

## Et pour quelques runtimes de plus...
Présentée par Thomas Gérardin - Cloud Builder chez Wescale.
Cette intervention commence par un historique de la conteneurisation, depuis 1979 avec Chroot jusqu'à maintenant avec les différents runtimes ainsi que l'émergence d'un standard (Container Runtime Interface - CRI) proposé par l'OCI (Open Container Initiative).
Après un listing et quelques informations sur les des différents runtimes :
- **Docker** qui s'appuie sur le runtime **containerd**,
- **containerd** qui lui même s'appuie sur **runc**,
- **runc** ou **crun** qui sont des runtimes de bas niveau (proche du kernel et respectivement écrit en Go et en C),
- **rkt** (dont le projet a été arrêté en Février 2020),
- **cri-o** qui gère de nombreuses fonctionnalités, dont les images, la sécurité, les métriques, le système de fichiers, etc...
- **Kata-container** qui s'engage vers une nouvelle voie, celle des micro-VM (qui met l'accent sur une isolation plus profonde).

Il nous rappelle que les runtimes apportent différentes fonctionnalités selon leur niveau et leur "proximité" avec le kernel, décrit un exemple de stack avec un orchestrateur de conteneur lié à un ou plusieurs runtimes de conteneur (en fonction des fonctionnalités recherchées).

Plus d'infos sur :
- [Docker](https://www.docker.com/)
- [containerd](https://containerd.io/)
- [runc](https://github.com/opencontainers/runc)
- [cri-o](https://cri-o.io/)
- [kata-container](https://katacontainers.io/)

---

## Les conteneurs détournés
Présenté par **Emmanuel Pluot** et **Olivier Cloirec** - **Site Reliability Engineer** chez **Publicis Sapient**.  
Ce talk aborde des sujets assez variés tels que :
- Builder : ils se sont concentré sur un retour d'expérience avec le langage C++ et la compilation lors du *build* de l'image du conteneur.
- Interpreteur : cette fois-ci, ils se sont également concentrés sur un return d'expérience avec le langage Groovy lors du *build* du conteneur. Le principe est assez identique, un exécutable est généré dans le but de tester un cas pratique,
- nsenter: Il s'agit plus ou moins de hacker la machine hôte en volant les namespace d'un processus hôte grâce au PID d'un processus. Cela ressemble à une faille de sécurité et peut être mitigée via un contrôleur d'admission,
- LogForwarder : Cette partie nous présente comment rediriger les logs vers la sortie **stdout** à l'aide de **Fluentbit**,
- X11 / IDE : L'idée derrière ce détournement est de démarrer une application graphique dans un conteneur. La présentation fait la part belle à Visual Studio Code, mais au vu des packages installés, n'importe quelle application graphique peut être installable. Cette application des fonctionnalités relatives aux conteneurs fait partie des moins connues et des moins utiles.

---

## Créer et distribuer un plugin pour Kubernetes
Présenté Par Aurélie Vache - Cloud Developper chez StackLabs - et Gaëlle Acas - Site Reliability engineer chez Stack Labs.
Aurelie Vache, déjà très connue pour son travail de vulgarisation sur Kubernetes et l'écosystème des conteneurs en général, a commencé par expliquer le principe de **kubectl** et son fonctionnement *ultra simple*...par contre, l'ajout de plugins étant lié au core de **Kubernetes**, la *release* peut avoir un cycle relativement long.
Cependant, la partie la plus longue de l'intervention est liée à la création de plugin qui se décompose en :
- Tout d'abord créer un fichier nommé *kubectl-myplugin*,
- Le rendre exécutable,
- Le placer dans le **PATH** (/usr/local/bin, par exemple),
- Exécuter le plugin.

Le tout avec des langages relativement simple à utiliser tels que Python, Bash, Go, Rust ou Quarkus (Ceux avec lesquels ont peut coder un CLI)

Une fois le plugin développé et testé, il reste le packaging...et c'est à ce moment que **Krew**, le package manager de plugins à destination **Kubernetes**, entre en scène.

Plus d'infos sur [Krew](https://krew.sigs.k8s.io/)

---

## Des conteneurs dans des conteneurs (maritime)
Présentée par **Guilhem Lettron** - ***Site Relibaility Engineer Freelance*** - et **Barthelemy Vessemont** - ***Head of Infrastructure*** chez **Storelift**.
Le but de la présentation étant de créer  un magasin autonome sans caisse grâce à des conteneurs maritimes en gérant l'hétérogénéité de plusieurs détails tels que le matériel (Raspberry, Arduino, GPU pour le Machine Learning) et de logiciels conteneurisés.
De plus, via l'utilisation de nombreux framework de Computer Vision, le suivi des client à l'intérieur des magasins est tout à fait possible.

Dans ce but, ils se sont organisés de la manière suivante :
- Build des Conteneurs : Des images d'une taille d'environ dix à douze gigaoctects contenant tous les frameworks nécessaire et divisés en trois étapes. De plus, via l'utilisation de Docker-in-Docker, la charge de travail sur les machines est beaucoup plus configurable et moins impactant financièrement (surtout dans le cas d'utilisation du Cloud, d'instances disposant de ressources assez importantes).
- Pour la partie physique, ce fut plus compliqué car le choix technique au départ était d'utiliser des machines Raspberry...mais ils se sont rendus compte de quelques limitations technique (l'installation de package Python extrêmement chronophage) et se sont finalement dirigés vers des émulateurs ARM64 conteneurisés sur des serveurs X86 classiques.

La stack finale, au dela de Kubernetes, rassemble de nombreuses technologies telles que :
- Observabilité : Thanos, Prometheus, Fluentd, Fluentbit et cAdvisor,
- Réseau : Après avoir testé ROS, Flannel et autre, ils ont finalement opté pour le Host-Network à cause de certaines limitations techniques des plugins CNI mentionnés,
- Déploiement : Ils avaient commencé par tester Ansible et ont basculé sur Kubernetes pour son côté modulaire,


Plus d'infos sur [Boxy](https://www.getboxy.co/)

---

## K3S, le Kubernetes allégé construit mes images sans Docker
Présentée par Romain Boulanger - Cloud Engineer chez Sfeir.  
Il nous présente au cas pratique idéal d'un client souhaitant migrer ses applications sur Kubernetes tout en utilisant ses propres machines sans oublier l'accent sur la sécurité.
Créé par Rancher (déjà à l'oeuvre sur l'orchestrateur de cluster Kubernetes bien connu), K3S est un Kubernetes allégé de certaines fonctionnalités, d'installation très rapide et présenté comme un simple exécutable et...recommandé pour tout ce qui est IoT, Intégration Continue ou les architectures RAM.
K3S présenté comme "allégé" car les fonctionnalités suivantes diffèrent par rapport à Kubernetes :
- Legacy, alpha, non-default et les addons,
- Utilisatation de sqlite3,
- gestion automatique du TLS,
- Peu de dépendances
- Pas de port à exposer pour l'API.

De plus, Docker (tout comme ETCD) est optionnel.

Plus d'infos sur [K3S](https://k3s.io/)

---

Vous pouvez retrouver les sessions de la Paris Container Day édition 2021 (ainsi que les précédentes) [ici](https://www.youtube.com/channel/UC2CaqD0MyyDadpD1ipneW6w/videos)