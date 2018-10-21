+++
date = "2017-11-11T15:29:49+02:00"
draft = "false"
title = "Le Labo #26 | Deployer sur Openstack via Terraform, Jenkins et Ansible"

+++

Pour une fois, je ne vais pas aborder le déploiement sur <b>DigitalOcean</b>, <b>Azure</b> ou même <b>Google Cloud</b>...Non, c'est fois ci, ce sera <b>Openstack</b>. Mais pas n'importe comment, ce sera toujours avec <b>Terraform</b> et sur plusieurs environnements différents impliquant donc plusieurs fichiers de variables différents.  
Je n'avais pas encore démontré l'utilisation des <i>dépendances implicites</i> ou d'utilisations de l'instruction <i>lookup</i> pour ittérer sur les listes de variables...Mais assez de tricotage/brodage, passons à l'action proprement dite.

## Terraform in action
Entrons dans le vif du sujet assez rapidement car j'imagine que vous connaissez déjà assez bien la technologie.  
Sur <b>Openstack</b> avant de pouvoir créer des serveurs, il est nécessaire de commencer par l'infrastructure, en l'occurence :  
- <b>Network</b>  
```hcl
resource "openstack_networking_network_v2" "network" {
  count			 = "${length(var.network)}"
  name           = "${lookup(var.network[count.index],"name")}"
  admin_state_up = "${lookup(var.network[count.index],"admin_state_up")}"
  region         = "${lookup(var.network[count.index],"region")? 1 : 0}"
}
```
- <b>Subnet</b>  
```hcl
resource "openstack_networking_subnet_v2" "subnet" {
  count		 = "${length(var.subnet)}"
  name       = "${lookup(var.subnet[count.index],"name")}"
  cidr       = "${lookup(var.subnet[count.index],"cidr")}"
  network_id = "${element(openstack_networking_network_v2.network.*.id,lookup(var.subnet[count.index],network_id))}"
  ip_version = "${lookup(var.subnet[count.index],"ip_version")}"
  region     = "${lookup(var.network[count.index],"region")? 1 : 0}"
}

```
- <b>Router</b>
```hcl
resource "openstack_networking_router_v2" "router" {
  count				  = "${length(var.router)}"
  name                = "${lookup(var.router,"name")}"
  admin_state_up      = "${lookip(var.router,"admin_state_up")}"
  external_network_id = "${element(openstack_networking_network_v2.network.*.id,lookup(var.router[count.index],"network_id"))}"
  region              = "${lookup(var.network,"region")? 1 : 0}"
}
```
- <b>Router Interface</b>  
```hcl
resource "openstack_networking_router_interface_v2" "router_interface" {
  count     = "${ "${length(var.router)}" == "0" ? "0" : "${lenght(var.subnet)}" }"
  router_id = "${element(openstack_networking_router_v2.routeur.*.id,lookup(var.router_interface[count.index],"routeur_id"))}"
  subnet_id = "${element(openstack_networking_subnet_v2.subnet.*.id,lookup(var.router_interface[count.index],"subnet_id"))}"
}
```  
- <b>Floating IP</b>  
```hcl
resource "openstack_networking_floatingip_v2" "floating_ip" {
  count  = "${lenght(var.floating_ip)}"
  pool   = "${lookup(var.floating_ip[count.index],"pool")}"
  region = "${lookup(var.network,"region")? 1 : 0}"
}
```

Une fois la partie réseau définie, autant passer à l'étape suivante : Les <b>sec_group</b> et <b>sec_group_rule</b> :  
```hcl
resource "openstack_compute_secgroup_v2" "sec_group" {
  count       = "${length(var.sec_group)}"
  description = "${lookup(var.sec_group[count.index],"description")}"
  name        = "${lookup(var.sec_group[count.index],"name")}"
  region      = "${lookup(var.network,"region")? 1 : 0}"
}
resource "openstack_networking_secgroup_rule_v2" "sec_group_rule" {
  count             = "${ "${length(var.sec_group)}" == "0" ? "0" : "${length(var.sec_group_rule)}" }"
  direction         = "${lookup(var.sec_group_rule[count.index],"direction")}"
  ethertype         = "${lookup(var.sec_group_rule[count.index],"ethertype")}"
  protocol          = "${lookup(var.sec_group_rule[count.index],"protocol")}"
  port_range_min    = "${lookup(var.sec_group_rule[count.index],"port_range_min")}"
  port_range_max    = "${lookup(var.sec_group_rule[count.index],"port_range_max")}"
  remote_ip_prefix  = "${lookup(var.sec_group_rule[count.index],"remote_ip_prefix")}"
  security_group_id = "${element(openstack_compute_secgroup_v2.sec_group.*.id,lookup(sec_group_rule[count.index],"sec_group_id"))}"
}
```

