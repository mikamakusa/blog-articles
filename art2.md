+++
date = "2017-11-29T13:35:49+02:00"
draft = false
title = "Comparatif Entre SaltStack, Ansible et Puppet"

+++

Alors oui, je sais...Certais vont crier au troll bien velu venant des montagnes.  
Certains vont se souvenir que je travaille sur **Saltstack** depuis un peu plus d'un an et demi...Mais ce sont ceux là même qui ont oublié que je fais parti des *early adopters* d'**Ansible** (quand **Tower** et **Galaxy** n'était pas encore sorti).  
Lors d'une de mes missions, j'ai pu aussi travailler sur **Puppet**...deux semaines, pas vraiment suffisant pour maîtriser la technologie, mais assez pour me faire une idée du manque de 'naturel' de la syntaxe (si je compare à **Ansible** et à **SaltStack**).  
Cet article n'a pas pour vocation d'être un lab, parce que je ne vais faire que de la comparaison de certaines features...un peu comme je l'avait fais sur le traitement de données json en **powershell** ou en **python** il y un peu plus de deux ans.  


## La communication entre les serveurs
Pour commencer, on va se pencher sur la relation entre les serveurs.  
Pour **SaltStack**, il s'agit d'un token que le minion transmet au serveur maître. Si les paramètres adéquats sont bien renseignés au niveau de la configuration du minion, les deux serveurs arriveront à communiquer sans aucun problèmes.  
Pour **Ansible**, il ne s'agit que de communication sur le protocole SSH avec clé privé. Donc, si le port SSH a été configuré différemment que par défaut (le port 22), ca fonctionnera aussi. La configuration des serveurs cibles est quasi inexistante, il y a juste besoin de l'agent **Ansible**.  
Du côté de **Puppet**, il s'agit d'une connection qui semble aussi sécurisée que peuvent l'être celles entre les minions et master **Saltstack**  avec un certificat SSL généré côté client et à accepter du côté serveur.


## L'utilisation de variables
**Saltstack** et ses *pillars*...ou **Ansible**/**Puppet** et leurs variables...  
Lorsque vous avez besoin de définir une variable qui pourra potentiellement être utilisée par toutes les étapes de provisionning d'un composant, autant éviter de la hardcoder dans un fichier bien précis.  
Concernant **Saltstack**, on crée un fichier *init.sls* dans le dossier pillar (dans la configuration du serveur, c'est au niveau *pillar_roots*) qui définira quel pillar sera appelé en fonction du nom du serveur tel qu'enregistré dans les clés du **salt master**, ensuite on crée un autre fichier *init.sls* dans un dossier correspondant au composant, et l'on enregistré les variables dedans selon le principe **clé: variable**...et peuvent aussi être définies en fonction d'une hiérarchie:  
```
var1:
  var1_1:
    var1_1_0: 'test'
```

Ensuite, pour les appeler dans les states, il suffit 'juste' de les appeler de la manière suivante:
```
{{ set var = salt['pillar.get']('var1:var1_1')}}
``` 

**Ansible** et **Puppet** offrent aussi la possibilité de définir une hierarchie de variable, la syntaxe relative à **Ansible** est identique à **SaltStack**, par contre pour **Puppet**, c'est complètement différent et ça donne ça: `$var1::var1_1 = 'test'`

Avec **Ansible**, pour appeler et injecter des variables dans une action définie dans un playbook, il faut d'abord définir le chemin du fichier contenant les variables à l'aide du keyword **var_files**...et ensuite appeler la variable ainsi (aussi bien dans un playbook que dans un template) :  

- Dans un playbook :
```
"{{ var1.var1_1 }}"
```

- Dans un template:
```
<%= var1.var1_1 %>
```

Pour **Puppet** c'est un peu différent.  
Je cherche encore si il y a la possibilité d'externaliser les variables, si vous avez l'information, je suis tout ouïe :)  
Néanmoins pour appeler les variables dans un module, il suffit juste de l'appeler de la même manière qu'elle a été définie, c'est à dire : `@var1::var1_1`


## Les Templates
En fait, je vais plutôt aborder les fichiers qui vont être utilisés en tant que templates ou en tant que fichier de configuration fixe (sans variables à injecter lors du provisionning).  
**SaltStack** vous permettra de placer le fichier en question absolument nimport où dans le dossier relatif au composant que vous souhaitez installer, il suffira juste de préciser la source.  
Prenons un exemple avec *file.managed* :  
```
file.managed:
  source: https://rubygems.org/rubygems/rubygems-2.7.2.tgz
  name: /tmp/rubygems-2.7.2.tgz
  skip_verify: True
```

Certes, c'est une archive provenant d'internet mais si le dossier de votre composant contient lui même plusieurs dossiers, il est possible de procéder ainsi (en étant dans le dossier *configure* par exemple:  
```
file.managed: 
  source: /install/template/config.xml
  name: /tmp/config.xml
```

Du côté d'**Ansible**, c'est complètement différent. Idéalement, chaque dossier de composant contient un dossier *templates* et si une *tasks* contien le keyword *template*, **Ansible** ira chercher le fichier en question dans le dossier susnommé lors de l'exécution du playbook (malin, non ?).  
Un petit exemple :  
```
  template:
    src: api.json.j2
    dest: "{{ base.config_dir }}/conf.d/api.json"
    owner: root
    group: root
    mode: 0644
```

En ce qui concerne **Puppet**, le fonctionnement semble identique à celui d'**Ansible** (selon la documentation officielle), un keyword *template* est disponible lors de l'utilisation du module *file*. 
Un petit exemple :  
```
file { '$uchiwa_config::path/$uchiwa_config::file':
	ensure 	=> file,
	owner	=> $uchiwa_config::owner,
	group 	=> $uchiwa_config::owner,
	mode 	=> $uchiwa_config::mode,
	content => template('templates/uchiwa.json.erb'),
	require => Class['sensu::prerequisites','sensu::install'],
}
```


## Les informations relative à l'hôte
**Saltstack**, **Ansible** et **Puppet** peuvent gérer des variables relatives à l'hôte provisionnable et les injecter dans des fichiers ou dans le but de piloter un déploiement de composant.
En ce qui concerne **Saltstack**, il est possible d'ajouter des *grains* directement dans le fichier de configuration du *minion* et de les appeler de la même manière que ceux d'origine.  
Il est possible que **Ansible** et **Puppet** proposent un fonctionnement similaire, mais je manque d'informations sur le sujet.  
Je ferais un update dès que possible.


## Les modules disponibles
Je ne vais pas faire une liste des modules disponibles pour chacune des technologies, un tour sur les sites officiels vous donneront toutes les informations nécessaires. Mais si on prend en compte le fait que **Saltstack** permet de gérer des serveurs influxdb, grafana, et tant d'autres...La liste est réellement impressionnante.  
Concernant **Ansible**, la liste est moins longue, certes...mais il se trouve qu'**Ansible** et **Saltstack** sont assez complémentaire à ce niveau là.

Concernant **Puppet**, c'est assez différent. De base, très peu de modules sont disponible...Mais la communauté autour de l'outil étant hyperactive, il se trouve que de nombreux modules sont trouvables un peu partout, fonctionnels et relativement bien documentés.  
Néamoins pour les développer, il faut se mettre au ruby.


## Intégration à certains outils
L'outil dont je me sers le plus souvent pour mes labs de tests est celui qui en intègre le plus dans la partie **Provisionning**...En effet, **Vagrant** gère aussi bien **Salt** (Masterless ou non), **Ansible**, **Puppet** et même **Chef**...La configuration de chacun est également très simple (voir ci-dessous) : 

- Pour **Ansible**:  
```
nc.vm.provision :ansible_local do |ansible|
  ansible.playbook = "ansible-scripts/playbook.yml"
  ansible.install = true
  ansible.install_mode = "pip"
end
```

- Pour **Saltstack** : 
```
nc.vm.provision :salt do |salt|
  salt.masterless = true
  salt.run_highstate = false
  salt.minion_config = "salt-scripts/minion"
end
```

- Pour **Puppet** : 
```
nc.vm.provision :puppet do |puppet|
  puppet.manifest_file = "default.pp"
  puppet.manifests_path = "puppet-scripts/manifests/"
end
```

Certes je n'ai mis que les options dont je me sers le plus souvent, si vous allez faire un tour sur la (documentation officielle de **Vagrant**)[], vous trouverez toutes les options de configuration disponible.

**Terraform**, de son côté, en propose moins...pour l'instant il ne propose que **Saltstack** et **Chef** (aussi bien en mode *masterless* que *master/minion*). J'imagine qu'à l'avenir, les gars de chez **Hashicorp** trouveront le temps d'en ajouter afin de completer la liste des provisionners disponible.  
Mais le système des provisionners est tellement bien réalisé qu'il est possible de passer outre cette limitation technique...à l'aide de *remote-exec* qui; lui, permet d'éxecuter des commandes sur l'hôte distant...et donc d'utiliser n'importe quel outil de configuration managment (oui oui, même **Puppet**).


## Finalement
J'avoue avoir une grande préférence pour **Saltstack** et **Ansible**, le premier je travaille avec depuis un peu plus d'un an et demi, le second je l'ai découvert alors qu'il n'était pas encore en version **2.0** (et en même temps que **Docker**).  
En revanche, je trouve **Puppet** relativement compliqué à utiliser et doté d'une syntaxe moins naturelle que les deux premiers cités...probablement parce que je maîtrise beaucoup moins **Ruby** que **Python**.  
Ceci explique peut être cela, après tout.