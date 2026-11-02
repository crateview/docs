---
title: 'Image de marque et thème'
description: 'Utilisez votre propre identité tout en conservant le système de conception natif de l’interface utilisateur Nuxt.'
---

![Image de marque et thème](https://crateview.de/images/docs/next/settings-light.webp)

## Valeurs par défaut

Crateview utilise la palette primaire `red` de Nuxt UI et la palette neutre `zinc`. Le mode couleur par défaut est Système ; les utilisateurs peuvent choisir Clair ou Foncé indépendamment de votre configuration de couleurs.

## Variables du thème d'exécution

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

La palette principale accepte les couleurs d'accent Tailwind standard telles que `blue`, `emerald` ou `violet`. Le neutre accepte `slate`, `gray`, `zinc`, `neutral` ou `stone`. Radius accepte les valeurs `px` ou `rem` non négatives. La configuration des polices sélectionne une pile de polices ; regroupez les fichiers de polices et déclarez `@font-face` si votre police n’est pas installée sur le client.

Redémarrez le serveur après avoir modifié les variables de l'environnement de production. Aucune reconstruction n'est nécessaire pour ces paramètres d'exécution.

## Configuration des applications et des composants

Les valeurs par défaut sont définies dans `app/app.config.ts` :

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Utilisez la configuration Nuxt normale de l'interface utilisateur `ui` pour les variantes et les icônes de composants. Des jetons de conception CSS supplémentaires et des règles de surface claire/foncée se trouvent dans `app/assets/css/main.css`. Les modifications de source nécessitent la reconstruction de l'application de production.

## Remplacer la marque

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Placez les ressources locales dans `public/` avant de créer ou utilisez les URL des ressources HTTP(S). Les liens de produits, de documentation et de référentiel facultatifs existants peuvent être configurés via leurs variables `NUXT_PUBLIC_BRAND_*_URL` correspondantes. La configuration publique est visible dans le navigateur et ne doit jamais contenir de secrets.

### Fichiers serveur sans recompilation

Vous pouvez aussi configurer des chemins de fichiers privés côté serveur. Chaque fichier configuré remplace son URL publique correspondante ; une variable de fichier vide conserve le réglage URL. Les chemins absolus sont acceptés ; les chemins relatifs partent du répertoire de travail du serveur (`/app` dans l’image Docker). Les chemins ne sont jamais envoyés au navigateur. Les ressources restent publiques, même avant la connexion.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Extensions acceptées : SVG, PNG, JPG/JPEG, WebP, GIF et ICO. Les fichiers doivent être ordinaires, lisibles et non vides ; une configuration invalide empêche le démarrage du serveur. Les fichiers sont chargés au démarrage : redémarrez après tout changement de chemin ou de contenu. Aucune recompilation n’est nécessaire.

Pour Docker Compose, créez un fichier de surcharge `compose.branding.yaml` :

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_LOGO_LIGHT_FILE: /app/branding/logo-light.svg
      NUXT_BRAND_LOGO_DARK_FILE: /app/branding/logo-dark.svg
      NUXT_BRAND_FAVICON_FILE: /app/branding/favicon.ico
```

Créez les trois fichiers dans `./branding` sur l’hôte et autorisez l’utilisateur `node` du conteneur à les lire. Lancez `docker compose -f compose.yaml -f compose.branding.yaml up -d`. Le montage est en lecture seule ; les chemins configurés désignent les fichiers dans le conteneur. Ces variables de fichiers privées concernent l’interface du registre.

Le site Web Crateview réutilise la marque de l'interface utilisateur du registre, les valeurs par défaut du thème et les noms des variables de marque. Le site Web est géré séparément ; sa source et sa configuration de déploiement sont privées. Ses sources de documentation sont publiques dans [crateview/docs](https://github.com/crateview/docs).

## Personnaliser l’application installée

L’application installée utilise `NUXT_PUBLIC_BRAND_NAME` et `NUXT_PUBLIC_BRAND_DESCRIPTION`. Les icônes PWA sont indépendantes des logos clair/sombre et du favicon. Le PNG de 192 pixels sert aussi d’icône tactile Apple. Les icônes par défaut utilisent le symbole Crateview.

Créez sur l’hôte les deux PNG d’exactement 192×192 et 512×512 pixels, lisibles par l’utilisateur du conteneur. La surcharge suivante les monte en lecture seule ; combinez-la à votre fichier Compose existant. Pour des URL, utilisez `NUXT_PUBLIC_BRAND_PWA_ICON_192` et `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Les changements d’icônes peuvent apparaître avec un délai, car les navigateurs gèrent les métadonnées des applications installées. Redémarrer Crateview applique la configuration, mais ne garantit pas une mise à jour immédiate des icônes existantes.
