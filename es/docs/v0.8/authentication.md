---
title: "Autenticación"
description: "Acceso anónimo, Basic Auth, sesiones cifradas y HTTPS."
---

## Autenticación admitida

Crateview admite registros anónimos, htpasswd Basic Auth nativo y el servicio opcional Crateview auth en modo token. Sondea `/v2/` y muestra el formulario de inicio de sesión cuando el registro requiere autenticación básica.

Los servicios externos de token Bearer y OIDC no están integrados. Los servicios de token externos no configurados generan un aviso explícito de no admitido-auth.

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

## Gestión de cuentas opcional

Un servicio de administración/auth independiente admite la creación de cuentas y los cambios de contraseña propia. Docker se conecta directamente al registro y funciona sin el contenedor UI. Los usuarios y los hashes bcrypt permanecen en `htpasswd`; Los permisos de token se encuentran en `permissions.json`. El almacenamiento de la cuenta no utiliza SQLite.

- **Modo básico:** Distribution comprueba el archivo htpasswd compartido; todos los usuarios tienen el mismo acceso al registro. Docker no requiere que se ejecute el servicio de administración.
- **Modo token:** El servicio independiente auth emite tokens firmados de cinco minutos; Distribution aplica los permisos del repositorio. Docker requiere el registro y el servicio auth. Inicialmente, los nuevos usuarios no tienen acceso al repositorio.
- **UI:** La lista de permisos del servidor controla la consulta, creación, restablecimiento de contraseñas y eliminación de cuentas. `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. Un valor vacío deniega la administración; se permiten los nombres indicados y el comodín permite a todos los usuarios autenticados.
- **Permisos:** CLI otorga repositorios exactos de destino, espacios de nombres (`team/*`) o todo (`*`); Los perfiles son `reader` (pull), `writer` (pull/push) y `manager` (pull/push/delete). El modo básico no tiene funciones de registro.

Habilite explícitamente `NUXT_AUTH_MODE=htpasswd` o `token` y configure el `NUXT_AUTH_SERVICE_URL` interno y el `NUXT_AUTH_SERVICE_SECRET` solo de servidor. El modo token además requiere que `NUXT_AUTH_TOKEN_REALM` coincida exactamente con el punto final del token público. Los registros anónimos y los servicios de tokens externos no admiten la administración de cuentas locales. Los permisos de los repositorios se gestionan mediante la CLI de autenticación; la interfaz los muestra en modo de solo lectura.

Consulte la [guía de servicio de cuenta](https://github.com/crateview/registry-ui/blob/244ad8eb2cea0317f9ab149c238def5b414af565/auth/README.md) para superposiciones, configuración, migración y comandos CLI de Compose. Enrute el acceso público Docker directamente al registro independientemente de la interfaz de usuario. Los tokens emitidos previamente siguen siendo válidos hasta cinco minutos después de cambiar la contraseña o revocar el permiso.

Administración agrupa Ajustes y Usuarios. El menú de cuenta también ofrece idioma y tema. Cuenta cambia tu contraseña previa confirmación de la actual. Usuarios solo permite listar y crear cuentas según la lista del servidor; los permisos de token son de solo lectura. Las contraseñas nuevas requieren 12 caracteres como mínimo y 72 bytes UTF-8 como máximo. Las cuentas de token nuevas no tienen acceso inicial.

La lista de permisos del servidor controla la consulta, creación, restablecimiento de contraseñas y eliminación de cuentas. El menú de tres puntos alineado a la derecha permite a los administradores restablecer la contraseña de otro usuario sin la anterior o eliminar su cuenta tras confirmar. El restablecimiento conserva los permisos de los repositorios; la eliminación los retira. Tu propio menú abre Cuenta para cambiar la contraseña con la actual; no puedes eliminar tu propia cuenta.

![Versión de documentación](https://crateview.de/images/docs/v0.8/users-light.webp)

![Comportamiento](https://crateview.de/images/docs/v0.8/user-actions-light.webp)

![Cambiar la contraseña](https://crateview.de/images/docs/v0.8/user-password-light.webp)

![Eliminar usuario](https://crateview.de/images/docs/v0.8/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![Versión de documentación](https://crateview.de/images/docs/v0.8/account-light.webp)

![Versión de documentación](https://crateview.de/images/docs/v0.8/login-light.webp)
