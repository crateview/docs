---
title: 'Branding & Theming'
description: 'Eigene Identität mit dem nativen Designsystem von Nuxt UI verbinden.'
---

![Branding & Theming](https://crateview.de/images/docs/v0.8/settings-light.webp)

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

### Serverdateien ohne Neubuild

Alternativ kannst du private serverseitige Dateipfade konfigurieren. Jede konfigurierte Datei hat Vorrang vor ihrer öffentlichen URL; eine leere Datei-Variable behält die URL-Einstellung bei. Absolute Pfade werden unterstützt; relative Pfade beziehen sich auf das Arbeitsverzeichnis des Servers (`/app` im Docker-Image). Dateipfade werden nie an den Browser gesendet. Die Assets selbst sind öffentlich, auch vor der Anmeldung.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Unterstützte Endungen: SVG, PNG, JPG/JPEG, WebP, GIF und ICO. Dateien müssen lesbare, nicht leere reguläre Dateien sein; ungültige Einstellungen verhindern den Serverstart. Dateien werden beim Start geladen: Starte nach Änderungen an Pfaden oder Dateiinhalten neu. Ein Neubuild ist nicht erforderlich.

Erstelle für Docker Compose eine Override-Datei `compose.branding.yaml`:

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

Lege die drei Dateien auf dem Host unter `./branding` an und erlaube dem Container-Benutzer `node` den Lesezugriff. Starte mit `docker compose -f compose.yaml -f compose.branding.yaml up -d`. Der Mount ist schreibgeschützt; konfigurierte Pfade beziehen sich auf Dateien im Container. Diese privaten Datei-Variablen gelten für das Registry-UI.

Die Crateview-Website übernimmt Logo, Theme-Defaults und Branding-Variablennamen aus dem Registry-UI. Die Website wird separat gepflegt; ihr Quellcode und ihre Deployment-Konfiguration sind privat. Die Dokumentationsquellen sind öffentlich in [crateview/docs](https://github.com/crateview/docs).

## Branding der installierten App

Die installierte App übernimmt `NUXT_PUBLIC_BRAND_NAME` und `NUXT_PUBLIC_BRAND_DESCRIPTION`. Eigene PWA-Icons sind unabhängig von Hell-/Dunkel-Logos und Favicon. Das PNG mit 192 Pixeln dient auch als Apple-Touch-Icon. Standardmäßig wird das Crateview-Zeichen verwendet.

Erstelle die beiden PNG-Dateien mit exakt 192×192 und 512×512 Pixeln auf dem Host und gewähre dem Containerbenutzer Lesezugriff. Das folgende Overlay bindet sie schreibgeschützt ein; kombiniere es mit deiner bestehenden Compose-Datei. URL-Overrides verwenden stattdessen `NUXT_PUBLIC_BRAND_PWA_ICON_192` und `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Icon-Änderungen erscheinen in bestehenden Installationen möglicherweise verzögert, da Browser die App-Metadaten verwalten. Ein Neustart von Crateview übernimmt die Laufzeitkonfiguration, garantiert aber keine sofortige Aktualisierung installierter Icons.
