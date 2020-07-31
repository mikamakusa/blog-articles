+++
date = "2020-07-31T21:00:00+02:00"
draft = false
title = "La certification Kubernetes Administrator - CKA"
+++

## Intro
Depuis le début de l'année, j'enchaine les certifications sur des technologies avec lesquelles j'ai beaucoup travaillé, mention spéciale à Terraform et Azure...Mais il y en a une avec laquelle j'ai travaillé il y a quelques années et qui m'a vraiment donné des sueurs froides, il s'agit de **Kubernetes**.  
Ma compagne s'en souvient encore, je n'arrêtais pas d'en parler parce que je n'arrivais pas à faire fonctionner un déploiement, un service, ou n'importe quel autre *objet* disponible dans l'API.  
Et pourtant, avec le recul et pas mal de préparation (je prépare la certification depuis 1 mois et demi, environ), je dois dire que tout semble super simple.

Le but de cet article est de partager mon expérience ainsi que quelques astuces pour être vraiment à l'aise sur le sujet.

## Les MOOCS
J'en ai croisé beaucoup sur la plateforme **Udemy**, très bien conçu pour certains, avec des labs à base de question et certains autres un peu trop en mode survol sans pour autant creuser un minimum...et il y a **killer.sh** (je reviendrais un peu plus tard sur...cette expérience).

**Sur Udemy** :  
* **Certified Kubernetes Administrator with Pratice Tests** de **KodeKloud** : Bien qu'il y en ait d'autres qui creusent probablement beaucoup plus loin dans les fonctionnalités de **Kubernetes**, celui-ci propose un lab à chaque fin de section. La difficulté est progressive et ce n'est pas seulement un petit quizz pour vérifier les connaissances engrangées durant le cours (comme l'on peut trouver sur d'autres cours).  
* **Kubernetes On the Cloud & The CNCF CKA Certification** de **Loony Corn** : Voici un example de cours sur **Kubernetes** préparant à la certification sans même proposer de lab pour se *mettre les mains dans le cambouis*. Environ 9h de vidéos (soit moitié moins qu'avec **KodeKloud**), un quizz à chaque fin de section et quelques infos sur l'examen en lui même. En gros : Peu de contenu (même si ça aborde **Kubernetes** sur **AWS** et **Azure**...A croire que **GCP** ne le propose pas).  
* **Learn DevOps: The Complete Kubernetes Course** de **Edward Viaene**: Cours vraiment très complet, à peu de choses près aussi long que de celui de **KodeKloud** sans les Labs (encore une fois), mais c'est une première partie parce qu'il propose également une seconde partie articulée autour d'usages plus avancés avec **spinnaker**, **CSI**, **automatisation**, utilisation de **terraform** pour déployer sur **kubernetes** (et j'oublie certainement pas mal de choses).

**Sur MyMooC.com** :  
J'en ai trouvé 6 :  
- Introduction to **Kubernetes**  
- Fundamentals of containers, **Kubernetes** and **RedHat OpenShift**  
- Scalable Microservices with **Kubernetes**  
- Getting started with **Google Kubernetes Engine**  
- **IBM Cloud**: deploying microservices with **Kubernetes**  
- **IBM Cloud Private**: deploying microservices with **Kubernetes**  
- Administrer des conteneurs dans Azure

**Sur CloudGuru** :  
Le contenu est assez pauvre, surtout si on compare avec le cours de chez **KodeKloud** sur **Udemy** (même le *Deep Dive*).  
- **Kubernetes** Deep Dive  
- **Google Kubernetes Engine**

**Sur Whizlabs** :  
Les cours sont aussi complets que celui que j'ai suivi avec **KodeKloud** (les Labs/HandsOn en moins).
- Certified Kubernetes Administrator  
- Certified Kubernetes Application Developper  
- Learn Kubernetes with AWS and Docker

**Sur Coursera** :   
Les cours sont nombreux, que ce soit du **fundamentals** / **getting started**, des cours orientés **Workloads** ou **Architecting**...et ceci en plusieurs langues.

