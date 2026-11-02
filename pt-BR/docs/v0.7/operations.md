---
title: "Operações e exclusão"
description: "Gerencie os manifestos com cuidado e mantenha o armazenamento do registro sob seu controle."
---

## Habilitar exclusão de manifesto

A exclusão está desabilitada na UI do Crateview por padrão. Tanto o frontend quanto o Distribution devem permitir:

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution também pode ler `REGISTRY_STORAGE_DELETE_ENABLED=true`. Reinicie os serviços relevantes após alterar seu ambiente ou configuração. O registro continua responsável pela autorização, inclusive pelas solicitações feitas diretamente por meio de `/v2/**`.

## Entenda a exclusão de digest

A operação de exclusão padrão tem como alvo um **manifesto digest**. Cada tag que aponta para esse digest pode ser removida. Crateview visualiza as tags afetadas antes da confirmação, mas um gravador simultâneo pode alterar as referências após a visualização: o registro API não fornece uma transação que bloqueie aliases.

Trate uma exclusão como uma alteração no manifesto e em todos os seus aliases, não como a remoção de um rótulo. Verifique seus backups e o escopo pretendido antes de confirmar.


Selecione uma ou mais tags na tabela e confirme a exclusão. A prévia atualiza tags afetadas, elimina cada digest uma vez e informa falhas parciais. Todos os aliases são afetados. A coleta de lixo permanece uma tarefa separada.

## Coleta de lixo e backups

A exclusão de um manifesto não promete recuperação imediata do disco. A coleta de lixo é uma tarefa separada de administração de registro; Crateview não o executa.

Faça backup do armazenamento do registro e de sua configuração usando um método apropriado ao driver de armazenamento. Mantenha os volumes ao substituir os recipientes. Siga as instruções da Distribution para coleta de lixo e coordenação de gravação:

- [Coleta de lixo Distribution](https://distribution.github.io/distribution/about/garbage-collection/)
- [Configuração Distribution](https://distribution.github.io/distribution/about/configuration/)

## Atualizar Crateview

Revise as notas de versão, defina `CRATEVIEW_VERSION` para a tag de versão com prefixo `v` desejada em `.env` e substitua o contêiner de front-end:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Por exemplo, a versão inicial usa `CRATEVIEW_VERSION=v0.7.0`. Para reverter, restaure a tag anterior e repita esses comandos. Mantenha o ambiente de tempo de execução e o volume `registry-data`; não execute `docker compose down --volumes`. As compilações de origem permanecem disponíveis por meio de `compose.build.yaml`.

O Dependabot é configurado no repositório de origem para propor atualizações de dependências; ele não atualiza sua implantação auto-hospedada automaticamente. Teste novamente a conectividade do registro, o login e as ações afetadas após uma atualização.

## Grandes registros

A pesquisa de repositório lê o catálogo paginado completo. Os metadados para repositórios visíveis são obtidos sob demanda. Classificar uma grande lista de tags por hora de criação da imagem pode exigir a leitura de muitos manifestos e configurações. Crateview não possui índice de metadados em segundo plano ou serviço de pesquisa externo.
