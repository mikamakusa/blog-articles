+++
date = "2024-01-09T22:23:00+02:00"
draft = false
title = "Découverte : Trellis"
+++ 

Un vent de fraicheur souffle depuis environ deux ans sur la manière de créer des images **Docker** : **Trellis**.  

# Kesako ?
C'est un outil d'intégration et déploiement continu...portable.  
Le principe de **Trellis** est de permettre de définir des *Dockerfiles* ainsi que les pipelines d'intégration et déploiement en *Typescript* et de les exécuter absolument n'importe où.

# Des prérequis ?
Evidemment il y a quelques prérequis à l'utilisation d'un tel outil :  
- **deno**, le runtime **javascript/typescript** base sur **javascript** version 8 et sur **Rust** et disponible depuis Mai 2018,  
- **docker**, evidemment,  
- Et...c'est tout.  

# Les mains dans le cambouis...
## Deno
Pour installer **deno**, rien de très compliqué...Il est disponible aussi bien sous forme de paquet **nix** (Pour [NixOS](https://nixos.org/)), [**asdf**](https://asdf-vm.com/), [**Cargo**](https://crates.io/crates/deno), exécutable **shell** ou [**conteneur Docker**](https://github.com/denoland/deno_docker).  
- Nix : `nix-shell -p deno`  
- asdf : `asdf plugin-add deno https://github.com/asdf-community/asdf-deno.git && asdf install deno latest`  
- Cargo : `cargo install deno --locked`  
- shell : `curl -fsSL https://deno.land/install.sh | sh`  

## Trellis
L'installation de **Trellis** est...tout aussi enfantine avec la commande suivante :  
```shell
deno install \
    --allow-run=docker \
    --allow-net \
    --allow-write \
    --allow-env \
    --allow-read \
    https://deno.land/x/trellis@v0.0.7/cli.ts
```

# Création d'un pipeline
## L'image
Comme tout pipeline d'intégration/déploiement, on commence par la définition de l'image avec laquelle toutes les commandes vont s'exécuter :  
```javascript
import { Image } from "https://deno.land/x/trellis@v0.0.6/mod.ts";

const UBUNTU_VERSION = "20.04";

export const buildStage = Image.from(`ubuntu:${UBUNTU_VERSION}`)
  .workDir("/root")
  .aptInstall([
    "curl",
    "wget",
    "jq",
    "git",
  ]);
```

## Le pipeline
Une fois l'image définie, on crée les étapes du pipeline...de la même manière qu'avec **Gitlab-CI** ou **Github Actions**.  
```javascript
import { build, Image, run } from "https://deno.land/x/trellis@v0.0.6/mod.ts";
import { buildStage } from "./mod.ts";

export default async function runChecks() {
  await build(buildStage);

  const checkCurl = Image.from(buildStage).run(
    "curl --help",
  );
  const checkJq = Image.from(buildStage).run(
    "jq --help",
  );
  const checkGit = Image.from(buildStage).run(
    "git --help",
  );
  const checkWget = Image.from(buildStage).run(
    "wget --help",
  );

  await Promise.all([
    run(checkCurl),
    run(checkJq),
    run(checkGit),
    run(checkWget)
  ]);
}
```  

# Quelques commandes
Quelques commandes bien utiles sont disponible avec **Trellis** :  
- Lister les images ainsi que les tâches du pipeline : `trellis ls`,  
```bash
>>> trellis ls mod.ts
Images:
- buildStage (trellis build --target buildStage)
```  
- Générer un Dockerfile à partir du pipeline : `trellis preview`,  
```bash
>>> trellis preview --target buildStage
#syntax=docker/dockerfile:1.4

FROM ubuntu:20.04 AS stage-0
WORKDIR /root
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked --mount=type=cache,target=/var/lib/apt,sharing=locked apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends curl git jq
```  
- Créer l'image Docker définie dans le pipeline : `trellis build`,  
```bash
>>> trellis build --target buildStage
[+] Building 0.6s (11/11) FINISHED
 => [internal] load build definition from Dockerfile                                                 0.0s
 => => transferring dockerfile: 335B                                                                 0.0s
 => [internal] load .dockerignore                                                                    0.0s
 => => transferring context: 2B                                                                      0.0s
 => resolve image config for docker.io/docker/dockerfile:1.4                                         0.2s
 => CACHED docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a858632727cf7a112fbf  0.0s
 => [internal] load build definition from Dockerfile                                                 0.0s
 => [internal] load .dockerignore                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                      0.2s
 => [stage-0 1/3] FROM docker.io/library/ubuntu:20.04@sha256:35ab2bf57814e9ff49e365efd5a5935b6915ee  0.0s
 => CACHED [stage-0 2/3] WORKDIR /root                                                               0.0s
 => CACHED [stage-0 3/3] RUN --mount=type=cache,target=/var/cache/apt,sharing=locked --mount=type=c  0.0s
 => exporting to image                                                                               0.0s
 => => exporting layers                                                                              0.0s
 => => writing image sha256:17f750ba9a4becf38ce4d584d0de4793bfd6a8139674c3b332cdcdf6525ea8d9         0.0s
 => => naming to docker.io/trellis/db112e211de238c035a9fd3bbcbd5c417aafc5ee96a8c24d99d4caf81a759903  0.0s
√ Build: trellis/db112e211de238c035a9fd3bbcbd5c417aafc5ee96a8c24d99d4caf81a759903
```
- Exécuter une ou plusieurs tâches du pipeline : `trellis run`,  
```bash
>>> trellis run tasks.ts
[+] Building 1.1s (13/13) FINISHED
 => [internal] load build definition from Dockerfile                                                 0.0s
 => => transferring dockerfile: 335B                                                                 0.0s
 => [internal] load .dockerignore                                                                    0.0s
 => => transferring context: 2B                                                                      0.0s
 => resolve image config for docker.io/docker/dockerfile:1.4                                         0.5s
 => [auth] docker/dockerfile:pull token for registry-1.docker.io                                     0.0s
 => CACHED docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a858632727cf7a112fbf  0.0s
 => [internal] load .dockerignore                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                      0.3s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                        0.0s
 => [stage-0 1/3] FROM docker.io/library/ubuntu:20.04@sha256:35ab2bf57814e9ff49e365efd5a5935b6915ee  0.0s
 => CACHED [stage-0 2/3] WORKDIR /root                                                               0.0s
 => CACHED [stage-0 3/3] RUN --mount=type=cache,target=/var/cache/apt,sharing=locked --mount=type=c  0.0s
 => exporting to image                                                                               0.0s
 => => exporting layers                                                                              0.0s
 => => writing image sha256:17f750ba9a4becf38ce4d584d0de4793bfd6a8139674c3b332cdcdf6525ea8d9         0.0s
 => => naming to docker.io/trellis/adf8a603d1ab539848d89f68491e1b9213c1ca498f3f68d871e1b59c4c7de601  0.0s
√ Build: trellis/adf8a603d1ab539848d89f68491e1b9213c1ca498f3f68d871e1b59c4c7de601
√ Run: git --help
√ Run: jq --help
√ Run: curl --help
√ Run: wget --help
```

# Mon avis
Etant un *early adopter* de **Docker** et ayant observé son évolution...et son lent déclin au sein de l'écosystème **Kubernetes**, je me pose énormément de questions concernant **Trellis** de part son originalité et sa jeunesse toute relative. Pour un non développeur, le fait de devoir se pencher sur une technologie inconnue avec un ticket d'entrée assez élevé peut également en freiner l'adoption alors que la création d'un Dockerfile de manière classique est simplifiée grâce à la syntaxe en elle-même. 