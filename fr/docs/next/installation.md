---
title: "Installation"
description: "Démarrez la pile complète ou connectez l'interface utilisateur à un registre existant."
---

Utilisez la révision de cette version, pas la branche par défaut pour une installation archivée.

<!-- release:installation:start -->
Construisez les conteneurs d'aperçu depuis la révision fixée ; aucune image d'aperçu publiée n'est supposée.

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker build -t crateview-registry-ui:next .
```
<!-- release:installation:end -->


## Exigences

Utilisez Node.js 26.13.0 ou plus récent dans la v26 pour le développement local, ou Docker avec Compose pour la pile complète. Un registre existant doit exposer le registre HTTP API V2 ; la pile fournie utilise l'image officielle `registry:3.1.2`.

## Docker Composer des exemples

Choisissez **une** des variantes suivantes et enregistrez-la sous le nom `compose.yaml` dans son propre répertoire. Aucun clone de référentiel n'est requis. Créez un fichier `.env` à côté avec les valeurs affichées pour cette variante. Compose transmet explicitement ces valeurs au conteneur.

Générez une valeur aléatoire d'au moins 32 caractères pour `NUXT_SESSION_PASSWORD` :

```sh
openssl rand -base64 32
```

Gardez cette clé stable et protégez `.env` ; ne le confiez pas au contrôle de version. Les images Crateview publiées prennent en charge Linux amd64.

### Interface utilisateur avec un registre existant

Nécessite un point de terminaison HTTP API V2 du registre accessible depuis le conteneur. Exemple `.env` (remplacez l'adresse ; n'incluez pas les informations d'identification dans l'URL) :

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

Open **http://localhost:3000**. Un amont de `127.0.0.1` fait référence au conteneur lui-même. Utilisez un nom DNS accessible ou une adresse de service sur un réseau Docker partagé.

### UI avec son propre registre

Pile locale sans authentification. Seul le port UI est publié en bouclage ; le registre reste sur le réseau Docker. `.env` :

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

Les données du registre ouvert **http://localhost:3000**. persistent dans le volume `registry-data`. Push et pull une image passant par la même origine :

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, registre et Traefik avec HTTPS

Pile autonome comprenant Traefik et Let's Encrypt. Nécessite un serveur accessible au public, des ports TCP 80/443 disponibles et autorisés et un domaine dont les enregistrements DNS pointent vers ce serveur. Le port 80 doit rester accessible pour le défi HTTP-01. `.env` :

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Entrez le domaine sans schéma ni chemin. Remplacez tous les exemples de valeurs. Créez ensuite de manière interactive un fichier bcrypt `htpasswd` ; remplacez `your-user` par votre nom d'utilisateur :

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

Le mot de passe est demandé et n'apparaît pas dans la commande. `-c` crée un nouveau fichier ; omettez-le lors de l’ajout d’utilisateurs. Protégez et sauvegardez le répertoire `auth`. Enregistrez ce `compose.yaml` à côté :

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

Ouvrez **https://registry.example.com** à l'aide de votre domaine et connectez-vous avec l'utilisateur de registre que vous avez créé. Traefik obtient le certificat automatiquement et redirige HTTP vers HTTPS. Les certificats persistent dans `letsencrypt`, les données de registre dans `registry-data`. Sauvegardez les deux volumes et n'utilisez pas `docker compose down -v` pour les arrêts de routine.

Tous les chemins, y compris `/v2/`, passent par Crateview. Ni le registre ni l'interface utilisateur ne publient leurs propres ports hôtes. Le registre applique Basic Auth ; aucun Traefik Basic Auth supplémentaire n'est requis. Traefik préserve l'hôte public et transmet le protocole HTTPS à Crateview. Le socket Docker donne accès au Traefik au Docker API ; exécutez cette pile sur un hôte de confiance.

Connexion CLI et transfert d'image (remplacer le domaine) :

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

La CLI demande le mot de passe séparément de la connexion à l'interface utilisateur. Voir [authentification](/fr/docs/next/authentication) et [Traefik avec Docker et Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) pour plus de détails.

## Utiliser un registre existant

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
npm ci
cp .env.example .env
```

Définissez l'amont dans `.env`, puis démarrez le mode développement :

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

L'interface utilisateur fonctionne sur le port 3000 avec remplacement du module à chaud. Lorsque le frontend s'exécute dans un conteneur, `127.0.0.1` signifie ce conteneur. Utilisez plutôt une adresse de service accessible, telle que `http://registry:5000`.

## Serveur de nœud de production

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

Le processus du nœud de production lit automatiquement les variables d'environnement réelles, et non `.env`. Configurez votre gestionnaire de processus ou votre environnement de conteneur en conséquence. Servez HTTPS pour une utilisation authentifiée à distance.

Crateview a besoin de son serveur Nitro pour l'authentification et le proxy API. Servir uniquement les fichiers frontaux en tant que site Web statique ne fournira pas d'interface utilisateur de registre fonctionnelle.

Continuez avec [configuration](/fr/docs/next/configuration) avant d'exposer le service au-delà de votre propre machine.

## Construire à partir des sources

Pour une build locale, utilisez les fichiers Compose du référentiel source :

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```

<a id="pwa"></a>

## Installer comme application (PWA)

Installez Crateview avec l’action native d’installation ou d’ajout à l’écran d’accueil d’un navigateur compatible. L’application s’ouvre en mode autonome. Elle nécessite toujours son serveur Nitro et un registre accessible.

Utilisez HTTPS en production ; localhost convient aux essais locaux. Le service worker est actif uniquement dans les builds de production, pas pendant `npm run dev`. Compilez et démarrez le serveur de production avant de tester l’installation ou le mode hors ligne.

Après une visite en ligne réussie et l’installation du service worker, un démarrage ou rechargement sans réseau affiche un message de connexion. Réessayer recharge l’URL d’origine lorsque la connexion revient. La langue enregistrée prime sur les langues du navigateur, avec l’anglais par défaut ; l’arabe et l’ourdou utilisent RTL.

Le message hors ligne ne concerne que les navigations de documents de l’application. Les réponses du registre, des comptes et des sessions ne sont pas enregistrées dans Cache Storage ; la consultation hors ligne du registre est indisponible. Les écrans ouverts conservent leurs erreurs habituelles. Le trafic API et `/v2/` n’est pas remplacé par du HTML hors ligne ; les mises à jour du worker ne rechargent pas les onglets de force.
