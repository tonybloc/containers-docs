# Introduction et présentation des conteneurs

Décrire les principes de base des conteneurs

En quoi sont-ils différent des machine virtuelle

Décrire l'orchestration des conteneurs et les fonctionalités de RedHat OpenShift.

L'ensemble des librairie servant à l'exécution du contenenur sont contenu dans le container.

Système de fichier Union
Les couche du conteneur sont immuable (non modifiabe). Le moteur de conteneur rajoute une couche accessibles en écriture pour les modifications du fichier d'exécution.
Les conteneur sont éphémères par défaut, ce qui signifie que le moteur de conteneur supprime la couche accessible en écriture lorsque vous supprimez le conteneur.

Fonctionalité du noyau Linux utilisés :
- Espace de noms => utilisé pour isollé les processus des conteneurs des uns et des autres (et du système hôte)
- Groupe de contrôle (cGroup) => utilisé pour la gestion des ressources machine
- chroot

Lorque le système hôte est un système non Linux, les fonctionalité untilisé par les conteneurs sont virutalisé par l'implementation du moteur de conteneur.

Le moteur de conteneur sont normalisé par un organisme de gouvenrance (OCI Open Container Initiative)

Différentier les images de conteneur des instance de conteneur.
On utilie des images de conteneur pour créer des instance de conteneur. Les images sont immuable tandisque 

Les images de conteneur OCI sont définie par les spécificiation image-spec
Les instance de conteneur OCI sont définie par les spécification runtime-spec.

Podman est un moteur de conteneur, tout commme docker.

## Présentation de Kubernetes et d'OpenShift

Kubernetes est un service d'orcestation qui simplifie le déploiement, la gestion et la mise à l'échelle des applications en conteneur.

La plus petite unité dans Kubenetes est le pod qui represente une seul application et se compose de plusieurs conteneur.
- Equilibrage de charge
- Mise à l'échelle horizontal
- Auto cicatrisant
- Déploiement automatisé
- Secret et gestion de la configuration

RHOCP est une surcouche de Kubernetess et rajoute les foncitonalités suivantes : 
- Workflow de développeur (construction d'artefacts, accès à des registre, pipileine CI/CD)
- Routes (expose les services au monde exterieur)
- Métrique et journalisation
- Interface utilisateur unifiée
