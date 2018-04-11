+++

date = "2018-04-10T09:20:00+02:00"
draft = false
title = "Ansible - les inventaires dynamiques"

+++

Je ne vais pas revenir sur ce qu'est **Ansible**, vous savez tous ce que c'est (j'espère...sinon je peux vous suggérer de faire un tour sur le site officiel).  
Par contre, il y a un moyen de rendre l'utilisation d'**Ansible** plus *fun* (je n'ai pas trouvé d'autres mot) à l'aide des *inventaires dynamiques*.  

## Qu'est-ce que cela implique ?  
- Gestion dynamique des hosts via un simple script  
En fait, dans le cas de **Vagrant** oui...mais en ce qui concerne **OpenStack** et **Azure**, ce n'est pas vraiment le cas puisqu'il faut aussi exporter quelques variables d'environnement.  

- Possibilité d'exporter des variables d'un hôte à un autre...sans avoir besoin de les définir au préalable (dans *defaults* ou dans *vars*).  

## Comment ca marche ?
Tout d'abord : il n'y a pas de script d'inventaire dynamique universel...à moins de le faire soi même. Par contre, les scripts pour chaque plateformes sont disponible sur internet, certains sont en lien sur le site officiel d'**Ansible** (**Cobbler**, **AWS**, **OpenStack** notamment).

Pour les utiliser, il suffit juste de l'ajouter dans la commande `ansible-playbook` : `ansible-playbook <playbook>.yml -i </path/to/dynamic-inventory>.py`.  
Parfois les scripts de dynamic inventory ne suffisent pas et ils faut, dans ce cas là, ajouter une configuration additionnelle du type :  
```
ansible:
  use_hostnames: true
  expand_hostvars: false
  use_floatingip: false
```

## Et les bastions ?
Je préfère anticiper toute questions sur les bastions dans le cadre d'une utilisation d'un bastion : Ca fonctionne de la même manière, il faut juste préciser à **Ansible** que toute les communication doivent passer par le *jump host* à l'aide d'un fichier **ansible.cfg** (exemple ci-dessous):  
```
[defaults]
hash_behaviour=merge
host_key_checking=false
#relative to CWD
deprecation_warnings=False
timeout=60
[ssh_connection]
ssh_args = -o ProxyCommand="ssh -W %h:%p -l centos -i </path/to/bastion_key> <Bastion.IP.address>"
```


