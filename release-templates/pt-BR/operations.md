Revise as notas de versão, defina `CRATEVIEW_VERSION` para a tag de versão com prefixo `v` desejada em `.env` e substitua o contêiner de front-end:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Por exemplo, a versão inicial usa `CRATEVIEW_VERSION={{RELEASE_TAG}}`. Para reverter, restaure a tag anterior e repita esses comandos. Mantenha o ambiente de tempo de execução e o volume `registry-data`; não execute `docker compose down --volumes`. As compilações de origem permanecem disponíveis por meio de `compose.build.yaml`.

O Dependabot é configurado no repositório de origem para propor atualizações de dependências; ele não atualiza sua implantação auto-hospedada automaticamente. Teste novamente a conectividade do registro, o login e as ações afetadas após uma atualização.
