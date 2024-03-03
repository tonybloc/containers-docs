# Red Hat OpenShift Development I : Introduction to containers with Podman

Préparation à l'examin `Red Hat Certified Specialist in containerized application`


OCI : Open Container Initiative

Manulpulation avec Podman:
- CLI
- Api
- Podman Desktop

## Commandes

### Gestion des images

Utilisation de **Quay.io** pour stocker nos propres images. Docker/Podman
Recherche les images selon le registre paramétré dans `/etc/containers/registries.conf`
### Gestion des conteneur

#### Mise en réseaux

Par defaut les conteneurs sont liée au réseau nommé `podman` et peuvent l'utiliser pour communiquer entre eux.
Cependant dans le cadre d'une application web par exemple. Il est fort probable que vous ayez besoin de créer d'autre réseau, pour permettre séparer les communication entre : 
UI - API - Datanase
```bash 
$ podman network create [name]
```
Crée un réseau Podman. Cette commande accepte différentes options pour configurer les propriétés du réseau, dont l’adresse de la passerelle, le masque de sous-réseau et l’utilisation d’IPv4 ou d’IPv6.

```bash 
$ podman network ls
```
Liste les réseaux existants et présente brièvement chacun d’eux. Les options de cette commande incluent divers filtres et un format de sortie permettant de lister d’autres valeurs pour chaque réseau.

```bash 
$ podman network inspect
```
Génère un objet JSON détaillé contenant des données de configuration pour le réseau.

```bash 
$ podman network rm
```
Supprime un réseau.

```bash 
$ podman network prune
```
Supprime tous les réseaux qui ne sont utilisés actuellement par aucun conteneur en cours d’exécution.

```bash 
$ podman network connect [network] [container]
$ podman network disconnect [network] [container]
```
Connecte/Déconnecte un conteneur en cours d’exécution à un réseau existant ou effectue une connexion à partir de celui-ci. Vous pouvez également connecter des conteneurs à un réseau Podman lors de leur création à l’aide de l’option `--net`. Si le conteneur est déjà en cours d'exécution, il est possible d'utiliser cette commande pour liée le conteneur à un nouveau réseaux.

Par defaut, avec le réseau `podman`, le service DNS est désactivé. Lorsque le service DNS est actif, le nom d'hôte du conteneur correspond au nom du conteneur (dans le réseau)

#### Accès à un conteneur


#### Cycle de vie d'un conteneur

