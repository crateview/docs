---
title: "验证"
description: "匿名访问、Basic Auth、加密会话和 HTTPS。"
---

## 支持的身份验证

Crateview 支持令牌模式下的匿名注册表、本机 htpasswd Basic Auth 和可选的 Crateview auth 服务。当注册表需要基本身份验证时，它会探测 `/v2/` 并显示登录表单。

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

将生成的文件挂载到位于 `/auth/htpasswd` 的注册表容器中。注册表配置与 Crateview 配置分开。使用HTTPS进行远程访问； Basic Auth 是编码，而不是传输加密。避免在共享 shell 历史记录或公共示例中留下真实密码。

## 用户界面会话

Nitro 服务器根据注册表验证凭据。它将它们存储在加密的 HttpOnly、SameSite=Strict cookie 中。浏览器 JavaScript 和 localStorage 不会收到凭据。会话在八小时后到期。

设置随机生产会话密钥：

```sh
openssl rand -base64 32
```

使用生成的值 `NUXT_SESSION_PASSWORD`，至少 32 个字符。负载均衡器后面的每个前端实例必须使用相同的值。如果没有配置，Crateview 会生成进程本地机密，因此重新启动会使现有会话失效。

## HTTPS 和注销

当公共请求协议为 HTTPS 时，会话 cookie 使用 Secure。正确配置反向代理的转发协议。注销会清除 UI 会话；它不会删除或修改注册表用户。

Docker 和其他 CLI 客户端使用其正常的注册表凭据直接通过 `/v2/**` 代理进行身份验证。登录 Web UI 不会自动登录 Docker CLI。

## 可选账户管理

独立的auth/管理服务支持帐户创建和自己的密码更改。 Docker 直接连接到注册表，无需 UI 容器即可工作。用户和bcrypt哈希值保留在`htpasswd`中；令牌权限位于 `permissions.json` 中。账户存储不使用SQLite。

- **基本模式：** Distribution 检查共享的htpasswd 文件；所有用户都具有平等的注册表访问权限。 Docker 不需要管理服务来运行。
- **代币模式：** 独立auth服务发行签名五分钟代币； Distribution 强制执行存储库权限。 Docker 需要注册表和 auth 服务。新用户最初没有存储库访问权限。
- **UI：** 用户通过提供当前密码来更改自己的密码。 `NUXT_USER_CREATION_ALLOWED_USERS` 默认为空； `alice,bob` 仅允许这些用户创建帐户，`*` 允许每个经过身份验证的用户。该策略是在服务器端强制执行的。
- **权限：** CLI 授予目标确切的存储库、命名空间 (`team/*`) 或所有内容 (`*`)；配置文件为 `reader` (pull)、`writer` (pull/push) 和 `manager` (pull/push/删除)。基本模式没有注册表角色。

显式启用 `NUXT_AUTH_MODE=htpasswd` 或 `token` 并配置内部 `NUXT_AUTH_SERVICE_URL` 和仅服务器 `NUXT_AUTH_SERVICE_SECRET`。令牌模式还需要 `NUXT_AUTH_TOKEN_REALM` 与公共令牌端点完全匹配。匿名注册表和外部令牌服务不支持本地帐户管理。权限和管理密码重置使用 auth CLI； UI 无法更改其他用户的密码或分配权限。

请参阅[帐户服务指南](https://github.com/crateview/registry-ui/blob/main/auth/README.md)，了解 Compose 覆盖、设置、迁移和 CLI 命令。将公共 Docker 访问直接路由到注册表，独立于 UI。之前颁发的令牌在密码更改或权限撤销后最多五分钟内仍然有效。
