+++
date = "2021-01-01T15:29:49+02:00"  
draft = "false"
title = "Le Labo #34 | Consul et Vault en TLS"
+++

J'avais une grande hésitation sur le titre...et puis j'ai voulu faire un peu d'humour en remplacant *PLS* par *TLS*...mais ce ne fait pas le même effet (j'en suis bien conscient).

Dans [l'article précédent](http://www.ageekslab.com/lab/labo39/) j'avais continué mon exploration de Vault en vous proposant un moyen relativement simple de remplacer le *Secret Manager* de **Kubernetes** par un cluster **Vault** externe...et d'après les quelques retours que j'ai eu sur **LinkedIn**, vous avez apprécié.    
Par conséquent, j'ai décidé d'explorer un peu plus le sujet en proposant, cette fois-ci, une approche plus orientée sécurité en ajoutant le chiffrement des communications entre **Consul** et **Vault** via **TLS**.  
Potentiellement, le principe est le même que lors de l'article précédent...cependant la configuration sera différente car **Consul** et **Vault** seront déployés dans **Kubernetes**.

## Consul
Précédemment seuls les **ACLs** étaient activés dans le configuration du serveur et du client, pour cela il suffisait juste d'ajouter le bloc **ACL** et de definir la *policy* par défaut sur **deny** et de générer le *token* par la suite.  
Jusqu'ici vous avez bien suivi l'article précédent.  
Maintenant nous allons nous pencher sur deux éléments nouveaux, à savoir :  
- **Secure Gossip Communication with Encryption**,  
- et **Secure Consul Agent Communication with TLS Encryption**.  

Si le premier ne requiert qu'un simple *token* pour fonctionner, la second peut donner des sueurs froides par la création de clés SSL...qui peuvent être créées à l'aide de **Consul** ou de tout autre outils prévu à cet effet (**OpenSSL** par exemple).

![img1](/images/consul_secure.png)

#### Secure Gossip Communication Encryption
Il s'agit ici de sécuriser les communications entre tous les membres du cluster sans exception.  
Comme évoqué plus haut dans cet article, un simple *token* est nécessaire composé d'une chaîne de caractère de 32 bits codée en base64...que tout le monde se rassure, la commande **keygen** de **Consul** s'en occupe très bien :  
```shell
consul keygen
```
Le résultat de ce *token* est à ajouter dans la configuration de **Consul** avec la clé **encrypt**. Il est également récommandé d'ajouter ce qui suit :  
```json
encrypt_verify_incoming = true
encrypt_verify_outgoing = true
verify_server_hostname = true
```

#### Secure Consul Agent Communication with TLS Encryption
Vous le savez, nous le savons...La sécurisation d'un datacenter via le chiffrement **TLS** est un détail très important pour les déploiements production.  
Ceci étant dit, la configuration TLS de **Consul** est un peu plus compliquée car elle requiert des certificats TLS qui peuvent être générés par de nombreux outils, dont **Consul**...et la génération de ces certificats peut parfois donner des résultats assez aléatoires.  
D'autant plus qu'en ce qui concerne **Consul**, la CLI et l'UI web devront être sécurisées séparément.  
Les certificats TLS sont utilisés par **Consul** pour vérifier l'authenticité des serveurs et de tous les clients qui doivent eux-même disposer de leurs propres certificats.

Si vous souhaitez utiliser **Consul** pour générer vos certificats TLS, il y a deux commandes à exécuter. La première permet de générer l'autorité de certification qui servira, par la suite, à créer les certificats eux même. Dans l'ordre, nous avons :  
- `consul tls ca create`,  
- `consul tls cert create -server`

Parmi les quatres certificats créés à l'aide des deux commandes ci-dessus, seuls trois seront utiles dans la configuration de **Consul** : le **ca public** généré à l'aide de la première commande, le certificat public ainsi que la clé privée générés à l'aide de la seconde.

Ici vous avez le choix de la méthode entre **auto_encryption** et la méthode **operator**. Très peu de différences entre les deux, si ce n'est la génération des certificats clients pour la seconde méthode à l'aide de la commande `consul tls cert create -client`.

Une fois que vos serveurs et agents sont démarrés, vous devriez pouvoir les voir s'afficher dans l'UI web de **Consul**...ou vérifier qu'il n'y aucune erreur dans les logs de chaque serveur **Consul**.

## Vault
Il y a de nombreuses options dans la configuration de **Vault** qui font référence à **Consul** :  
- Le **backend** (à ne pas confondre avec le *backend storage*),  
- Le **storage** (cependant, dans notre exemple, je n'utilise pas **Consul** mais **postgresl**),  
- **Service Registration**

Chacune de ses options proposent les mêmes principes relatif à la sécurisation des communications, nous retrouvons *token* (relatif aux **ACLs**) ainsi que *tls_skip_verify*, *tls_key_file*, *tls_ca_file*, *tls_cert_file* et *tls_min_version*.

## Kubernetes
Comme évoqué au début de l'article **Consul** et **Vault** seront déployés dans **Kubernetes**, le premier en tant que **Service Mesh**, le second en tant que **Secret Manager**. Je vais évidemment faire appel à la toute puissance de **Helm** pour déployer rapidement les deux services et je vais détailler un minimum chaque *value file*.  
Je précise que la version de **Helm** utilisée ici est la version 3...après avoir tenté avec la version 2...L'avantage est l'absence de **Tiller**.  

![img2](/images/consul_vault_kube.png)

#### Consul Helm Chart
Tous les charts **Helm** sont entièrement personnalisable, celui-ci ne déroge pas à la règle. Cependant, il ne sera pas extrêmement différent de la version *non kubernetisée* sur laquelle j'ai pu travailler.

Le paramètre **Global** contient des variables affectant une grande partie des composants du *chart*.  
C'est d'ailleurs dans cette partie que sont abordés tout ce dont nous avons abordé plus haut, à savoir **Gossip Encryption**, **ACLs** et **TLS** via des secrets qui seront créés au préalable.
```yaml
global:
  enabled: true
  datacenter: dc1
  enablePodSecurityPolicies: true
  gossipEncryption:
    secretName: consulSecrets
    key: gossipSecret
  acls:
    bootstrapToken:
      secretName: consulSecrets
      key: bootstrapSecret
  tls:
    enabled: true
    verify: true
    httpsOnly: true
    caCert:
      secretName: consulSecrets
      key: caCertSecret
    caKey:
      secretName: consulSecrets
      key: caKeySecret
```

Dans les parties **Server**, **Client** et **UI**, rien de particulier à signaler puisque une grande partie des options de sécurisation ont déjà été définies plus haut.

#### Vault Helm Chart
Le chart **Helm** pour **Vault** répond au même principe que pour le service précédent : une partie **global** contenant des variables impactant de nombreuses partie du chart, puis les parties **injector** (relatif à **Vault** Agent Injector), server et ui.  

#### Démarrer les services
Avant de songer à démarrer le déploiement des services sur **Kubernetes**, je vous invite à créer le secret intégrant le token Gossip, ACL et les certificats...si vous oubliez ce détail, ca risque de faire désorde dans vos logs.

## Conclusion
A part le fait que **Vault** et **Consul** soient déployés dans **Kubernetes** avec un accent *sécurité* un peu plus prononcé sur les communications entre serveurs **Consul** et **Vault**, il n'y a que très peu de différences avec l'article précédent...Et je n'ai pas encore abordé l'aspect **Service Mesh** de **Consul** qui fera peut-être l'objet d'un prochain article.  
