---
title: "Utilisation de Crateview"
description: "Parcourez les images, changez de plate-forme et inspectez les données derrière une balise."
---

## Rechercher des référentiels et des balises

Le champ de recherche au-dessus des résultats filtre pendant la saisie. Il accepte les noms de dépôts et `repository:tag`. La vue Explorer et la liste des tags affichent la plage actuelle et le total.

Les dépôts utilisent un tableau par défaut, avec une vue grille disponible. Les noms avec barres forment des espaces de noms. La barre latérale se redimensionne et se replie, mémorise son état localement et contient la recherche.

/ ou Ctrl+K (Cmd+K sur macOS) ouvre la palette de recherche et navigation. Les dossiers se replient. Paramètres contient langue, Système/Clair/Sombre et À propos avec recherche des licences et mentions originales.

Administration regroupe Paramètres et Utilisateurs. Le menu du compte propose aussi langue et thème. Compte modifie votre mot de passe après vérification de l'actuel. La liste serveur contrôle consultation et création des comptes ; les droits des jetons sont en lecture seule. Un nouveau mot de passe exige au moins 12 caractères et au plus 72 octets UTF-8. Les nouveaux comptes à jetons n'ont aucun accès initial aux dépôts.

La liste d’autorisation du serveur contrôle la consultation, la création, la réinitialisation des mots de passe et la suppression des comptes. Le menu à trois points aligné à droite permet aux administrateurs de réinitialiser le mot de passe d’un autre utilisateur sans son ancien mot de passe ou de supprimer son compte après confirmation. La réinitialisation conserve les droits sur les dépôts ; la suppression les retire. Votre propre menu ouvre Compte pour changer votre mot de passe avec le mot de passe actuel ; la suppression de votre propre compte est bloquée.

![Image de marque et thème](https://crateview.de/images/docs/next/settings-light.webp)

## Inspecter une image

Ouvrez un référentiel et sélectionnez une balise. La vue de l'image montre son digest, les plates-formes disponibles, les tailles de couches compressées et les métadonnées de création.

Pour une image multiplateforme, choisissez le système d'exploitation et l'architecture dans le sélecteur de plateforme. L'historique des couches et la configuration de l'image sélectionnée suivent ce choix. L'onglet Manifeste brut affiche le manifeste ou l'index racine de l'image ; Configuration affiche la configuration de la plate-forme sélectionnée.

![Détails de l'image alpine avec sélection de plate-forme, historique des couches et commande pull.](https://crateview.de/images/docs/next/image-light.webp)

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
