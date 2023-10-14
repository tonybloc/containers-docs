# Red Hat OpenShift Developper - Introduction to Containers with Podman (D0188)


## Principes de base Podman

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


### Création de conteneurs avec Podman

### Notion de base sur la mise en réseau de conteneurs

### Accès aux services réseau en conteneur

### Accès aux conteneurs

### Gestion du cycle de vie des conteneurs