## Les Labs
### Killer.sh
Je l'avais évoqué plus haut parce qu'il est important de savoir qu'il existe des simulateurs d'examens pour les certifications **CKA** et **CKAD**.  
Mon retour d'expérience sur **killer.sh** est assez mitigé...autant c'est un très bon simulateur avec des questions relativement simple et d'autres bien compliquées...autant j'ai aussi l'impression que même en reprennant toutes les réponses, on peut arriver à obtenir un score ridiculement petit (et je n'ai pas compris pourquoi).
Le type de question est assez varié :   
- Arrêt du scheduler, démarrage d'un pod puis modification du pod pour qu'il démarre sur le master. Redémarrage du scheduler et création d'un nouveau pod.  
- Création et utilisation d'un scheduler.  
- Récupération des metadata telles que **UID** ou **AGE** du pod ou des ressources nécessaire au pod (afin d'observer si le pod peut s'arrêter si le *worker* n'a plus de resources)  
- Les **Job** et **Cronjob**  
- Les **DaemonSet**  
- Les **Pod** avec plusieurs conteneurs et volumes partagés  
- La commande **kubectl get events** dans le but d'observer les événements survenu sur le cluster (en fonction d'un namespace)  
- Les resources contenues dans un namespace (via la commande `kubectl api-resources --namespaced=true -o name`)  
- le nombre de resources par namespace
```
RESOURCES=configmaps,endpoints,events,limitranges,persistentvolumeclaims,pods,podtemplates,replicationcontrollers,resourcequotas,secrets,serviceaccounts,services,controllerrevisions.apps,daemonsets.apps,deployments.apps,replicasets.apps,statefulsets.apps,horizontalpodautoscalers.autoscaling,cronjobs.batch,jobs.batch,leases.coordination.k8s.io,events.events.k8s.io,ingresses.extensions,ingresses.networking.k8s.io,networkpolicies.networking.k8s.io,poddisruptionbudgets.policy,rolebindings.rbac.authorization.k8s.io,roles.rbac.authorization.k8s.io

kubectl get $RESOURCES -A -o jsonpath="{range .items[*]}{.metadata.namespace}{'\n'}" | sort | grep kube-system | wc -l
```
- Il y a également des questions relative aux snapshots **ETCD** ou à **kubelet**

### KodeKloud
Les labs de **Kodekloud** sont assez nombreux, il y a 12 chaptitres et au moins 5 labs par chapitre ainsi que 3 examens tests avec solution vidéo pour les deux derniers. Je dirais sans aucune hésitation que ces labs sont plus facile que le simulateur de **killer.sh**.

