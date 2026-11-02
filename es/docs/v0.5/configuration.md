---
title: "Configuración"
description: "Conexión de registro, variables de entorno de ejecución y comportamiento de implementación."
---

## Capas de configuración

Los valores predeterminados de la aplicación se definen en `nuxt.config.ts` y `app/app.config.ts`. Las variables de entorno anulan los valores predeterminados del tiempo de ejecución cuando se inicia el servidor. Los comandos de desarrollo y construcción leen `.env`; el punto de entrada del nodo de producción no lo lee automáticamente.

Reinicie el servidor después de cambiar las variables del entorno de ejecución. La edición de la configuración de componentes a nivel de fuente requiere una nueva compilación de producción.

## Configuración de registro y sesión

| Variable | Por defecto | Objetivo |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM` | `http://127.0.0.1:5000` | Origen del registro del lado del servidor. |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false` | Exponer la eliminación manifiesta en la interfaz de usuario. |
| `NUXT_SESSION_PASSWORD` | Aleatorio por proceso | Secreto utilizado para cifrar sesiones de UI. |
| `HOST` | Dependiente del tiempo de ejecución | Interfaz de escucha del servidor de nodo. |
| `PORT` | `3000` | Puerto de escucha del servidor de nodo. |

El flujo ascendente debe ser un origen HTTP(S) accesible desde el servidor frontend. Mantenga las credenciales fuera de la URL. Las solicitudes utilizan la autenticación del usuario actual; Crateview no reemplaza el control de acceso al registro.

## Variables de marca pública

Todos los nombres siguientes utilizan el prefijo `NUXT_PUBLIC_BRAND_`:

| Sufijo | Objetivo |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME` | Nombre de la aplicación. |
| `DESCRIPTION` | Descripción de la aplicación. |
| `LOGO_LIGHT`, `LOGO_DARK` | Rutas de logotipos o URL HTTP(S). |
| `FAVICON` | Ruta o URL del icono del navegador. |
| `TITLE_TEMPLATE` | Título de la página; `%s` se reemplaza por el título de la página. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Enlaces de navegación opcionales. |

La configuración pública se envía al navegador. Nunca almacene contraseñas o tokens en variables `NUXT_PUBLIC_*`. Consulte [marca y temática](/es/docs/v0.5/theming) para conocer las opciones de color y tipografía.

## Diseño de red

El navegador se comunica con Crateview en un origen. Nitro representa `/v2/**` para clientes de registro y lee metadatos de registro a través de `/api/registry/**`. Esto evita requerir acceso de navegador de origen cruzado al propio registro.

Detrás de un proxy inverso, conserve el origen público y los encabezados de protocolo. La dirección de interfaz pública también es la dirección de registro que se muestra en los comandos pull copiados. Para uso con autenticación remota, finalice HTTPS correctamente antes de aceptar las credenciales del navegador o CLI.

## Compose versus configuración independiente

El archivo Compose incluido se conecta intencionalmente a su propio servicio `registry` con `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. Para utilizar otro registro en una implementación de contenedor, adapte ese entorno de servicio o ejecute la interfaz por separado. No asuma que se puede acceder a un registro de bucle invertido de host a través de una puerta de enlace de contenedor.
## Configuración de esta versión

La monitorización es opcional y está desactivada por defecto. Resumen abre el panel si está activada; si no, /home redirige a /explore. Activarla da datos operativos globales a todos con acceso a la UI, incluso visitantes anónimos de registros públicos. El modo token gestionado requiere además lectura global.

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[Monitorización](/es/docs/v0.5/monitoring)

| ENV | Default |
| --- | --- |
| `NUXT_AUTH_MODE` | `off` |
| `NUXT_AUTH_SERVICE_URL` | — |
| `NUXT_AUTH_SERVICE_SECRET` | — |
| `NUXT_AUTH_TOKEN_REALM` | — |
| `NUXT_USER_CREATION_ALLOWED_USERS` | — |

Administración agrupa Ajustes y Usuarios. El menú de cuenta también ofrece idioma y tema. Cuenta cambia tu contraseña previa confirmación de la actual. Usuarios solo permite listar y crear cuentas según la lista del servidor; los permisos de token son de solo lectura. Las contraseñas nuevas requieren 12 caracteres como mínimo y 72 bytes UTF-8 como máximo. Las cuentas de token nuevas no tienen acceso inicial.
