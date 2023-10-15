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


