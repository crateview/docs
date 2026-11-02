---
title: "Descripción general"
description: "Qué hace Crateview, cómo se conecta a su registro y dónde están sus límites."
---

## Una visión más clara de su registro

Crateview es una interfaz web para registros de contenedores Docker y OCI. Le ayuda a encontrar repositorios y etiquetas, inspeccionar imágenes multiplataforma, leer manifiestos y documentos de configuración, y comprender las capas comprimidas y su historial.

Se ejecuta junto con su registro. El registro almacena imágenes y aplica el control de acceso; Crateview proporciona la interfaz y un proxy del mismo origen. Docker y otros clientes OCI pueden utilizar el origen de la interfaz como dirección de registro.

![Crateview descripción general del repositorio con Alpine y repositorios de demostración anidados.](https://crateview.de/images/docs/v0.4/repositories-light.webp)

## Arquitectura

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

El catálogo no necesita base de datos; la monitorización opcional usa SQLite persistente.

## Funciones compatibles

- Búsqueda de nombres de repositorio, sugerencias de etiquetas y espacios de nombres anidados.
- Tablas de etiquetas, selección de plataforma, tamaños de capas comprimidas e historial de imágenes.
- Anotaciones OCI para descripciones, URL de origen, documentación y licencias.
- Manifiesto sin formato y configuración JSON con soporte para portapapeles.
- Referencias OCI, incluido el índice de reserva de etiqueta digest estándar mantenido por los clientes OCI.
- Acceso anónimo y nativo htpasswd Basic Auth.
- Eliminación de manifiesto opcional, con las etiquetas afectadas mostradas antes de la confirmación.
- Dos idiomas de interfaz (inglés y alemán), Sistema/Claro/Oscuro y marca configurable.


- La monitorización es opcional y está desactivada por defecto. Resumen abre el panel si está activada; si no, /home redirige a /explore. Activarla da datos operativos globales a todos con acceso a la UI, incluso visitantes anónimos de registros públicos.

## Conoce los límites

El API estándar no proporciona marcas de tiempo push, estadísticas de descarga, favoritos ni escaneo de vulnerabilidades. Crateview no fabrica estos valores. La fecha de creación de una imagen proviene de los metadatos de la imagen y no es su hora de carga. Los tamaños mostrados describen las capas de imágenes comprimidas, no el uso del disco del registro.

Los servicios externos de token Bearer, OIDC, administración de usuarios y gestión de claves API no están integrados. Los artefactos de firma pueden enumerarse como referentes; Esta no es una verificación de firma criptográfica.

Comience con [installation](/es/docs/v0.4/installation), luego revise [configuration](/es/docs/v0.4/configuration) y [authentication](/es/docs/v0.4/authentication).
