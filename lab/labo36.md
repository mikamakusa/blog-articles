+++
date = "2020-08-26T15:29:49+02:00"
draft = "false"
title = "Le Labo #30 | Ejabberd et l'autoscaling sur AWS"

+++

Il y a quelque années, lorsque j'étais chez **ArubaCloud** (Entre 2012 et 2016), nous utilisions **jabber** pour communiquer entre toutes les équipes (en Italie et en France), mais je n'avais jamais pu me faire les mains sur le serveur (afin de savoir comment ça se configure).  
Je vais vous avouer que lorsque l'on m'a demandé si je connaissais le protocole **XMPP**, j'avais un doute mais je me suis renseigné avant de répondre.  
Et c'est à ce moment que ça m'est revenu (en parcourant les résultats proposés par **Google**) : **Jabber** est basé sur le protocole **XMPP**.  
En parcourant les différents résultats, j'ai pu découvrir quelques outils sur le site [xmpp.org]('https://xmpp.org/software/servers.html') :  
- [ejabberd]('https://www.ejabberd.im/') : opensource et complètement gratuit avec une documentation relativement complète - disponible pour windows, Linux et MacOS  
- [Iot Broker]('https://waher.se/Broker.md') : Uniquement sur Windows  
- [Isode M-Link]('https://isode.com/products/m-link.html') : Linux et Windows  
- [Prosody]('http://prosody.im/') : Pour BSD, Linux et MacOS  
- [Tigase]('https://tigase.net/xmpp-server') : Pour Linux, MacOS, Solaris et Windows  

Seul **ejabberd** est gratuit et, bien évidemment, c'est grace à celui-là que je peux vous proposer cet article.  

