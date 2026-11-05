---
title: "Authentification"
description: "Accès anonyme, Basic Auth, sessions cryptées et HTTPS."
---

## Authentification prise en charge

Crateview prend en charge les registres anonymes, le service natif htpasswd Basic Auth et le service optionnel Crateview auth en mode jeton. Il sonde `/v2/` et affiche le formulaire de connexion lorsque le registre requiert une authentification de base.

Les services de jetons Bearer externes et OIDC ne sont pas intégrés. Les services de jetons externes non configurés génèrent une notification explicite non prise en charge-auth.

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

## Gestion de compte facultative

Un service d'authentification/gestion indépendant prend en charge la création de compte et les modifications de votre propre mot de passe. Docker se connecte directement au registre et fonctionne sans le conteneur d'interface utilisateur. Les utilisateurs et les hachages bcrypt restent dans `htpasswd` ; les autorisations de jeton résident dans `permissions.json`. Le stockage du compte n'utilise pas SQLite.

- **Mode de base :** Distribution vérifie le fichier htpasswd partagé ; tous les utilisateurs ont un accès égal au registre. Docker ne nécessite pas l'exécution du service de gestion.
- **Mode jeton :** Le service indépendant auth émet des jetons signés de cinq minutes ; Distribution applique les autorisations du référentiel. Docker nécessite le registre et le service auth. Les nouveaux utilisateurs n'ont initialement aucun accès au référentiel.
- **UI:** La liste d’autorisation du serveur contrôle la consultation, la création, la réinitialisation des mots de passe et la suppression des comptes. `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. Une valeur vide interdit l’administration ; les noms listés sont autorisés, et le joker autorise tout utilisateur authentifié.
- **Autorisations :** La CLI accorde les référentiels cibles exacts, les espaces de noms (`team/*`) ou tout (`*`) ; les profils sont `reader` (pull), `writer` (pull/push) et `manager` (pull/push/delete). Le mode de base n'a aucun rôle de registre.

Activez explicitement `NUXT_AUTH_MODE=htpasswd` ou `token` et configurez le `NUXT_AUTH_SERVICE_URL` interne et le `NUXT_AUTH_SERVICE_SECRET` serveur uniquement. Le mode jeton nécessite en outre que `NUXT_AUTH_TOKEN_REALM` corresponde exactement au point de terminaison du jeton public. Les registres anonymes et les services de jetons externes ne prennent pas en charge la gestion des comptes locaux. Les droits sur les dépôts sont gérés via la CLI d’authentification ; l’interface les affiche en lecture seule.

Consultez le [guide de service de compte](https://github.com/crateview/registry-ui/blob/244ad8eb2cea0317f9ab149c238def5b414af565/auth/README.md) pour les superpositions, la configuration, la migration et les commandes CLI de Compose. Acheminez l’accès public Docker directement au registre indépendamment de l’interface utilisateur. Les jetons précédemment émis restent valides jusqu'à cinq minutes après la modification du mot de passe ou la révocation de l'autorisation.

Administration regroupe Paramètres et Utilisateurs. Le menu du compte propose aussi langue et thème. Compte modifie votre mot de passe après vérification de l'actuel. La liste serveur contrôle consultation et création des comptes ; les droits des jetons sont en lecture seule. Un nouveau mot de passe exige au moins 12 caractères et au plus 72 octets UTF-8. Les nouveaux comptes à jetons n'ont aucun accès initial aux dépôts.

La liste d’autorisation du serveur contrôle la consultation, la création, la réinitialisation des mots de passe et la suppression des comptes. Le menu à trois points aligné à droite permet aux administrateurs de réinitialiser le mot de passe d’un autre utilisateur sans son ancien mot de passe ou de supprimer son compte après confirmation. La réinitialisation conserve les droits sur les dépôts ; la suppression les retire. Votre propre menu ouvre Compte pour changer votre mot de passe avec le mot de passe actuel ; la suppression de votre propre compte est bloquée.

![Version de documentation](https://crateview.de/images/docs/v0.8/users-light.webp)

![Actes](https://crateview.de/images/docs/v0.8/user-actions-light.webp)

![Changer le mot de passe](https://crateview.de/images/docs/v0.8/user-password-light.webp)

![Supprimer l’utilisateur](https://crateview.de/images/docs/v0.8/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![Version de documentation](https://crateview.de/images/docs/v0.8/account-light.webp)

![Version de documentation](https://crateview.de/images/docs/v0.8/login-light.webp)
