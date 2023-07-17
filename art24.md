+++
date = "2023-07-17T22:23:00+02:00"
draft = false
title = "Le hack - Retour d'expérience du salon parisien"
+++

Suite à la **Toulouse Hacking Convention** que j'avais particulièrement apprécié pour son côté technique d'un niveau assez relevé, j'ai eu envie de participer au salon **Le Hack** se déroulant, cette fois-ci, à la **Cité des Sciences et de l'Industrie** à Paris. Le fait que **SQUAD** proposait des invitations à leurs employés fut une bonne occasion d'y aller. Loin de moi l'idée de revenir sur les événements assez chaotiques de ce week-end, qui pourraient même faire l'objet d'un film à grand suspense, je vais plutôt revenir sur les conférences auxquelles j'ai pu assister.  

## Du driver Windows à l'EDR
Lors de ce talk, **Aurelien CHALOT**, *Sysadmin et Security Researcher* chez **Orange CyberDefense** nous a expliqué le fonctionnement des antivirus et le fait qu'ils aient besoin de moyens pour interpréter les actions de ces derniers à l'aide d'une fonctionnalité de l'OS Windows et appelée **Kernels Callbacks**. Cependant pour les entreprises développant les antivirus (**McAfee**, **Symantec**, **Kaspersky**, etc...), cela n'a pas toujours été facile de travailler main dans la main avec **Microsoft** suite au déploiement de**PatchGuard** (sur **Windows XP**).  
En effet, celui-ci empêchait l'accès et la modification du kernel...et donc le fonctionnement des antivirus.  
Depuis **Microsoft** a proposé une alternative : Le framework **WDF** (pour **Windows Driver Framework**) comprenant le **Kernel Mode Driver Framework** et le **User Mode Driver Framework**.  
Le premier donne accès à toutes les fonctionnalités du kernel mais semble difficile et très risqué à prendre en main (à cause de potentiels *kernel panic*) contrairement au second. 

## Lambda Malware: The Hidden Threat In Excel Spreadsheets
Le duo composé de **Yonathan BAUM** et de **Daniel WOLFMAN**, deux chercheurs en cybersécurité, sont venus nous faire un historique des types de malwares que peut contenir un fichier Excel. Depuis les **macros** jusqu'à l'utilisation de **Lambdas**, en passant par le **VBA** et **.NET** avec l'obfuscation d'information dans le but de masquer une charge utile malveillante. Ils nous ont également gratifié d'une petite démonstration technique de *Comment est-ce qu'un fichier Excel contenant un malware peut-il échapper à une détection anti-virus ?*.

## Prototype Pollution And Where To Find Them
Ici, il s'agit de **BitK**, un ambassadeur technique, et de **SakillR**, un développeur R&D, sont venu nous explique ce que sont les *prototypes* en **Javascript** et comment les trouver de manière très simple, même pour les non-initiés (tels que moi, par exemple). Ils ont expliqué également comment polluer ces même prototype dans un but malveillant, afin d'introduire des propriétés exotiques à certains objets conduisant à un comportement inattendu et permettant potentiellement à l'attaquant d'exécuter du code arbitraire.  
L'objectif réel de cette intervention est, via l'utilisation d'un outil développé par eux même, est d'aider à identifier les objets potentiellement dangereux en instrumentant le code source via une démonstration technique ciblant certaines des bibliothèques javascript les plus utilisées.   

## Parasitage De Serveur For Fun And Profit
L'intervention de **Damien CAUQUIL**, nous venant de **Virtualabs** est peut-être la plus...amusante...même si ce n'est pas vraiment le bon mot pour décrire comment parasiter un serveur web en y stockant des fichiers...à l'insu de l'administrateur en charge de celui-ci.  
Evidemment à ne pas reproduire, car potentiellement illegal même si ce n'est pas vraiment dangereux car ne portant pas atteinte aux données présente sur le serveur en question, bien qu'une démonstration soit présentée ainsi que des bonnes pratiques dans le but d'éviter ce type de situations. 

## ZFS Raiders Of The Lost File
**Nikolaos TSAPAKIS**, un ancien de **Citrix** et de **Symantec** (pour ne citer que les plus récents) nous explique, lors de ce talk comment, après avoir voulu jeter un vieux disque dur, il a cherché à récupérer un fichier sur la partition ZFS du disque en question.  
Après avoir créé une image du disque, il a commencé à rechercher le fichier via sa signature...hexadécimale (en se souvenant l'extension et la taille du fichier) et, une fois le bon *header* trouvé, décompresser le bloc de données et ré-écrire le fichier.  
Le tout à l'aide d'un script en Python. 

## DPAPI - Don't Put Administration Passwords In
**Thomas SEIGNEURET** et **Pierre-Alexandre VANDEWOESTYNE**, tous deux provenant de **Login Sécurité**, sont venu nous présenter **DPAPI**, une **API** de **windows** mise à disposition des développeurs dans le but de stocker les secrets des utilisateurs afin qu'ils n'aient pas la partie cryptographique à gérer. Mais ce n'est pas tout, en ayant également un point de vue **securité offensive**, ils nous ont également détaillé et démontré comment un attaquant peut exploiter les possibles failles de cette **API** en expliquant les différentes mesures de protections pouvant être mise en place dans le but de limiter l'exploitations des failles en question.

## OSINT Village - Hardcore OSINT : Reverse Social Media Mechanisms
J'ai également fais un tour du côté du **Village OSINT**, en pensant que l'intervention de **Dmitry Danilov** sur le **Reverse Social Media Engineering** serait passionnante et technique, mais ce ne fut pas vraiment le cas. A part faire le listing des réseaux sociaux, des types d'IDs recensés et des recommandations sur l'utilisations des APIs dans le but de protéger ses données...il n'y a pas eu grand chose à signaler.
