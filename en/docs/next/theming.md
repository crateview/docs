---
title: 'Branding & theming'
description: 'Use your own identity while keeping Nuxt UI’s native design system.'
---

![Branding & theming](https://crateview.de/images/docs/next/settings-light.webp)

## Defaults

Crateview uses Nuxt UI's `red` primary palette and `zinc` neutral palette. The default color mode is System; users can choose Light or Dark independently of your color configuration.

## Runtime theme variables

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

The primary palette accepts standard Tailwind accent colors such as `blue`, `emerald` or `violet`. Neutral accepts `slate`, `gray`, `zinc`, `neutral` or `stone`. Radius accepts nonnegative `px` or `rem` values. Font configuration selects a font stack; bundle font files and declare `@font-face` if your font is not installed on the client.

Restart the server after changing production environment variables. No rebuild is needed for these runtime settings.

## Application and component configuration

Defaults are defined in `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Use normal Nuxt UI `ui` configuration for component variants and icons. Additional CSS design tokens and light/dark surface rules live in `app/assets/css/main.css`. Source changes require rebuilding the production application.

## Replace the brand

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Place local assets in `public/` before building, or use HTTP(S) asset URLs. Existing optional product, documentation and repository links can be configured through their corresponding `NUXT_PUBLIC_BRAND_*_URL` variables. Public configuration is visible in the browser and must never contain secrets.

The Crateview website reuses the registry UI's mark, theme defaults and branding variable names. The website is maintained separately; its source and deployment configuration are private. Its documentation sources are public in [crateview/docs](https://github.com/crateview/docs).
