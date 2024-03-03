# Red Hat OpenShift Development : Introduction to containers with Podman

Préparation à l'examin `Red Hat Certified Specialist in containerized application`

#### Cycle de vie d'un conteneur

![Schema Cycle de vie](https://rol.redhat.com/rol/static/static_file_cache/do188-4.12/basics/lifecycle/assets/images/basics/lifecycle-actions.svg)



### Commandes - Utilitaires / Rappels
| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ podman -v` | version de podman | |
| `$ printenv VARIABLE`| Affiche la valeur d'une variable d'environnement| |
| `$ export VARIABLE='value'` | Défini la valeur d'une variable d'environnement | |
| `$ nginx -s reload`| Après avoir mise à jours la configuration de nginx, redemarage | |
| `$ cp {source} {destination}`| Copie de fichier | |
| | | |

| Chemain de fichier  | Description          | Options |
| :------------------------ |:----------------------- | -----:|

| `/etc/nginx/nginx.conf` | Fichier de configuration | |
| | | |
| `podman unshare`| | |
| | | |
| | | |
| | | |


### Commande - Manipulation des images

| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ podman pull name:tag` | Récupère une image depuis un registre | La configuration des registres est spécifiée dans `/etc/containers/registries.conf`. Pour gérer et publier des images dans un registre, vous pouvez utiliser les commande skopeo ainsi que Quay.io stocker vos images |
| `$ podman images` | Liste toutes les images récupérés en local | |
| `$ podman build -t {imagename:tag} {file}` | Compiler une image | `--file` ou `-f` pour spécifier le fichier à compiler. <br>`--tag` ou `-t` pour définir le nom du l'image et du tag. <br>`--build-arg KEY=VALUE` pour définir ou surcharger la valeurs des arguments dans le cas ou aucune valeur par défaut n'est spécifiée |
| `$ podman image inspect [registry/user/image:tag]` | Inspect la configuraiton de l'image| `--format` |
| `$ podman image rm` | Supprimer une image du registre local | `--all` suprime toutes les images |
| `$ podman image prune` | Supprime les image inutilisé | `-a` pour supprimer ceux suspendues |
| `$ podman image tree {image:tag}` | Afficher la liste des couches d'images |  |

### Commande - Manipulation des volumes

| Commande  | Description          | Options |
| :------------------------ |:----------------------- | -----:|
| `$ podman volume create {name} ?path`  | Création d'un volume  |  |
| `$ podman volume prune` | Supprime les volumes inutilisés  |  |
| `$ podman volume inspect {name}` | Affiche la configuration d'un volume |  |
| `$ podman volume import {name} {volume.tar.gz}` | Import un volume |  |
| `$ podman volume export {name} --output {volume.tar.gz}` | Export un volume  |  |
| `` |  |  |
| | | |


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



### Commande - Manipulation des conteneurs

| Commande  | Description          | Options |
| :------------------------ |:----------------------- | :-----|
| `$ podman run [options] container [commande...]`  | Execute un processus dans un conteneur en cours d'execution | Options :  <br>`--name=toto` attribue un nom au conteneur. <br>`--rm` Supprime automatiquement le conteneur lors de sa fermeture. <br>`-p host-port:container-port` Map un port du système hot avec un port du conteneur. <br>`-d` permet d'executer le conteneur en arrière plan (libérant ainsi le terminal). <br>`-e NAME=value` Détermine un variable d'environnement dans le conteneur. <br>`--net networkA,networkB` Associer le conteneur à un réseau. <br>`--volume /path/on/host:/path/in/container:OPTIONS {imagename}` associer un volume au conteneur, voici les options `OPTION` : `:Z` `:z`. <br>`--mount type=TYPE,source=/path/on/host,destination=/path/in/container` Attache un volume au conteneur `TYPE`: *bind*, *volume*, *tmpfs* |
| `$ podman create [options] container` | Creation d'un conteneur | |
| `$ podman start [options] container` | Mise en route d'un conteneur | |
| `$ podman pause [options] container` | Mets en pause tous les processus d'un conteneur en cours d'exécution | |
| `$ podman unpause [options] container` | Désactive la mise en pause de tous les processus d'un conteneur | |
| `$ podman kill [options] container` | Détruit un conteneur en cours d'execution | |
| `$ podman stop [options] container`  | Stop l'execution d'un conteneur  |  `--all` ou `-a` pour stoper l'execution de tout les conteneurs. `--time` pour définir le temps d'attente avant la l'arret du conteneur |
| `$ podman restart [options] container`  | Redémarre un conteneur          |     |
| `$ podman rm [options] container`  | Supprime un conteneur stopé          |    `--force` ou `-f` force la suppression d'un conteneur en cours d'execution ou mise en pause |
| `$ podman rmi [options] container`  | Supprime une images de l'espace de stockage local          |     |
| `$ podman exec [options] container [command ...]`  | Execute un processus dans un conteneur en cours d'execution          | `--env` ou `-e` défini une variable d'environnement (temporaire). Pour rentre la variable d'environnement persistance utilier l'option `-e` de `$ podman run`. `--interactive` ou `-i` indique au conteneur d'accepter les entrées utilisateur. `-tty` ou `-t` attache un pseudo terminal au conteneur. `--latest` ou `-l` execute la commande dans le dernier conteneur créée. On combine l'ensemble de ces options pour ouvrir un terminal interactive dans un conteneur : `$ podman exec -til /bin/bash`. `--net` rattache un réseau au conteneur |
| `$ podman inspect [options] container`  | Affiche la configuration d'un conteneur ou d'une image. Paramétrage réseaux, usage CPU, variable d'environnement, status, mappage de port, volumes, ...         | Pour simplifier la recherche d'information dans la configuration du conteneur, on peut utiliser l'option `--format` et en utilisant l'annotation GO avec `{{` et `}}`. Exemple `$ podman inspect --format='{{.State.Status}}' 1a2de915f` |
| `$ podman ps [options]`  | Liste les conteneurs | `-all` ou `-a`: Affiche également les conteneurs stopés. `--format=json` format l'output de la commande |
| `$ podman rename [options] container`  | Modifie le nom d'un conteneur          |     |
| `$ podman stats [options] container`  | Affiche les statistique en temps réel associées aux ressources consommées par le conteneur          |     |
| `$ podman logs [options] container`  | Recherche les logs d'un conteneur          |     |
| `$ podman top [options] container`  | Affiche les processus en cours d'execution d'un conteneur          |     |
| `$ podman cp [option] [container:]source [container]destination` | Copie un fichier dans un conteneur | Récupère les fichiers de logs d'un contenu `$ podman cp a3db6c810:/tmp/logs .` (. correspond au repertoire de travail courent) |
| `$ podman network create {networkname}` | Création d'un réseau | |
| `$ podman network rm {networkname}` | Suppression d'un réseau | |
| `$ podman network prune {networkname}` | Suppression des réseaux non utilisés | |
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



### Instruction - Création d'une image personalisées

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

#### Utilisations Avances

##### Configuration de ARG et ENV
Exemple d'utilisation aver ARG et ENV
```bash
ARG VERSION="1.16.8" \
    BIN_DIR=/usr/local/bin/

ENV VERSION=${VERSION} \
    BIN_DIR=${BIN_DIR}

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" & \ 
    -o ${BIN_DIR}/example

RUN mkdir /etc/gitea && \
    chown root:gitea /etc/gitea && \
    chmod 770 /etc/gitea
```

#### ENTRYPOINT et CMD
Lorsqu'on utiliser ENTRYPOINT, les argement passer lors de l'exécution du contenenur ceux si sont passé en paramètre à l'ENTRYPOINT
Lorsqu'on utiliser CMD, les arguement passées remplace totalement la CMD défini dans l'image
Lorsque ENTRYPOINT et CMD sont utilisé conjoitement, les arguement remplace CMD qui sont ensuite passé dans l'ENTRYPOINT

pour outrepasser complement ENTRYPOINT on peut exécuter podman run --entrypoint env imagename (c'est comme un --force, pour remplacer la commande exécuter dans L'ENTRYPOINT)

#### Sous étapes

Pour éviter d'empacter trop de dépendance et donc reduire la taille de l'image

```bash
FROM registry.access.redhat.com/ubi8/nodejs-14:1 as builder

COPY ./ /opt/app-root/src/
RUN npm install
RUN npm run build 

FROM registry.access.redhat.com/ubi8/nginx-120 

COPY --from=builder /opt/app-root/src/ /usr/share/nginx/html
```


Liste des filtres de recherche usuel avec l'option `--format` de `$ podman inspect`

`'{{.NetworkSettings.Networks.apps.IPAddress}}'` Récupère l'adresse ip du conteneur sur le réseau apps
`'{{.State.Status}}'` Statut du conteneur
`'{{.State.Running}}'` Pour savoir si le conteneur est en cours d'execution


### Instruction - Création de conteneur avec Compose



| Instruction | Description| Remarque |
| :---- | :----| :----|
| `version` (obsolète) | spécifie la version de Compose utilisée.| |
| `services` | définit les conteneurs utilisés. | |
| `networks` | définit les réseaux utilisés par les conteneurs. | |
| `volumes` | spécifie les volumes utilisés par les conteneurs. | |
| `configs` | spécifie les configurations utilisées par les conteneurs. | |
| `secrets` | définit les secrets utilisés par les conteneurs. | |
 

#### Déclaration et utilisation de Volumes

Dans l'exemple ci-dessous, on bind le volume `db-val` avec le répertoire local `/var/lib/postgresql/data`.  `my-volume` a qu'en a lui été créé préalablement créer avec la commande `$ podman volume create db-vol`, c'est pourquoi on indique `externe:true`
```yaml
services:
  db:
    image: registry.redhat.io/rhel8/postgresql-13
    environment:
      POSTGRESQL_ADMIN_PASSWORD: redhat
    ports:
      - "5432:5432"
    volumes:
      - db-vol:/var/lib/postgresql/data
      - my-volume:/var/lib/exemple/data
      - my-bind-volume:/var/lib/src/data:Z

volumes:
  db-vol: {}
  my-volume: 
    external:true
  my-bind-volume: {}
```

```yaml
services:
  wiremock:
    container_name: "quotes-provider"
    image: "registry.ocp4.example.com:8443/redhattraining/wiremock"
    volumes:
      - ~/DO188/labs/compose-lab/wiremock/stubs:/home/wiremock:Z
    networks:
      - lab-net
  quotes-api:
    container_name: "quotes-api"
    image: "registry.ocp4.example.com:8443/redhattraining/podman-quotesapi-compose"
    ports:
      - "8080:8080"
    networks:
      - lab-net
    environment:
      QUOTES_SERVICE: "http://quotes-provider:8080"
  quotes-ui:
    container_name: "quotes-ui"
    image: "registry.ocp4.example.com:8443/redhattraining/podman-quotes-ui"
    ports:
      - "3000:8080"

networks:
  lab-net: {}
```


| Commande | Description| Remarque |
| :---- | :----| :----|
| `$ podman-compose up` | Execute le fichier `compose.yaml`, qui crée les objets définis, tels que les volumes ou les réseaux et démarre les conteneurs définis. | `-d` ou `--detach` : démarre les conteneurs en arrière-plan.<br> `--force-recreate` : recrée les conteneurs au démarrage. <br>`-V` ou `--renew-anon-volumes` : recrée des volumes anonymes. <br> `--remove-orphans` : supprime les conteneurs qui ne correspondent pas aux services définis dans le fichier Compose actuel. |
| `podman-compose stop` | Stop l'execution des services compose | |
| `podman-compose down` | Arrête et supprime les services compose. Les réseaux et volumes sont eux percistant  | |
| `` |  | |
| `` |  | |
| `` |  | |
| `` |  | |
| `` |  | |
| `podman generate kube` | Génère le fichier de configuration Kubernetess selon les conteneur,pod, volume actifs| |
| `podman play kube` | Démarre la configuration Kube | |