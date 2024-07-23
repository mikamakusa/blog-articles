+++
date = "2024-07-23T15:29:49+02:00"
draft = false
title = "Un nouveau neurone"

+++

La révolution de l'IA générative en 2016 avec **Tay** (de **Microsoft**) puis l'avènement de **ChatGPT** en 2018 a bouleversé les usages...si bien que Apple a finalement décidé de l'intégrer a ses futurs appareils (sans oublier les potentielles failles de sécurité inhérentes a ce type d'usage). Bientôt nous verrons débarquer sur le marché de nombreux ordinateurs, smartphones ou tablettes intégrant cette nouvelle technologie *révolutionaire*.  
Mais la révolution passe-t-elle par l'utilisation d'un service en ligne ou par un programme complètement personnalisable et local ?

N'étant pas un grand fan de l'intelligence artificielle générative sans pour autant être traumatisé par les films de la saga Terminator, j'ai tout de même décidé de me pencher sur le sujet et j'ai croisé la route de **Jan**.

**Jan** est un logiciel opensource, installable localement sur :
- [**Windows**](https://github.com/janhq/jan/releases/download/v0.5.2/jan-win-x64-0.5.2.exe),
- [**Linux Debian**](https://github.com/janhq/jan/releases/download/v0.5.2/jan-linux-amd64-0.5.2.deb) ou [**Linux AppImage**](https://github.com/janhq/jan/releases/download/v0.5.2/jan-linux-x86_64-0.5.2.AppImage) et
- [**MacOS Intel**](https://github.com/janhq/jan/releases/download/v0.5.2/jan-mac-x64-0.5.2.dmg) ou [**MacOS Silicon**](https://github.com/janhq/jan/releases/download/v0.5.2/jan-mac-arm64-0.5.2.dmg), inclue une interface relativement simple à utiliser, un modèle d'API mais pas de LLM prédéfini. Il est possible de le connecter aux modèles les plus reconnus tels que ceux de :
- **OpenAI**,
- **Mistral**,
- **Nvidia**
- ou bien **Groq**.

Cependant si vous privilégiez la transparence et la tracabilité des sources d'entraînement ainsi que le respect de votre vie privée, il est recommandé d'utiliser **Jan** avec votre propres données, de manière déconnectée...par contre, si votre ordinateur est un peu ancien, ne vous attendez pas à des performances digne d'une formule 1.

## L'interface
L'interface du logiciel est très épurée : quatres icônes sur la gauche, autant en haut (avec les classiques icônes de fermeture d'application, d'aggrandissement et de rétrécissement de fenêtre), les liens vers l'espace discord et git ainsi que le *moniteur système* pour veiller à la consommation de ressources par le LLM en cours d'utilisation et un lien pour accéder aux journaux d'événement de l'application.

*mon avis* : A part le visionnage des journaux d'événements que j'aurais intégré directement dans l'interface, je n'ai aucune objection particulière.

### Le HUB
Il s'agit de la fonctionnalité qui servira à télécharger les différents modèles disponibles avec des indications sur le poids du modèle et, très important, sur sa rapidité par rapport au matériel utilisé :
- *slow on your device*,
- *Not enough RAM*,
- *Recommended*

Il est évidemment important de sélectionner le modèle le plus adapté a sa configuration sous peine de mettre son ordinateur à genoux, de le voir rendre ses tripes et de devoir redémarrer violemment sa machine parce qu'elle ne reponds plus aux différents stimulis.  
La connexion à Huggingface n'étant pas encore implémentée nativement, vous avez également la possibilité d'importer des modèles provenant de cette plateforme...ou bien des modèles *textuels* permettant de générer du texte ou des ligne de code dans différents langages.

### Thread
Cette partie de l'interface se décompose en 4 parties, les deux premières devraient être assez familières aux utilisateurs de ChatGPT puisqu'il s'agit d'une colonne contenant les différents prompts et la seconde affiche la *conversation* avec le modèle. Les deux parties restantes, à savoir **Assistant** et **Modèle** correspondent respectivement à la contextualisation des prompts et à la sélection du modèle que vous souhaiterez utiliser...ainsi qu'aux *paramètres d'inférence*, en l'occurence le nombre de jetons maximum ainsi que la *temperature*, plus elle sera proche de zéro et moins l'agent conversatiolnel sera sensible aux hallucinations.

### Local API Server
**JAN** peut également faire office de serveur HTTP mais les modes **Thread** et **Local API Server** ne peuvent absolument pas être utilisés en même temps

### Settings
Certaines pages de la partie settings se ressemblent toutes, notamment *my models* et *extensions*. Ces deux pages regroupent les fournisseurs de modèles préconfigurés, tels que **Nvidia**, **OpenAI**, **Anthropic**, **Cohere** ou bien **Groq** avec un lien vers leur configuration (symbolisée par une roue crantée), les autres pages d'options réellement utiles sont les suivantes :
- **Appearance** : Sélection du thème d'affichage (Dark Dimmed, Joi Dark, Joi Light, Night Blue), Thème d'interface (disponible uniquement pour Joi Dark et Light),
- **Keyboard Shortcuts** : Les différents raccourcis clavier,
- **Avanced Settings** : Activation du mode expérimental (qui supprime les options *clear logs* et *reset to factory default*), **GPU Acceleration** (très utile si vous êtes équipé d'un GPU Nvidia relativement récent), **Jan Data Folder** (dossier de stockage des messages, modèles et autres données utilisateur), Configuration du proxy et du comportement du logiciel face aux certificats TLS/SSL,
- **Model Management** : Y renseigner le token de votre compte Hugginface...plateforme sur laquelle vous pouvez également accéder au LLM disponible dans l'application (entre autre),
- **System Monitoring** : Vous pouvez désactiver la journalisation de l'application et définir un intervale de secondes entre deux remise à zéro des journaux d'événements.

### Faisons le grand saut
Puisque les modèles sont légion, choisissons-en un de préférence assez léger, puisqu'il risque de faire ramer ma machine.  
Mon choix s'est donc porté sur TinyLlama Chat 1.1B Q4.  
Disponible également sur la plateforme HuggingFace, ce modèle a été entraîné sur 3000 milliards de jeton durant 90 jours à partir de Septembre 2023.  
Le dépôt Github du projet est [ici](https://github.com/jzhang38/TinyLlama).  
Cependant, vous pouvez en utiliser d'autres parmi la liste des modèles du **Hub** de l'application, certains sont téléchargeable et utilisable en local, d'autres nécessitent d'être connectés en permanence (que ce soit à HuggingFace ou à un autre service).


## Au final
De ce que j'ai pu tester, le modèle est assez réactif.  
Le logiciel est certes relativement jeune mais semble assez prometteur.  
Pouvoir utiliser localement et sans être connecté en permanence à un service d'IA Générative évite d'avoir à partager ses données confidentielles et peut être utilisé pour toute sortes d'actions, en fonction du modèle sélectionné.  
Il semble possible d'utiliser un modèle *fait maison* à condition qu'il soit au format **gguf** (et donc, provenant de la plateforme **HuggingFace**).  