---
title: "Instalação"
description: "Inicie a pilha completa ou conecte a UI a um registro existente."
---

Use a revisão desta versão; não clone a branch padrão para uma instalação arquivada.

<!-- release:installation:start -->
Compile contêineres de prévia localmente da revisão fixada; não se presume imagem publicada.

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker build -t crateview-registry-ui:next .
```
<!-- release:installation:end -->


## Requisitos

Use Node.js 26.13.0 ou mais recente na v26 para desenvolvimento local ou Docker com Compose para a pilha completa. Um registro existente deve expor o Registro HTTP API V2; a pilha empacotada usa a imagem oficial `registry:3.1.2`.

## Exemplos de Docker Compose

Escolha **uma** das variantes a seguir e salve-a como `compose.yaml` em seu próprio diretório. Nenhum clone de repositório é necessário. Crie um arquivo `.env` junto com os valores mostrados para essa variante. O Compose passa explicitamente esses valores para o contêiner.

Gere um valor aleatório de pelo menos 32 caracteres para `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

Mantenha esta chave estável e proteja `.env`; não o comprometa com o controle de versão. As imagens Crateview publicadas suportam Linux amd64.

### UI com um registro existente

Requer um endpoint HTTP API V2 do Registro acessível a partir do contêiner. Exemplo `.env` (substitua o endereço; não inclua credenciais na URL):

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

Abrir **http://localhost:3000**. Um upstream de `127.0.0.1` refere-se ao próprio contêiner. Use um nome DNS acessível ou um endereço de serviço em uma rede Docker compartilhada.

### UI com seu próprio registro

Pilha local sem autenticação. Somente a porta UI é publicada em loopback; o registro permanece na rede Docker. `.env`:

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

Abrir **http://localhost:3000**. Os dados do registro persistem no volume `registry-data`. Push e pull uma imagem através da mesma origem:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, registro e Traefik com HTTPS

Pilha autônoma incluindo Traefik e Let's Encrypt. Requer um servidor acessível publicamente, portas TCP 80/443 disponíveis e permitidas e um domínio cujos registros DNS apontem para esse servidor. A porta 80 deve permanecer acessível para o desafio HTTP-01. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Insira o domínio sem esquema ou caminho. Substitua todos os valores de exemplo. Em seguida, crie interativamente um arquivo bcrypt `htpasswd`; substitua `your-user` pelo seu nome de usuário:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

A senha é solicitada e não aparece no comando. `-c` cria um novo arquivo; omita-o ao adicionar usuários. Proteja e faça backup do diretório `auth`. Salve este `compose.yaml` ao lado dele:

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

Abra **https://registry.example.com** usando seu domínio e faça login com o usuário de registro que você criou. Traefik obtém o certificado automaticamente e redireciona HTTP para HTTPS. Os certificados persistem em `letsencrypt`, os dados de registro em `registry-data`. Faça backup de ambos os volumes e não use `docker compose down -v` para encerramentos de rotina.

Todos os caminhos, incluindo `/v2/`, passam por Crateview. Nem o registro nem a UI publicam suas próprias portas de host. O registro impõe Basic Auth; nenhum Traefik Basic Auth adicional é necessário. Traefik preserva o host público e encaminha o protocolo HTTPS para Crateview. O soquete Docker dá acesso Traefik ao Docker API; execute esta pilha em um host confiável.

Login CLI e transferência de imagem (substitua o domínio):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

A CLI solicita a senha separadamente do login da UI. Consulte [autenticação](/pt-BR/docs/next/authentication) e [Traefik com Docker e Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) para obter detalhes.

## Use um registro existente

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
npm ci
cp .env.example .env
```

Defina o upstream em `.env` e inicie o modo de desenvolvimento:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

A UI é executada na porta 3000 com substituição dinâmica do módulo. Quando o frontend é executado dentro de um contêiner, `127.0.0.1` significa esse contêiner. Use um endereço de serviço acessível, como `http://registry:5000`.

## Servidor do nó de produção

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

O processo do nó de produção lê variáveis ​​de ambiente reais, não `.env` automaticamente. Configure seu gerenciador de processos ou ambiente de contêiner adequadamente. Sirva HTTPS para uso autenticado remotamente.

Crateview precisa de seu servidor Nitro para autenticação e proxy API. Servir apenas os arquivos de front-end como um site estático não fornecerá uma IU de registro funcional.

Continue com [configuration](/pt-BR/docs/next/configuration) antes de expor o serviço além de sua própria máquina.

## Construir a partir da fonte

Para uma compilação local, use os arquivos Compose do repositório de origem:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
