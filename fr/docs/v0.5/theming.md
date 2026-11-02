---
title: "Image de marque et thème"
description: "Utilisez votre propre identité tout en conservant le système de conception natif de l’interface utilisateur Nuxt."
---

![Image de marque et thème](https://crateview.de/images/docs/v0.5/settings-light.webp)

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

Le site Web Crateview réutilise la marque de l'interface utilisateur du registre, les valeurs par défaut du thème et les noms des variables de marque. Le site Web est géré séparément ; sa source et sa configuration de déploiement sont privées. Ses sources de documentation sont publiques dans [crateview/docs](https://github.com/crateview/docs).
