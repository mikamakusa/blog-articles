+++
date = "2023-04-24T22:23:00+02:00"
draft = false
title = "Retour sur la Toulouse Hacking Convention"
+++

## Retour sur deux jours de conférences
Le salon Toulouse Hacking Convention s’est tenu sur deux jours, les 20 et 21 Avril 2023 dans le cadre de l’auditorium Marthe Condat, au sein de l’Université Paul Sabatier de Toulouse.
Lors des quinze sessions, entre rétro ingénierie physique, hacking de matériel, démonstrations techniques et retours d’expériences relatifs à certaines situations rencontrées sur le terrain.

Bien que j'ai pu assiter à toutes les conférences...excepté à celle d'Axelle Apvrille (pour cause de bouchons sur la route), je ne vais pas revenir sur toutes mais uniquement sur celles qui m'ont le plus marquées.

### Mask roms and masks of abstraction
Histoire de mettre un peu d’ambiance tôt le matin, Travis Goodspeed, rétro-ingénieur de l’est du Tennessee, nous a expliqué sa recette pour prendre des micropuces en photo, en extraire les bit physique et les convertir en octets…le tout à l’aide de divers acides (fluorhydrique, nitrique, peroxynitrique, sulfurique) d’un microscope et des logiciels adéquats.
Évidemment à ne pas reproduire, cette technique de rétro-ingénierie physique est peut-être la meilleure façon de comprendre les nouvelles technologies aux plus bas niveaux d’abstraction…tout comme les cartouches de vieux jeux vidéo.

### Automating the extraction of secrets stored inside CI/CD
Le duo de chez Synacktiv, Théo Louis-Tisserand et Hugo Vincent, a expliqué l’importance de bien sécuriser les informations sensibles telles que les identifiants et mots de passe lors de l’utilisation d’outils de CICD.
Les informations en question transitent dans les pipelines d’intégration/déploiements de manière extrêmement peu sécurisée et leur extraction en devient très simple.
A ce titre, ils ont également développé un outil pour faciliter le travail d’extraction de ces données.
En parallèle, ils ont également rappelé l’importance d’utiliser des tokens dans le but de sécuriser au maximum les échanges entre applications et d’éviter d’injecter les mots de passe dans le code applicatif.

### Hash cracking : automation driven by laziness, 10 years after
Ou comment à l’aide d’un outil codé par lui-même, David Soria nous a présenté les différents types de mots de passe, du plus simple au plus complexe, qu’ils soient prédictible ou non, craquable facilement ou l’aide de techniques ou d’outils d’un niveau assez élevé…et en est arrivé à coder son propre outil intégrant et combinant à peu près toutes les stratégies de mot de passe afin de percer, à la volée, un grand nombre de secrets.
Depuis il utilise ses compétences et cet outil auprès des entreprises désireuses de connaître les vulnérabilités de leur système d’information.


### Weaponinzing ESP32 RF Stacks
En découvrant l’ESP32, une série de microcontrôleurs de type SoC, intégrant la gestion du Bluetooth et du Wifi, Romain Cayre et Damien Cauquil se sont également demandés jusqu'où il était possible d’aller avec.
Par exemple espionner/renifler les communications Bluetooth, faire supporter d’autres protocoles que ceux prévu à l’origine par le matériel, en faire un outil de hacking…ou tout plein d’autres choses “un peu exotiques”.
Après leur intervention, la réponse semble être positive : ce type de contrôleur, entre de “bonnes mains” est capable de tout.


### Hammerscope : Observing dram power consumption using rowhammer
Venu tout droit des labos d’Intel, le duo israélien Yaakov Chien et Arie Haenel nous ont présenté Hammerscope, une technique permettant de mesurer la consommation d’énergie des modules de RAM grâce à Rowhammer.
Pour rappel, Rowhammer est une vulnérabilité matérielle affectant une grande partie des systèmes modernes. A haut niveau et en effectuant certains schémas d’accès à la mémoire un attaquant peut y insérer des bits sans même les écrire.
Plus d’informations sont disponible [ici](https://angelosk.github.io/Papers/2022/hammerscope.pdf)


### Why there is more to today’s attacks against online games than meets the eye
Probablement la moins technique de toutes les interventions de cette événement, Marc Dacier revient sur les deux principales techniques de triche dans les jeux vidéos en ligne et nous explique pourquoi elles sont relativement facile à mettre en oeuvre et difficile à contrer aussi bien pour les joueurs que pour les opérateurs de ces mêmes jeux…le tout en ayant biensûr fait de la publicité quasi gratuite pour l’université des sciences et technologies d’Arabie Saoudite.


### A study on Windows authentication and Proxy-EZ
Un retour en force des ninjas de Synactiv avec une présentation d’un outil développé en interne. Proxy-EZ est un proxy HTTP gérant toutes les authentifications HTTP à la place de l’utilisateur et supporte de nombreux protocoles, tels que kerberos, pass-the-hash-overpass-the-hash ou NTLM EPA.
Plus d’informations [ici](https://github.com/synacktiv/Prox-Ez)


### Software defined vehicle security-challenges, risks and rewards
Fort de son expérience chez Renault autour des modèles de véhicules électriques, dans lesquels l’informatique et le Cloud ont évolué jusqu’à prendre une place de plus en plus important, Redouane Soum revient sur l’émergence du Software Defined Véhicule et comment cette évolution s’accompagne également de potentiels risques liés à la cybersécurité relative à des véhicules de plus en plus connectés et, par conséquent, à la sécurité de leurs utilisateurs.

### Les Rumps
En gros : des Speed talks.
Impossible d'aborder un spped talk en particulier...car en 5 min difficile d'aborder un sujet en profondeur.  
Certains ont assez bien survolé leur sujet.  
Palme d'or à l'intervention de l'ANSSI qui a duré 15 secondes...pour annoncer qu'ils recrutent.