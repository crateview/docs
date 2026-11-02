---
title: "品牌和主题"
description: "使用您自己的身份，同时保留 Nuxt UI 的原生设计系统。"
---

![品牌和主题](https://crateview.de/images/docs/v0.4/settings-light.webp)

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

Crateview 网站重复使用镜像仓库 UI 的标记、主题默认值和品牌变量名称。网站单独维护；它的源和部署配置是私有的。其文档来源在 [crateview/docs](https://github.com/crateview/docs).
