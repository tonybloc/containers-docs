# Principes de base de podman

Par defaut Podman ne possède pas de demon, pour cause de sécurité

Pour interagire avec les conteneurs, plusieurs manière s'offre à vous : 
-  Par ligne de commande (CLI Podman)
-  Par webservice (API RESTful Podman)
-  Par l'application de bureau (Podman Desktop)


Afficher la version de podman :
`$ podman -v`



Récupérer une image de conteneur depuis un registre d'images avec la commande `podman pull`.
Une image de conteneur est référencée sous la forme : `NAME:VERSION`
par exemple on récupère ici la version `7.9` de l'image `rhel` depuis le registre `registry.redhat.io`
```bash
$ podman pull registry.redhat.io/rhel7/rhel:7.9
```
Les images récupérées sont ensuite stocké en local sur votre système. Les images sont stocké dans le répertoire : `~/.local/share/containers`

Pour les utilisateur root, les images sont stocké dans le répertoire : `/var/lib/containers` (ces images sont affiché uniquement avec la commande : `$ podman images ls`

Pour lister les images existantes sur votre système, vous devez utilier la commande `podman images`
```bash
$ podamn images
REPOSITORY                        TAG         IMAGE ID      CREATED       SIZEÒ
registry.redhat.io/rhel7/rhel     7.9         52617ef413bd  4 weeks ago   216 MB
```

Ensuite pour exécuter une image de conteneur que vous avez récupéré, vous pouvez utiliser la commande `podman run` avec le nom de l'image et les potentiels arguments nécessaire.
```bash
$ podman run registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

Options disponible : 

`$ podman run --rm [image]` Supprime automatiquement le conteneur lors de sa fermeture (il ne sera donc plus listé avec podman ps -all)
`$ podman run --name [image]` Attribue un nom au conteneur
`$ podman run -p [host-port:container-port] [image]` Map un port du système hote avec un port du conteneur
`$ podman run -d [image]` Permets d'exécuter le conteneur en arrière plan, libère ainsi le terminal
`$ podman run -e NAME='Red hat'` Permets de transmettre des variable d'environnement au conteneur. Celle ci est imprimé à l'aide la commande printenv à l'intereur du conteneur.
`$ podman run --net networkA,networkB` Associe le contenuer à un réseau


> **Note** :
> Si on exécute la commande précedente sans voir récupérer l'image de contenuer auparavant, podamn essaie d'extraire l'image avant d'exécuter le conteneur. Ainsi il n'est pas nécessaire d'utilser la commande `podman pull`

Pour lister les conteneurs en **cours d'exécution** vous devez utiliser la commande `podamn ps`
```bash
$ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

Options disponible :
`$ podman ps --all` Affiche les couteneur en cours d'exécution et arrêtés
`$ podman ps --format=json` Affiche le contenu de la commande podman ps au format json




## Notion de base pour la mise en réseau de conteneurs

Par defaut les conteneurs sont liée au réseau nommé `podman` et peuvent l'utiliser pour communiquer entre eux.
Un reseaux entre deux ou plusieurs container permet d'isoler leurs communication.

Cependant dans le cadre d'une application web par exemple. Il est fort probable que vous ayez besoin de créer d'autre réseau, pour permettre séparer les communication entre : UI - API - Database

`$ podman network create [name]`
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
