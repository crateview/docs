---
title: "概述"
description: "Crateview 的用途、它如何连接到您的镜像仓库以及它的边界在哪里。"
---

## 更清晰地查看您的镜像仓库

Crateview 是 Docker 和 OCI 容器镜像仓库的 Web 界面。它可以帮助您查找存储库和标签、检查多平台映像、读取清单和配置文档以及了解压缩层及其历史记录。

它与您的镜像仓库一起运行。镜像仓库存储图像并强制执行访问控制； Crateview提供接口和同源代理。 Docker 和其他 OCI 客户端可以使用前端源作为其镜像仓库地址。

![Crateview 存储库概述，包含 Alpine 和嵌套演示存储库。](https://crateview.de/images/docs/v0.1/repositories-light.webp)

## 建筑学

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

前端读取标准镜像仓库 HTTP API V2。 UI 元数据源自目录、标签列表、清单和图像配置 blob。它不需要单独的搜索索引或应用程序数据库。

## 支持的功能

- 存储库名称搜索、标签建议和嵌套命名空间。
- 标签表、平台选择、压缩层大小和图像历史记录。
- OCI 说明、源 URL、文档和许可证的注释。
- 带有剪贴板支持的原始清单和配置 JSON。
- OCI 引荐来源网址，包括 OCI 客户端维护的标准 digest 标签索引回退。
- 匿名访问和本机 htpasswd Basic Auth。
- 可选的清单删除，并在确认之前显示受影响的标签。
- 两种界面语言（英语和德语）、系统/浅色/深色及可配置品牌。


## 知道界限

标准API不提供push时间戳、下载统计、收藏夹或漏洞扫描。 Crateview 不会伪造这些值。图像创建日期来自图像元数据，而不是其上传时间。显示的大小描述的是压缩图像层，而不是镜像仓库的磁盘使用情况。

外部 Bearer 令牌服务、OIDC、用户管理和 API 密钥管理未集成。签名工件可能会被列为引用者；这不是加密签名验证。

从[安装](/zh-CN/docs/v0.1/installation)]开始，然后查看[配置](/zh-CN/docs/v0.1/configuration)和[身份验证](/zh-CN/docs/v0.1/authentication)。