### Katacoda
Il y a bien évidemment [**Katacoda**](https://www.katacoda.com/courses/kubernetes) qui propose de nombreux scénarii très utile pour se familiariser avec **Kubernetes** mais, à mon avis, c'est beaucoup trop *clicodrome* pour vraiment être utile dans le but de passer une des deux certifications **Kubernetes**.

### D'autres Environnements types "Lab"
Il y en a deux autres recensés sur le blog de [Pierre-Mickaël Chancrin](https://piermick.wordpress.com/) :  
  - https://github.com/hub-kubernetes/kubernetes-CKA : propose principalement des *use-case* à tester dans un lab que vous auriez monté au préalable.  
  - https://github.com/arush-sal/cka-practice-environment : C'est un simulateur d'examen en mode **Bring Your Own Cluster**...donc le même type que celui cité précédemment.  
Si vous optez pour ce type d'approche, je ne peux que vous recommander d'utiliser **Vagrant** / **VirtualBox** pour monter votre propre cluster Kubernetes.

## Le contenu couvert
Comme toujours : très exhaustif.  
- Les commandes de base **kubernetes**  
- L'unité de base : le **pod** ainsi que toutes les possibilités (*tolerations*, *volumes*, *nodeSelector*, *serviceAccount*, *multicontainer*, *initcontainer*, etc...)  
- Les **Deployments** et **Services** (ainsi que comment créer un **DaemonSet** facilement)  
- Les **Certificats**, les **roles**, **rolebinding** et tout ce qui est en rapport avec la gestion des droits utilisateurs (*RBAC*)  
- La sécurité (aussi bien relative aux pods qu'au réseau)  

Les thématiques sont nombreuses, donc ne vous en faites pas si je n'ai pas tout mis dans l'article...tout est visible sur le site du **CNCF**.

## Les livres
Il y a de nombreux livres disponibles : 
- **Kubernetes - Maîtrisez l'orchestrateur des infrastructures du futur** - De Kelsey Hightower et Brendan Burns  
- **Kubernetes - Gérez la plateforme de déploiement de vos applications conteneurisées** - De Yannig Perré  
- **Kubernetes: Up and Running** - de Brendan Burns  
- **The Kubernetes Book** - De Nigel Poulton  
- **Kubernetes: A simple Guide to Master Kubernetes for Beginners and Advanced Users** - De Brian Docker  
- **Understaning Kubernetes in a visual Way** - de Aurelie Vache  
- **Kubernetes Administration: A hands-On Guide to Certified Kubernetes Administrator Exam** - De Swapnil Jain  
- **Kubernetes Overview - Prepare CKA & CKAD Certifications** - De Philippe Martin  

J'avoue ne pas les avoir tous lu...Juste trois d'entre eux (les trois derniers).  
Ils sont tous très complets et j'ai une préférence pour celui d'**Aurelie Vache**. Il est présenté sous forme de Sketch Book.  
Je ne vais pas critiquer les dessins, je ne fais pas mieux de mon côté...Mais au moins l'information présentée est aérée, claire et précise.

Celui de **Swapnil Jain**  donne énormément d'info et propose des labs à faire soi-même.  

Le dernier s'est arrêté à la version **1.15** de **Kubernetes** et semble donc un peu ancien (mi Juin 2019) mais reste très utile.

## Avant les examens
### Renseignements
Pas de panique, des REX sont disponibles partout sur internet :  
- [Zenika](https://blog.zenika.com/2018/12/18/certification-kubernetes-ils-passent-la-cka-et-vous-disent-tout/)  
- Medium :  
  - https://medium.com/@ContinoHQ/the-ultimate-guide-to-passing-the-cka-exam-1ee8c0fd44cd  
  - https://medium.com/platformer-blog/how-i-passed-the-cka-certified-kubernetes-administrator-exam-8943aa24d71d  
  - https://medium.com/@danielmittelman1/the-comprehensive-guide-to-passing-the-certified-kubernetes-administrator-cka-exam-ba7ef2cec27f  
  - https://medium.com/faun/certified-kubernetes-administrator-cka-ultimate-guide-238710f9ba73  
- [Pierre-Mickael Chancrin](https://piermick.wordpress.com/) (de Ippon Technologies)

Et si vous cherchez encore des informations, n'hésitez pas à soliciter vos collègues (il y en a forcément un qui l'a passé).

### Révisions et entrainements
Je vais forcément me répéter par rapport aux articles relatifs aux certifications que j'ai passé précédemment :   
- [Google Associate Cloud Egineer](http://www.ageekslab.com/art11/)  
- [Hashicorp Terraform Associate](http://www.ageekslab.com/art14/)  
- [Azure Solutions Architect Expert](http://www.ageekslab.com/art10/)  
- AWS Solutions Architect Associate (pas d'article pour celui-là, désolé)

Soyez sérieux lors de vos révisions...d'autant plus que ce n'est **absolument pas** un examen portant sur la théorie autour de **Kubernetes** mais uniquement sur la pratique. L'examen vous demandera une bonne dose de connaissances bien que vous ayez accès à l'intégralité de la documentation officielle (sinon, c'est absolument infaisable à moins d'être atteint d'hypermnésie), il s'agit surtout de savoir quoi utiliser, quand l'utiliser et le plus efficacement possible.

### Réservation
Ca se passe [ici](https://www.cncf.io/certification/cka/), le site est très bien conçu contrairement a *WebAssessor* (pour les certifications **GCP**) et vérifie que la plage horaire sélectionnée est bien libre lorsque vous effectuez votre choix.  
Comptez 300€ pour l'examen...ou moins si vous parvenez à trouver un voucher (par vos collègues ou bien **KodeKloud** en propose parfois).

## Pendant l'examen
Comme les autres examens, vous n'avez droit qu'a un seul navigateur...et deux onglets :  
- L'examen en lui même
- La documentation officielle (comme évoqué précédemment)

Vous disposez de trois heures, avez accès à plusieurs clusters de configuration différentes (nombre de worker, CNI différent).  
Chaque tache doit être effectuée sur un contexte donné (vous allez user et abuser de la commande `kubectl config use-context [NAME]`), par chance, les taches sont groupées par contexte.

L'interface est très *eye-candy*, très lisible (ce qui est un très bon point).  
Comme pour tout autre examen, vous avez la possibilité de *marquer* une question pour y revenir plus tard...très utile si vous butez sur l'une d'entre elle trop longtemps.  

## Le final
Une fois l'examen terminé...encore trente-six heures (maximum) d'angoisse avant le verdict final.  
Une fois ce délai passé, vous saurez si vous pouvez sortir le champagne.

## Conclusion
Je vais forcément me répéter sur certains points, mais :  
- Restez sérieux dans vos révisions,  
- Lisez bien attentivement toutes les questions, certaines sont tordues, d’autres peuvent sembler relativement simple…Mais n’oubliez surtout pas que c’est une certif et rien n’est jamais simple dans ces cas là,  
- Faites un tour sur la documentation officielle, tout est super bien documenté,  
- Never stop learning