Pour le reste (les serveurs), je vous laisse vous en occuper...Après tout, vous devriez avoir compris le principe des dépendances implicites et des fichiers de variables externes (les <b>.tfvars</b>). Si vraiment vous avez besoin d'aide, je suis toujours joignable via <b>twitter</b> ou <b>LinkedIn</b>.

Travailler sur <b>Openstack</b> est relativement simple :  
- Un réseau de base sur lequel <i>brancher</i> celui que vous allez créer (vous ne pourrez pas brancher vos serveurs dessus),  
- Un Security Group de base...à ne surtout pas oublier de définir pour vos serveurs (sinon, vous ne pourrez pas vous connecter dessus ni même essayer de contacter chacun d'entre eux),  
- Un fichier à télécharger pour pouvoir déployer tout ce dont vous avez besoin pour travailler et qui évite d'avoir a <i>hardcoder</i> des variables d'environnement dans vos states <b>Terraform</b>.

## Lets industrialize it  
Passons à <b>Jenkins</b>...Pour le moment et contrairement à d'autres outils/Providers Cloud, il n'existe pas encore de plugin Terraform, ce qui fait que pour <i><b>Terraformer</b></i> son infrastructure Cloud, il faut le faire <i>manuellement</i> dans <b>Jenkins</b> :  
```groovy
stage("Terraform") {
	steps {
		sh '''
		cd terraform/resources
		terraform init
		terraform apply -var-files vars.tfvars -auto-apply
		'''
	}
}
```
Sans biensûr oublier de définir les variables d'environnement adequates...ce qui revient à les <i>hardcoder</i> dans le pipeline...Mais il y a un autre moyen : Déployer via <b>Jenkins</b> un conteneur <b>Docker</b> qui s'occupera de tout le travail de <b>Terraform</b>.  
Non, ce n'est pas tiré par les cheveux...Le <i>Dockerfile</i> est extrêment simple :  
```Dockerfile
FROM node:10.10.0-jessie

WORKDIR /opt/

ARG URL_PROXY
ARG LOGIN_PROXY
ARG PASS_PROXY
ARG TARGET
ARG TENANT
ARG TF_VERSION

ENV http_proxy http://${LOGIN_PROXY}:${PASS_PROXY}@${URL_PROXY}
ENV https_proxy http://${URL_PROXY}
ENV HTTPS_PROXY http://${URL_PROXY}
ENV HTTP_PROXY http://${URL_PROXY}

COPY ${TARGET}01_${TENANT}-openrc.sh .
COPY terraform.sh terraform.sh

RUN echo Acquire::http::proxy \"http://${LOGIN_PROXY}:${PASS_PROXY}@${URL_PROXY}\"; > /etc/apt/apt.conf.d/Proxy && \
    echo Acquire::https::proxy \"http://${LOGIN_PROXY}:${PASS_PROXY}@${URL_PROXY}\"; >> /etc/apt/apt.conf.d/Proxy && \
    apt-get update && \
    apt-get install -y git unzip && \
    curl https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip && \
    unzip terraform_${TF_VERSION}_linux_amd64.zip && \
    chmod +x terraform && chmod +x terraform.sh && chmod +x ${TARGET}01_${TENANT}-openrc.sh

CMD ["./terraform.sh"]

```  
J'ai sélectionné une image avec nodejs, mais on peut utiliser n'importe quoi...je vous rassure.
Autre chose : J'ai défini des <i>build-arg</i> pour gérer des informations relatives à un éventuel proxy à intégrer dans votre conteneur, le ficheir contenant les variable d'environnement ainsi que la version de <b>Terraform</b> à télécharger depuis le site officiel.  
Comme vous pouvez le constater, il y a aussi un <i>terraform.sh</i> qui se lancera au démarrage du conteneur...et c'est lui qui va gérer tout le déploiement.  
```
#!/bin/sh
set -x

pid=0

_stopStrapi() {
  echo "Stopping strapi"
  kill -SIGINT "$strapiPID"
  wait "$strapiPID"
  exit 143;
}

trap 'kill ${!}; _stopStrapi' SIGTERM SIGINT

cd /tmp/

ACTION=${ACTION}
PROXY_LOGIN=${PROXY_LOGIN}
PROXY_PASSWORD=${PROXY_PASSWORD}
PROXY_URL=${PROXY_URL}
GIT_URL=${GIT_URL}
PROJECT=${PROJECT}
TARGET=${TARGET}
TENANT=${TENANT}
ELEMENT=${ELEMENT}

VARFILE="$(echo $TARGET | awk '{print tolower($0)}')_vars.tfvars"
ENVFILE="$(echo $TARGET)01_$(echo $TENANT)-openrc.sh"

export HTTP_PROXY="http://$PROXY_LOGIN:$PROXY_PASSWORD@${PROXY_URL}"
export HTTPS_PROXY="http://$PROXY_LOGIN:$PROXY_PASSWORD@${PROXY_URL}"
export http_proxy="http://$PROXY_LOGIN:$PROXY_PASSWORD@${PROXY_URL}"
export https_proxy="http://$PROXY_LOGIN:$PROXY_PASSWORD@${PROXY_URL}"

git clone https://$PROXY_LOGIN:$PROXY_PASSWORD@$GIT_URL/$PROJECT/terraform.git 

. /opt/$ENVFILE

for element in $ELEMENT; do
	cd /tmp/terraform/resources/$element && \
		/opt/terraform init && \
		/opt/terraform plan -var-file=/tmp/terraform/resources/$VARFILE && \
		/opt/terraform $ACTION -var-file=/tmp/terraform/resources/$VARFILE
done

pid="$!"
```

En démarrant <b>Terraform</b> depuis un conteneur <b>Docker</b>, il devient possible de ne plus être dépendant de diverses limites techniques relative au serveur <b>Jenkins</b> et à ses slaves...surtout si ce n'est pas vous qui les gérez.

## Let's Ansible it
Contrairement à <b>Terraform</b>, <b>Ansible</b> dispose d'un plugin pour <b>Jenkins</b> et est extrêmement bien documenté...On peut l'utiliser aussi bien avez un pipelnine <i>déclaratif</i> que <i>scripté</i> (qui eux sont relativement compliqué à monter car necessitant une connaissanc)e relativement étendue de <b>Groovy</b> - Voir ci-dessous un petit exemple).  
```groovy
ansiblePlaybook(
        playbook: 'playbook.yml',
        inventory: 'inventory.ini',
        tags: 'tags',
        colorized: true,
        sudoUser: 'root',
        extraVars: 'ansible_python_interpreter=/usr/bin/python'
)
```

