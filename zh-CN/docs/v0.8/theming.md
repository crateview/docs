---
title: '品牌和主题'
description: '使用您自己的身份，同时保留 Nuxt UI 的原生设计系统。'
---

![品牌和主题](https://crateview.de/images/docs/v0.8/settings-light.webp)

## 默认值

Crateview 使用 Nuxt UI 的 `red` 主调色板和 `zinc` 中性调色板。默认颜色模式为System；用户可以选择浅色或深色，与您的颜色配置无关。

## 运行时主题变量

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

主调色板接受标准 Tailwind 强调色，例如 `blue`、`emerald` 或 `violet`。中性接受 `slate`、`gray`、`zinc`、`neutral` 或 `stone`。 Radius 接受非负 `px` 或 `rem` 值。字体配置选择字体堆栈；如果客户端未安装您的字体，则捆绑字体文件并声明 `@font-face`。

更改生产环境变量后重新启动服务器。这些运行时设置不需要重建。

## 应用程序和组件配置

默认值在 `app/app.config.ts` 中定义：

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

对组件变体和图标使用正常的 Nuxt UI `ui` 配置。 `app/assets/css/main.css` 中包含其他 CSS 设计令牌和明暗表面规则。源更改需要重建生产应用程序。

## 更换品牌

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

在构建之前将本地资源放置在 `public/` 中，或使用 HTTP(S) 资源 URL。现有的可选产品、文档和存储库链接可以通过其相应的 `NUXT_PUBLIC_BRAND_*_URL` 变量进行配置。公共配置在浏览器中可见，并且绝不能包含机密。

### 无需重新构建即可使用服务器文件

也可以配置服务器端私有文件路径。配置的文件优先于对应的公开 URL；文件变量为空时保留 URL 设置。支持绝对路径；相对路径以服务器工作目录为基准（Docker 镜像中为 `/app`）。文件路径不会发送给浏览器。资源本身是公开的，登录前也可访问。

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

支持的扩展名：SVG、PNG、JPG/JPEG、WebP、GIF 和 ICO。文件必须是可读、非空的普通文件；无效配置会阻止服务器启动。文件在启动时加载：更改路径或文件内容后请重启。无需重新构建。

对于 Docker Compose，创建覆盖文件 `compose.branding.yaml`：

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

在主机的 `./branding` 中创建这三个文件，并允许容器的 `node` 用户读取。运行 `docker compose -f compose.yaml -f compose.branding.yaml up -d`。挂载为只读；配置的路径指向容器内的文件。这些私有文件变量适用于注册表 UI。

Crateview 网站重复使用镜像仓库 UI 的标记、主题默认值和品牌变量名称。网站单独维护；它的源和部署配置是私有的。其文档来源在 [crateview/docs](https://github.com/crateview/docs).

## 自定义已安装应用的品牌

已安装应用使用 `NUXT_PUBLIC_BRAND_NAME` 和 `NUXT_PUBLIC_BRAND_DESCRIPTION`。专用 PWA 图标独立于浅色/深色标志和 favicon。192 像素 PNG 同时用作 Apple 触控图标。默认安装图标使用 Crateview 标志。

在主机上创建准确为 192×192 和 512×512 像素的两个 PNG 文件，并允许容器用户读取。以下覆盖配置将文件只读挂载；请与现有 Compose 文件组合使用。若使用 URL 覆盖，请设置 `NUXT_PUBLIC_BRAND_PWA_ICON_192` 和 `NUXT_PUBLIC_BRAND_PWA_ICON_512`。

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

浏览器负责管理已安装应用的元数据，因此图标更改可能延迟显示。重启 Crateview 会应用运行时配置，但不能保证现有安装的图标立即更新。
