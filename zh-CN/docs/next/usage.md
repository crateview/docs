---
title: "使用Crateview"
description: "浏览图像、切换平台并检查标签背后的数据。"
---

## 查找存储库和标签

探索页面列出镜像仓库。搜索支持仓库名称和 repository:tag。打开仓库可查看标签。

镜像仓库默认以表格显示，可切换网格。带斜线的名称构成命名空间。侧栏可调整宽度、折叠并在本地保存状态，搜索位于侧栏。

/ 或 Ctrl+K（macOS 为 Cmd+K）打开命令面板，搜索镜像仓库、标签或导航。命名空间文件夹可折叠。设置包含语言、系统/浅色/深色，以及可搜索依赖许可证和原始声明的关于页面。

管理区域包含设置和用户。账户菜单也提供语言和主题。账户页面要求当前密码才能修改自己的密码。只有服务器允许名单授权的用户才能列出和创建账户；令牌权限只读。新密码至少12个字符且最多72个UTF-8字节。新令牌账户最初无仓库访问权。

![品牌和主题](https://crateview.de/images/docs/next/settings-light.webp)

## 检查图像

打开存储库并选择一个标签。图像视图显示了其 digest、可用平台、压缩层大小和创建元数据。

对于多平台映像，请在平台选择器中选择操作系统和体系结构。图层历史记录和选定的图像配置遵循该选择。原始清单选项卡显示图像的根清单或索引；配置显示所选平台的配置。

![Alpine 图像详细信息，包括平台选择、图层历史记录和 pull 命令。](https://crateview.de/images/docs/next/image-light.webp)

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
