+++
date = "2024-08-12T15:29:49+02:00"
draft = false
title = "Ce qui a ébranlé le monde l'IT en juillet 2024"

+++

Quatre événements ont secoué le monde de l'IT en Juillet 2024, il s'agit de :  
- la panne mondiale provoquée par une mise à jour de Crowdstrike,  
- l'attaque DDoS provoquant une panne chez Microsoft Azure,
- L'exploitation d'une faille de sécurité dans les hyperviseurs VMWare ESXi,
- Le Texas poursuit Meta pour utilisation de données biométriques sans consentement  


## Crowdstrike et la mise-à-jour qui a déclenché une apocalypse.
Comment est-ce arrivé ?  
Cela serait dû a une "simple" erreur de code de Crowdstrike Falcon, celle-ci a provoqué un redémarrage en boucle de plus de huit millions de machines (ordinateurs personnels et serveur) équipé du système d'exploitation de Microsoft. Les versions n'ont pas été communiqué, mais nous savons que les versions les plus anciennes (Windows 95 ou NT) n'ont pas été touchées (1).

Mais qui est Crowdstrike ?  
Peu connu du grand public avant les faits, Crowdstrike est une entreprise de cybersecurité Cloud Native implantée au Texas et fournissant des outils de réponses aux attaques via sa plateforme Crowdstrike Falcon Sensor.

Quels furent les impacts ?  
On estime à plus de huit millions le nombre de machines touchées sans compter les dommages financiers non négligeables au sein de nombreux secteurs économique dont le plus touché fut celui de la santé (deux milliards de dollars), mais les secteurs bancaire, des transports et informatiques ont également subit le contrecoup de la panne.  
Au final, les pertes financières devraient s'élever entre cinq et dix milliards de dollars, dont un peu moins de la moitié pour les seuls secteurs de la santé et bancaire.  
Bien que le secteur de la santé serait le plus touché...le secteur accusant la perte par entreprise la plus élevée est celui des transports (143 millions de dollars contre 64). 
L'impact a été tellement significatif pour les compagnies aériennes que l'effet s'est fait ressentir pendant près d'une semaine avec plus quarante mille vols annulés ou retardés, quelques soit la distance (2).   
Ensuite en terme d'image, Crowdstrike a tenté de limiter les dégats en souhaitant dédommager leur client via une "carte cadeau" de dix dollars (3) ainsi que par l'intermédiaire du CEO, George Kutz, auprès de la Sécurité Intérieure de Etats Unis d'Amérique (4)...le tout entraînant une baisse de 32% du cours de l'action Crowdstrike et une plainte des investisseurs pour fausses déclarations à propos de leur plateforme Falcon (5) (6).

Sources :  
- (1) : https://www.lemondeinformatique.fr/actualites/lire-comment-windows-95-a-preserve-des-entreprises-de-la-panne-crowdstrike-94355.html  
- (2) : https://www.cio.com/article/3476789/crowdstrike-failure-what-you-need-to-know.html#:~:text=On%20July%2024%2C%20CrowdStrike%20reported,Channel%20File%20291%20content%20update  
- (3) : https://techcrunch.com/2024/07/24/crowdstrike-offers-a-10-apology-gift-card-to-say-sorry-for-outage/?guccounter=1  
- (4) : https://www.theregister.com/2024/07/23/crowdstrike_ceo_to_testify/  
- (5) : https://www.lesechos.fr/tech-medias/hightech/apres-la-panne-mondiale-crowdstrike-poursuivi-en-justice-par-ses-actionnaires-2111878  
- (6) : https://securityaffairs.com/166480/security/investors-have-sued-crowdstrike.html?utm_source=dlvr.it&utm_medium=tumblr

