+++
date = "2023-10-04T22:23:00+02:00"
draft = false
title = "Retour d'expérience IDM Europe 2023 (Utrecht)"
+++ 

Voici un article que j'aurais dû publier l'an dernier, en effet, **Whitehall Media** m'avait invité à l'événement de 2022 (avec un code d'accès réutilisable pour toutes les autres conférences) et je n'avais pas eu le temps de vous proposer un retour d'expérience sur le sujet.  
Initialement je pensais que ce type de conférence axé sur des retours d'expériences d'entreprises clientes de solutions autour des sujets **IAM (Identity Access Management)**, **PIM (Privileged Identity Management)**, **PAM (Privileged Access Management)**, mais il s'avère que les intervenants sont relativement hétéroclytes puisque l'on y retrouve des acteurs du marché de l'**Identity Management**, tels que **PingIdentity**, **Okta**, **BeyondTrust**, **SailPoint** ainsi que des acteurs provenant d'autres domaines, tels que **ABN AMRO Bank**, les laboratoires **ROCHE**, **Vodafone** (cette année) ou **Swisscom**, **Jumbo Supermarket** ou **Firmenich** (l'an dernier).  
Par contre, n'ayant pris de notes sur l'édition précédent, cet article traitera exclusivement de celle qui vient de s'achever à **Utrecht**.

## Will we use IAM thinking to implement CIAM
Voici l'approche de **Bruno Falcao** des Laboratoires **Roche** sur l'implementation d'une solution de **Customer Identity and Access Management** : ne pas aborder l'expérience client de la même manière que celle des employés. Durant son intervention, il a pris en exemple deux cas assez différents l'un de l'autre :  
- Celui de **Patricia**, probablement très jeune et au courant de ce que peut être le mécanisme d'authentification **MFA**, avec un téléphone fourni par l'entreprise.  
- Celui du **Docteur Hans Fritz**, certainement plus très jeune, avec un téléphone relativement ancien et non compatible avec **MFA**.  

Ce cas de figure permet assez facilement d'illustrer pourquoi l'approche IAM n'est pas pertinente pour implémenter un CIAM car les clients n'ont pas les mêmes besoins que les utilisateurs internes de l'entreprise...le tout en effectuant un parallèle avec **Netflix** et leur politique client avortée plus tôt dans l'année.

## Adrift in a sea of Cyber-terror : Are you looking for a life-jacket or a life boat ?
Commencer sa présentation par une photo de ses enfants en maillot de bain et de la mer qui leur fait face dans de différentes condition, c'est un parallèle qu'il fallait oser pour décrire le type de protection à choisir selon le type de menace qui nous fait face...et c'est également l'approche qu'**Hans-Robert Vermeulen** de **Sailtpoint** a choisi pour illustrer son propos.  
Autant dire qu'il fallait oser.  
Parmi les solutions disponibles sur le marché, décrypter le marketing de chaque distributeurs est indispensable car certains offrent plus de fonctionnalités que d'autres et il devient parfois difficile de s'y retrouver. 

## Intro to N.I.S 2 compliance for Privileged Access
La directive N.I.S. 2 est enfin parmi nous, c'est une bonne nouvelle...mais comment garantir la compliance de nos système d'information par rapport à celle-ci ?  
C'est ce que nous explique **Chris Dearden** de **Ping Identity**. Il a commenté chaque paragraphe important, contenant des éléments plus technique que d'autres, en ajoutant que ces éléments techniques peuvent être adressé par **Ping Identity** (et certainement chez d'autres).

## Prevention of Lateral Movement using MFA and Service Account Protection in Active Directory
*Lateral Movement* ? Il s'agit des techniques, utilisées par les hackers estampillés **Red Team** (autrement dit *offensifs*) exploitant des failles de sécurité leur permettant, une fois entrés dans un système d'information, de se déplacer librement à l'intérieur.  
Durant cette intervention, **Frank Leavis** de **Silverfort** nous explique qu'il est possible de réduire (et non d'annuler complètement) les risques, provenant de failles de sécurité inhérents aux services mis en oeuvre (tels qu'Active Directory), à l'aide des mécanismes de **MFA** (multi-factor authentication) et de **service account**.

## MFA Migration Lessons Learned
Selon **Martin Sandren**, son équipe serait la plus détestée de toute l'entreprise. Certes, mais pourquoi ?  
En fonction des contraintes techniques inhérentes à la localisation, son équipe a été contrainte de réviser en profondeur la politique de MFA afin qu'elle puisse s'appliquer à tous et proposer **l'expérience MFA** la plus appropriée à certaines circonstances.  
Par exemple, dans le cas où les SMS ainsi que l'application ne permettraient pas l'authentification multifacteurs, il reste toujours d'autres méthodes telles que les certificats ou la localisation réseau.

## Merger & Acquisitions : How a modern IGA provides a smooth and efficient process
Lorsque plusieurs entreprises fusionnent, leur système d'information est également voué à subir le même sort et le processus peut devenir compliqué.  
Durant son intervention, **Anette Lavu** de **Valmet** nous a révélé avoir fait appel à la puissance de l'outil d'**Administration et de Gouvernance des Identités (IGA)** afin de rendre l'opération plus simple et rapide, en réduisant la durée à environ deux heures.  
Durant cette session, nous avons également eu droit à quelques bonnes pratiques relatives à l'utilisation de la solution d'**IGA** de **Savyint**.