查看发行说明，将 `CRATEVIEW_VERSION` 设置为 `.env` 中所需的 `v` 前缀发行标签，然后替换前端容器：

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

例如，初始版本使用 `CRATEVIEW_VERSION={{RELEASE_TAG}}`。要回滚，请恢复以前的标签并重复这些命令。保留运行环境和`registry-data`体积；不要运行 `docker compose down --volumes`。源版本仍然可以通过 `compose.build.yaml` 获得。

Dependabot 在源存储库中配置以建议依赖项更新；它不会自动升级您的自托管部署。升级后重新测试镜像仓库连接、登录和受影响的操作。
