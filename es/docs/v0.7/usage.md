---
title: "Usando Crateview"
description: "Navegue por imágenes, cambie de plataforma e inspeccione los datos detrás de una etiqueta."
---

## Encuentra repositorios y etiquetas

Explorar enumera repositorios. La búsqueda acepta nombres y repository:tag. Abre un repositorio para ver sus etiquetas.

Los repositorios usan tabla por defecto; puedes cambiar a cuadrícula. Los nombres con barras forman espacios de nombres. La barra lateral se redimensiona y contrae, guarda su estado localmente y contiene la búsqueda.

/ o Ctrl+K (Cmd+K en macOS) abre la paleta para buscar repositorios/etiquetas y navegar. Las carpetas se contraen. Ajustes contiene idioma, Sistema/Claro/Oscuro y Acerca de con licencias y avisos originales consultables.

Administración agrupa Ajustes y Usuarios. El menú de cuenta también ofrece idioma y tema. Cuenta cambia tu contraseña previa confirmación de la actual. Usuarios solo permite listar y crear cuentas según la lista del servidor; los permisos de token son de solo lectura. Las contraseñas nuevas requieren 12 caracteres como mínimo y 72 bytes UTF-8 como máximo. Las cuentas de token nuevas no tienen acceso inicial.

La lista de permisos del servidor controla la consulta, creación, restablecimiento de contraseñas y eliminación de cuentas. El menú de tres puntos alineado a la derecha permite a los administradores restablecer la contraseña de otro usuario sin la anterior o eliminar su cuenta tras confirmar. El restablecimiento conserva los permisos de los repositorios; la eliminación los retira. Tu propio menú abre Cuenta para cambiar la contraseña con la actual; no puedes eliminar tu propia cuenta.

![Marca y temática](https://crateview.de/images/docs/v0.7/settings-light.webp)

## Inspeccionar una imagen

Abra un repositorio y seleccione una etiqueta. La vista de imagen muestra su digest, plataformas disponibles, tamaños de capa comprimida y metadatos de creación.

Para una imagen multiplataforma, elija el sistema operativo y la arquitectura en el selector de plataforma. El historial de capas y la configuración de la imagen seleccionada siguen esa elección. La pestaña Manifiesto sin formato muestra el índice o manifiesto raíz de la imagen; Configuración muestra la configuración de la plataforma seleccionada.

![Detalles de la imagen alpina con selección de plataforma, historial de capas y comando pull.](https://crateview.de/images/docs/v0.7/image-light.webp)

## Comandos Pull

El panel de comandos proporciona comandos Docker, Podman y Skopeo para imágenes, y comandos de artefactos ORAS/Podman para artefactos. Copie el comando y ejecútelo en su propia terminal.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Reemplace el host de ejemplo con su origen público Crateview. La autenticación CLI es independiente de la sesión de su navegador.

## Metadatos y referencias OCI

Las descripciones, licencias y enlaces de origen se leen de las anotaciones OCI o de las etiquetas de configuración de imágenes cuando están disponibles. Los metadatos faltantes permanecen vacíos en lugar de ser adivinados.

La pestaña Referentes lee los referentes OCI API. Si el registro no implementa ese punto final, Crateview lee el índice de etiquetas estándar derivado de digest mantenido por los clientes OCI. No crea su propio índice escaneando cada manifiesto. Los artefactos de firma enumerados no se verifican ni se marcan como confiables automáticamente.

## Interpretar marcas de tiempo y tamaños.

El tiempo de creación de la imagen lo proporcionan la configuración de la imagen o las anotaciones. No le indica cuándo se envió una etiqueta a este registro. Los tamaños de las capas comprimidas no miden el consumo de disco, porque las capas se pueden compartir y los datos eliminados pueden permanecer hasta la recolección de basura.
