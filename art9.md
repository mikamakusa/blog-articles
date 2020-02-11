+++
date = "2020-02-11T15:29:49+02:00"
draft = "false"
title = "AWS EC2 Image Builder"

+++

Le service **EC2** - Service phare parmi la galaxie de services proposés par AWS - bénéficie de tout un écosystème de fonctionnalités répondant à de nombreux besoins grâce au marketplace sur lequel nous pouvons retrouver des *AMI* - Amazon Machine Images - fonctionnelles proposées par des acteurs phare du marché, tels que :
- NetApp
- Dell
- Bitnami
- Citrix
- Plesk
- OpenVPN


Il y en a tellement que je pourrais pas tous les citer, mais sachez tout de même que si vous avez un besoin particulier, une AMI est disponible pour y répondre.
Néanmoins AWS propose aussi de rendre disponible sur la plateforme des images créées, à l’aide de **Packer**, en donnant des maux de tête a ceux qui ne sont pas familiers avec ce type d’outil.
Et je ne fais pas allusion aux cas de partage d’AMI entre plusieurs comptes... qui pourrait donner des sueurs froides à n’importe qui.


Le principe de Packer est de définir une AMI à l’aide d’un template comprenant :
- builders : Il s’agit de la création de l’image en elle-même en précisant toutes les informations relatives au compte ainsi que l’image.
- provisioner : Relatif à l’automatisation de l’installation des composants qui vont intégrer l’image, que ce soit des outils externe (Ansible, SaltStack, Chef, Puppet, etc...) ou via des scripts shell.
- post-processor : Les actions postérieures à la création de l’image, telles que Amazon Import qui nécessite la création, au préalable, d’une archive d’image et d’un bucket S3.

Toujours est-il qu’AWS a entendu notre appel et a annoncé, pendant leur conférence annuelle **re:Invent**, la release de **EC2 Image Builder**.


## EC2 Image Builder
Dans le but de simplifier et d’automatiser au maximum la création d’AMI, **EC2 Image Builder** propose plusieurs étapes allant de la sélection de composant (tels que Python, Nodejs, etc…) à la création de composants personnalisés (via un éditeur similaire à celui de Gitlab-CI) en passant par la gestion des licences externes (dans le cas de création d’AMI basée sur Windows Server).
Rappelons que lors du build de l’image, le système se charge de monter une petite infrastructure permettant la connexion à Internet (dans le but de télécharger diverses mises-à-jour) ou d’initier une connexion SSH (qui serait parfaite pour de la gestion de configuration à l’aide de logiciels tiers tels qu’Ansible).

Voilà pour la petite présentation... Maintenant passons à la prise en main du service.

#### Première étape
La première étape consiste à sélectionner le type d’AMI a partir de laquelle votre image customisée sera construite.

![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img1.png)




Bien que le choix des systèmes d’exploitation soit relativement limité du côté Linux - seule la version d’Amazon est disponible - pas d’inquiétude, celle-ci est un dérivé de CentOS/RedHat.
Concernant Windows, de nombreuses versions sont disponible[e][f]s, qu’elles incluent SQL Server ou bien l’interface graphique.


A savoir : Pour ceux qui n’ont jamais touché aux dernières versions de Windows Server (en fait, depuis la version 2016), l’interface graphique est optionnelle... mais la connexion à un serveur Windows se fera toujours via l’outil MSTSC (sur Windows) ou Remote Desktop Connexion (Sur Linux/MacOs).
Toujours pas de SSH en natif sur Windows, ce qui est bien dommage.


Ensuite il faudra sélectionner les composants que l’on peut également créer soi-même (je reviendrai dessus) :
![alt-text]()https://github.com/mikamakusa/blog-articles/blob/master/images/img2.png  



Les composants disponibles sont contextualisés. Ainsi vous n’aurez pas la même liste de composants selon le type de système d’exploitation désiré.
Pour le moment, très peu de composants préconfigurés sont disponibles et incluent :
* Powershell (Windows et Linux)
* Python 3 (linux)
* Php (Linux)
* Docker (Linux)
* JDK/JRE (Linux)
* Corretto (OpenJDK - Linux)
* Stig (Windows et Linux)


Vu le peu de composants disponibles à l’heure actuelle, AWS nous donne la possibilité d’en créer (voir ci-dessous) :
![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img3.png)  

Ici on peut définir le système d’exploitation pour lequel le composant sera disponible et, à l’aide de l’option KMS keys, la possibilité de chiffrer l’AMI.
![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img4.png)  

Cette seconde partie est peut être la plus intéressante car elle reprend le principe de Gitlab-CI avec les phases de pipeline (bien que la syntaxe soit assez différente, voir l’exemple ci-dessous)

