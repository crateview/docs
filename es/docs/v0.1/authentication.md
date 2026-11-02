---
title: "Autenticación"
description: "Acceso anónimo, Basic Auth, sesiones cifradas y HTTPS."
---

## Autenticación admitida

Se admite acceso anónimo y htpasswd Basic Auth nativo. No se admiten servicios Bearer externos ni OIDC. Esta versión no administra usuarios.

## Basic Auth del lado del registro

Cree un archivo htpasswd con un hash de contraseña de bcrypt. El siguiente ejemplo utiliza un contenedor Apache oficial temporal; reemplace las credenciales de ejemplo y proteja el archivo resultante:

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Ejemplo de configuración de Distribution:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Monte el archivo generado en el contenedor de registro en `/auth/htpasswd`. La configuración del registro es independiente de la configuración de Crateview. Utilice HTTPS para acceso remoto; Basic Auth es una codificación, no una codificación de transporte. Evite dejar contraseñas reales en el historial de shell compartido o en ejemplos públicos.

## sesiones de interfaz de usuario

El servidor Nitro valida las credenciales con el registro. Los almacena en una cookie cifrada, HttpOnly, SameSite=Strict. El navegador JavaScript y localStorage no reciben las credenciales. Las sesiones caducan después de ocho horas.

Establezca un secreto de sesión de producción aleatoria:

```sh
openssl rand -base64 32
```

Utilice el valor generado como `NUXT_SESSION_PASSWORD`, con al menos 32 caracteres. Cada instancia de frontend detrás de un balanceador de carga debe usar el mismo valor. Sin configuración, Crateview genera un secreto de proceso local, por lo que un reinicio invalida las sesiones existentes.

## HTTPS y cerrar sesión

La cookie de sesión utiliza Secure cuando el protocolo de solicitud pública es HTTPS. Configure correctamente el protocolo de reenvío del proxy inverso. Al cerrar sesión se borra la sesión de UI; no elimina ni modifica los usuarios del registro.

Docker y otros clientes CLI se autentican directamente a través del proxy `/v2/**` utilizando sus credenciales de registro normales. Iniciar sesión en la interfaz de usuario web no inicia sesión automáticamente en la CLI Docker.
