---
title: "Marca y temática"
description: "Utilice su propia identidad manteniendo el sistema de diseño nativo de la interfaz de usuario Nuxt."
---

![Marca y temática](https://crateview.de/images/docs/v0.4/settings-light.webp)

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

El sitio web Crateview reutiliza la marca de la interfaz de usuario del registro, los valores predeterminados del tema y los nombres de las variables de marca. El sitio web se mantiene por separado; su configuración de origen e implementación son privadas. Sus fuentes de documentación son públicas en [crateview/docs](https://github.com/crateview/docs).
