+++
date = "2024-07-15T22:23:00+02:00"
draft = false
title = "Retour sur Le Hack - Edition 2024"
+++ 

Encore un salon que j'avais particulièrement apprécié l'an dernier et, contrairement à la **Toulouse Hacking Convention** qui se déroulait le même jour que l'**AWS Summit**, je ne pouvais pas la manquer pas cette année. Certes ce ne sera pas pour le compte de la même ESN que l'an dernier (et je les ai croisé pendant le salon), mais si **Valeuriad** m'a laissé y aller, c'est probablement par intêret pour les sujets porteurs, à savoir la cybersécurité et le hacking ethique.  
Cette fois-ci, le contexte sera certainement moins chaotique que l'an dernier...mais vu que c'est encore à Paris, je m'attendais à tout (et tout s'est bien passé).  
Les conférences sont nombreuses, toutes aussi intéressantes les unes que les autres, et j'ai dû faire un choix assez cornélien entre plusieurs types de sujets.

## Keynote d'ouverture
Oui...nous sommes des geeks boutonneux et le thème lancé par le remix de **All Your Are Belong To Us** nous le rappelle (accessoirement, le thème provient d'un jeu vidéo de la même génération que votre serviteur). Nous voici sur la vingtième édition, autrefois appelée **La nuit du Hack**, de l'événement organisé par l'association **HZV** et orienté sur le hacking et la cybersécurité.  
Entre bornes d'arcades pour fans de retro-gaming, flipper ayant pour thème la plus iconique des séries autour de l'univers créé par Georges Lucas et de nombreuses conférences et workshops très intéressants, personne ne devrait s'ennuyer sur les trois jours que vont durer le salon.


## PHISHING FOR POTENTIAL: THE “RTFM” GUIDE TO HACKING YOUR BRAIN-FRAME
Le parcours de **Kayley Melton** brise les barrières conventionnelles et peut également servir d'exemple en ce qui concerne l'inclusion de personnes atteintes de divers handicaps. Neuro divergente et se définissant comme non binaire, **Kayley** est parvenue a adapter son environnement à sa personne et à son handicap.  
La présentation a offert des outils pratiques et des pistes en vue d'optimiser son style de travail et ses besoins.


## AI FOR CYBERSECURITY: APPLYING MACHINE LEARNING TO ENHANCE MALWARE ANALYSIS
Cette présentation démontre que, contrairement à l'utilisation seule des moyens de détection classique comme les antivirus, la plus value de l'intelligence artificielle et des modèles de *machine learning*, tels que [**MABEL**](https://github.com/action-ai-institue/MABEL-dataset), permet d'analyser plus profondément les logiciels potentiellement malveillants, les classer ainsi que produire des données et rapports les plus uniformes possible.  


## DÉCOUVERTE DU GROUPE APT-C36 SUR LES RÉSEAUX D’UNE PROFESSION LIBÉRALE RÉGLEMENTÉE
Avant de rentrer dans le vif du sujet, je vais d'abord revenir sur plusieurs acronymes très utilisés dans la cybersecurité :  
- **APT** pour **Advanced Peristent Threat**,  désigne initialement un type de piratage informatique furtif et continu, ciblant une entité spécifique.  
- **Lateral Movement** : Action consistant à se déplacer de machines en machines sur un réseau par un hacker.  
- **IOC** pour **Indicator of Compromise** : il s'agit d'un artefact observé sur un réseau ou une machine indiquant une intrusion informatique.  
- **EDR** pour **Endpoint Detection and Response** : Il s'agit d'une solution de protection de terminaux qui, contrairement aux antivirus, scanne tous les terminaux présent sur un réseau.

Rentrons dans le vif du sujet...Ce talk fut l'occasion de nous expliquer le déroulement d'une attaque, sur plusieurs mois, détectée à l'aide d'IoC ayant permis la corrélation avec un groupe d'attaquant (**Blind Eagle**)...ce dernier ayant même servi à cacher les agissements potentiels d'un autre groupe (**Hagga Threat Actor**), Les actions détectées furent découpées en plusieurs étapes :  
- Campagnes de phishing,  
- Installation de plusieurs keylogger...parfois en plusieurs exemplaires,  
- Installation de plusieurs outils de prise en main à distance,  
- Créations de multiples utilisateurs ayant des droits administrateurs,  
- Mouvement latéraux vers d'autres serveurs, tels qu'Active Directory.  

Durant l'enquête, il s'est avéré qu'une tentative de ransomware était en cours (des outils de cryptage de données avaient déjà été installés et pas encore utilisés)...mais stoppée à temps.


## J'ai testé pour vous : SERIOUS HAC - SERIOUS GAME POUR L’HOMOLOGATION ET L’APPRENTISSAGE DE LA CYBERSÉCURITÉ
Durant cette session de découverte, nous avons pu essayer un jeu de rôle assez particulier.  
Le principe du jeu est le suivant :  
- Deux équipes sont formées, les attaquants et les défenseurs,  
- Le jeu se déroule sur 20 tours.  

### Du côté des défenseurs
- Au départ, les défenseurs choisissent une entité, ce qui leur octroie un personnage. En fonction de ce personnage, une ou plusieurs compétences sont débloquée mais, en contrepartie, cela impacte le nombre de crédit alloué à l'équipe en début de partie.  
- le plateau de jeu se compose de 7 bandes : 2 de préparation, 4 de retour et la dernière signifie que la mesure est en place.  
- En plus du personnage de départ, l'équipe des défenseurs comporte un autre personnage **de base**, ces deux personnages ne coûtent pas de crédit lorsque l'on fait appel à eux pour débloquer une mesure...contrairement aux prestataires (au nombre de sept).  
- Les mesures sont symbolisées par des cartes, il y a en a une petite trentaine, démarrant par la connaissance du système (en quatre niveaux) au maintient en conditions de sécurité (**MCS**), en passant par la ségrégation des réseau ou la sensibilisation des utilisateurs.  
- Chaque cartes **mesures** impose un temps de préparation et un temps de retour (plus ou moins élevé) ainsi que des contraintes relatives au personnage pouvant jouer cette carte (en fonction de la couleur ou de l'initiale sur les cartes) et de son coût en crédit, certaines sont gratuites et peuvent être lancée avec un des deux personnages *principaux*, les autres d'un niveau plus élevées ne peuvent être jouées qu'avec les prestataires...et peuvent s'avérer coûteuses.  

### Du côté des attaquants
- Au départ, les attaquants choisissent également une entité leur octroyant trois personnages. Ils auront également accès à des *prestataires*,  
- L'équipe démarre le jeu sans aucun crédit mais en gagne deux à chaque tours (des revenus illegaux),  
- Leur plateau de jeu est un peu différent, en plus des 7 cases similaire à l'équipe adverse, une case en plus est présente pour indiquer qu'un lancer de dé est nécessaire pour que l'attaque soit réalisée. Dans ce cas là, les deux équipes lancent leur dé. Si les deux équipes sont a égalité (peu importe les bonus), les défenseurs l'emportent.  
- Ils ont également une trentaine de carte symbolisant les attaques en fonction de la **killchain**, depuis l'**OSINT Général** (condition préalable a toute autres actions) à l'attaque DDoS en passant par le *Phishing* ou le *vidage de comptes bancaires*. A l'instar de l'équipe adverse, ces cartes imposent des contraintes de préparation, de retour, liées au personnages pouvant jouer la carte...ainsi que le coût de la carte (pouvant être cumulé à celui du prestataire).

### Le déroulement du jeu
- Le jeu se déroule sur vingt tours,  
- Tous les 5 tours un événement est déclenché par le maître du jeu : les deux équipes devront choisir une carte et annoncer l'effet applicable sur l'équipe adverse (sans annoncer l'effet sur sa propre équipe),  
- Comment expliqué précédemment, lors qu'une carte **attaque** arrive sur la case précédent sa mise en place effective, il faut lancer les dé pour voir si l'attaque est couronnée de succès ou non. Au bout de 3 echecs, la carte retourne en phase préparation,   
- Lors de chaque tour, chaque action placée sur le plateau de jeu avance d'un case, chaque équipe peut lancer une nouvelle action en fonction du budget disponible et des personnages pouvant exécuter l'action,  

### Le Endgame
Le jeu se termine lorsque :  
- Les attaquants ont déclenché une attaque vraiment critique en fonction du système ciblé, selon l'arbe des mesures il s'agit des attaques de la dernière colone (dont **USB Killer** et **Attaque DDoS** font partie),  
- Les attaquants n'ont plus de personnels disponible. Lors de la partie, il se peut qu'ils se fassent arrêter en cas d'echec critique (1 au lancer de dé) lors d'une attaque de type **Red Team**,  
- Les défenseurs n'ont plus de budget pour exécuter des mesures défensives.

Une fois la partie terminée, le maître du jeu fait le bilan avec l'équipe des **défenseurs** pour connaître leur avis sur une possible homologation de leur système d'information en fonction des mesures débloquées, celles qui sont en cours sur le plateau de jeu et celles qui n'ont pas été lancées.

### Mon avis
Pour pouvoir vous faire ce retour d'expérience le plus précis possible sur ce jeu, j'ai testé les deux camps...et gagné à chaque fois (on appelle ça de la chance)...et pourtant, dans le camps des **attaquants**, nous étions assez mal parti car les dieux du hasard semblaient être contre nous :  
- L'OSINT général fut assez long : 6 tours,  
- Nous avons raté également d'autres choses (DDoS, par exemple),  
- Nous avons perdu un attaquant lors d'un événement aléatoire (alors qu'une action que lui seul pouvait effectuer était engagée) mais nous avons gagné un nouveau membre quelques tours plus tard,  
- Mais nous avons également eu plusieurs bonus lors des tours 10 et 15 (plusieurs attaques débloquées d'un coup).

Le jeu est encore en cours de finalisation, l'équilibrage est encore en cours, il reste encore quelques coquilles mais devrait être disponible pour la fin de l'année.
En tout cas, merci à Floralie (pour l'idée du jeu de plateau) et à son mari pour sa contribution.


## SUPPLY CHAIN ATTACK : LE CAS DU REGISTRE PRIVÉ DOCKER
Durant ce talk, [Geoffrey Sauvageot-Berland](https://linkedin.com/in/geoffrey-sb/) - ingénieur cybersécurité chez Orange Cyberdéfense - nous a expliqué l'importance de sécuriser tout les composants d'une usine logicielle...et la registry privée **Docker** en particulier qui, de par sa configuration par défaut, permet des accès anonymes sans aucun contrôle.  
Par conséquent, il devient possible de viser une image Docker, de la modifier en y dissimulant une backdoor via un script afin de compromettre son cycle de développement.


## TROUVER SA PLACE DANS L'INFOSEC
Durant ce talk, [Charlie Bromberg](https://twitter.com/_nwodtuhs) nous explique avec une bonne dose d'humour, tout ce qu'il y a à savoir sur le milieu de l'INFOSEC, que ce soit les études, les jobs disponibles, les différents statuts (Freelance, salariés), les différents types d'entreprises (Public, Privé, etc...), les communautés et les resources disponible sur internet ainsi que le mindset à avoir pour vraiment réussir et s'épanouir dans cet univers.