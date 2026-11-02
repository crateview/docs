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


Selecciona una o varias etiquetas en la tabla y confirma la eliminación. La vista previa actualiza etiquetas afectadas, elimina cada digest una vez e informa fallos parciales. Se afectan todos los alias. La recolección de basura sigue siendo independiente.

## Recolección de basura y copias de seguridad.

Eliminar un manifiesto no promete una recuperación inmediata del disco. La recolección de basura es una tarea de administración del registro separada; Crateview no lo ejecuta.

Haga una copia de seguridad del almacenamiento de su registro y su configuración utilizando un método apropiado para el controlador de almacenamiento. Mantenga los volúmenes al reemplazar los contenedores. Siga las instrucciones de Distribution para la recolección de basura y la coordinación de escritura:

- [Distribution recolección de basura](https://distribution.github.io/distribution/about/garbage-collection/)
- [Configuración Distribution](https://distribution.github.io/distribution/about/configuration/)

## Actualizar Crateview

<!-- release:operations:start -->
Construye primero los contenedores de vista previa desde la revisión fijada; no se presupone una imagen publicada.

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
<!-- release:operations:end -->

## Grandes registros

La búsqueda en el repositorio lee el catálogo paginado completo. Los metadatos de los repositorios visibles se obtienen bajo demanda. Ordenar una lista de etiquetas grande por tiempo de creación de imágenes puede requerir la lectura de muchos manifiestos y configuraciones. Crateview no tiene índice de metadatos en segundo plano ni servicio de búsqueda externo.
