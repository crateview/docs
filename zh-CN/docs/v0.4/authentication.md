---
title: "验证"
description: "匿名访问、Basic Auth、加密会话和 HTTPS。"
---

## 支持的身份验证

支持匿名访问和原生 htpasswd Basic Auth。不支持外部 Bearer 令牌服务及 OIDC。此版本没有用户管理。

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
