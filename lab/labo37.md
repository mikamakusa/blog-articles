+++
date = "2020-09-22T15:29:49+02:00"  
draft = "false"
title = "Le Labo #31 | Serveur RDS et Autoscaling des Session Hosts"
+++

# A l'origine
A l'origine, le composant RDS s'appelait Terminal Services et était principalement destiné aux réseaux locaux ou régionaux offrant de meilleures latences que sur un réseau bien plus large.
Qu'est-ce que Terminal Server?  

Il s'agit d'un composant destiné à une architecture de client léger où les applications (incluant selon les cas le bureau Windows) sont rendues disponibles à l'aide d'un terminal client à distance. Ce type de technologie à évolué avec le temps et a tout d'abord été introduit sous Windows NT 4.0, amélioré sous Windows 2000 puis Windows 2003 pour devenir Remote Desktop Services dès Windows 2008.
  
Ceux qui ont travaillé principalement sur des parcs à base de Windows XP/7 ont forcément connu la version client MSTSC (pour Microsoft Terminal Service Client) qui permettait de prendre la main à distance sur n'importe quel PC d'un réseau local.
  
Mais peu connaissent et ont déployé la version serveur…dans le Cloud.
  
Pour l'anecdote, j'ai travaillé pendant quelques années pour un fournisseur de services Cloud qui proposait des templates à base de services Remote Desktop Services auxquels, une fois installés, il fallait ajouter les licences CALs (les fameuses licences utilisateurs) pour bénéficier d'un service RDS prêt à l'emploi (sans avoir à configurer profondément le service).
  
Dans cet article, nous allons déployer une infrastructure RDS en éclatant les composants entre les serveurs.

# Le besoin
L'idée derrière cet article est de proposer un service de bureau à distance qui s'adapte à la demande en fonction des ressources consommées…Tout simplement.
Pour cela, je vais devoir déployer plusieurs services :  
- **Active Directory**: Afin de bénéficier d'une gestion des utilisateurs totalement intégrée,  
- **DNS**: Certes ce sera installé en même temps que le service Active Directory, mais il va me permettre de gérer tout l'aspect adressage des ressources,  
- **Connection Broker**: C'est le composant le plus important, ou presque, car il gère une grande partie de l'aspect sécurité, haute disponibilité et rendu graphique,  
- **Licensing Server**: Ce composant peut s'installer sur n'importe quel serveur (y compris sur un serveur Active Directory, par exemple) et gère tout l'aspect licences utilisateur,  
- **Gateway**: C'est le serveur qui permet à des clients de pouvoir se connecter de manière sécurisée aux hôtes sur lesquels ils sont reliés,  
- **WebAccess**: Tout simplement le composant qui offre une interface web sur laquelle les utilisateurs vont se connecter afin de retrouver les services disponibles (bureau à distance complet ou applications),  
- **SessionHost**: Il s'agit du composant qui sera installé sur les hôtes.

Il existe également un composant qui ne sera pas utilisé ici: le **VDI** pour *Virtual Desktop Infrastructure*. Ce composant est assez particulier car pour pouvoir fonctionner, le hardware est fortement mis à contribution dans la mesure où, lors de la configuration, il est obligatoire de préciser la quantité de ressource qui sera allouée par utilisateur.  
Les composants ci-dessus ne sont que les briques logicielles nécessaires…mais je n'ai pas encore évoqué tout l'aspect infrastructure.  
Il s'avère que déployer une solution de ce genre est bien plus simple sur **Azure** ou sur **AWS** car ils proposent tout deux un service de **Remote Desktop Gateway** incluant:  
- **Autoscaling Group**,  
- **Security Group** prêt à l'emploi,  
- Routes et translations d'adresses pré-configurées,  
- Un tier application vide pour les instances SessionHost,  
- Un certificat SSL,  
- Des stratégies **CAP**(Client Access Protection) et **RAP**(Resource Access Protection) également configurées.  

Par contre, rien de tout cela n'est disponible sur **Google Cloud**…Ce qui représente un challenge technique plus élevé.

# Comment y arriver?
Puisque **Google Cloud** ne propose pas de solution clé en main (ou presque) afin de déployer ce genre de service, j'ai eu à faire quelques choix techniques parmi les suivants:  
- **Stackdriver Logging** permet de créer des sink, il s'agit d'une possibilité d'intégration du service vers d'autres services et d'export de logs vers le service sélectionné. Parmi les services, nous avons: Pub/Sub, Cloud Storage, BigQuery, Splunk ou bien un autre projet,    
- **Pub/Sub**: qui permet de gérer les messages transmis par Stackdriver Logging (dans le cas présent) pour le diriger vers un autre service,  
- **Cloud Functions**: Encore un service Serverless, celui-ci traite le message provenant de Pub/Sub et parse le json pour isoler certaines informations contenues et effectuer des actions bien précises à partir de là,  
- **VPC Access Connector**: Afin de connecter un service Serverless à votre VPC, par exemple à Cloud Functions, Cloud Run ou bien App Engine. En l'occurence, Cloud Functions pourra déclencher l'exécution d'un Playbook Ansible via l'instance sur laquelle le service a été déployé.    

