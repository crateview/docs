---
title: "故障排除"
description: "解决连接、身份验证、元数据和部署问题。"
---

## UI 无法到达镜像仓库

从前端服务器的网络环境检查镜像仓库来源。在容器中，`localhost` 指向容器本身。常见的 Compose 上游是 `http://registry:5000`。

```sh
curl -i http://127.0.0.1:5000/v2/
```

匿名镜像仓库应该返回成功。经过身份验证的用户可以通过 `WWW-Authenticate` 质询返回 `401`。连接拒绝意味着侦听器、地址或网络路径未准备好。

## 登录失败

确认镜像仓库使用本机 htpasswd Basic Auth 并且其 bcrypt 文件已正确安装。当前 UI 不支持外部 Bearer 令牌质询。

对于重复的会话丢失，请检查所有前端实例是否共享 `NUXT_SESSION_PASSWORD`。进程本地生成的秘密在重新启动后会发生变化。还验证 HTTPS 和转发的协议标头；浏览器会话需要 cookie 返回同一源。

## 我的镜像没有描述或创建日期

这些值必须存在于清单注释或图像配置中。 Crateview 不会推断丢失的元数据或替换镜像仓库 push 次。构建图像时添加适当的 OCI 注释。

## 推荐人为空

镜像仓库需要公开 OCI 引荐来源网址端点或标准 digest 标签后备索引。 OCI 客户端负责在需要时维护该后备索引。仅将带有主题的清单推送到没有引用者支持的镜像仓库中并不会使 Crateview 扫描它的所有标签。

## 删除缺失或被拒绝

启用 Crateview 的 `NUXT_REGISTRY_DELETE_ENABLED` 和 Distribution 的存储删除设置。注册管理机构仍可以根据其权限或实施情况拒绝该请求。删除通过 digest 进行，可能会影响多个标签。

## 生产环境忽略我的 .env 文件

`node .output/server/index.mjs` 不会自动加载 `.env`。通过流程管理器、容器环境或 shell 导出变量。更改运行时间值后重新启动。纯静态 Web 主机无法替代所需的 Nitro 服务器。

## 复制的pull命令不起作用

该命令使用公共前端地址。确认 CLI 可以到达该源并信任其 TLS 证书。需要时单独验证 CLI。浏览器登录不会创建 Docker CLI 登录。

如果您发现可重现的 UI 问题，请打开 [issue](https://github.com/crateview/registry-ui/issues)，其中包含受影响的版本和已清理的重现详细信息。使用【私人安全通道](https://github.com/crateview/registry-ui/security/policy)查找漏洞。
