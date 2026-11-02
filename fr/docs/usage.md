---
title: "Utilisation de Crateview"
description: "Parcourez les images, changez de plate-forme et inspectez les données derrière une balise."
---

## Rechercher des référentiels et des balises

La présentation répertorie les référentiels du catalogue du registre. Explorez les noms de référentiels de filtres ; une requête `repository:tag` recherche les balises dans ce référentiel. Les noms imbriqués tels que `team/service` sont pris en charge.

Appuyez sur `/` en dehors d'un champ modifiable, ou sur `Ctrl+K`, pour concentrer la recherche dans la barre de navigation. La barre latérale peut être réduite à partir de la bascule à côté de la recherche. Les contrôles de langue et de mode couleur se trouvent au bas de la barre latérale développée.

## Inspecter une image

Ouvrez un référentiel et sélectionnez une balise. La vue de l'image montre son digest, les plates-formes disponibles, les tailles de couches compressées et les métadonnées de création.

Pour une image multiplateforme, choisissez le système d'exploitation et l'architecture dans le sélecteur de plateforme. L'historique des couches et la configuration de l'image sélectionnée suivent ce choix. L'onglet Manifeste brut affiche le manifeste ou l'index racine de l'image ; Configuration affiche la configuration de la plate-forme sélectionnée.

![Détails de l'image alpine avec sélection de plate-forme, historique des couches et commande pull.](https://crateview.de/images/image-light.webp)

## Commandes Pull

Le panneau de commandes fournit les commandes Docker, Podman et Skopeo pour les images, ainsi que les commandes d'artefact ORAS/Podman pour les artefacts. Copiez la commande et exécutez-la dans votre propre terminal.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Remplacez l'exemple d'hôte par votre origine publique Crateview. L'authentification CLI est distincte de votre session de navigateur.

## Métadonnées et référents OCI

Les descriptions, licences et liens sources sont lus à partir des annotations OCI ou des étiquettes de configuration d'image lorsqu'elles sont disponibles. Les métadonnées manquantes restent vides plutôt que devinées.

L'onglet Référents lit les référents OCI API. Si le registre n'implémente pas ce point de terminaison, Crateview lit l'index de balise standard dérivé de digest géré par les clients OCI. Il ne crée pas son propre index en analysant chaque manifeste. Les artefacts de signature répertoriés ne sont pas automatiquement vérifiés ou marqués comme fiables.

## Interpréter les horodatages et les tailles

Le temps de création de l'image est fourni par la configuration de l'image ou les annotations. Il ne vous indique pas quand une balise a été poussée vers ce registre. La taille des couches compressées ne mesure pas la consommation du disque, car les couches peuvent être partagées et les données supprimées peuvent rester jusqu'à la récupération de place.
