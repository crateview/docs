Revise las notas de la versión, configure `CRATEVIEW_VERSION` con la etiqueta de versión con el prefijo `v` deseada en `.env` y reemplace el contenedor de interfaz:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Por ejemplo, la versión inicial utiliza `CRATEVIEW_VERSION={{RELEASE_TAG}}`. Para retroceder, restaure la etiqueta anterior y repita estos comandos. Mantenga el entorno de ejecución y el volumen `registry-data`; no ejecute `docker compose down --volumes`. Las compilaciones de origen siguen estando disponibles a través de `compose.build.yaml`.

Dependabot está configurado en el repositorio de origen para proponer actualizaciones de dependencia; no actualiza automáticamente su implementación autohospedada. Vuelva a probar la conectividad del registro, el inicio de sesión y las acciones afectadas después de una actualización.
