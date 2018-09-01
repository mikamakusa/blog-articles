+++

date = "2018-09-01T10:05:00+02:00"
draft = false
title = "Un CMF avec Strapi"

+++

# Un CMF ? Kesako ?
Tout le monde connait les **CMS**, ou *Content Managment System*, et il en existe vraiment tout plein comme, par exemple :  
- Magento : boutique en ligne
- Wordpress : Que l'on ne présente plus...et qui est une usine à gaz
- Drupal
- PrestaShop

Il y en a pour tout les goûts et certains sont tellement populaire qu'ils disposent aussi bien d'une communauté professionnelle reconnue que d'une communauté d'utilisateurs lambda (comme vous et moi).  
Pourtant, il existe aussi des **CMF** pour *Content Managment Framework* dont il semblerait que **Drupal** fasse ausi partie, tout comme **OpenOrchestra** ainsi que **Strapi**.

**Strapi** est relativement nouveau bien qu'ils en soient à déjà la version 3.0.0-alpha.14...En fait, leur rythme de release est assez soutenu (environ une par mois) et est français...Cependant le fait qu'ils soient français n'est pas vraiment un gros avantage dans la communication avec les développeurs (étant donné qu'ils ne repondent presque jamais sur *Slack*).

Après ce petit troll totalement gratuit (néanmoins justifié), je vais évoquer quelques fonctionnalités de l'outil :  
- Totalement modulable : Des plugins sont disponible sur leur *market place*, mais vous pouvez développer les votres et les installer à la place de ceux disponibles.  
- CMF réalisé en nodejs, utilisable avec d'autres packages, tels que **pm2** (haute disponibilité *à portée de l'homme*) ou d'autres packages.  
- Dispose d'une interface très *eye candy*.  
- S'interface avec différentes base de données très simplement : MongoDB, MySQL et PostgreSQL.  
- Génération de projets très facile : un minimum de quatre parametres requis pour créer un projet, dont trois pour la connexion à une base de données

# Comment On l'installe ?
En fait, il y a deux manières d'installer une instance Strapi :  
- *Classique* : Directement sur le serveur via la commande `npm install -g strapi@v[version]`  
Une fois **Strapi** installé, il faudra exécuter les commandes suivantes :  

```
strapi new [project_name] --dbclient=[DB_CLIENT] --dbhost=[DB_IP] --dbport=[DB_PORT] --dbname=[DB_NAME] --dbusername="" --dbpassword=""  
cd [project_name]  
strapi start
```

- *Docker* : via un conteneur, je recommande de créer vous même votre image **Strapi** intégrant un script de démarrage.

---
**Attention**  

Durant l'étude de **Strapi**, votre serviteur a rencontré de nombreux obstacles...cet article à aussi bien pour but de vous faire découvrir l'outil que de vous permettre d'éviter de rencontrer les mêmes obstacles :  
- Je vous recommande la version *3.0.0-aplha.14*, d'autres sont disponibles, plus anciennes, et fonctionnent très mal.  
- le site officiel de **Strapi** recommande certaines versions de node, npm et mongo...oubliez ce que vous avez pu voir sur le site officiel. Mention spéciale à la nodejs 9.1, **Strapi** plante dès la création du projet.  
- Vous trouverez pas mal d'aide sur le *Slack* **Strapi**, mais pas venant des développeurs ni du community manager...Ceux qui répondent le plus souvent/efficacement sont des utilisateurs lambda,  
- Je vous recommande la version 4 de MongoDB (si vous souhaitez utiliser celle-ci), j'avoue ne pas avoir testé avec MySQL ou PostgreSQL.

---

# Que faire après l'installation ?
**Strapi** est installé ? Parfait...vous allez pouvoir commencer à jouer avec.  
Pour cela, si vous n'avez pas installé un *reverse proxy*, rendez-vous sur l'adresse du serveur, port 1337 et enregistrez l'admin user afin de vous connecter à l'admin panel:  
Vous pourrez créer d'autres utilisateurs par la suite sur la page **Users**.

---
**Attention**  

Si vous avez installé une autre version que celle que je vous ai recommandé plus haut, il se peut que cette page soit totalement vide lors de votre permière connexion...et à chaque fois que vous créez un nouvel utilisateur.
Etrangement chaque utilisateur créé sera visible en base de données.

---

- La gestion des permissions s'effectue via la page **Roles and Persmissions**.  

---
**Attention**

Tout comme pour la page **Users**, cette page peut afficher les items **Public**, **Authenticated** et **Administrator**, mais n'affichera rien lorsque vous cliquez sur chaque items. C'est un défaut des anciennes versions de **Strapi** sur lesquelles, évidemment, les développeurs ne communiquent pas.

---

- La gestion des API s'effectue via le **Content Type Builder**...il s'agit en fait du coeur de l'application à partir duquel vous pourrez définir les relations entre chaque API, ainsi que les différents champs à ajouter à chacune d'entre elles.

---
**Attention**

Les API créées à partir d'une version plus ancienne de **Strapi** **NE SONT PAS MIGRABLE** vers une version de **Strapi** plus récente...l'information n'est communiquée nulle part sur le site officiel.  

---

# Finalement,
L'outil est bien conçu, surtout si on fait abstraction des différents problèmes que l'on peut rencontrer avec les versions les plus anciennes.  
Je rappelle que cet article n'est pas un troll pour vous détourner de **Strapi**, après tout...c'est un produit développé par des français, encore très jeune...et certainement promis à un grand avenir.  
Je pense que, dans un avenir proche, je ferai un autre article dessus pour présenter plus en détail telle ou telle fonctionnalités *built-in*. N'empêche que, dans l'immédiat, les différents problèmes rencontrés lors de l'installation ou lors de son utilisation ne donnent pas vraiment envie de se pencher dessus plus sérieusement.