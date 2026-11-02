---
title: "使用Crateview"
description: "浏览图像、切换平台并检查标签背后的数据。"
---

## 查找存储库和标签

探索页面列出镜像仓库。搜索支持仓库名称和 repository:tag。打开仓库可查看标签。

概览列出镜像仓库，探索页面过滤名称。/ 或 Ctrl+K 聚焦导航栏搜索。主题和语言位于侧栏底部。

## 检查图像

打开存储库并选择一个标签。图像视图显示了其 digest、可用平台、压缩层大小和创建元数据。

对于多平台映像，请在平台选择器中选择操作系统和体系结构。图层历史记录和选定的图像配置遵循该选择。原始清单选项卡显示图像的根清单或索引；配置显示所选平台的配置。

![Alpine 图像详细信息，包括平台选择、图层历史记录和 pull 命令。](https://crateview.de/images/docs/v0.1/image-light.webp)

## Pull 命令

命令面板提供用于图像的 Docker、Podman 和 Skopeo 命令，以及用于工件的 ORAS/Podman 工件命令。复制该命令并在您自己的终端中运行它。

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

将示例主机替换为您的公共 Crateview 源。 CLI 身份验证与您的浏览器会话是分开的。

## OCI 元数据和引荐来源网址

描述、许可证和源链接是从 OCI 注释或图像配置标签（如果可用）中读取的。丢失的元数据保持为空而不是被猜测。

“引用者”选项卡读取 OCI 引用者 API。如果镜像仓库未实现该端点，则 Crateview 读取由 OCI 客户端维护的标准 digest 派生标签索引。它不会通过扫描每个清单来构建自己的索引。列出的签名工件不会自动验证或标记为可信。

## 解释时间戳和大小

图像创建时间由图像配置或注释提供。它不会告诉您标签何时被推送到此镜像仓库。压缩层大小不测量磁盘消耗，因为层可以共享，并且删除的数据可能会保留到垃圾收集为止。
