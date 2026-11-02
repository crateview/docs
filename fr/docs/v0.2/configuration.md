---
title: "Configuration"
description: "Connexion au registre, variables d'environnement d'exécution et comportement de déploiement."
---

## Couches de configuration

Les valeurs par défaut de l'application sont définies dans `nuxt.config.ts` et `app/app.config.ts`. Les variables d'environnement remplacent les valeurs par défaut d'exécution au démarrage du serveur. Les commandes de développement et de construction lisent `.env` ; le point d'entrée du nœud de production ne le lit pas automatiquement.

Redémarrez le serveur après avoir modifié les variables d'environnement d'exécution. La modification de la configuration des composants au niveau source nécessite une nouvelle version de production.

## Paramètres du registre et de la session

| Variable | Défaut | But |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM` | `http://127.0.0.1:5000` | Origine du registre côté serveur. |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false` | Exposez la suppression du manifeste dans l’interface utilisateur. |
| `NUXT_SESSION_PASSWORD` | Aléatoire par processus | Secret utilisé pour chiffrer les sessions d'interface utilisateur. |
| `HOST` | Dépend du temps d'exécution | Interface d'écoute du serveur de nœuds. |
| `PORT` | `3000` | Port d'écoute du serveur de nœud. |

L'amont doit être une origine HTTP(S) accessible depuis le serveur frontend. Conservez les informations d'identification en dehors de l'URL. Les requêtes utilisent l'authentification de l'utilisateur actuel ; Crateview ne remplace pas le contrôle d'accès au registre.

## Variables de marque publique

Tous les noms ci-dessous utilisent le préfixe `NUXT_PUBLIC_BRAND_` :

| Suffixe | But |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME` | Nom de l'application. |
| `DESCRIPTION` | Description de l'application. |
| `LOGO_LIGHT`, `LOGO_DARK` | Chemins de logo ou URL HTTP(S). |
| `FAVICON` | Chemin d’accès ou URL de l’icône du navigateur. |
| `TITLE_TEMPLATE` | Titre de la page ; `%s` est remplacé par le titre de la page. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Liens de navigation facultatifs. |

La configuration publique est envoyée au navigateur. Ne stockez jamais de mots de passe ou de jetons dans des variables `NUXT_PUBLIC_*`. Voir [branding et thème](/fr/docs/v0.2/theming) pour les options de couleur et de typographie.

## Disposition du réseau

Le navigateur communique avec Crateview sur une seule origine. Nitro proxy `/v2/**` pour les clients de registre et lit les métadonnées du registre via `/api/registry/**`. Cela évite de nécessiter un accès du navigateur d’origine croisée au registre lui-même.

Derrière un proxy inverse, préservez l'origine publique et les en-têtes de protocole. L'adresse publique du frontend est également l'adresse de registre affichée dans les commandes pull copiées. Pour une utilisation authentifiée à distance, terminez correctement HTTPS avant d'accepter les informations d'identification du navigateur ou de la CLI.

## Compose par rapport à une configuration autonome

Le fichier Compose fourni se connecte intentionnellement à son propre service `registry` avec `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. Pour utiliser un autre registre dans un déploiement de conteneur, adaptez cet environnement de service ou exécutez le frontend séparément. Ne présumez pas qu’un registre de bouclage hôte est accessible via une passerelle de conteneur.