## Une attaque DDoS aurait provoqué une panne chez Microsoft
En toute fin de mois de Juillet, certains services de Microsoft ont, de nouveau, subit une panne durant une petite dizaine d'heures. Nous avons appris par la suite que seuls certains services d'Azure (à savoir App Service, Application Insights, IoT Central, Log Search Alerts, Policy et le portail Azure lui-même) de même que certains services de Microsoft 365 (Admin Center, Intune, Entra, PowerBI et Power Platform) venaient d'être victime d'une attaque en déni de service.  
Il s'avère, en fait, que ce n'est pas vraiment l'attaque en elle-même qui a causé le plus de dégats mais plutôt les systèmes de réponse à l'attaque qui ont amplifié l'impact au lieu de l'attenuer.  

Source : https://www.securityweek.com/microsoft-says-azure-outage-caused-by-ddos-attack-response/


## Une faille de sécurité dans l'hyperviseur VMWare ESXi
Vulnerabilities and Common Exposures, ou Vulnérabilités et Expositions Communes - en Français - est un dictionnaire des informations publiques relatives à la sécurité maintenu par [Mitre](https://attack.mitre.org/) et soutenu par le [Département de la Sécurité Intérieur de Etats Unis](https://www.dhs.gov/).  
Notée CVE-2024-37085, il s'agit d'une faille de sécurité observée dans les hyperviseurs VMWare ESXi au sein d'un domaine Active Directory, et est utilisée par plusieurs opérateurs de ransomware pour effectuer des élévations de privilèges et, ainsi, chiffrer le système de fichier et par conséquent corrompre le fonctionnement général de l'hyperviseur, accéder aux machines virtuelles et extraire des données ou effectuer des mouvements latéraux.  
En matière de cybersécurité, un mouvement latéral est une action visant à se déplacer du serveur infiltré vers une machine cible dont l'importance stratégique pourrait être plus importante.  

Sources :  
- https://www.microsoft.com/en-us/security/blog/2024/07/29/ransomware-operators-exploit-esxi-hypervisor-vulnerability-for-mass-encryption/  
- https://arstechnica.com/security/2024/07/hackers-exploit-vmware-vulnerability-that-gives-them-hypervisor-admin/

## Meta contre l'état du Texas
Le point de départ de cette affaire :  
En 2022, le procureur général du Texas, Ken Paxton, a poursuivi en justice la société Meta (Facebook, Instagram, Whatsapp, etc...) pour capture illegale de données biométriques. Le champs d'action de la plainte ne s'est limité qu'au Texas mais il est tout à fait probable que la société incriminée ait agit ainsi dans d'autres parties du monde.  
Rappel des faits :  
Le logiciel de reconnaissance faciale développé par Meta et introduit en 2011 en tant que fonctionnalité de suggestion de tags...scannait et enregistrait diverses données biométriques à partir des photos téléchargées sur Facebook (et probablement sur Instagram également). 
Il faut savoir qu'au Texas, la loi CUBI (pour Capture or Use of Biometric Identifier), votée en 2007, interdit à toute personne, morale ou physique, de capturer les identifiants biométriques d'un individu à des fins commerciales à moins d'en informer explicitement ce dernier et d'obtenir son consentement éclairé.  
Les identifiants biométriques comprennent le scan rétinien, l'empreinte vocale, la géométrie faciale, etc...
Loin devant le record précédent s'élevant à 390 Millions de dollars déténu par Google contre une quarantaine d'états (fin 2022), avec cet accord dont le montant est d'1,4 Milliard de dollars, le procureur général du Texas à souhaité souligner l'importance de tenir les multinationales du secteur de l'IT responsable de toute violations des lois sur la confidentialité des données.  

Sources :  
- https://gbhackers.com/meta-paid-a-1-4-billion-settlement/  
- https://www.texasattorneygeneral.gov/news/releases/attorney-general-ken-paxton-secures-14-billion-settlement-meta-over-its-unauthorized-capture  

Loi **CUBI** :  
- https://www.texasattorneygeneral.gov/consumer-protection/file-consumer-complaint/consumer-privacy-rights/biometric-identifier-act  
- https://statutes.capitol.texas.gov/docs/bc/htm/bc.503.htm

