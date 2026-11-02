---
title: 'Pencitraan merek & tema'
description: 'Gunakan identitas Anda sendiri sambil tetap mempertahankan sistem desain asli Nuxt UI.'
---

![Pencitraan merek & tema](https://crateview.de/images/docs/next/settings-light.webp)

## Default

Crateview menggunakan palet utama `red` Nuxt UI dan palet netral `zinc`. Mode warna default adalah Sistem; pengguna dapat memilih Terang atau Gelap terlepas dari konfigurasi warna Anda.

## Variabel tema runtime

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

Palet utama menerima warna aksen Tailwind standar seperti `blue`, `emerald`, atau `violet`. Netral menerima `slate`, `gray`, `zinc`, `neutral` atau `stone`. Radius menerima nilai `px` atau `rem` non-negatif. Konfigurasi font memilih tumpukan font; bundel file font dan nyatakan `@font-face` jika font Anda tidak diinstal pada klien.

Mulai ulang server setelah mengubah variabel lingkungan produksi. Tidak diperlukan pembangunan kembali untuk pengaturan runtime ini.

## Konfigurasi aplikasi dan komponen

Defaultnya ditentukan di `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Gunakan konfigurasi Nuxt UI `ui` normal untuk varian komponen dan ikon. Token desain CSS tambahan dan aturan permukaan terang/gelap ada di `app/assets/css/main.css`. Perubahan sumber memerlukan pembangunan kembali aplikasi produksi.

## Ganti mereknya

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Tempatkan aset lokal di `public/` sebelum membangun, atau gunakan URL aset HTTP(S). Tautan produk, dokumentasi, dan repositori opsional yang ada dapat dikonfigurasi melalui variabel `NUXT_PUBLIC_BRAND_*_URL` yang sesuai. Konfigurasi publik terlihat di browser dan tidak boleh mengandung rahasia.

### Berkas server tanpa membangun ulang

Anda juga dapat mengatur jalur berkas privat di server. Setiap berkas yang dikonfigurasi diutamakan dibanding URL publik terkait; variabel berkas kosong mempertahankan pengaturan URL. Jalur absolut didukung; jalur relatif dihitung dari direktori kerja server (`/app` pada image Docker). Jalur berkas tidak pernah dikirim ke browser. Asetnya bersifat publik, termasuk sebelum masuk.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Ekstensi yang didukung: SVG, PNG, JPG/JPEG, WebP, GIF, dan ICO. Berkas harus berupa berkas biasa yang dapat dibaca dan tidak kosong; konfigurasi tidak valid menggagalkan startup server. Berkas dimuat saat startup: mulai ulang setelah mengubah jalur atau isinya. Tidak perlu membangun ulang.

Untuk Docker Compose, buat berkas override `compose.branding.yaml`:

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_LOGO_LIGHT_FILE: /app/branding/logo-light.svg
      NUXT_BRAND_LOGO_DARK_FILE: /app/branding/logo-dark.svg
      NUXT_BRAND_FAVICON_FILE: /app/branding/favicon.ico
```

Buat ketiga berkas di `./branding` pada host dan izinkan pengguna `node` dalam container membacanya. Jalankan `docker compose -f compose.yaml -f compose.branding.yaml up -d`. Mount bersifat hanya baca; jalur yang dikonfigurasi merujuk pada berkas di dalam container. Variabel berkas privat ini berlaku untuk UI registry.

Situs web Crateview menggunakan kembali tanda UI registri, default tema, dan nama variabel merek. Situs web dikelola secara terpisah; konfigurasi sumber dan penerapannya bersifat pribadi. Sumber dokumentasinya bersifat publik di [crateview/docs](https://github.com/crateview/docs).

## Identitas aplikasi terpasang

Aplikasi memakai `NUXT_PUBLIC_BRAND_NAME` dan `NUXT_PUBLIC_BRAND_DESCRIPTION`. Ikon PWA terpisah dari logo terang/gelap dan favicon. PNG 192 piksel juga menjadi ikon sentuh Apple. Ikon bawaan memakai tanda Crateview.

Buat kedua PNG berukuran tepat 192×192 dan 512×512 piksel pada host dan izinkan pengguna kontainer membacanya. Overlay berikut memasangnya hanya-baca; gabungkan dengan Compose yang ada. Untuk URL, gunakan `NUXT_PUBLIC_BRAND_PWA_ICON_192` dan `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Perubahan ikon mungkin terlambat muncul karena browser mengelola metadata aplikasi terpasang. Memulai ulang Crateview menerapkan konfigurasi, tetapi tidak menjamin ikon yang sudah terpasang langsung diperbarui.
