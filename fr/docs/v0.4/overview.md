---
title: "Aperçu"
description: "Que fait Crateview, comment il se connecte à votre registre et où se trouvent ses limites."
---

## Une vue plus claire de votre registre

Crateview est une interface Web pour les registres de conteneurs Docker et OCI. Il vous aide à trouver des référentiels et des balises, à inspecter les images multiplateformes, à lire les manifestes et les documents de configuration, et à comprendre les couches compressées et leur historique.

Il fonctionne parallèlement à votre registre. Le registre stocke les images et applique le contrôle d'accès ; Crateview fournit l'interface et un proxy de même origine. Docker et d'autres clients OCI peuvent utiliser l'origine du frontend comme adresse de registre.

![Aperçu du référentiel Crateview avec les référentiels de démonstration Alpine et imbriqués.](https://crateview.de/images/docs/v0.4/repositories-light.webp)

## Architecture

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

Le catalogue n'exige pas de base ; la supervision facultative utilise SQLite persistant.

## Fonctionnalités prises en charge

- Recherche de nom de référentiel, suggestions de balises et espaces de noms imbriqués.
- Tableaux de balises, sélection de plate-forme, tailles de couches compressées et historique des images.
- Annotations OCI pour les descriptions, les URL sources, la documentation et les licences.
- Manifeste brut et configuration JSON avec prise en charge du presse-papiers.
- Référents OCI, y compris le repli d'index de balise digest standard géré par les clients OCI.
- Accès anonyme et natif htpasswd Basic Auth.
- Suppression facultative du manifeste, avec les balises concernées affichées avant confirmation.
- Deux langues d'interface (anglais et allemand), Système/Clair/Sombre et identité visuelle configurable.


- La supervision est facultative et désactivée par défaut. La vue d'ensemble affiche le tableau de bord si elle est active ; sinon /home redirige vers /explore. L'activer expose les données globales à toute personne ayant accès à l'interface, y compris les visiteurs anonymes des registres publics.

## Connaître les limites

Le API standard ne fournit pas d'horodatage push, de statistiques de téléchargement, de favoris ou d'analyse des vulnérabilités. Crateview ne fabrique pas ces valeurs. La date de création d'une image provient des métadonnées de l'image et ne correspond pas à son heure de téléchargement. Les tailles affichées décrivent les couches d'images compressées, et non l'utilisation du disque du registre.

Les services externes de jeton Bearer, OIDC, l'administration des utilisateurs et la gestion des clés API ne sont pas intégrés. Les artefacts de signature peuvent être répertoriés comme référents ; il ne s’agit pas d’une vérification de signature cryptographique.

Commencez par [installation](/fr/docs/v0.4/installation), puis examinez [configuration](/fr/docs/v0.4/configuration) et [authentication](/fr/docs/v0.4/authentication).
