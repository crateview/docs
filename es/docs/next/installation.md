---
title: "Instalación"
description: "Inicie la pila completa o conecte la interfaz de usuario a un registro existente."
---

Usa la revisión de esta versión; no clones la rama predeterminada para una instalación archivada.

<!-- release:installation:start -->
Construye primero los contenedores de vista previa desde la revisión fijada; no se presupone una imagen publicada.

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker build -t crateview-registry-ui:next .
```
<!-- release:installation:end -->


## Requisitos

Utilice Node.js 26.13.0 o posterior dentro de v26 para el desarrollo local, o Docker con Compose para la pila completa. Un registro existente debe exponer el Registro HTTP API V2; la pila incluida utiliza la imagen oficial `registry:3.1.2`.

## Ejemplos de Docker Compose

Elija **una** de las siguientes variantes y guárdela como `compose.yaml` en su propio directorio. No se requiere ninguna clonación del repositorio. Cree un archivo `.env` junto con los valores que se muestran para esa variante. Compose pasa explícitamente estos valores al contenedor.

Genere un valor aleatorio de al menos 32 caracteres para `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

Mantenga esta clave estable y proteja `.env`; no lo comprometas con el control de versiones. Las imágenes Crateview publicadas son compatibles con Linux amd64.

### UI con un registro existente

Requiere un punto final de Registro HTTP API V2 accesible desde el contenedor. Ejemplo `.env` (reemplace la dirección; no incluya credenciales en la URL):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: crateview-registry-ui:next
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: ${NUXT_REGISTRY_UPSTREAM:?Set NUXT_REGISTRY_UPSTREAM}
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    restart: unless-stopped
```

```sh
docker compose config --quiet
docker compose up -d
```

Abrir **http://localhost:3000**. Un flujo ascendente de `127.0.0.1` se refiere al contenedor en sí. Utilice un nombre DNS accesible o una dirección de servicio en una red Docker compartida.

### UI con registro propio

Pila local sin autenticación. Sólo el puerto UI se publica en loopback; el registro permanece en la red Docker. `.env`:

```dotenv
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - registry-data:/var/lib/registry
    restart: unless-stopped
  frontend:
    image: crateview-registry-ui:next
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
```

```sh
docker compose config --quiet
docker compose up -d
```

Abrir **http://localhost:3000**. Los datos del registro persisten en el volumen `registry-data`. Push y pull una imagen a través del mismo origen:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, registro y Traefik con HTTPS

Pila independiente que incluye Traefik y Let's Encrypt. Requiere un servidor accesible públicamente, puertos TCP 80/443 disponibles y permitidos y un dominio cuyos registros DNS apunten a ese servidor. El puerto 80 debe permanecer accesible para el desafío HTTP-01. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Ingrese al dominio sin esquema ni ruta. Reemplace todos los valores de ejemplo. Luego cree interactivamente un archivo bcrypt `htpasswd`; reemplace `your-user` con su nombre de usuario:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

Se solicita la contraseña y no aparece en el comando. `-c` crea un nuevo archivo; omítalo al agregar usuarios. Proteja y haga una copia de seguridad del directorio `auth`. Guarde este `compose.yaml` junto a él:

```yaml
services:
  traefik:
    image: traefik:v3.7.17
    command:
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL:?Set ACME_EMAIL}
      - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
      - --certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt
    restart: unless-stopped
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    volumes:
      - registry-data:/var/lib/registry
      - ./auth:/auth:ro
    restart: unless-stopped
  frontend:
    image: crateview-registry-ui:next
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    labels:
      traefik.enable: "true"
      traefik.http.routers.crateview.rule: Host(`${CRATEVIEW_DOMAIN:?Set CRATEVIEW_DOMAIN}`)
      traefik.http.routers.crateview.entrypoints: websecure
      traefik.http.routers.crateview.tls.certresolver: letsencrypt
      traefik.http.routers.crateview.service: crateview
      traefik.http.services.crateview.loadbalancer.server.port: "3000"
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
  letsencrypt:
```

```sh
docker compose config --quiet
docker compose up -d
```

Abra **https://registry.example.com** usando su dominio e inicie sesión con el usuario de registro que creó. Traefik obtiene el certificado automáticamente y redirige HTTP a HTTPS. Los certificados persisten en `letsencrypt`, los datos de registro en `registry-data`. Haga una copia de seguridad de ambos volúmenes y no utilice `docker compose down -v` para apagados de rutina.

Todas las rutas, incluido `/v2/`, pasan por Crateview. Ni el registro ni la interfaz de usuario publican sus propios puertos de host. El registro aplica Basic Auth; no se requiere ningún Traefik Basic Auth adicional. Traefik conserva el host público y reenvía el protocolo HTTPS a Crateview. El zócalo Docker le da acceso al Traefik al Docker API; ejecute esta pila en un host confiable.

Inicio de sesión CLI y transferencia de imágenes (reemplace el dominio):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

La CLI solicita la contraseña por separado del inicio de sesión de la interfaz de usuario. Consulte [authentication](/es/docs/next/authentication) y [Traefik con Docker y Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) para obtener más detalles.

## Utilice un registro existente

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
npm ci
cp .env.example .env
```

Configure el upstream en `.env`, luego inicie el modo de desarrollo:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

La interfaz de usuario se ejecuta en el puerto 3000 con reemplazo de módulo en caliente. Cuando la interfaz se ejecuta dentro de un contenedor, `127.0.0.1` significa ese contenedor. En su lugar, utilice una dirección de servicio accesible, como `http://registry:5000`.

## Servidor de nodo de producción

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

El proceso de producción del Nodo lee automáticamente las variables de entorno reales, no `.env`. Configure su administrador de procesos o entorno de contenedor en consecuencia. Sirve HTTPS para uso autenticado remoto.

Crateview necesita su servidor Nitro para la autenticación y el proxy API. Servir solo los archivos frontend como un sitio web estático no proporcionará una interfaz de usuario de registro que funcione.

Continúe con [configuration](/es/docs/next/configuration) antes de exponer el servicio más allá de su propia máquina.

## Construir desde la fuente

Para una compilación local, utilice los archivos Compose del repositorio de origen:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```

<a id="pwa"></a>

## Instalar como aplicación (PWA)

Instala Crateview mediante la acción nativa de instalación o de añadir a la pantalla de inicio de un navegador compatible. Se abre en modo independiente. La aplicación sigue necesitando su servidor Nitro y un registro accesible.

Usa HTTPS en producción; localhost sirve para las pruebas locales. El service worker solo está activo en las compilaciones de producción, no durante `npm run dev`. Compila e inicia el servidor de producción antes de probar la instalación o el comportamiento sin conexión.

Tras una visita en línea y la instalación del service worker, abrir o recargar la aplicación sin red muestra un aviso de conexión. Reintentar recarga la URL original cuando vuelve la conexión. El idioma guardado tiene prioridad sobre los idiomas del navegador, con inglés como alternativa; árabe y urdu usan RTL.

La página sin conexión solo se utiliza para navegaciones de documentos de la aplicación. Las respuestas del registro, de cuentas y de sesiones no se guardan en Cache Storage; no hay navegación del registro sin conexión. Las pantallas abiertas conservan sus errores habituales. El tráfico de API y `/v2/` no se sustituye por HTML sin conexión y las actualizaciones del worker no fuerzan la recarga de pestañas.
