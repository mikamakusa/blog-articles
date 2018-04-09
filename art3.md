+++

date = "2018-04-09T10:05:00+02:00"
draft = false
title = "Google Skaffold"

+++

Bonjour à tous ceux qui me lisent. Il est très rare que je fasse la pub d'un outil qui vient à peine de naître, mais dans le cas de **Skaffold**, je dirais que ça vaut le détour.  
Oublions donc le nom bizarre (qui signifie *échaffaud* dans la langue de Shaskespear), attardons nous plutôt sur ses fonctionnalités.

## Qu'est-ce ?  
**Skaffold** est un outils de CD en ligne de commande s'intégrant directement dans **Kubernetes**...Entendons CD non pas *continuous deployment* mais *continuous development*.  
En gros, vous codez localement et il se charge, lui même, de déployer dans votre instance **Kubernetes**. Ce qui peut être très utile pour tester un pipeline lorsque l'on se prépare à déployer en production.

## Fonctionnalités  

- Pas de composant serveur, rien à installer sur l'instance **Kubernetes**,  
- Détecte les updates et *build/push/deploy* automatiquement,  
- Gestion des *tag* image,  
- Supporte tout types d'outils et de workflows,  
- Supporte de nombreux type de composants applicatifs,

**Skaffold** s'adapte aux outils que vous utilisez dans votre workflow et s'y intègre très simplement:

## Commandes  
**skaffold dev**  
"observe" en permance tout update réalisé sur le code, le build et le déploie sur l'instance **kubernetes** cible.
Boucle "continuous build-deploy".  
Affichage des logs des conteneurs déployés.

**skaffold run**
A utiliser pour démarrer le pipeline **skaffold** occasionnelement. Tout comme **Jenkins** (sauf si l'on intègre du *try and catch*), le pipeline s'arrête à la moindre erreur.

## Comment ça fonctionne ?
Avant d'installer **Skaffold**, nous allons avoir besoin des éléments suivants :  
- Un cluster **Kubernetes** (Captain obvious is obvious),  
- **Docker** et un **Docker Registry**

Pour installer **skaffold**, procédons ainsi :  
```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64  
chmod +x skaffold  
sudo mv skaffold /usr/local/bin
```

## Petit Exemple
Commençons pas cloner le repo suivant pour récupérer les exemples: `https://github.com/GoogleCloudPlatform/skaffold`  
Changez de dossier et démarrez **skaffold**:  
```
cd examples/getting-started  
skaffold dev  
```

*Ouput de la commande skaffold dev*
---  
```
Starting build...
Found [minikube] context, using local docker daemon.
Sending build context to Docker daemon  6.144kB
Step 1/5 : FROM golang:1.9.4-alpine3.7
 ---> fb6e10bf973b
Step 2/5 : WORKDIR /go/src/github.com/GoogleCloudPlatform/skaffold/examples/getting-started
 ---> Using cache
 ---> e9d19a54595b
Step 3/5 : CMD ./app
 ---> Using cache
 ---> 154b6512c4d9
Step 4/5 : COPY main.go .
 ---> Using cache
 ---> e097086e73a7
Step 5/5 : RUN go build -o app main.go
 ---> Using cache
 ---> 9c4622e8f0e7
Successfully built 9c4622e8f0e7
Successfully tagged 930080f0965230e824a79b9e7eccffbd:latest
Successfully tagged gcr.io/k8s-skaffold/skaffold-example:9c4622e8f0e7b5549a61a503bf73366a9cf7f7512aa8e9d64f3327a3c7fded1b
Build complete in 657.426821ms
Starting deploy...
Deploying k8s-pod.yaml...
Deploy complete in 173.770268ms
[getting-started] Hello world!
```
---

**skaffold** a effectué les actions suivantes lors de son démarrage :  
- Build d'une image depuis un code source local
- Tag avec sha256
- Précise l'image, définie dans *skaffold.yaml*, dans les manifests **Kubernetes**,  
- Déploie le manifest **Kubernetes** à l'aide de la commande `kubectl apply -f`

Et une fois déployé, on obtient ce qui suit :  
```
[getting-started] Hello world!  
[getting-started] Hello world!  
[getting-started] Hello world!  
```

Ensuite on modifie le fichier *main.go* :  
```
diff --git a/examples/getting-started/main.go b/examples/getting-started/main.go
index 64b7bdfc..f95e053d 100644
--- a/examples/getting-started/main.go
+++ b/examples/getting-started/main.go
@@ -7,7 +7,7 @@ import (

 func main() {
        for {
-               fmt.Println("Hello world!")
+               fmt.Println("Hello jerry!")
                time.Sleep(time.Second * 1)
        }
 }
```

Une fois le fichier sauvegardé, le pipeline **skaffold** est démarré automatiquement, et on obtient ce qui suit :  
```
[getting-started] Hello jerry!
[getting-started] Hello jerry!
```

C'est un outil qui semble assez prometteur bien que très jeune.  
Je vais continuer à le suivre et, pourquoi pas dans un futur relativement proche, vous proposer un article/lab plus poussé sur le sujet.