# L'idée
Après avoir lu un bonne partie de la documentation, je peux partir sur le constat suivant : **ejabberd est stable, très stable, et peut héberger près de deux millions d'utilisateurs du moment qu'il ne tourne pas sur un serveur asthmatique**. Par contre, aucune indication de ce peut être un **serveur asthmatique** (c'est de moi, ça ne vient pas du tout de la documentation officielle).  
J'imagine que tout dépend de l'activité des utilisateurs sur le serveur ainsi que des modules actifs.  
Il y a toute une liste de Use Cases possible sur cette [page]('https://docs.ejabberd.im/use-cases/'), ce qu'il laisse imaginer des recommandations techniques relativement élevées en fonction des cas.  
Si vous avez eu, tout comme moi, l'idée de parcourir la documentation, vous avez certainement remarqué la partie [clustering]('https://docs.ejabberd.im/admin/guide/clustering/') qui reste tout de même limitée car non automatique.  

# Du côté de l'infra
Une fois l'idée de départ prise en compte...autant aller plus loin en se focalisant sur l'infrastructure.  
J'exposerais les lignes directrices (en schematisant) afin de rester totalement générique (et éviter un effet de *vendor-locking*).

![img1](/images/xmpp_schema.png)  

En fait, il y aura deux groupes de serveurs :  
- Le master  
idéalement, il faudrait en avoir plusieurs afin d'éviter tout point de blocage si le master venait à tomber. Pour le moment, je n'ai pas encore vu dans la documentation si il était possible de *scaler* le serveur maitre derrière un *load balancer*.  
C'est un point de détail à creuser par la suite.  
- Les workers  
Dans le but de garantir la stabilité du serveur **ejabberd**, j'ai prévu d'utiliser la fonction d'autoscaling sur les workers templatisé.  

Le template en question contient la configuration nécessaire à :  
- Le téléchargement et l'installation de **ejabberd**,  
- La récupération du cookie provenant du master,  
- La modification du fichier de configuration **/opt/ejabberd/conf/ejabberdctl.cfg**,  
- Le démarrage du service **ejabberd**,  
- L'action *join_cluster*

# Un peu de technique
J'ai pu remarquer que le principe d'autoscaling fonctionne sensible de la même manière sur les différent Cloud Providers du marché (les trois principaux, à savoir **AWS**, **GCP** et **Azure**). En gros, cela s'appuie sur les trois actions suivantes :  
- Création d'un script d'installation/configuration du template.  
Ca peut être un *cloud init* tout simple ou l'initilisation d'une configuration via **Ansible** (ou tout autre outil de *Configuration Management*),  
- Création du template.  
Il s'agit de définir les élements de configuration de l'instance ( **CPU**, **RAM**, **DISK**, **OS**, **RESEAU** ainsi que le script d'installation sus-mentionné),  
- Création du groupe d'instance à partir du template (incluant bien-sur la fonction d'autoscaling ainsi que la définition des métriques servant a actionner les mécanismes cités précédemment).  

## Le script d'installation
Contrairement à **GCP** et **Azure**, sur **AWS** nous avons la possibilité de configurer :  
- Des **Launch Templates** ou,  
- Des **Launch Configurations**.

Le principe est sensiblement identique à une exception : Les *user data* (ou *cloud-init*) ne fonctionnent pas avec les **launch configurations**...bien que les deux puissent être utilisés avec les **Auto Scaling Groups**.

Pour les besoins de cet article, la configuration est réduite au strict minimum, c'est à dire :  
- Pas de module superflu,  
- Pas de création d'utilisateur autre que l'administrateur.  

Le script de configuration ressemble à cela :  

### Sur le master
```bash
#!/bin/bash
sudo wget https://www.process-one.net/downloads/downloads-action.php?file=/20.07/ejabberd_20.07-0_amd64.deb -O /opt/ejabberd_20.07-0_amd64.deb
sudo dpkg -i /opt/ejabberd_20.07-0_amd64.deb
NODE=$(hostname -s)
sudo sed -i 's/#ERLANG_NODE=ejabberd@localhost/ERLANG_NODE=ejabberd@'$NODE'/g' /opt/ejabberd/conf/ejabberdctl.cfg
sudo sed -i 's/#FIREWALL_WINDOW=/FIREWALL_WINDOW=4200-4300/g' /opt/ejabberd/conf/ejabberdctl.cfg
sudo cp /opt/ejabberd-20.07/bin/ejabberd.service /lib/systemd/system
sudo systemctl daemon-reload
sudo systemctl start ejabberd
```

### Sur les workers
La configuration sur les worker est sensiblement identique à un détail près : le cookie.  
En effet, afin de permettre la connexion des workers sur le master, il est nécessaire de copier le cookie du master sur le worker...exactement au même endroit.  
```bash
#!/bin/bash
sudo wget https://www.process-one.net/downloads/downloads-action.php?file=/20.07/ejabberd_20.07-0_amd64.deb -O /opt/ejabberd_20.07-0_amd64.deb
sudo dpkg -i /opt/ejabberd_20.07-0_amd64.deb
NODE=$(hostname -s)
sudo sed -i 's/#ERLANG_NODE=ejabberd@localhost/ERLANG_NODE=ejabberd@'$NODE'/g' /opt/ejabberd/conf/ejabberdctl.cfg
sudo sed -i 's/#FIREWALL_WINDOW=/FIREWALL_WINDOW=4200-4300/g' /opt/ejabberd/conf/ejabberdctl.cfg
sudo cp /opt/ejabberd-20.07/bin/ejabberd.service /lib/systemd/system
sudo scp root@master:/opt/ejabberd/.erlang.cookie /opt/ejabberd/.erlang.cookie
sudo chown ejabberd:ejabberd /opt/ejabberd/.erlang.cookie
sudo chmod 400 /opt/ejabberd/.erlang.cookie
sudo systemctl daemon-reload
sudo systemctl start ejabberd
sudo /opt/ejabberd-20.07/bin/ejabberdctl --no-timeout join_cluster ejabberd@'MASTER'
```

## Le template
Je ne vais pas vous apprendre ce qu'est un template ni ce qu'il doit contenir.  
Que ce soit sur **AWS**, **Azure** ou **GCP**, vous pouvez le créer avec **Terraform**, via la console ou avec **AWS/Cloudformation**, **GCP/Cloud Deployment** ou bien **Azure/Blueprints**...Mais concentrons nous surtout sur **AWS**.

### Avec Terraform
Le provider **AWS** pour **Terraform** propose la ressource *aws_launch_template* avec la documentation extremement bien détaillée, on peut y retrouver les éléments suivants :  
- block_device_mappings,  
- capacity_reservation_specification,  
- cpu_options,  
- credit_specification,  
- elastic_gpu_specifications,  
- elastic_inference_calculator,  
- iam_instance_profile,  
- instance_market_options,  
- license_specification,  
- metadata_options,  
- monitoring,  
- network_interface,  
- placement,  
- tag_specifications  

En plus des classique *user_data*, *ram_disk_id*, *vpc_security_group_ids*, *key_name*, *kernel_id*, *instance_type*, *image_id*, *ebs_optimized*, *disable_api_termination* et *name*.  

En gros, une ressource *aws_launch_template* classique peut ressembler à cela :  

```hcl
resource "aws_launch_template" "launch_template" {
  count                     = length(var.launch_template)
  name                      = lookup(var.launch_template[count.index], "name")
  description               = lookup(var.launch_template[count.index], "description")
  default_version           = lookup(var.launch_template[count.index], "default_version")
  disable_api_termination   = lookup(var.launch_template[count.index], "disable_api_termination")
  image_id                  = lookup(var.launch_template[count.index], "image_id")
  instance_type             = lookup(var.launch_template[count.index], "instance_type")
  user_data                 = filebase64(join("/", [path.root, "scripts", lookup(var.launch_template[count.index], "user_data")]))

  dynamic "iam_instance_profile" {
    for_each = lookup(var.launch_template[count.index], "iam_instance_profile")
    content {
      arn = element(var.arn, lookup(iam_instance_profile.value, "arn_id"))
    }
  }

  dynamic "monitoring" {
    for_each = lookup(var.launch_template[count.index], "monitoring")
    content {
      enabled = lookup(monitoring.value, "monitoring")
    }
  }
}
```

### Avec CloudFormation
Il semblerait que toutes les options disponible avec **Terraform** le soient également avec **CloudFormation**. Donc, une ressource *launch_template* pour **CloudFormation** peut ressembler à ce qui suit :  

```yaml
MyLaunchTemplate:
  Type: AWS::EC2::LaunchTemplate
  Properties: 
    InstanceType: c4.large
    DisableApiTermination: 'true'
    KeyName: MyKeyPair
    ImageId: ami-04d5cc9b88example
    IamInstanceProfile:
      Arn:
        Fn::GetAtt:
          - MyIamInstanceProfile
          - Arn
    SecurityGroupIds:
      - sg-083cd3bfb8example
    LaunchTemplateName: MyLaunchTemplate
```

## Le groupe d'instances
Une fois le script d'installation ainsi que le template créés, on peut passer la dernière partie : Le groupe d'instance sur lequel se basera toute la mécanique d'autoscaling (ainsi que les métriques sur lesquelles l'autoscaling se basera pour fonctionner).

