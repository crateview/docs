---
title: "Instalasi"
description: "Mulai tumpukan lengkap atau sambungkan UI ke registri yang ada."
---

Gunakan revisi versi ini; jangan klon cabang default untuk instalasi arsip.

## Persyaratan

Gunakan Node.js 26.13.0 atau yang lebih baru dalam v26 untuk pengembangan lokal, atau Docker dengan Compose untuk tumpukan lengkap. Registri yang ada harus mengekspos Registri HTTP API V2; tumpukan yang dibundel menggunakan gambar resmi `registry:3.1.2`.

## Contoh Docker Compose

Pilih **salah satu** dari varian berikut dan simpan sebagai `compose.yaml` di direktorinya sendiri. Tidak diperlukan klon repositori. Buat file `.env` di sampingnya dengan nilai yang ditampilkan untuk varian tersebut. Compose secara eksplisit meneruskan nilai-nilai ini ke container.

Hasilkan nilai acak minimal 32 karakter untuk `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

Jaga agar kunci ini tetap stabil dan lindungi `.env`; jangan komit ke kontrol versi. Gambar Crateview yang diterbitkan mendukung Linux amd64.

### UI dengan registri yang ada

Memerlukan titik akhir Registry HTTP API V2 yang dapat dijangkau dari kontainer. Contoh `.env` (ganti alamat; jangan sertakan kredensial di URL):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.5.0
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

Buka **http://localhost:3000**. Hulu `127.0.0.1` mengacu pada kontainer itu sendiri. Gunakan nama DNS yang dapat dijangkau atau alamat layanan di jaringan Docker bersama.

### UI dengan registrinya sendiri

Tumpukan lokal tanpa otentikasi. Hanya port UI yang dipublikasikan pada loopback; registri tetap berada di jaringan Docker. `.env`:

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
    image: ghcr.io/crateview/registry-ui:v0.5.0
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

Buka **http://localhost:3000**. Data registri tetap ada di volume `registry-data`. Push dan pull gambar melalui asal yang sama:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, registri dan Traefik dengan HTTPS

Tumpukan mandiri termasuk Traefik dan Let's Encrypt. Memerlukan server yang dapat dijangkau secara publik, port TCP 80/443 yang tersedia dan diizinkan, serta domain yang data DNS-nya mengarah ke server tersebut. Port 80 harus tetap dapat dijangkau untuk tantangan HTTP-01. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Masukkan domain tanpa skema atau jalur. Ganti semua nilai contoh. Kemudian secara interaktif buat file bcrypt `htpasswd`; ganti `your-user` dengan nama pengguna Anda:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

Kata sandi diminta dan tidak muncul dalam perintah. `-c` membuat file baru; hilangkan saat menambahkan pengguna. Lindungi dan buat cadangan direktori `auth`. Simpan `compose.yaml` ini di sampingnya:

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
    image: ghcr.io/crateview/registry-ui:v0.5.0
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

Buka **https://registry.example.com** menggunakan domain Anda dan masuk dengan pengguna registri yang Anda buat. Traefik memperoleh sertifikat secara otomatis dan mengalihkan HTTP ke HTTPS. Sertifikat tetap ada di `letsencrypt`, data registri di `registry-data`. Cadangkan kedua volume dan jangan gunakan `docker compose down -v` untuk pematian rutin.

Semua jalur, termasuk `/v2/`, melalui Crateview. Baik registri maupun UI tidak memublikasikan port hostnya sendiri. Registri menerapkan Basic Auth; tidak diperlukan Traefik Basic Auth tambahan. Traefik mempertahankan host publik dan meneruskan protokol HTTPS ke Crateview. Soket Docker memberikan Traefik akses ke Docker API; jalankan tumpukan ini pada host tepercaya.

Login CLI dan transfer gambar (ganti domain):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

CLI meminta kata sandi secara terpisah dari login UI. Lihat [autentikasi](/id/docs/v0.5/authentication) dan [Traefik dengan Docker dan Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) untuk detailnya.

## Gunakan registri yang ada

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.5.0
npm ci
cp .env.example .env
```

Atur upstream di `.env`, lalu mulai mode pengembangan:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

UI berjalan pada port 3000 dengan penggantian modul panas. Ketika frontend berjalan di dalam sebuah container, `127.0.0.1` berarti container tersebut. Gunakan alamat layanan yang dapat dijangkau, seperti `http://registry:5000`.

## Server Node Produksi

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

Proses Node produksi membaca variabel lingkungan aktual, bukan `.env` secara otomatis. Konfigurasikan manajer proses atau lingkungan kontainer Anda dengan tepat. Sajikan HTTPS untuk penggunaan autentikasi jarak jauh.

Crateview memerlukan server Nitro untuk otentikasi dan proksi API. Hanya menyajikan file frontend sebagai situs web statis tidak akan menyediakan UI registri yang berfungsi.

Lanjutkan dengan [configuration](/id/docs/v0.5/configuration) sebelum menampilkan layanan di luar mesin Anda sendiri.

## Bangun dari sumber

Untuk build lokal, gunakan Compose file dari repositori sumber:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.5.0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
