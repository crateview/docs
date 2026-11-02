---
title: 'Branding & Theming'
description: 'Eigene Identität mit dem nativen Designsystem von Nuxt UI verbinden.'
---

![Branding & Theming](https://crateview.de/images/docs/v0.4/settings-light.webp)

## Standardwerte

Crateview verwendet die Nuxt-UI-Paletten `red` für Primary und `zinc` für Neutral. Der Standard-Farbmodus ist System. Benutzer können unabhängig von deiner Farbkonfiguration Hell oder Dunkel wählen.

## Theme-Variablen zur Laufzeit

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

Primary akzeptiert die üblichen Tailwind-Akzentfarben wie `blue`, `emerald` oder `violet`. Neutral akzeptiert `slate`, `gray`, `zinc`, `neutral` oder `stone`. Der Radius muss ein nicht negativer `px`- oder `rem`-Wert sein. Die Schriftkonfiguration wählt einen Font-Stack. Liefere Font-Dateien mit und definiere `@font-face`, wenn die Schrift auf den Clients nicht installiert ist.

Starte den Server nach Änderungen an produktiven Umgebungsvariablen neu. Für diese Laufzeiteinstellungen ist kein neuer Build erforderlich.

## App- und Komponenten-Konfiguration

Die Standardwerte stehen in `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Verwende die normale Nuxt-UI-Konfiguration unter `ui` für Komponentenvarianten und Icons. Weitere CSS-Designtokens und Regeln für helle/dunkle Oberflächen liegen in `app/assets/css/main.css`. Änderungen am Quellcode benötigen einen neuen Produktionsbuild.

## Branding ersetzen

```dotenv
NUXT_PUBLIC_BRAND_NAME=Meine Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · Meine Registry
```

Lege lokale Assets vor dem Build in `public/` ab oder verwende HTTP(S)-URLs. Optionale Links zu Produkt, Dokumentation und Repository lassen sich über die passenden `NUXT_PUBLIC_BRAND_*_URL`-Variablen setzen. Öffentliche Konfiguration ist im Browser sichtbar und darf keine Geheimnisse enthalten.

Die Crateview-Website übernimmt Logo, Theme-Defaults und Branding-Variablennamen aus dem Registry-UI. Die Website wird separat gepflegt; ihr Quellcode und ihre Deployment-Konfiguration sind privat. Die Dokumentationsquellen sind öffentlich in [crateview/docs](https://github.com/crateview/docs).