Par contre, il est tout à fait possible d'utiliser un conteneur <b>Docker</b>, bien que pour <b>Openstack</b> il y ait besoin de nombreux packages installables via <b>pip</b> :  
```
python-openstackclient  
shade   
argparse   
python-swiftclient  
appdirs   
iso8601   
os_service_types   
requestsexceptions   
munch   
deprecation  
ipaddress   
jsonpatch   
dogpile.cache   
jmespath   
netifaces   
decorator  
```

Selon L'infrastructure sur laquelle vous êtes, cette image <b>Docker</b> peut être très longue à builder...je recommande de le faire localement et de la publier sur un <b>Docker Registry</b> avant de l'utiliser dans <b>Jenkins</b>.

## Return to Jenkins
Maintenant que vous avez vos images <b>Docker</b> pour <b>Terraform</b> et <b>Ansible</b>, retournons sur <b>Jenkins</b> et son plugin <b>Docker</b>. Son utilisation est assez simple :  
- Pour build une image : `docker.build("name","/build_args/ --build-arg ENVIRONMENT=${TARGET} ./Ansible/)`  
- Pour démarrer un conteneur : `docker.image("image_name").withRun(/run_args/)`

## Let's rock
Ca faisait longtemps que je n'avais pas écrit d'article sur Jenkins...Je me suis dis ca pouvait être une idée d'aborder cette technologie malgré la percée effectuée par Bamboo, Circle-CI et autres.