```yaml
phases:
  -
    name: 'build'
    steps:
      -
        name: SampleS3Download
        action: S3Download
        timeoutSeconds: 60
        onFailure: Abort
        maxAttempts: 3
        inputs:
          -
            source: 's3://sample-bucket/sample1.ps1'
            destination: 'C:\Temp\sample1.ps1'
          -
            source: 's3://sample-bucket/sample2.ps1'
            destination: 'C:\Temp\sample2.ps1'
```




La documentation relative à la création de composants se trouve ici : https://docs.aws.amazon.com/imagebuilder/latest/userguide/managing-image-builder-console.html#image-builder-create-component


La dernière partie, relative aux tests à ajouter à la création de l’image, se déroule de la même manière que celle relative aux composants : des tests sont déjà disponibles [i][j]mais il est possible d’en créer soi même.


#### Seconde étape
Lors de cette étape, nous allons définir plusieurs éléments tels que :
- le type d’instance, le gabarit tel que présenté par AWS lors de la création d’instance (m2.small,c1.xlarge, g3s.xlarge, etc…)
- Le topic SNS sur lequel envoyer les alertes générées par EC2 Image Builder,
- VPC, Subnet et Security Groups,
- Le bucket de logs,
- La clé SSH qui servira à se connecter à la future instance,
- Le rôle IAM à associer au profil d’instance.


Cette étape est très important car elle va poser les bases d’une construction d’AMI sécurisée.

![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img5.png)
![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img6.png)  



#### Troisième étape
Lors de cette dernière étape (la quatrième n’est qu’un rappel de toutes les options sélectionnées auparavant), vous pourrez choisir les paramètres de distribution de l’AMI tels que le nom de sortie de l’AMI, les régions sur lesquelles vous souhaitez la rendre disponible ainsi que les comptes qui pourront y accéder.

![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img7.png)




Autre remarque importante : j’avais évoqué plus haut les difficultés qu’il pouvait y avoir à partager une AMI créée pour un compte particulier, bien que cette étape puisse simplifier en apparence, le partage d’AMI entre plusieurs comptes, le fait d’ajouter des comptes dans launch permissions ne sera pas suffisant pour que les AMI ainsi buildées soient disponibles pour chacun des comptes (mais je reviendrais dessus en fin d’article).


#### Dernière étape
En fait, de dernière étape, il ne s’agit que d’un rappel de toute la configuration définie lors des étapes précédente[k][l]s. De plus, il ne s’agit pas du démarrage d’une création d’AMI mais la création d’un pipeline.
Le pipeline en lui-même peut être démarré de plusieurs manières différentes (défini lors de la seconde étape) :
- manuellement
- Via une expression cron
- Via un “Job Scheduler”, il s’agit en fait d’une expression cron simplifiée au maximum (day, week, month, avec sélection du jour pour les deux derniers et de l’heure).


## Conclusion
EC2 Image Builder est une excellente idée (sur le papier), il pourrait remplacer Packer (un peu comme le fait CloudFormation pour Terraform) mais ne fonctionne que sur AWS (une bonne dose de vendor locking). Il est proposé dans toutes les régions mais il n’en est pas moins relativement limité.

- La création des AMI n’est pas automatique, ce n’est pas un handicap étant donné qu’il est possible de définir un job cron qui va s’en charger.
- Tout comme Packer, Il est soumis aux erreurs de typage et de syntaxe et, qui dit template dit versionning.
- Par contre, la validation du template se fait à chaque étape de création.
- Là où un template Packer peut être versionné, c’est impossible de le faire avec EC2 Image Builder (même pour les scripts de build de composants et de tests). Par conséquent, impossible de lier la création d’AMI à un event github/gitlab.
- Les logs d’erreurs ne sont pas disponible comme peuvent l’être ceux de Packer (voir ci-dessous), et le peu d’information disponible n’est pas assez explicite pour savoir comment débugger.

![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img8.png)




Dernier point sur lequel je devais revenir, le partage d’AMI entre compte n’est pas parfait. Si vous avez oublié de créer une clé KMS (Key Management Service), l’AMI ainsi créée ne sera visible que sur le compte avec lequel elle a été créée (voir ci-dessous) :

![alt-text](https://github.com/mikamakusa/blog-articles/blob/master/images/img9.png)




Et si vous souhaitez utiliser l’AMI avec un autoscalling group (par exemple), l’instance ne démarrera pas du tout (sans donner plus d’info dans CloudTrail/CloudWtach), la commande suivante permettra de résoudre ce problème :
```
aws kms create-grant \
--key-id arn:aws:kms:eu-west-1:xxxxxxxx:key/39aa32e1-13a9-4705-86a5-9fa802911edd \
--grantee-principal arn:aws:iam::xxxxxxxxxxx:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
--operations "Encrypt" "Decrypt" "ReEncryptFrom" "ReEncryptTo" "GenerateDataKey" "GenerateDataKeyWithoutPlaintext" "DescribeKey" "CreateGrant" \
--region eu-west-1
```
