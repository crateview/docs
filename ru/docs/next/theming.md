---
title: 'Брендинг и тематика'
description: 'Используйте свою индивидуальность, сохраняя при этом собственную систему дизайна пользовательского интерфейса Nuxt.'
---

![Брендинг и тематика](https://crateview.de/images/docs/next/settings-light.webp)

## По умолчанию

Crateview использует основную палитру `red` пользовательского интерфейса Nuxt и нейтральную палитру `zinc`. Цветовой режим по умолчанию — Системный; пользователи могут выбрать «Светлый» или «Темный» независимо от вашей цветовой конфигурации.

## Переменные темы времени выполнения

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

Основная палитра поддерживает стандартные акцентные цвета Tailwind, такие как `blue`, `emerald` или `violet`. Нейтраль принимает `slate`, `gray`, `zinc`, `neutral` или `stone`. Радиус принимает неотрицательные значения `px` или `rem`. Конфигурация шрифтов выбирает стек шрифтов; сгруппируйте файлы шрифтов и объявите `@font-face`, если ваш шрифт не установлен на клиенте.

Перезапустите сервер после изменения переменных производственной среды. Для этих параметров времени выполнения пересборка не требуется.

## Конфигурация приложения и компонентов

Значения по умолчанию определены в `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Используйте обычную конфигурацию Nuxt пользовательского интерфейса `ui` для вариантов компонентов и значков. Дополнительные токены дизайна CSS и правила светлой/темной поверхности находятся в `app/assets/css/main.css`. Изменения исходного кода требуют перестройки рабочего приложения.

## Заменить бренд

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Перед сборкой поместите локальные ресурсы в `public/` или используйте URL-адреса ресурсов HTTP(S). Существующие дополнительные ссылки на продукт, документацию и репозиторий можно настроить с помощью соответствующих переменных `NUXT_PUBLIC_BRAND_*_URL`. Публичная конфигурация видна в браузере и никогда не должна содержать секретов.

### Файлы сервера без пересборки

Можно также настроить приватные пути к файлам на сервере. Каждый указанный файл имеет приоритет над соответствующим публичным URL; пустая файловая переменная сохраняет настройку URL. Поддерживаются абсолютные пути; относительные отсчитываются от рабочего каталога сервера (`/app` в образе Docker). Пути никогда не отправляются в браузер. Сами ресурсы общедоступны, в том числе до входа.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Поддерживаемые расширения: SVG, PNG, JPG/JPEG, WebP, GIF и ICO. Это должны быть читаемые непустые обычные файлы; неверная конфигурация препятствует запуску сервера. Файлы загружаются при запуске: после изменения путей или содержимого перезапустите сервер. Пересборка не нужна.

Для Docker Compose создайте файл переопределения `compose.branding.yaml`:

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

Создайте три файла в `./branding` на хосте и разрешите пользователю контейнера `node` читать их. Запустите `docker compose -f compose.yaml -f compose.branding.yaml up -d`. Каталог подключается только для чтения; настроенные пути указывают на файлы внутри контейнера. Эти приватные файловые переменные относятся к интерфейсу реестра.

Веб-сайт Crateview повторно использует метку пользовательского интерфейса реестра, настройки темы по умолчанию и имена переменных фирменного оформления. Веб-сайт поддерживается отдельно; его источник и конфигурация развертывания являются частными. Источники его документации общедоступны в [crateview/docs](https://github.com/crateview/docs).

## Оформление установленного приложения

Приложение использует `NUXT_PUBLIC_BRAND_NAME` и `NUXT_PUBLIC_BRAND_DESCRIPTION`. Значки PWA не зависят от светлого/тёмного логотипов и favicon. PNG размером 192 пикселя также служит значком Apple. По умолчанию используется знак Crateview.

Создайте на хосте PNG размером ровно 192×192 и 512×512 пикселей и разрешите пользователю контейнера читать их. Следующий дополнительный Compose-файл монтирует их только для чтения; объедините его с существующим. Для URL используйте `NUXT_PUBLIC_BRAND_PWA_ICON_192` и `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Браузеры управляют метаданными установленных приложений, поэтому значки могут обновиться с задержкой. Перезапуск Crateview применяет настройки, но не гарантирует немедленное обновление уже установленного значка.