## Du côté de Stackdriver
Comme évoqué plus haut, les sink permettent d'exporter les logs vers d'autres services et ceci de manière extrêmement simple, aussi bien via la console qu'à l'aide de Terraform.  

### Via la console
Plusieurs étapes :  
![Créer le filtre puis cliquer sur Create Sink](/images/Screenshot%202020-09-22%20at%2023.05.07.png)
  
![Sélectionner la destination du *Sink* dans le cas actuel:PubSub](/images/Screenshot%202020-09-22%20at%2023.06.22.png)    

![Nommer le Sink et sélectionner/créer le topic PubSub](/images/Screenshot%202020-09-22%20at%2023.07.43.png)  

### Avec Terraform
Seulement trois ressources à créer pour cela:

```hcl
resource "google_pubsub_topic" "example" {
  name = "example-topic"
}
resource "google_logging_project_sink" "my-sink" {
  name = "my-pubsub-instance-sink"
  destination = google_pubsub_topic.example.name
  filter = "resource.type = gce_instance AND jsonPayload.event_subtype = compute.instances.insert"
  unique_writer_identity = true
}
resource "google_pubsub_subscription" "example" {
  name  = "example-subscription"
  topic = google_pubsub_topic.example.name
}
```

## Du côté de PubSub
En fait, sur le *topic* **PubSub** il n'y a rien de plus à faire que ce qui a été déjà effectué dans Stackdriver. Il y a évidemment la possibilité de créer un trigger pour Cloud Functions…mais même cette action est faisable depuis l'interface de ce dernier.

![img1](/images/Screenshot%202020-09-22%20at%2023.23.21.png)

## Du côté de Cloud Functions
Si on ne la crée pas directement depuis **PubSub**, il n'y a aucune différence ni difficulté à la créer depuis l'interface du service. La partie qui demandera le plus de travail reste le code de la fonction en lui même, bien que vous pourrez toujours faire des tests avec hello_pubsub.  
Pour configurer le *trigger*, suivez les étapes ci-dessous:  

![Sélectionner PubSub dans TriggerType](/images/Screenshot%202020-09-22%20at%2023.30.09.png)  

![Et sélectionner le topic, puis sauvegarder pour accéder à la partie Code](/images/Screenshot%202020-09-22%20at%2023.31.05.png)  

## Du côté du VPC Access Connector
Une fois les *Sink* **Stackdriver**, *Topic/Subscription* (en mode Push) pour **PubSub** et **Cloud Function** créés, il est temps de passer au **VPC Access Connector**. L'API du service n'étant pas activée automatiquement lorsque l'on accède au service, une simple commande permet de débloquer ce dernier:
`gcloud services enable vpcaccess.googleapis.com`  
Une fois le service **VPC Access Connector** activé, une simple commande permet de créer le connecteur entre les services Serverless et ceux liés au VPC (comme Compute Engine ou GKE):
```
gcloud beta compute networks vpc-access connectors create cache-connector \
--network default \
--region us-central1 \
--range 10.8.0.0/28 \
--min-throughput 200 \
--max-throughput 400
```

## Du côté de Compute Engine
Comme évoqué plus haut, Remote Desktop Services nécessite le déploiement de plusieurs briques logicielles, chacune d'elles devraient idéalement être déployées sur une seule instance.
![img1](/images/1_5WQB4ADOlK7Rz9KQc9R_8A.png)

### Techniquement…
#### Les serveurs RDS Master et Workers
Il faut savoir qu'Ansible ne s'installe pas sur Windows mais peut exécuter des commandes Powershell/Batch et déployer des Rôles/Features…Le tout grâce à WinRM. Cependant, cela ne fonctionne pas tout à fait de la même manière qu'avec Linux (pour lequel il n'y a rien à faire, ou presque) car il faut tout d'abord l'activer.    
Dans le cas d'une instance du service Compute Engine, il faut ajouter ce qui suit dans le StartupScript:  

```powershell
Set-ExecutionPolicy -ExecutionPolicy Restricted -Force

$reg_winlogon_path = "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon"
Set-ItemProperty -Path $reg_winlogon_path -Name AutoAdminLogon -Value 0
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultUserName -ErrorAction SilentlyContinue
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultPassword -ErrorAction SilentlyContinue
$selector_set = @{
    Address = "*"
    Transport = "HTTPS"
}
$value_set = @{
    CertificateThumbprint = "E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE"
}

New-WSManInstance -ResourceURI "winrm/config/Listener" -SelectorSet $selector_set -ValueSet $value_set
```

