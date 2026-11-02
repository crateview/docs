---
title: "操作与删除"
description: "仔细管理清单并让镜像仓库存储处于您的控制之下。"
---

## 启用清单删除

默认情况下，Crateview UI 中禁用删除。前端和 Distribution 都必须允许：

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution 也可以读取 `REGISTRY_STORAGE_DELETE_ENABLED=true`。更改环境或配置后重新启动相关服务。注册管理机构仍然负责授权，包括直接通过 `/v2/**` 提出的请求。

## 了解digest删除

标准删除操作的目标是**清单 digest**。每个指向 digest 的标签都可以删除。 Crateview 在确认之前预览受影响的标签，但并发编写器可以在预览后更改引用：镜像仓库 API 不提供锁定别名的事务。

将删除视为对清单及其所有别名的更改，而不是删除一个标签。在确认之前验证您的备份和预期范围。


在仓库表格中选择一个或多个标签并确认删除。预览会刷新受影响的标签，相同摘要仅删除一次，并报告部分失败。删除摘要会影响所有别名。垃圾回收仍是独立的注册表管理任务。

## 垃圾收集和备份

删除清单并不保证立即回收磁盘。垃圾收集是一项单独的镜像仓库管理任务； Crateview 不运行它。

使用适合存储驱动程序的方法备份镜像仓库存储及其配置。更换容器时保留卷。请遵循 Distribution 的垃圾收集和写入协调说明：

- [Distribution 垃圾收集](https://distribution.github.io/distribution/about/garbage-collection/)
- 【Distribution配置](https://distribution.github.io/distribution/about/configuration/)

## 更新Crateview

<!-- release:operations:start -->
预览容器需先从固定源码修订本地构建，不假定存在已发布预览镜像。

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
<!-- release:operations:end -->

## 大型镜像仓库

存储库搜索读取完整的分页目录。可见存储库的元数据是按需获取的。按图像创建时间对大型标签列表进行排序可能需要阅读许多清单和配置。 Crateview没有后台元数据索引或外部搜索服务。
