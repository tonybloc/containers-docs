# Red Hat OpenShift Developper - Introduction to Containers with Podman (D0188)


## Notions de base conteneur

Récupération une image de conteneur, depuis un registre d'images avec la commande `podman pull`
Une image de conteneur est référencée sous la forme : `NAME:VERSION`
```bash
$ podman pull registry.redhat.io/rhel7/rhel:7.9
```
Les images récupérer sont stocké (pout les utilsiateur non root) dans le répertoire : `~/.local/share/containers`
Pour les utilisateur root, les images sont stocké dans le répertoire : `/var/lib/containers` (ces images sont affiché uniquement avec la commande : `$ podman images ls`

L'image est ensuite stockée en local sur le système.
Pour lister les images existantes sur le système, vous devez utilier la commande `podman image`
```bash
$ podamn images
REPOSITORY                        TAG         IMAGE ID      CREATED       SIZE
registry.redhat.io/rhel7/rhel     7.9         52617ef413bd  4 weeks ago   216 MB
```

Pour exécuter une image de conteneur, utiliser la commande `podman run`
```bash
$ podman run registry.redhat.io/rhel7/rhel:7.9
```

Options disponible : 
```bash
$ podman run --rm [image]        // Supprime automatiquement le conteneur lors de sa fermeture
$ podman run --name [image]      // Attribue un nom au conteneur
$ podman run -p [host-port:container-port] [image]    // Map un port du système hote avec un port du conteneur
$ podman run -d [image]         // Permets d'exécuter le conteneur en arrière plan, libère ainsi le terminal
$ podman run -e NAME='Red hat'  // Permets de transmettre des variable d'environnement au conteneur. Celle ci est imprimé à l'aide la commande printenv à l'intereur du conteneur.
$ podman run --net networkA,networkB      // Associe le contenuer à un réseau 
```

> **Note** :
> Si on exécute la commande précedente sans voir récupérer l'image de contenuer auparavant, podamn essaie d'extraire l'image avant d'exécuter le conteneur. Ainsi il n'est pas nécessaire d'utilser la commande `podman pull`

Pour lister les conteneurs en cours d'exécution vous devez utiliser la commande `podamn ps`
```bash
$ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

Options disponible
```bash
$ podman ps --all    // Affiche les couteneur en cours d'exécution et arrêtés
$ podman ps --format=json     // Affiche le contenu de la commande podman ps au format json
```


Etats de conteneur possible


## Notion de base pour la mise en réseau de conteneurs

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

La commande suivante permet de lister les mappages de ports pour tout les conteneur : 
```bash
$ podman port --all
```
Si un conteneur appelé my-app est associé au réseau apps. La commande suivante récupère l’adresse IP privée du conteneur dans le réseau apps
```bash
$ podman inspect my-app \
  -f '{{.NetworkSettings.Networks.apps.IPAddress}}'
```

La commande suivante, permet d'exécuter une nouveau processus dans un conteneur en cours d'exécution
```bash
$ podman exec [options] container [commande ...]
```
Exemple :
```bash
$ podman exec httpd cat /etc/httpd/conf/httpd.conf      // httpd étant le conteneur, et cat /etc/httpd/... la commande
```

Options disponible
```bash
$ podman exec --env (ou -e) [container] [commande]                  // pour spécifier des variable d'environnement
$ podman exec --interactive (ou -i) [container] [commande]          // pour indiquer au conteneur d'accepter l'entrée
$ podman exec --tty (ou -t) [container] [commande]                  // pour allouer un pseudo terminal
$ podman exec --latest (ou -l)                                      // pour exécuter la commande dans le dernier conteneur crée
```

Exemple, on définie une nouvelle variable d'environnement (temporaire, cad qu'a la fin du process env, la variable d'env ne sera pas percisté) dans le dernier container créer puis on execture la commande env pour lister les variable d'environnement du conteneur.
```bash
$ podman exec -e ENVIRONMENT=dev -l env
```

Pour ouvrir un shell dans le container cible, vous pouvez utiliser la commande suivante : (il fautdra utiliser la commande exit pour fermer la session interative
```bash
$ podman exec -ti httpd /bin/bash
```

Certain conteneur (notement pour la production), ne dispose pas des utilitaire `cat`, ou `vi` pour éditer/affiche le contenu d'un fichier. Mais il est possible de copier des fichier de l'hote vers le conteneur et invesement avec les commande suivante : 
```bash
$ podman cp [options] [container:]source_path [container:]destination_path
```
Exemple : Copier le fichier /tmps/logs à du conteneur 'a3b4564a687e' dans le répertoire actuel
```bash
$ podman cp a3b4564a687e:/tmp/logs .
```
Exemple : Copier le fichier nginx.conf du system dans le répertoir /etc/nginx du conteneur 'a3b4564a687e'
```bash
$ podman cp ./nginx.conf a3b4564a687e:/etc/nginx
```


Il vous est possible de récupérer l'ensemble des informations d'un conteneur avec la commande : 
```bash
$ podman inspect [container]
```

Cette commande retourn un tableau JSON qui affiche des information sur different aspect du conteneur (Paramètre réseau, utilisation du processus, variable d'environnement, mappage de port, volumes, ...)
Pour recherche une information précessis dans ce tableau vous pouvez utiliser l'option suivante

```bash 
$ podman insepect --format='{{.State.Status}}' [container]
```

Commande pour gérer le cycle de vie d'un conteneur: 

```bash
$ podman stop [container]
$ podman stop --all [container]
$ podman stop --time=100 [container]

$ podman kill [container]
$ podman pause [container]
$ podman unpause [container]
$ podman restart [container]
$ podman rm [container]         // Suppression de conteneur
$ podman rm --force
$ podman rm --all
```


## Gestion des registres d'images (avec Skopeo)

Exemple de registre existant : Quay.io, Red Hat Registry (https://catalog.redhat.com/), Docker Hub et Amazon ECR.

`registry.access.redhat.com` : ne nécessite aucune authentification
`registry.redhat.io` : nécessite une authentification

Lorsqu'on utilise la commande `$ podman pull [registre][image-path][:tag]` sans spécifier de registre, par défaut podman utilise le fichier `/etc/containers/registries.conf` pour recherche le registre de conteneur susceptible de contenir le nom de l'image rechercher 
Par exemple, avec la configuration suivante, Podman effectue une recherche dans Red Hat Registry. Si l’image est introuvable dans Red Hat Registry, Podman effectue une recherche dans le registre de Docker Hub.
`unqualified-search-registries == ['registry.redhat.io', 'docker.io']`
On pouvez également bloquer un registre
`[[registry]]
location="docker.io"
blocked=true`

Skopeo peut inspecter des images distantes ou transférer des images entre des registres sans utiliser le stockage local. La commande `skopeo` utilise le format `transport:image`, notamment `docker://remote_image, dir:path` ou `oci:path:tag`.
Utilisez la commande `skopeo copy` pour copier des images entre des registres:
```bash
$ skopeo copy \
 docker://registry.access.redhat.com/ubi9/nodejs-18 \
 docker://quay.io/myuser/nodejs-18
```

Pour télécharger l'image en local (utilisez dir:...)
```bash
$ skopeo copy \
 docker://registry.access.redhat.com/ubi9/nodejs-18 \
 dir:/var/lib/images/nodejs-18
```

Pour se connecter à un registre nécessitant une authentication pour récupérer des images utiliser la commande
```bash
$ podman login [registre]
```
Les informations d'identification sont enregistré dans un fichier `${XDG_RUNTIME_DIR}/containers/auth.json` (skopeo utilise le même fichier)
```bash
$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
{
	"auths": {
		"registry.redhat.io": {
			"auth": "dXNlcjpodW50ZXIy"
		}
	}
}

$ echo -n dXNlcjpodW50ZXIy | base64 -d
user:hunter2

```
Pour créer de nouvelle baslise pour les images locales, on doit utiliser la commande suivante : 
```bash
$ podman image tag LOCAL_IMAGE:TAG LOCAL_IMAGE:NEW_TAG
```
Possible de recherche une image avec podamn. La recherche s'effectue sur les registre défini comme `unqualified-search-registries` du fichier `registries.conf`
```bash
$ podman search nginx
```

Pour compiler une image pour ensuite la poussé sur le registre Red Hat Quay.io, il est nécessaire d'effectuer les commandes suivante
```bash
$ podman build --file Containerfile --tag quay.io/YOUR_QUAY_USER/IMAGE_NAME:TAG
$ podman push quay.io/YOUR_QUAY_USER/IMAGE_NAME:TAG
```

Comme pour la container/network/... la commande suivante permet d'afficher les informations d'une image local
```bash
$ podman image inspect [REGISTRY/USER/IMAGE8_PAHT:TAG]
[
   {
    "Id": "6683...98ea",
    ...output omitted...
    "Config": {
      "User": "27",                                                   // Utilisateur par défaut de l'image
      "ExposedPorts": {                                               // Port que l'application expose
           "3306/tcp": {}
      },
      "Env": [ 3
           "PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            ...output omitted...
          ],
      "Entrypoint": [                                               // Le point d’entrée, une commande exécutée au démarrage du conteneur
           "container-entrypoint"
      ],
      "Cmd": [                                                     // La commande que le script container-entrypoint exécute
           "run-mysqld"
      ],
      "WorkingDir": "/opt/app-root/src",                           // Le répertoire de travail des commandes dans l’image
      "Labels": {                                                  // Des étiquettes fournissant des métadonnées supplémentaires
         ...output omitted...
         "release": "177.1654147959",
         "summary": "MariaDB 10.3 SQL database server",
         "url": "https://access.redhat.com/containers/#/registry.access.redhat.com/rhel8/mariadb-103/images/1-177.1654147959",
        ...output omitted...
    "Architecture": "amd64", 8
    "Os": "linux",
    "Size": 573593952,
```

Suppression d'image avec la commande
```bash
$ podman image rm [REGISTRY/NAMESPACE/IMAGE_NAME:TAG]
$ podman image rm --all [REGISTRY/NAMESPACE/IMAGE_NAME:TAG]
$ podman image prune
$ podman image prune -a       // Pour supprimer les images inutilisé et ceux suspendues
```

# [ReadHat] Création de container personalisées


Un "ContainerFile" liste les instruction permettant de construire une image de conteneur.
Chaque instruction utilisé entraîne une modification qui est capturé dans une couche d'image.
Ces couches sont empilées pour former l'image du container.

## Les images de bases

La création d'une image consiste à appliquer des modification sur une image de base.
Cette image de base determine :
> Gestionnaire de paquetage
> Système d'initialisation
> Disposition du système de fichier
> Dépendances et environnement d'exécution préinstallés.


Red Hat fournit des images de container conçus spécialement comme point de départ. On les nomme "UBI" Univeral Base Image.
Quatre variantes de ces images existe :
> standard      : Image UBI principal qui comprend DNF, systemd, et des utilitaire tels que gzip et tar.
> init          : Simplifie l'exécution de plusieurs application sau sein d'un même conteneur en les gérant avec systemd.
> minimal       : Version plus petite que init et utilise le gesetionnaire micodnf.
> micro         : Version la plus petite qui n'inclut pas de gestionnaire de paquetage.

Certaine d'entre elles incluent des environnement d'exécution courant tels que Python et Node.js


## Instruction concernant les ContainerFile

> FROM          : Défini l'image de base
> WORKDIR       : Défini le répertoire de travail actuel à l'interieur du conteneur. Les instruction qui suivent s'execute dans ce répertoire
> COPY          : Copie les fichiers de l'hôte de compilation dans le système de fichier du conteneur.
> ADD           : Alternative à COPY, 
                        => Permet la copie de fichier à partir d'une URL, 
                        => Décompression des archives tar dans l'image de destination
> RUN           : Exécute une commande dans le conteneur et valide l'état du conteneur dans une nouvelle couche à l'interieur de l'image
> ENTRYPOINT    : Défini l'exécutable à exécuter au démarage du conteneur
> CMD           : Exécute une commande au démarage du conteneur  Cette commande est transmise à l’exécutable défini par ENTRYPOINT. Les images de base définissent un ENTRYPOINT par défaut qui est généralement un exécutable shell, tel que Bash
> USER          : Modifie l'utilisateur actif à l'interieur du conteneur. Les instructions qui suivents sont exécutées avec l'identité de cette utilisateur.y comprie CMD.
> LABEL         : Défini un paire de clé valeur aux métadonnée de l'image (pour l'organisation et la recherche d'image)
> EXPOSE        : Ajoute un port aux métadonnées d'image, qui indique quelle application du conteneur est liée à ce port. (Uniquement pour la documentation)
> ENV           : Définir des variable d'environnements
> ARG           : Définit des variable ou moment de la compilation
> VOLUME        : Définit l'emplacement de stockage des données en dehors du conteneur.




Chaque instruction Containerfile s’exécute dans un conteneur indépendant à l’aide d’une image intermédiaire créée à partir de chaque commande précédente. 
Cela signifie que chaque instruction est indépendante des autres instructions du Containerfile. 
Voici un exemple de Containerfile permettant de créer un conteneur de serveur Web Apache simple :

# This is a comment line                                            1
FROM        registry.redhat.io/ubi8/ubi:8.6                         2
LABEL       description="This is a custom httpd container image"    3
RUN         yum install -y httpd                                    4
EXPOSE      80                                                      5
ENV         LogLevel "info"                                         6
ADD         http://someserver.com/filename.pdf /var/www/html        7
COPY        ./src/   /var/www/html/                                 8
USER        apache                                                  9
ENTRYPOINT  ["/usr/sbin/httpd"]                                     10
CMD         ["-D", "FOREGROUND"]                                    11


1   : Les lignes qui commencent par le signe « dièse » (#) sont des commentaires.
2   : L’instruction FROM déclare que la nouvelle image de conteneur étend l’image de base du conteneur registry.redhat.io/ubi8/ubi:8.6ubi8/ubi:8.6.
3   : L'instruction LABEL ajoute des métadonnées à l'image.
4   : RUN exécute des commandes dans une nouvelle couche au-dessus de l’image en cours. Le shell utilisé pour exécuter les commandes est /bin/sh.
5   : EXPOSE configure les métadonnées indiquant que le conteneur écoute sur le port réseau spécifié au moment de l’exécution.
6   : ENV est responsable de la définition des variables d’environnement qui sont disponibles dans le conteneur.
7   : L’instruction ADD copie les fichiers ou les répertoires à partir d’une source locale ou distante et les ajoute au système de fichiers du conteneur.
8   : COPY copie les fichiers à partir d'un chemin relatif vers le répertoire de travail et les ajoute au système de fichiers du conteneur.
9   : USER spécifie le nom d’utilisateur ou l’UID à utiliser lors de l’exécution de l’image de conteneur pour les instructions RUN, CMD et ENTRYPOINT.
10  : ENTRYPOINT spécifie la commande par défaut à exécuter lorsque les utilisateurs instancient l’image en tant que conteneur.
11  : CMD fournit les arguments par défaut pour l’instruction ENTRYPOINT.


Création de l'image [ImageName] selon les instruction du fichier [ContainerFile]
`$ podman build --tag [imageName] --file [Containerfile]`

Visualisation de la liste des couches d'images intermédiaire
`$ podman image tree [imageName][:tag]`

Inspection de la configuration de l'image
`$ podman inspect [imageName][:tag] --format '{{ [SearchExpression] }}'`

Création du conteneur
`$ podman build -d --rm --name [ContainerName] -p [portHost]:[portContainer] [ImageName][:tag]`



## ENV 
L’instruction ENV vous permet de spécifier une configuration dépendant de l’environnement ; par exemple, des noms d’hôte, des ports ou des noms d’utilisateur. 
L'application en conteneur peut utiliser les variables d'environnement lors de l'exécution

## ARG
Utilisez l’instruction ARG pour définir des variables au moment de la compilation, généralement pour rendre une compilation de conteneur personnalisable.
Vous pouvez optionnellement configurer une valeur de variable de compilation par défaut si le développeur ne la fournit pas au moment de la compilation avec :
ARG key[=default value] exemple : 


ARG VERSION="1.16.8" \
    BIN_DIR=/usr/local/bin/

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" \
        -o ${BIN_DIR}/example


Pour créer l'image de conteneur avec argument, utiliser --build-arg
$podman build --build-arg key=exemple-value

Si vous ne configurez pas les valeurs par défaut de l’instruction ARG, mais que vous devez configurer les valeurs par défaut pour ENV, 
utilisez la syntaxe ${VARIABLE:-DEFAULT_VALUE}, par exemple
ARG VERSION \
    BIN_DIR

ENV VERSION=${VERSION:-1.16.8} \
    BIN_DIR=${BIN_DIR:-/usr/local/bin/}

RUN curl "https://dl.example.io/${VERSION}/example-linux-amd64" \
        -o ${BIN_DIR}/example

## VOLUME
Utilisez l'instruction VOLUME pour stocker des données de manière persistante. 
La valeur est le chemin où Podman monte un volume persistant à l’intérieur du conteneur. 
L’instruction VOLUME accepte plusieurs chemins pour créer plusieurs volumes

FROM registry.redhat.io/rhel9/postgresql-13:1
VOLUME /var/lib/pgsql/data

Pour récupérer le répertoire local utilisé par un volume, vous pouvez utiliser la commande podman inspect VOLUME_ID.
[user@host ~]$ podman inspect VOLUME_ID
[
    {
        "Name": "VOLUME_ID",
        "Driver": "local",
        "Mountpoint": "/home/your-name/.local/share/containers/storage/volumes/VOLUME_ID/_data",
        ...output omitted...
        "Anonymous": true
    }
]

Pour supprimer les volumes inutilisés, utilisez la commande:  $podman volume prune
Pous pouvez créer un volume nommé à l'aide de la commande: $podman volume create [volumeName].
Pour lister les volumes utilisez le commande : $podman volume ls 
[user@host ~]$ podman volume ls \
 --format="{{.Name}}\t{{.Mountpoint}}"


## ENTRYPOINT et CMD
Les instructions ENTRYPOINT et CMD spécifient la commande à exécuter au démarrage du conteneur. Un Containerfile valide doit contenir au moins l’une de ces instructions.


L’instruction ENTRYPOINT définit un exécutable, ou une commande, qui fait toujours partie de l’exécution du conteneur. 
Cela signifie que des arguments supplémentaires sont transmis à la commande fournie.
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.5
ENTRYPOINT ["echo", "Hello"]

[user@host ~]$ podman run my-image
Hello

[user@host ~]$ podman run my-image Red Hat
Hello Red Hat

Si vous utilisez uniquement l’instruction CMD, la transmission d’arguments au conteneur remplace la commande fournie dans l’instruction CMD.
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.5
CMD ["echo", "Hello", "Red Hat"]

[user@host ~]$ podman run my-image
Hello Red Hat

L’exécution d’un conteneur avec l’argument whoami remplace la commande echo.

[user@host ~]$ podman run my-image whoami
root

Lorsqu’un Containerfile spécifie à la fois ENTRYPOINT et CMD, CMD modifie son comportement. 
Dans ce cas, les valeurs fournies à CMD sont transmises en tant qu’arguments par défaut à ENTRYPOINT.

FROM registry.access.redhat.com/ubi8/ubi-minimal:8.5
ENTRYPOINT ["echo", "Hello"]
CMD ["Red", "Hat"]

[user@host ~]$ podman run my-image
Hello Red Hat

## Compilation en plusieurs étapes

# First stage
FROM registry.access.redhat.com/ubi8/nodejs-14:1 as builder         1
COPY ./ /opt/app-root/src/
RUN npm install
RUN npm run build                                                   2

# Second stage
FROM registry.access.redhat.com/ubi8/nginx-120                      3
COPY --from=builder /opt/app-root/src/ /usr/share/nginx/html        4


1 : Définissez la première étape avec un alias. La deuxième étape utilise l'alias builder pour référencer cette étape.
2 : Compilez l'application.
3 : Définissez la deuxième étape sans alias. Elle utilise une image de base ubi8/nginx-120 pour diffuser la version de l’application prête pour la production.
4 : Copiez les fichiers de l'application dans un répertoire de l'image finale. L’indicateur --from signale que Podman copie les fichiers à partir de l’étape builder.


## Inspection des couches de données du conteneur

Les images de conteneur utilisent un système de fichiers en couches COW (Copy-On-Write). 
Lorsque vous créez un Containerfile, les instructions RUN, COPY et ADD créent des couches (que l’on désigne parfois collectivement sous le nom de blobs).

Le système de fichiers COW en couches garantit l’immuabilité d’un conteneur. 
Lorsque vous démarrez un conteneur, Podman crée et monte une couche fine, éphémère et accessible en écriture au-dessus des couches d’image du conteneur. 
Lorsque vous supprimez le conteneur, Podman supprime cette couche.
Cela signifie que toutes les couches de conteneur restent identiques, à l’exception de la couche fine et accessible en écriture.
Les données sont donc effemaire s'ils ne sont pas stocker sur un volume dédiée

### Mise en cache des couches d’image
Étant donné que toutes les couches de conteneur sont identiques, plusieurs conteneurs peuvent les partager. 
Cela signifie que Podman peut mettre en cache des couches et compiler uniquement celles qui ont été modifiées ou qui n’ont pas été mises en cache.

Vous pouvez utiliser la mise en mémoire cache pour réduire le temps de compilation. Prenons l’exemple d’un Containerfile d’application Node.js :

...content omitted...
COPY . /app/
...content omitted...
L’instruction précédente copie chaque fichier et répertoire du répertoire Containerfile dans le répertoire /app du conteneur. Cela signifie que toute modification apportée à l’application entraîne la recompilation de chaque couche après la couche COPY.

Vous pouvez modifier les instructions comme suit :

...content omitted...
COPY package.json /app/
RUN npm ci --production
COPY src ./src
...content omitted...
Cela signifie que si vous modifiez le code source de votre application dans le répertoire src et recompilez votre image de conteneur, la couche de dépendance est mise en cache et ignorée, ce qui réduit le temps de compilation. Podman ne recompile la couche de dépendance que lorsque vous modifiez le fichier package.json.




### Réduction du nombre de couche
Vous pouvez également créer des Containerfiles qui n’utilisent pas de commandes chaînées et configurer Podman pour « écraser » les couches.
Utilisez l’option --squash pour écraser les couches déclarées dans le Containerfile. 
Vous pouvez également utiliser l’option --squash-all pour écraser les couches de l’image parente.



## Manipulation

FROM registry.ocp4.example.com:8443/redhattraining/podman-random-numbers as generator
RUN python3 random_generator.py

FROM registry.ocp4.example.com:8443/ubi8/python-38:1-96

ENV FILE="/redhat/materials/numbers.txt"
USER default
WORKDIR /redhat

COPY --from=generator --chown=default /app/numbers.txt materials/numbers.txt
COPY main.py .

VOLUME /redhat/materials

CMD python3 main.py


[student@workstation custom-advanced]$ podman inspect custom-advanced --format="{{ (index .Mounts 0).Source}}"




## ROOTLESS (sans privilège)

Isolement de la charge de travail du conteneur

Prérequis sur la machine hôte :
cgroup v2           => limiter les ressources de conteneur sans nécessiter une augmentation des privilèges
slirp4netns         => implémenter la mise en réseau en mode utilisateur pour les espace de noms réseau sans privilège
fuse-overlayfs      => gérer le système de fichier COW 


Lorsque vous créez un Containerfile, l’utilisateur est généralement root. 
Cela est dû au fait que vous avez besoin de privilèges élevés pour certaines opérations, comme l’installation de paquetages ou la modification de la configuration

Pour déterminer l’utilisateur actuel, exécutez la commande id :
$podman run registry.access.redhat.com/ubi9/ubi id


FROM registry.access.redhat.com/ubi9/ubi

RUN adduser \
   --no-create-home \
   --system \
   --shell /usr/sbin/nologin \
   python-server

USER python-server

CMD ["python3", "-m", "http.server"]

un conteneur de ce type démarre le même processus sans privilèges élevés, ce qui, pour une personne malintentionnée, rend plus difficiles l’exploitation d’une vulnérabilité et l’accès au système hôte. L’instruction RUN s’exécute en tant qu’utilisateur root, car elle précède l’instruction USER.


[user@host ~]$ cat /etc/subuid /etc/subgid
student:100000:65536
student:100000:65536

L'ID utilisateur 0 (root) est une exception, car l'utilisateur root est mappé sur l'ID utilisateur qui a démarré le conteneur. Par exemple, si un utilisateur avec l'ID 1000 démarre un conteneur qui utilise l'utilisateur root, l'utilisateur root est mappé à l'ID utilisateur hôte 1000.


# Persistance de données
Les volumes sont des montages de données gérés par Podman. Les montages de liaison sont des montages de données gérés par l’utilisateur.

Pour lier un volume à un conteneur, voici la commande
```bash
$ podman run -p 8080:8080 --volume [hostpath:containerpath:options]
$ podman run -p 8080:8080 -v [volume_name:containerpath]
$ podman run -p 8080:8080 --mount type=TYPE,source=[hostpath],destination=[containerpath]				
```

TYPE étant le type de volume. `bind` pour les montage de liaison, `volume` pour les montages de volume, `tmpfs` pour des montage éphémère en mémoire uniquement
> **Note** : Le paramètre mount et la méthode privilégiee pour monter des répertoires dans un container

La création de volume s'effectue avec la commande 
```bash
$ podman volume create [volume_name]
$ podman volume inspect [volume_name]
```

Pour les conteneurs en mode rootless, Podman stock les données de volume local dans le répertoire `$HOME/.local/share/containers/storage/volumes/`

Il est possible d'importer des données d'une archive tar dans un volume à l'aide de la commande. et d'exporter des données d'un volume sur la machine local avec la commande : 
```bash
$ podman volume import [volume_name] [archive_name]
$ podman volume export [volume_name] --output [archive_name]
```

Pour charger des données dans un container il es possible d'utiliser les instruction suivantes : (cela nécessite un client de base de données)
```bash
$ podman cp SQL_FILE TARGET_DB_CONTAINER:CONTAINER_PATH
$ podman exec -it DATABASE_CONTAINER \
	psql -U DATABASE_USER -d DATABASE_NAME -f CONTAINER_PATH
```
Si le conteneur ne possède pas ce client de base de donnée, il vous est possible d'utiliser un conteneur éphémaire pour effectuer le traitement
```bash
$ podman run -it --rm \
  -e PGPASSWORD=DATABASE_PASSWORD \
  -v ./SQL_FILE:/tmp/SQL_FILE:Z \
  --network DATABASE_NETWORK \
  registry.redhat.io/rhel8/postgresql-12:1-113 \
  	psql -U DATABASE_USER -h DATABASE_CONTAINER \
       	-d DATABASE_NAME -f /tmp/SQL_FILE
```
Cette commande utilise l’option SELinux Z afin de définir le contexte SELinux pour que le conteneur ait accès au SQL_FILE hôte.

La commande utilise également le réseau DATABASE_NETWORK, qui permet au client psql de ce conteneur d’utiliser le nom DATABASE_CONTAINER comme nom d’hôte pour le conteneur de base de données. Pour que le service DNS fonctionne, il doit être activé sur le réseau DATABASE_NETWORK.

Pour exporter les données de la base de données, vous pouvez utiliser les commandes de sauvegarde de la base de données présentes dans l’image de conteneur de la base de données. Par exemple, MySQL et PostgreSQL fournissent, respectivement, les commandes mysqldump et pg_dump.
Par exemple pour un conteneur PostreSQL
```bash
$ podman exec POSTGRESQL_CONTAINER \
  pg_dump -Fc DATABASE -f BACKUP_DUMP
```


## Journalisation
Il arrive qu’un conteneur ne parvienne pas à démarrer pour différentes raisons ; par exemple, en raison d’une configuration manquante ou d’un problème d’accès aux fichiers.
Pour accéder au logs du container vous pouvez utiliser la commande : 
```bash
$ podman logs [container]
```
Quelques commande utils pour débuger l'exécution dun container
```bash
$ podman port CONTAINER				// Accéder au mappage de port du conteneur
```
Pour vérifier les ports d’application en cours d’utilisation, listez les ports réseau ouverts dans le conteneur en cours d’exécution. 
(aide : socket statistics (ss)
 -p affiche le processus qui utilise le socket, 
 -a affiche les connexions établies et en cours d'écoute
 -n affiche les adresses IP.
 -t affiche les socket TCP.
)
```bash
$ podman exec -it CONTAINER ss -pant			
```
Pour cibler un espace de nom de conteneur:  (pour récuper le PID du conteneur `podman inspect CONTAINER --format '{{.State.Pid}}'`)
```bash
$ sudo nsenter -n -t CONTAINER_PID ss -pant
```
> **Note** : Les applications en conteneur doivent écouter sur l’adresse 0.0.0.0, qui fait référence à toute interface réseau à l’intérieur du conteneur. L’utilisation de l’interface de bouclage 127.0.0.1 empêche l’application de communiquer en dehors du conteneur.
