---
title: "Aperçu"
description: "Que fait Crateview, comment il se connecte à votre registre et où se trouvent ses limites."
---

## Une vue plus claire de votre registre

Crateview est une interface Web pour les registres de conteneurs Docker et OCI. Il vous aide à trouver des référentiels et des balises, à inspecter les images multiplateformes, à lire les manifestes et les documents de configuration, et à comprendre les couches compressées et leur historique.

Il fonctionne parallèlement à votre registre. Le registre stocke les images et applique le contrôle d'accès ; Crateview fournit l'interface et un proxy de même origine. Docker et d'autres clients OCI peuvent utiliser l'origine du frontend comme adresse de registre.

![Aperçu du référentiel Crateview avec les référentiels de démonstration Alpine et imbriqués.](https://crateview.de/images/docs/next/repositories-light.webp)

## Architecture

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

Le catalogue n'exige pas de base ; la supervision facultative utilise SQLite persistant. Les comptes utilisent htpasswd et un fichier de droits distinct.

## Fonctionnalités prises en charge

- Recherche de nom de référentiel, suggestions de balises et espaces de noms imbriqués.
- Tableaux de balises, sélection de plate-forme, tailles de couches compressées et historique des images.
- Annotations OCI pour les descriptions, les URL sources, la documentation et les licences.
- Manifeste brut et configuration JSON avec prise en charge du presse-papiers.
- Référents OCI, y compris le repli d'index de balise digest standard géré par les clients OCI.
- Accès anonyme et natif htpasswd Basic Auth.
- Suppression facultative du manifeste, avec les balises concernées affichées avant confirmation.
- 13 langues d'interface, Système/Clair/Sombre et personnalisation configurable.

- La supervision est facultative et désactivée par défaut. La vue d'ensemble affiche le tableau de bord si elle est active ; sinon /home redirige vers /explore. L'activer expose les données globales à toute personne ayant accès à l'interface, y compris les visiteurs anonymes des registres publics. Le mode à jetons géré exige aussi un droit de lecture global.

## Connaître les limites

Le API standard ne fournit pas d'horodatage push, de statistiques de téléchargement, de favoris ou d'analyse des vulnérabilités. Crateview ne fabrique pas ces valeurs. La date de création d'une image provient des métadonnées de l'image et ne correspond pas à son heure de téléchargement. Les tailles affichées décrivent les couches d'images compressées, et non l'utilisation du disque du registre.

Administration regroupe Paramètres et Utilisateurs. Le menu du compte propose aussi langue et thème. Compte modifie votre mot de passe après vérification de l'actuel. La liste serveur contrôle consultation et création des comptes ; les droits des jetons sont en lecture seule. Un nouveau mot de passe exige au moins 12 caractères et au plus 72 octets UTF-8. Les nouveaux comptes à jetons n'ont aucun accès initial aux dépôts.

La liste d’autorisation du serveur contrôle la consultation, la création, la réinitialisation des mots de passe et la suppression des comptes. Le menu à trois points aligné à droite permet aux administrateurs de réinitialiser le mot de passe d’un autre utilisateur sans son ancien mot de passe ou de supprimer son compte après confirmation. La réinitialisation conserve les droits sur les dépôts ; la suppression les retire. Votre propre menu ouvre Compte pour changer votre mot de passe avec le mot de passe actuel ; la suppression de votre propre compte est bloquée.

Commencez par [installation](/fr/docs/next/installation), puis examinez [configuration](/fr/docs/next/configuration) et [authentication](/fr/docs/next/authentication).
