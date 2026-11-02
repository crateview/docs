---
title: 'Marca y temática'
description: 'Utilice su propia identidad manteniendo el sistema de diseño nativo de la interfaz de usuario Nuxt.'
---

![Marca y temática](https://crateview.de/images/docs/v0.6/settings-light.webp)

## Valores predeterminados

Crateview utiliza la paleta principal `red` de Nuxt UI y la paleta neutra `zinc`. El modo de color predeterminado es Sistema; los usuarios pueden elegir Claro u Oscuro independientemente de su configuración de color.

## Variables del tema en tiempo de ejecución

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

La paleta principal acepta colores de acento estándar Tailwind como `blue`, `emerald` o `violet`. Neutral acepta `slate`, `gray`, `zinc`, `neutral` o `stone`. Radius acepta valores `px` o `rem` no negativos. La configuración de fuentes selecciona una pila de fuentes; empaquete los archivos de fuentes y declare `@font-face` si su fuente no está instalada en el cliente.

Reinicie el servidor después de cambiar las variables del entorno de producción. No es necesaria ninguna reconstrucción para estas configuraciones de tiempo de ejecución.

## Configuración de aplicaciones y componentes

Los valores predeterminados se definen en `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Utilice la configuración normal de Nuxt UI `ui` para las variantes de componentes e íconos. Tokens de diseño CSS adicionales y reglas de superficie clara/oscura se encuentran en `app/assets/css/main.css`. Los cambios de fuente requieren reconstruir la aplicación de producción.

## Reemplazar la marca

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Coloque los activos locales en `public/` antes de compilarlos o utilice URL de activos HTTP(S). Los enlaces de repositorio, documentación y productos opcionales existentes se pueden configurar a través de sus variables `NUXT_PUBLIC_BRAND_*_URL` correspondientes. La configuración pública es visible en el navegador y nunca debe contener secretos.

### Archivos del servidor sin recompilar

También puedes configurar rutas privadas a archivos del servidor. Cada archivo configurado tiene prioridad sobre su URL pública correspondiente; una variable de archivo vacía conserva la URL. Se admiten rutas absolutas; las relativas parten del directorio de trabajo del servidor (`/app` en la imagen Docker). Las rutas nunca se envían al navegador. Los recursos son públicos, incluso antes de iniciar sesión.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Extensiones admitidas: SVG, PNG, JPG/JPEG, WebP, GIF e ICO. Deben ser archivos normales, legibles y no vacíos; una configuración inválida impide iniciar el servidor. Los archivos se cargan al iniciar: reinicia después de cambiar rutas o contenidos. No es necesario recompilar.

Para Docker Compose, crea un archivo de sobrescritura `compose.branding.yaml`:

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

Crea los tres archivos en `./branding` del host y permite que el usuario `node` del contenedor los lea. Ejecuta `docker compose -f compose.yaml -f compose.branding.yaml up -d`. El montaje es de solo lectura; las rutas configuradas corresponden a archivos dentro del contenedor. Estas variables privadas de archivos se aplican a la interfaz del registro.

El sitio web Crateview reutiliza la marca de la interfaz de usuario del registro, los valores predeterminados del tema y los nombres de las variables de marca. El sitio web se mantiene por separado; su configuración de origen e implementación son privadas. Sus fuentes de documentación son públicas en [crateview/docs](https://github.com/crateview/docs).

## Personalizar la aplicación instalada

La aplicación instalada usa `NUXT_PUBLIC_BRAND_NAME` y `NUXT_PUBLIC_BRAND_DESCRIPTION`. Los iconos PWA son independientes de los logotipos claro/oscuro y del favicon. El PNG de 192 píxeles también es el icono táctil de Apple. Los iconos predeterminados muestran la marca de Crateview.

Crea los dos PNG de exactamente 192×192 y 512×512 píxeles en el host y permite su lectura al usuario del contenedor. La siguiente superposición los monta como solo lectura; combínala con tu archivo Compose existente. Para usar URL, configura `NUXT_PUBLIC_BRAND_PWA_ICON_192` y `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Los cambios de icono pueden tardar en aparecer porque el navegador administra los metadatos de la aplicación instalada. Reiniciar Crateview aplica la configuración de ejecución, pero no garantiza una actualización inmediata de iconos existentes.
