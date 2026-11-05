---
title: "配置"
description: "镜像仓库连接、运行时环境变量和部署行为。"
---

## 配置层

应用程序默认值在 `nuxt.config.ts` 和 `app/app.config.ts` 中定义。服务器启动时，环境变量会覆盖运行时默认值。开发和构建命令读取 `.env`；生产节点入口点不会自动读取它。

更改运行时环境变量后重新启动服务器。编辑源级组件配置需要新的生产版本。

## 日期和时间格式

在运行时设置 `NUXT_PUBLIC_DATE_FORMAT` 可统一界面日期格式。时区取自浏览器。变量为空或无效时，Crateview 使用浏览器的区域设置。

示例: `DD.MM.YYYY HH:mm`. 标记: `YYYY`, `MM`, `DD`, `HH`, `mm`, `ss`, `Z` (UTC ±HH:mm). 原样显示的文本请放在方括号内，例如 `[at]`。

## 镜像仓库和会话设置

| 变量 | 默认 | 目的 |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM` | `http://127.0.0.1:5000` | 服务器端镜像仓库起源。 |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false` | 在 UI 中公开清单删除。 |
| `NUXT_SESSION_PASSWORD` | 每个进程随机 | 用于加密 UI 会话的密钥。 |
| `HOST` | 依赖于运行时 | 节点服务器监听接口。 |
| `PORT` | `3000` | 节点服务器监听端口。 |

上游必须是可从前端服务器访问的 HTTP(S) 源。将凭据保留在 URL 之外。请求使用当前用户的身份验证； Crateview 不会取代镜像仓库访问控制。

## 公共品牌变量

以下所有名称均使用 `NUXT_PUBLIC_BRAND_` 前缀：

| 后缀 | 目的 |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME` | 应用程序名称。 |
| `DESCRIPTION` | 应用描述。 |
| `LOGO_LIGHT`、`LOGO_DARK` | 徽标路径或 HTTP(S) URL。 |
| `FAVICON` | 浏览器图标路径或 URL。 |
| `TITLE_TEMPLATE` | 页面标题； `%s` 被页面标题替换。 |
| `PRODUCT_URL`、`DOCS_URL`、`REPOSITORY_URL` | 可选的导航链接。 |

公共配置发送到浏览器。切勿将密码或令牌存储在 `NUXT_PUBLIC_*` 变量中。有关颜色和排版选项，请参阅[品牌和主题](/zh-CN/docs/v0.8/theming)。

## 网络布局

浏览器与 Crateview 在一个源上进行通信。 Nitro 为镜像仓库客户端代理 `/v2/**`，并通过 `/api/registry/**` 读取镜像仓库元数据。这避免了需要跨源浏览器访问镜像仓库本身。

在反向代理后面，保留公共来源和协议标头。公共前端地址也是复制的 pull 命令中显示的镜像仓库地址。对于远程身份验证使用，请在接受浏览器或 CLI 凭据之前正确终止 HTTPS。

## Compose 与独立配置

捆绑的 Compose 文件有意使用 `NUXT_REGISTRY_UPSTREAM=http://registry:5000` 连接到其自己的 `registry` 服务。要在容器部署中使用另一个镜像仓库，请调整该服务环境或单独运行前端。不要假设可以通过容器网关访问主机环回镜像仓库。
## 此版本的配置

监控可选，默认关闭。启用后概览显示仪表盘，否则 /home 重定向至 /explore。启用会向所有有界面访问权的用户开放整个注册表的运行数据，公共注册表也包括匿名访问者。托管令牌模式还要求全局读取权限。

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[监控](/zh-CN/docs/v0.8/monitoring)

| ENV | Default |
| --- | --- |
| `NUXT_AUTH_MODE` | `off` |
| `NUXT_AUTH_SERVICE_URL` | — |
| `NUXT_AUTH_SERVICE_SECRET` | — |
| `NUXT_AUTH_TOKEN_REALM` | — |
| `NUXT_USER_CREATION_ALLOWED_USERS` | — |

管理区域包含设置和用户。账户菜单也提供语言和主题。账户页面要求当前密码才能修改自己的密码。只有服务器允许名单授权的用户才能列出和创建账户；令牌权限只读。新密码至少12个字符且最多72个UTF-8字节。新令牌账户最初无仓库访问权。

服务器允许列表控制账户查看、创建、密码重置和删除。 右侧三点菜单允许管理员在不提供旧密码的情况下重置其他用户的密码，或在确认后删除其账户。重置密码保留仓库权限；删除账户会移除权限。自己的菜单会打开“账户”，需要当前密码才能更改密码；不能删除自己的账户。

## PWA 图标配置

| 变量 | 默认值 | 用途 |
| --- | --- | --- |
| `NUXT_PUBLIC_BRAND_PWA_ICON_192` | `/branding/pwa-192.png` | 192 像素安装图标 URL。 |
| `NUXT_PUBLIC_BRAND_PWA_ICON_512` | `/branding/pwa-512.png` | 512 像素安装图标 URL。 |
| `NUXT_BRAND_PWA_ICON_192_FILE` | — | 私有 192 像素 PNG 文件覆盖。 |
| `NUXT_BRAND_PWA_ICON_512_FILE` | — | 私有 512 像素 PNG 文件覆盖。 |

本地图标文件优先于相应的公开 URL。文件必须是可读、非空且具有准确正方形尺寸的 PNG。无效文件会阻止服务器启动。相对路径以服务器工作目录为基准；容器路径必须指向容器内可读的文件。修改配置后需要重启。

本地图标 URL 和清单遵循 `NUXT_APP_BASE_URL`，例如 `/registry/`；外部 HTTP(S) 图标 URL 保持不变。私有文件变量不能用于存储秘密：图片内容是公开的，只有文件系统路径保留在服务端。
