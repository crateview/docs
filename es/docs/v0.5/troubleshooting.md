---
title: "Solución de problemas"
description: "Resolver problemas de conexión, autenticación, metadatos e implementación."
---

## La interfaz de usuario no puede llegar al registro

Verifique el origen del registro desde el entorno de red del servidor frontend. En un contenedor, `localhost` apunta al propio contenedor. Un Compose común en sentido ascendente es `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

Un registro anónimo debería ser exitoso. Uno autenticado puede devolver `401` con un desafío `WWW-Authenticate`. Un rechazo de conexión significa que el oyente, la dirección o la ruta de red no están listos.

## El inicio de sesión falla

Crateview admite registros anónimos, htpasswd Basic Auth nativo y el servicio opcional Crateview auth en modo token. Sondea `/v2/` y muestra el formulario de inicio de sesión cuando el registro requiere autenticación básica.

Los servicios externos de token Bearer y OIDC no están integrados. Los servicios de token externos no configurados generan un aviso explícito de no admitido-auth.

En caso de pérdida repetida de sesiones, verifique que todas las instancias de frontend compartan `NUXT_SESSION_PASSWORD`. Un secreto generado localmente por el proceso cambia después de un reinicio. Verifique también HTTPS y los encabezados de protocolo reenviados; Las sesiones del navegador requieren que la cookie vuelva al mismo origen.

## Mi imagen no tiene descripción ni fecha de creación.

Estos valores deben existir en las anotaciones del manifiesto o en la configuración de la imagen. Crateview no deduce metadatos faltantes ni sustituye los tiempos del registro push. Agregue las anotaciones OCI apropiadas al crear sus imágenes.

## Los referentes están vacíos.

El registro debe exponer el punto final de referencia OCI o el índice de reserva de etiqueta digest estándar. Los clientes de OCI son responsables de mantener ese índice de reserva cuando sea necesario. El simple hecho de insertar un manifiesto con un tema en un registro sin soporte de referencias no hace que Crateview escanee todas las etiquetas en busca de él.

## Falta la eliminación o se ha rechazado

Habilite la configuración de eliminación de almacenamiento de Crateview, `NUXT_REGISTRY_DELETE_ENABLED` y Distribution. El registro aún puede rechazar la solicitud según sus permisos o implementación. La eliminación funciona mediante digest y puede afectar varias etiquetas.

## La producción ignora mi archivo .env

`node .output/server/index.mjs` no carga automáticamente `.env`. Exporte variables a través de su administrador de procesos, entorno de contenedor o shell. Reinicie después de cambiar los valores de tiempo de ejecución. Un servidor web solo estático no puede reemplazar el servidor Nitro requerido.

## El comando pull copiado no funciona

El comando utiliza la dirección de interfaz pública. Confirme que la CLI pueda llegar a ese origen y confíe en su certificado TLS. Autentique la CLI por separado cuando sea necesario. Un inicio de sesión en el navegador no crea un inicio de sesión CLI Docker.

Si encuentra un problema de interfaz de usuario reproducible, abra un [issue](https://github.com/crateview/registry-ui/issues) con la revisión afectada y los detalles de reproducción desinfectados. Utilice el [canal de seguridad privado](https://github.com/crateview/registry-ui/security/policy) para vulnerabilidades.
## Monitorización

Configuración de esta versión: [Monitorización](/es/docs/v0.5/monitoring).

Configuración de esta versión — La monitorización es opcional y está desactivada por defecto. Resumen abre el panel si está activada; si no, /home redirige a /explore. Activarla da datos operativos globales a todos con acceso a la UI, incluso visitantes anónimos de registros públicos. El modo token gestionado requiere además lectura global.
