---
title: "验证"
description: "匿名访问、Basic Auth、加密会话和 HTTPS。"
---

## 支持的身份验证

Crateview 支持令牌模式下的匿名镜像仓库、本机 htpasswd Basic Auth 和可选的 Crateview auth 服务。当镜像仓库需要基本身份验证时，它会探测 `/v2/` 并显示登录表单。

外部 Bearer 令牌服务和 OIDC 未集成。未配置的外部令牌服务会生成明确的不支持 auth 通知。

## 注册中心端 Basic Auth

使用 bcrypt 密码哈希创建 htpasswd 文件。以下示例使用临时官方 Apache 容器；替换示例凭据并保护生成的文件：

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Distribution 设置示例：

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

将生成的文件挂载到位于 `/auth/htpasswd` 的镜像仓库容器中。镜像仓库配置与 Crateview 配置分开。使用HTTPS进行远程访问； Basic Auth 是编码，而不是传输加密。避免在共享 shell 历史记录或公共示例中留下真实密码。

## 用户界面会话

Nitro 服务器根据镜像仓库验证凭据。它将它们存储在加密的 HttpOnly、SameSite=Strict cookie 中。浏览器 JavaScript 和 localStorage 不会收到凭据。会话在八小时后到期。

设置随机生产会话密钥：

```sh
openssl rand -base64 32
```

使用生成的值 `NUXT_SESSION_PASSWORD`，至少 32 个字符。负载均衡器后面的每个前端实例必须使用相同的值。如果没有配置，Crateview 会生成进程本地机密，因此重新启动会使现有会话失效。

## HTTPS 和注销

当公共请求协议为 HTTPS 时，会话 cookie 使用 Secure。正确配置反向代理的转发协议。注销会清除 UI 会话；它不会删除或修改镜像仓库用户。

Docker 和其他 CLI 客户端使用其正常的镜像仓库凭据直接通过 `/v2/**` 代理进行身份验证。登录 Web UI 不会自动登录 Docker CLI。

## 可选账户管理

独立的auth/管理服务支持帐户创建和自己的密码更改。 Docker 直接连接到镜像仓库，无需 UI 容器即可工作。用户和bcrypt哈希值保留在`htpasswd`中；令牌权限位于 `permissions.json` 中。账户存储不使用SQLite。

- **基本模式：** Distribution 检查共享的htpasswd 文件；所有用户都具有平等的镜像仓库访问权限。 Docker 不需要管理服务来运行。
- **代币模式：** 独立auth服务发行签名五分钟代币； Distribution 强制执行存储库权限。 Docker 需要镜像仓库和 auth 服务。新用户最初没有存储库访问权限。
- **UI:** 服务器允许列表控制账户查看、创建、密码重置和删除。 `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. 空值禁止管理；列出的用户名允许管理，通配符允许所有已认证用户。
- **权限：** CLI 授予目标确切的存储库、命名空间 (`team/*`) 或所有内容 (`*`)；配置文件为 `reader` (pull)、`writer` (pull/push) 和 `manager` (pull/push/删除)。基本模式没有镜像仓库角色。

显式启用 `NUXT_AUTH_MODE=htpasswd` 或 `token` 并配置内部 `NUXT_AUTH_SERVICE_URL` 和仅服务器 `NUXT_AUTH_SERVICE_SECRET`。令牌模式还需要 `NUXT_AUTH_TOKEN_REALM` 与公共令牌端点完全匹配。匿名镜像仓库和外部令牌服务不支持本地帐户管理。仓库权限通过认证 CLI 管理；界面仅以只读方式显示。

请参阅[帐户服务指南](https://github.com/crateview/registry-ui/blob/9954f9c33e34c65bab19378a9b488e9b8ed13b6d/auth/README.md)，了解 Compose 覆盖、设置、迁移和 CLI 命令。将公共 Docker 访问直接路由到镜像仓库，独立于 UI。之前颁发的令牌在密码更改或权限撤销后最多五分钟内仍然有效。

管理区域包含设置和用户。账户菜单也提供语言和主题。账户页面要求当前密码才能修改自己的密码。只有服务器允许名单授权的用户才能列出和创建账户；令牌权限只读。新密码至少12个字符且最多72个UTF-8字节。新令牌账户最初无仓库访问权。

服务器允许列表控制账户查看、创建、密码重置和删除。 右侧三点菜单允许管理员在不提供旧密码的情况下重置其他用户的密码，或在确认后删除其账户。重置密码保留仓库权限；删除账户会移除权限。自己的菜单会打开“账户”，需要当前密码才能更改密码；不能删除自己的账户。

![文档版本](https://crateview.de/images/docs/next/users-light.webp)

![行动](https://crateview.de/images/docs/next/user-actions-light.webp)

![更改密码](https://crateview.de/images/docs/next/user-password-light.webp)

![删除用户](https://crateview.de/images/docs/next/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![文档版本](https://crateview.de/images/docs/next/account-light.webp)

![文档版本](https://crateview.de/images/docs/next/login-light.webp)