![Schema Cycle de vie](https://rol.redhat.com/rol/static/static_file_cache/do188-4.12/basics/lifecycle/assets/images/basics/lifecycle-actions.svg)

### Résumé des commandes

| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ podman -v` | version de podman | |
| `$ podman pull name:tag` | Récupère une image depuis un registre | La configuration de registre est spécifiée dans `/etc/containers/registries.conf`  |
| `$ podman images` | Liste toutes les images récupérés en local | |
| `$ podman run [options] container [commande...]`  | Execute un processus dans un conteneur en cours d'execution        |  `--name=toto` attribue un nom au conteneur. `--rm` Supprime automatiquement le conteneur lors de sa fermeture. `-p host-port:container-port` Map un port du système hot avec un port du conteneur. `-d` permet d'executer le conteneur en arrière plan (libérant ainsi le terminal). `-e NAME=value` Détermine un variable d'environnement dans le conteneur. `--net networkA,networkB` Associer le conteneur à un réseau. `--volume /path/on/host:/path/in/container:OPTIONS {imagename}` associer un volume au conteneur, voici les options `OPTION` : `:Z` `:z`. `--mount type=TYPE,source=/path/on/host,destination=/path/in/container` Attache un volume au conteneur `TYPE`: *bind*, *volume*, *tmpfs* |
| `$ podman create [options] container`  | Creation d'un conteneur        |  Aligné à droite |
| `$ podman start [options] container`  | Execute un conteneur        |  Aligné à droite |
| `$ podman pause [options] container`  | Mets en pause tous les processus d'un conteneur en cours d'excution         |    Aligné à droite |
| `$ podman unpause [options] container`  | Désactive la mise en pause de tous les processus d'un conteneur          |    Aligné à droite |
| `$ podman kill [options] container`  | Détruit un conteneur en cours d'execution             |   Aligné à droite |
| `$ podman stop [options] container`  |   Stop l'execution d'un conteneur        |  `--all` ou `-a` pour stoper l'execution de tout les conteneurs. `--time` pour définir le temps d'attente avant la l'arret du conteneur |
| `$ podman restart [options] container`  | Redémarre un conteneur          |    Aligné à droite |
| `$ podman rm [options] container`  | Supprime un conteneur stopé          |    `--force` ou `-f` force la suppression d'un conteneur en cours d'execution ou mise en pause |
| `$ podman rmi [options] container`  | Supprime une images de l'espace de stockage local          |    Aligné à droite |
| `$ podman exec [options] container [command ...]`  | Execute un processus dans un conteneur en cours d'execution          | `--env` ou `-e` défini une variable d'environnement (temporaire). Pour rentre la variable d'environnement persistance utilier l'option `-e` de `$ podman run`. `--interactive` ou `-i` indique au conteneur d'accepter les entrées utilisateur. `-tty` ou `-t` attache un pseudo terminal au conteneur. `--latest` ou `-l` execute la commande dans le dernier conteneur créée. On combine l'ensemble de ces options pour ouvrir un terminal interactive dans un conteneur : `$ podman exec -til /bin/bash`. `--net` rattache un réseau au conteneur |
| `$ podman inspect [options] container`  | Affiche la configuration d'un conteneur ou d'une image. Paramétrage réseaux, usage CPU, variable d'environnement, status, mappage de port, volumes, ...         | Pour simplifier la recherche d'information dans la configuration du conteneur, on peut utiliser l'option `--format` et en utilisant l'annotation GO avec `{{` et `}}`. Exemple `$ podman inspect --format='{{.State.Status}}' 1a2de915f` |
| `$ podman ps [options]`  | Liste les conteneurs | `-all` ou `-a`: Affiche également les conteneurs stopés. `--format=json` format l'output de la commande |
| `$ podman rename [options] container`  | Modifie le nom d'un conteneur          |    Aligné à droite |
| `$ podman stats [options] container`  | Affiche les statistique en temps réel associées aux ressources consommées par le conteneur          |    Aligné à droite |
| `$ podman logs [options] container`  | Recherche les logs d'un conteneur          |    Aligné à droite |
| `$ podman top [options] container`  | Affiche les processus en cours d'execution d'un conteneur          |    Aligné à droite |
| `$ podman cp [option] [container:]source [container]destination` | Copie un fichier dans un conteneur | Récupère les fichiers de logs d'un contenu `$ podman cp a3db6c810:/tmp/logs .` (. correspond au repertoire de travail courent) |
| `$ podman network create {networkname}` | Création d'un réseau | |
| `$ podman network rm {networkname}` | Suppression d'un réseau | |
| `$ podman network prune {networkname}` | Suppression des réseaux non utilisés| |
| `$ podman network ls` | Lister l'ensemble des réseaux| |
| `$ podman network connect {networkname} {containername}` | Connecter un conteneur un réseau | |
| `$ podman network disconnect {networkname} {containername}` | Deconnecter un conteneur d'un réseau | |
| `$ podman port --all` | liste les mappage de port de tout les conteneurs | |


### GESTION DE REGISTRE
| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ skopeo copy [transport:image] [transport:image]`| | Copier l'image d'un repository en local `$ skopeo copy docker://registry.access.redhat.com/ubi9/nodejs-18 dir:/var/lib/images/nodejs-18`|
| `$ podman login {registry}` | Se connecter à un registre | Les informations d'identification sont enregistré dans un fichier `${XDG_RUNTIME_DIR}/containers/auth.json`. Utilisation de `$ echo -n dXNlcjpodW50ZXIy pipe base64 -d` pour décoder la clé |
| `$ podman search {imagename}` | | |
| `$ podman push quay.io/QUAY_USER/{imagename}` | Publication d'une image | |
| `$ podman search {imagename}` | | |

### GESTION DES IMAGES

`$ printenv VARIABLE` : Affiche la valeur d'une variable d'environnement
`$ export VARIABLE='value'` : Défini la valeur d'une variable d'environnement

| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ podman build -t {imagename:tag} {file}` | Compiler une image | `--file` ou `-f` pour spécifier le fichier à compiler. `--tag` ou `-t` pour définir le nom du l'image et du tag. `--build-arg key=example-value`pour définir la valeurs des arguments |
| `$ podman image inspect [registry/user/image:tag]` | Inspect la configuraiton de l'image| `--format` |
| `$ podman image rm` | Supprimer une image du registre local | `--all` suprime toutes les images |
| `$ podman image prune` | Supprime les image inutilisé | `-a` pour supprimer ceux suspendues |

| `$ podman image tree {image:tag}` | Afficher la liste des couches d'images |  |

| `$ podman volume create {name} ?path`  | Création d'un volume  |  |
| `$ podman volume prune` | Supprime les volumes inutilisés  |  |
| `$ podman volume inspect {name}` | Affiche la configuration d'un volume |  |
| `$ podman volume import {name} {volume.tar.gz}` | Import un volume |  |
| `$ podman volume export {name} --output {volume.tar.gz}` | Export un volume  |  |
| `` |  |  |


### Construction d'une image personalisées

Utilisation d'une image de base universelle (UBI)
Quatre version : standard, init, minimal et micro

Langage pour créer le containerfile : DSL (Domaine Specific Language)

| Instruction | Description| Remarque |
| :---- | :----| :----|
| `FROM` | Détermine l'image de base utilisées | Utilisation d'une image de base universelle UBI. Plusieurs version disponible : standard, init, minimal et micro. Cela influt sur la taille de l'image |
| `WORKDIR` | Détermine le répertoir de travail à l'interieur du contenur. | Les instrcution qui suivent s'exécute à partir de ce répertoir |
| `COPY` et `ADD` | Copie les fichier de l'hôte vers le système de fichier de l'image de conteneur  | à la différence de COPY, ADD permet l'ajout de fichier depuis une URL et permet la décompression  d'archive tar dans l'image |
| `RUN` | Exécute une commande dans le conteneur et valide l'état résultant du conteneur dans une nouvelle couche d'image| |
| `ENTRYPOINT` | Défini l'exécutable à exécuter au démarrage du conteneur||
| `CMD` | Exécute une commande au démarrage du conteneur. | La commande est transmis à l'exécutable de l'ENTRYPOINT. |
| `USER` | Modifie l'utilisateur actif dans le conteneur. | Les instruction suivante sont exécuter avec l'identité de cet utilisateur, y compris CMD|
| `LABLE` | Rajoute des métadonnée à l'image (paire de clé, valeur) | |
| `EXPOSE` | Ajout un port aux métadonnée d'image |  (/!\ c'est uniquement à des fins de documentation) |
| `ENV` | Défini des variable d'environnement | |
| `ARG` | Défini des variables au moment de la compilation |  |
| `VOLUME` | Défini l'emplacement de stockage des données en déhors du conteneur| |

Exemple d'utilisation aver ARG et ENV
```bash
ARG VERSION \
    BIN_DIR

ENV VERSION=${VERSION:-1.16.8} \
    BIN_DIR=${BIN_DIR:-/usr/local/bin/}

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" -o ${BIN_DIR}/example
```

Liste des filtres de recherche usuel avec l'option `--format` de `$ podman inspect`

`'{{.NetworkSettings.Networks.apps.IPAddress}}'` Récupère l'adresse ip du conteneur sur le réseau apps
`'{{.State.Status}}'` Statut du conteneur
`'{{.State.Running}}'` Pour savoir si le conteneur est en cours d'execution

NIDAVELLIR


Pourquoi vérifier si le DNS est désactivté sur le réseau par défaut podman ?

L'activation du DNS sur un réseau permet d'utiliser le nom du conteneur à la place de l'adresse IP privet (au sein du réseau).
`podman run --rm --network cities registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 curl -s http://times-app/BKK && echo`


# NGINX

Fichier de configuration `/etc/nginx/nginx.conf`
After updated configuration, reload nginx with : `nginx -s reload`


Astuces avancer pour la création de container file

#### Utiliser des variables d'environnement.

> Attributer des variables d'environnement à la compilation de l'image (et leurs défini des valeurs par défaut)
> ARG DB_HOST[=localhost]
> Utilisation de  podman build --build-arg key=example-value
Voici la manière propre

ARG VERSION="1.16.8"
    BIN_DIR=/usr/local/bin/

ENV VERSION=${VERSION}
    BIN_DIR=${BIN_DIR}

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" -o ${BIN_DIR}/example

> Alternative

ARG VERSION \
    BIN_DIR

ENV VERSION=${VERSION:-1.16.8} \
    BIN_DIR=${BIN_DIR:-/usr/local/bin/}

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" \
        -o ${BIN_DIR}/example


#### ENTRYPOINT et CMD
Lorsqu'on utiliser ENTRYPOINT, les argement passer lors de l'exécution du contenenur ceux si sont passé en paramètre à l'ENTRYPOINT
Lorsqu'on utiliser CMD, les arguement passées remplace totalement la CMD défini dans l'image
Lorsque ENTRYPOINT et CMD sont utilisé conjoitement, les arguement remplace CMD qui sont ensuite passé dans l'ENTRYPOINT

pour tourepasser complement ENTRYPOINT on peut exécuter podman run --entrypoint env imagename (c'est comme un --force, pour remplacer la commande exécuter dans L'ENTRYPOINT)


#### Utiliser des sous etape
Pour éviter d'empacter trop de dépendance et donc reduire la taille de l'image

> First stage
FROM registry.access.redhat.com/ubi8/nodejs-14:1 as builder 1
COPY ./ /opt/app-root/src/
RUN npm install
RUN npm run build 2

> Second stage
FROM registry.access.redhat.com/ubi8/nginx-120 3
COPY --from=builder /opt/app-root/src/ /usr/share/nginx/html 4


RUN, COPY, ADD => Créer des couche blobs

##### COmmande chainner
Pour éviter d'utiliser des commandes chainner comme
RUN mkdir /etc/gitea && \
    chown root:gitea /etc/gitea && \
    chmod 770 /etc/gitea

note pour l'anchainnement de commande utiliser & \
pout l'anchainnement de variable d'environnement utiliser \


## GESTION DES VOLUMES

Pour faciliter le controle des données d'un conteneur privilégier l'utilisation de volumes

`podman run --volume /path/on/host:/path/in/container:OPTIONS {imagename}`
`podman run --mount type=TYPE,source=/path/on/host,destination=/path/in/container`

Mount spécifie explicitement le type de volume.
*bind*: pour les montages de liaison
*volume*: pour les montages de volumes
*tmpfs*: pour des montages éphémère en mémoire uniquement



Lorsque'on utiliser un montage de liaison, on doit configurer manuellement les autorisation de fichier et d'accès SELinux.
`$ podman run -p 8080:8080 --volume /www:/var/www/html registry.access.redhat.com/ubi8/httpd-24:latest`
Par defaut le processus Httpd ne dispose pas des autorisation pour accéder au répertoir /var/www/html
Pour résoudre ce problème on vérifie les droits avec : 
`$ podman unshare ls -l /www/`
>-rw-rw-r--. 1 root root 21 Jul 12 15:21 index.html
`$ podman unshare ls -ld /www/`
>drwxrwxr-x. 1 root root 20 Jul 12 15:21 /www/

Pour résoudre les problème d'accès SELinux du répertoir /www
`ls -Zd /www`
system_u:object_r:default_t:s0:c228,c359 /www

Le type ici: object_r doit être remplacer par container_file_t pour avoir accés au montage de liaison.

pour cela il faut rajouter l'option :Z au volume
`$ podman run -p 8080:8080 --volume /www:/var/www/html:Z registry.access.redhat.com/ubi8/httpd-24:latest`


Montage volume tmpfs :
`podman run -e POSTGRESQL_ADMIN_PASSWORD=redhat --network lab-net --mount  type=tmpfs,tmpfs-size=512M,destination=/var/lib/pgsql/data registry.redhat.io/rhel9/postgresql-13:1`





Commande utilise
Copie de fichier (origin > dest):  $ cp ~/DO188/labs/persisting-mounting/index.html ~/www

option -ti de podman run

Problème de droit d'accès.
podman unshare ls -ld ~/www
drwxrwx---. 1 root root 20 Jun 28 14:56 /home/student/www

> Les autres utilisateur non pas le droit d'accés en lecture sur le répertoire
> Et le groupe propriétaire du répertoir est root

Verifier l'ID de groupe à l'interieur du conteneur podman-server
podman run --rm \
registry.ocp4.example.com:8443/redhattraining/podman-python-server id
uid=994(python-server) gid=994(python-server) groups=994(python-server)

> Groupe id 994 doit être propriétaire du dossier

Attribution du groupe au repertoir
podman unshare chgrp -R 994 ~/www

> Puis vérification
podman unshare ls -ln --directory ~/www
drwxrwx---. 1 0 994 20 Jun 28 14:56 /home/student/www






On peut indiquer à podman de supprimer les commandes chainner avec --squash

Liste d'image utilisable 

registry.access.redhat.com/ubi8/ubi-minimal
registry.access.redhat.com/ubi8/ubi-minimal
registry.access.redhat.com/ubi8/ubi-minimal
registry.access.redhat.com/ubi8/ubi-minimal
registry.access.redhat.com/ubi8/ubi-minimal