Il n'y a pas de méthode plus simple qu'une autre, encore une fois tout dépendra de votre sensibilité...à savoir si vous êtes **pro-CloudFormation** ou **Cloud Agnostique** (donc plus enclin à utiliser **Terraform** car, tout comme moi, vous travaillez sur plusieurs Cloud Providers différents).  

De la même manière que pour le **launch_template**, notre *instance group* sera le plus simple possible, je n'utiliserais absolument pas les options suivantes:  
- mixed_instance_policy : il s'agit de la possibilité d'utiliser aussi bien des instances *spot* que des instances dites *classique*,  
- placement_group : en gros, les instances ne seront pas liées à un hardware spécifique dans la zone sélectionnée,  
- load_balancers : pour ce lab, nous n'en avons pas besoin,
- launch_configuration (je ne pense pas avoir besoin de préciser pourquoi).

### Avec Terraform
```hcl
resource "aws_autoscaling_group" "autoscaling_group" {
  availability_zones = ["us-east-1a"]
  desired_capacity   = 2
  max_size           = 10
  min_size           = 1
  default_cooldown   = 30
  health_check_type  = "EC2"

  launch_template {
    id      = aws_launch_template.foobar.id
    version = "$Latest"
  }
}
```

Afin de mieux définir l'autoscaling, il est également bon de préciser quelles métriques utiliser...et par conséquent, d'ajouter autant de resources **aws_autoscaling_policy** que de métriques à surveiller.  
```hcl
resource "aws_autoscaling_policy" "policy" {
  name                   = "ejabberd-change-in-capacity"
  scaling_adjustment     = 4
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.autoscaling_group.name
}
```

### Avec CloudFormation
Sans surprise, le template de configuration d'un *autoscaling_group* à l'aide de **CloudFormation** est identique à ce que l'on peut retrouver avec **Terraform** (j'y ajoute également les *autoscaling_policy*) :  
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  LatestAmiId:
    Description: Region specific image from the Parameter Store
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
  myLaunchTemplateVersionNumber:
    Type: String
  Subnets:
    Type: CommaDelimitedList
Resources:
  myLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties: 
      LaunchTemplateData: 
        ImageId: !Ref LatestAmiId
        InstanceType: t3.micro
  myASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: '1'
      MaxSize: '10'
      DesiredCapacity: '2'
      LaunchTemplate:
        LaunchTemplateId: !Ref myLaunchTemplate
        Version: !Ref myLaunchTemplateVersionNumber
      VPCZoneIdentifier: !Ref Subnets
  myCPUPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref myASG
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: !Ref CPUPolicyTargetValue
```

# Et une fois déployé ?
Une fois que tout est déployé, autant aller vérifier que tout est *sous contrôle*.  
En fait, **ejabberd** propose une interface d'administration relativement bien conçue, assez *eye candy* (je trouve) et très claire dans laquelle on peut trouver toute les informations de configurations ainsi que, chose très importante dans le cas présenté ici, tous les workers connectés au master **ejabberd**.  
L'interface est accessible de cette manière : http://[MASTER_IP]:5280/admin, le login/password est celui qui a été défini lors de l'installation du master.

# Et si on veut aller plus loin ?
Bien que j'ai poussé un peu plus loin un cas d'utilisation classique de **ejabberd** (rare sont les cas d'utilisation classique utilisant des fonctions d'autoscaling), il est possible d'aller encore plus loin et ce sur plusieurs niveaux :  
- Réseau  
- Sécurité  
- Configuration  

En ce qui concerne le réseau, et que je n'ai pas expliqué ici, j'avais créé un **VPC** avec deux réseaux - privé et public - et je pense que vous avez déjà deviné quel type de réseau était réservé a quel type de serveur.  
Pour tout ce qui est Sécurité, je fais bien entendu référence au fait que **ejabberd** peut se connecter à un serveur **LDAP** pour tout ce qui est gestion des utilisateurs...mais il n'y a pas que ça et tout est documenté sur le site officiel (même si c'est fait de façon assez sommaire et peu détaillé).  
Pour le reste de la configuration, c'est également le cas : documenté de manière basique sur le site officiel mais en creusant un peu sur Internet on trouve des informations ou des tutoriels.
