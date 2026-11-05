---
title: "Descripción general"
description: "Qué hace Crateview, cómo se conecta a su registro y dónde están sus límites."
---

## Una visión más clara de su registro

Crateview es una interfaz web para registros de contenedores Docker y OCI. Le ayuda a encontrar repositorios y etiquetas, inspeccionar imágenes multiplataforma, leer manifiestos y documentos de configuración, y comprender las capas comprimidas y su historial.

Se ejecuta junto con su registro. El registro almacena imágenes y aplica el control de acceso; Crateview proporciona la interfaz y un proxy del mismo origen. Docker y otros clientes OCI pueden utilizar el origen de la interfaz como dirección de registro.

![Crateview descripción general del repositorio con Alpine y repositorios de demostración anidados.](https://crateview.de/images/docs/v0.8/repositories-light.webp)

## Arquitectura

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

El catálogo no necesita base de datos; la monitorización opcional usa SQLite persistente. Las cuentas usan htpasswd y un archivo de permisos separado.

## Funciones compatibles

- Búsqueda de nombres de repositorio, sugerencias de etiquetas y espacios de nombres anidados.
- Tablas de etiquetas, selección de plataforma, tamaños de capas comprimidas e historial de imágenes.
- Anotaciones OCI para descripciones, URL de origen, documentación y licencias.
- Manifiesto sin formato y configuración JSON con soporte para portapapeles.
- Referencias OCI, incluido el índice de reserva de etiqueta digest estándar mantenido por los clientes OCI.
- Acceso anónimo y nativo htpasswd Basic Auth.
- Eliminación de manifiesto opcional, con las etiquetas afectadas mostradas antes de la confirmación.
- 13 idiomas de interfaz, Sistema/Claro/Oscuro y marca configurable.

- La monitorización es opcional y está desactivada por defecto. Resumen abre el panel si está activada; si no, /home redirige a /explore. Activarla da datos operativos globales a todos con acceso a la UI, incluso visitantes anónimos de registros públicos. El modo token gestionado requiere además lectura global.

## Conoce los límites

El API estándar no proporciona marcas de tiempo push, estadísticas de descarga, favoritos ni escaneo de vulnerabilidades. Crateview no fabrica estos valores. La fecha de creación de una imagen proviene de los metadatos de la imagen y no es su hora de carga. Los tamaños mostrados describen las capas de imágenes comprimidas, no el uso del disco del registro.

Administración agrupa Ajustes y Usuarios. El menú de cuenta también ofrece idioma y tema. Cuenta cambia tu contraseña previa confirmación de la actual. Usuarios solo permite listar y crear cuentas según la lista del servidor; los permisos de token son de solo lectura. Las contraseñas nuevas requieren 12 caracteres como mínimo y 72 bytes UTF-8 como máximo. Las cuentas de token nuevas no tienen acceso inicial.

La lista de permisos del servidor controla la consulta, creación, restablecimiento de contraseñas y eliminación de cuentas. El menú de tres puntos alineado a la derecha permite a los administradores restablecer la contraseña de otro usuario sin la anterior o eliminar su cuenta tras confirmar. El restablecimiento conserva los permisos de los repositorios; la eliminación los retira. Tu propio menú abre Cuenta para cambiar la contraseña con la actual; no puedes eliminar tu propia cuenta.

Comience con [installation](/es/docs/v0.8/installation), luego revise [configuration](/es/docs/v0.8/configuration) y [authentication](/es/docs/v0.8/authentication).

## Instalar como aplicación (PWA)

Aplicación instalable con aviso de inicio sin conexión traducido, reintento e iconos configurables. Navegar por el registro y usar cuentas requiere conexión de red.

[Instalar como aplicación (PWA)](/es/docs/v0.8/installation#pwa)