Ensuite, pour faire le lien entre Ansible et les instances qui feront partie du groupe d'autoscaling, il sera plus judicieux de faire appel à un Inventaire Dynamique (voir la documentation officielle d'Ansible) afin d'éviter de devoir maintenir un inventaire statique.
Powershell ou Ansible ?
Pourquoi vouloir opposer les deux méthodes alors que nous pourrions utiliser les deux ? Il s'avère que certaines actions sont possible avec les deux technologies, par exemple :

**Installation de rôles/features:**
#### Via Ansible
```yaml
- name: Install RDS Licensing Feature
  win_feature:
    name: RDS-Licensing
    include_management_tools: true
    include_sub_feature: true
    state: present
```

#### Via Powershell
```powershell
Import-Module RemoteDesktop
Add-WindowsFeature RDS-RD-Licensing
```
**Promotion d'un domaine:**
#### Via Ansible
```yaml
- name: Domain Promotion
  win_domain:
    dns_domain_name: "{{ domain.name }}"
    safe_mode_password: "{{ domain.admin.password }}"
    database_path: "{{ domain.database }}"
    domain_netbios_name : "{{ domain.name.split[0] | upper }}"
    sysvol_path: "{{ domain.sysvol }}"
    forest_mode: "{{ domain.forest_mode }}"
```
#### Via Powershell
```powershell
$ForestConfiguration = @{
  '-DatabasePath'= 'C:\Windows\NTDS';
  '-DomainMode' = $DomainMode;
  '-DomainName' = $DomainNameDNS;
  '-DomainNetbiosName' = $DomainNameNetbios;
  '-ForestMode' = $DomainMode;
  '-InstallDns' = $true;
  '-LogPath' = 'C:\Windows\NTDS';
  '-NoRebootOnCompletion' = $false;
  '-SysvolPath' = 'C:\Windows\SYSVOL';
  '-Force' = $true;
  '-SafeModeAdministratorPassword' = $Password;
  '-CreateDnsDelegation' = $false }
Set-LocalUser -Name "Administrator" -Password $Password | Enable-LocalUser
Import-Module ADDSDeployment
Install-ADDSForest @ForestConfiguration
```

Par contre, la majorité des commandes relatives au déploiement d'une infrastructure Remote Desktop Services ne sont pas disponibles en tant que module Ansible (à l'exception de la configuration des RAP et CAP), à moins d'utiliser les modules win_shell et win_command (qui fonctionne de la même manière que shell ou command pour les OS Unix/Linux), ou psexec (qui permet l'élévation des privilèges lors de l'exécution de la commande).
Dans ces cas là, on va combiner Ansible et Powershell pour déployer et configurer les différents services RDS.
```yaml
- name: Configure Session Deployment
  ansible.windows.win_shell: New-RDSessionDeployment -Connection-Broker [RDS_MASTER] -WebAccessServer [RDS_MASTER] -SessionHost [RDS_MASTER]
    executable: powershell.exe
- name: Configure Licensing
  ansible.windows.win_shell: Add-RDServer -Server [LICENCE_SERVER] -Role RDS_LICENSING -Connection-Broker [RDS_MASTER]
    executable: powershell.exe
```

# Tout fonctionne ?
Une fois que tout est déployé, l'architecture Remote Desktop Services avec scaling automatique des SessionHost ressemble à ceci:  
![img1](/images/1_waCo0kkb0m9cEyyk-0Yu5w.png)

Il suffit de se connecter au Connection-Broker à l'aide de ses identifiants de connexion (provenant de l'Active Directory), de choisir la session et de s'y connecter. En fonction de la configuration des SessionsCollection, il peut y avoir une ou plusieurs sessions pour chaque utilisateur (cela dépend entre autre de la configuration d'Active Directory et des OU contenant les profils utilisateurs).
Cependant, le but de cet article était principalement de présenter une technologie encore bien présente dans certaines entreprises sous un angle un peu différent…et non d'effectuer une plongée en apnée dans les entrailles de Windows Server. Si vous cherchez de la documentation relative à la configuration de Resources Access Protection, du serveur de Licensing ou des SessionCollection (par exemple), voici quelques liens qui vous seront très utiles:  
- https://www.it-connect.fr/windows-server-2019-installer-un-serveur-rds-avec-powershell/  
- https://www.loadbalancer.org/applications/microsoft-remote-desktop-services/  
- https://msfreaks.wordpress.com/2018/10/06/step-by-step-windows-2019-remote-desktop-services-using-the-gui/  
- https://ryanmangansitblog.com/2013/03/27/deploying-remote-desktop-gateway-rds-2012/
