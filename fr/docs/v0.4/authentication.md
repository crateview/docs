---
title: "Authentification"
description: "Accès anonyme, Basic Auth, sessions cryptées et HTTPS."
---

## Authentification prise en charge

Accès anonyme et htpasswd Basic Auth natif pris en charge. Services Bearer externes et OIDC non pris en charge. Pas de gestion des utilisateurs dans cette version.

## Basic Auth côté registre

Créez un fichier htpasswd avec un hachage de mot de passe bcrypt. L'exemple suivant utilise un conteneur Apache officiel temporaire ; remplacez les exemples d'informations d'identification et protégez le fichier résultant :

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Exemples de paramètres Distribution :

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Montez le fichier généré dans le conteneur de registre sur `/auth/htpasswd`. La configuration du registre est distincte de la configuration Crateview. Utilisez HTTPS pour l'accès à distance ; Basic Auth est un codage et non un cryptage de transport. Évitez de laisser de vrais mots de passe dans l’historique du shell partagé ou dans des exemples publics.

## Sessions d'interface utilisateur

Le serveur Nitro valide les informations d'identification par rapport au registre. Il les stocke dans un cookie crypté, HttpOnly, SameSite=Strict. Le navigateur JavaScript et localStorage ne reçoivent pas les informations d'identification. Les sessions expirent après huit heures.

Définissez un secret de session de production aléatoire :

```sh
openssl rand -base64 32
```

Utilisez la valeur générée sous la forme `NUXT_SESSION_PASSWORD`, avec au moins 32 caractères. Chaque instance frontale derrière un équilibreur de charge doit utiliser la même valeur. Sans configuration, Crateview génère un secret local de processus, donc un redémarrage invalide les sessions existantes.

## HTTPS et déconnexion

Le cookie de session utilise Secure lorsque le protocole de requête publique est HTTPS. Configurez correctement le protocole transféré du proxy inverse. La déconnexion efface la session d'interface utilisateur ; il ne supprime ni ne modifie les utilisateurs du registre.

Docker et d'autres clients CLI s'authentifient directement via le proxy `/v2/**` à l'aide de leurs informations d'identification normales de registre. La connexion à l’interface utilisateur Web ne connecte pas automatiquement la CLI Docker.
