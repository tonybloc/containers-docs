# Red Hat OpenShift Developper - Introduction to Containers with Podman (D0188)


## Notions de base conteneur

Récupération une image de conteneur, depuis un registre d'images avec la commande `podman pull`
Une image de conteneur est référencée sous la forme : `NAME:VERSION`
```bash
$ podman pull registry.redhat.io/rhel7/rhel:7.9
```

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



