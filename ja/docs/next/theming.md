---
title: 'ブランディングとテーマ設定'
description: 'Nuxt UI のネイティブ デザイン システムを維持しながら、独自のアイデンティティを使用します。'
---

![ブランディングとテーマ設定](https://crateview.de/images/docs/next/settings-light.webp)

## デフォルト

Crateview は、Nuxt UI の `red` プライマリ パレットと `zinc` ニュートラル パレットを使用します。デフォルトのカラー モードはシステムです。ユーザーは、色の設定とは関係なく、明るいまたは暗いを選択できます。

## ランタイムテーマ変数

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

プライマリ パレットは、`blue`、`emerald`、`violet` などの標準の Tailwind アクセント カラーを受け入れます。ニュートラルは、`slate`、`gray`、`zinc`、`neutral`、または `stone` を受け入れます。半径は、負でない `px` または `rem` 値を受け入れます。フォント設定ではフォント スタックを選択します。フォント ファイルをバンドルし、フォントがクライアントにインストールされていない場合は `@font-face` を宣言します。

運用環境変数を変更した後、サーバーを再起動します。これらのランタイム設定では再構築は必要ありません。

## アプリケーションとコンポーネントの構成

デフォルトは `app/app.config.ts` で定義されています。

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

コンポーネントのバリアントとアイコンには、通常の Nuxt UI `ui` 構成を使用します。追加の CSS デザイン トークンと明るい/暗いサーフェス ルールは `app/assets/css/main.css` に存在します。ソースを変更するには、実稼働アプリケーションを再構築する必要があります。

## ブランドを置き換える

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

ビルドする前にローカル アセットを `public/` に配置するか、HTTP(S) アセット URL を使用します。既存のオプションの製品、ドキュメント、およびリポジトリのリンクは、対応する `NUXT_PUBLIC_BRAND_*_URL` 変数を通じて設定できます。パブリック設定はブラウザに表示され、シークレットを含めることはできません。

### 再ビルドせずにサーバー上のファイルを使用

サーバー側の非公開ファイルパスも設定できます。設定したファイルは対応する公開 URL より優先されます。ファイル変数が空の場合は URL 設定が維持されます。絶対パスに対応し、相対パスはサーバーの作業ディレクトリ（Docker イメージでは `/app`）を基準に解決されます。ファイルパスはブラウザーに送信されません。アセット自体はログイン前も公開されます。

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

対応する拡張子は SVG、PNG、JPG/JPEG、WebP、GIF、ICO です。読み取り可能で空ではない通常ファイルが必要です。設定が無効な場合、サーバーは起動しません。ファイルは起動時に読み込まれるため、パスや内容を変更した後は再起動してください。再ビルドは不要です。

Docker Compose では、オーバーライドファイル `compose.branding.yaml` を作成します。

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

ホストの `./branding` に 3 つのファイルを作成し、コンテナーの `node` ユーザーに読み取りを許可します。`docker compose -f compose.yaml -f compose.branding.yaml up -d` で起動します。マウントは読み取り専用です。設定するパスはコンテナー内のファイルを指します。これらの非公開ファイル変数はレジストリー UI に適用されます。

Crateview Web サイトは、レジストリ UI のマーク、テーマのデフォルト、およびブランド変数名を再利用します。ウェブサイトは別個に管理されます。そのソースと展開構成は非公開です。そのドキュメントのソースは [crateview/docs](https://github.com/crateview/docs).
