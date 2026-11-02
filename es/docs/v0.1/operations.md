---
title: "Operaciones y eliminación"
description: "Administre los manifiestos con cuidado y mantenga el almacenamiento del registro bajo su control."
---

## Habilitar eliminación de manifiesto

La eliminación está deshabilitada en la interfaz de usuario Crateview de forma predeterminada. Tanto el frontend como el Distribution deben permitirlo:

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution también puede leer `REGISTRY_STORAGE_DELETE_ENABLED=true`. Reinicie los servicios relevantes después de cambiar su entorno o configuración. El registro sigue siendo responsable de la autorización, incluidas las solicitudes realizadas directamente a través de `/v2/**`.

## Comprender la eliminación de digest

La operación de eliminación estándar tiene como objetivo un **manifiesto digest**. Todas las etiquetas que apunten a ese digest se pueden eliminar. Crateview obtiene una vista previa de las etiquetas afectadas antes de la confirmación, pero un escritor simultáneo puede cambiar las referencias después de la vista previa: el registro API no proporciona una transacción que bloquee los alias.

Trate una eliminación como un cambio en el manifiesto y todos sus alias, no como la eliminación de una etiqueta. Verifique sus copias de seguridad y el alcance previsto antes de confirmar.

## Recolección de basura y copias de seguridad.

Eliminar un manifiesto no promete una recuperación inmediata del disco. La recolección de basura es una tarea de administración del registro separada; Crateview no lo ejecuta.

Haga una copia de seguridad del almacenamiento de su registro y su configuración utilizando un método apropiado para el controlador de almacenamiento. Mantenga los volúmenes al reemplazar los contenedores. Siga las instrucciones de Distribution para la recolección de basura y la coordinación de escritura:

- [Distribution recolección de basura](https://distribution.github.io/distribution/about/garbage-collection/)
- [Configuración Distribution](https://distribution.github.io/distribution/about/configuration/)

## Actualizar Crateview

Revise las notas de la versión, configure `CRATEVIEW_VERSION` con la etiqueta de versión con el prefijo `v` deseada en `.env` y reemplace el contenedor de interfaz:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Por ejemplo, la versión inicial utiliza `CRATEVIEW_VERSION=v0.1.2`. Para retroceder, restaure la etiqueta anterior y repita estos comandos. Mantenga el entorno de ejecución y el volumen `registry-data`; no ejecute `docker compose down --volumes`. Las compilaciones de origen siguen estando disponibles a través de `compose.build.yaml`.

Dependabot está configurado en el repositorio de origen para proponer actualizaciones de dependencia; no actualiza automáticamente su implementación autohospedada. Vuelva a probar la conectividad del registro, el inicio de sesión y las acciones afectadas después de una actualización.

## Grandes registros

La búsqueda en el repositorio lee el catálogo paginado completo. Los metadatos de los repositorios visibles se obtienen bajo demanda. Ordenar una lista de etiquetas grande por tiempo de creación de imágenes puede requerir la lectura de muchos manifiestos y configuraciones. Crateview no tiene índice de metadatos en segundo plano ni servicio de búsqueda externo.
