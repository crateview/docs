---
title: "Descripción general"
description: "Qué hace Crateview, cómo se conecta a su registro y dónde están sus límites."
---

## Una visión más clara de su registro

Crateview es una interfaz web para registros de contenedores Docker y OCI. Le ayuda a encontrar repositorios y etiquetas, inspeccionar imágenes multiplataforma, leer manifiestos y documentos de configuración, y comprender las capas comprimidas y su historial.

Se ejecuta junto con su registro. El registro almacena imágenes y aplica el control de acceso; Crateview proporciona la interfaz y un proxy del mismo origen. Docker y otros clientes OCI pueden utilizar el origen de la interfaz como dirección de registro.

![Crateview descripción general del repositorio con Alpine y repositorios de demostración anidados.](https://crateview.de/images/docs/v0.1/repositories-light.webp)

## Arquitectura

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

La interfaz lee el Registro estándar HTTP API V2. Los metadatos de la interfaz de usuario se derivan del catálogo, las listas de etiquetas, los manifiestos y los blobs de configuración de imágenes. No necesita un índice de búsqueda independiente ni una base de datos de aplicaciones.

## Funciones compatibles

- Búsqueda de nombres de repositorio, sugerencias de etiquetas y espacios de nombres anidados.
- Tablas de etiquetas, selección de plataforma, tamaños de capas comprimidas e historial de imágenes.
- Anotaciones OCI para descripciones, URL de origen, documentación y licencias.
- Manifiesto sin formato y configuración JSON con soporte para portapapeles.
- Referencias OCI, incluido el índice de reserva de etiqueta digest estándar mantenido por los clientes OCI.
- Acceso anónimo y nativo htpasswd Basic Auth.
- Eliminación de manifiesto opcional, con las etiquetas afectadas mostradas antes de la confirmación.
- Dos idiomas de interfaz (inglés y alemán), Sistema/Claro/Oscuro y marca configurable.


## Conoce los límites

El API estándar no proporciona marcas de tiempo push, estadísticas de descarga, favoritos ni escaneo de vulnerabilidades. Crateview no fabrica estos valores. La fecha de creación de una imagen proviene de los metadatos de la imagen y no es su hora de carga. Los tamaños mostrados describen las capas de imágenes comprimidas, no el uso del disco del registro.

Los servicios externos de token Bearer, OIDC, administración de usuarios y gestión de claves API no están integrados. Los artefactos de firma pueden enumerarse como referentes; Esta no es una verificación de firma criptográfica.

Comience con [installation](/es/docs/v0.1/installation), luego revise [configuration](/es/docs/v0.1/configuration) y [authentication](/es/docs/v0.1/authentication).
