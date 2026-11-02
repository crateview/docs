---
title: "Solução de problemas"
description: "Resolva problemas de conexão, autenticação, metadados e implantação."
---

## A IU não consegue acessar o registro

Verifique a origem do registro no ambiente de rede do servidor front-end. Em um contêiner, `localhost` aponta para o próprio contêiner. Um upstream comum do Compose é `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

Um registro anônimo deve retornar sucesso. Um autenticado pode retornar `401` com um desafio `WWW-Authenticate`. Uma recusa de conexão significa que o ouvinte, o endereço ou o caminho da rede não estão prontos.

## Falha no login

Confirme se o registro usa htpasswd Basic Auth nativo e se seu arquivo bcrypt está montado corretamente. Um desafio de token Bearer externo não é suportado pela UI atual.

Para perdas repetidas de sessão, verifique se todas as instâncias de front-end compartilham `NUXT_SESSION_PASSWORD`. Um segredo gerado localmente pelo processo muda após uma reinicialização. Verifique também HTTPS e cabeçalhos de protocolo encaminhados; as sessões do navegador exigem que o cookie retorne à mesma origem.

## Minha imagem não tem descrição ou data de criação

Esses valores devem existir nas anotações do manifesto ou na configuração da imagem. Crateview não infere metadados ausentes nem substitui tempos de push do registro. Adicione as anotações OCI apropriadas ao criar suas imagens.

## Os referenciadores estão vazios

O registro precisa expor o ponto de extremidade dos referenciadores OCI ou o índice substituto da tag digest-tag padrão. Os clientes OCI são responsáveis ​​por manter esse índice de fallback quando necessário. Simplesmente enviar um manifesto de assunto para um registro sem suporte de referenciadores não faz com que o Crateview verifique todas as tags para ele.

## A exclusão está faltando ou foi rejeitada

Ative a configuração de exclusão de armazenamento de Crateview `NUXT_REGISTRY_DELETE_ENABLED` e Distribution. O registro ainda pode rejeitar a solicitação de acordo com suas permissões ou implementação. A exclusão funciona por digest e pode afetar várias tags.

## A produção ignora meu arquivo .env

`node .output/server/index.mjs` não carrega `.env` automaticamente. Exporte variáveis ​​através do seu gerenciador de processos, ambiente de contêiner ou shell. Reinicie após alterar os valores de tempo de execução. Um host somente estático não pode substituir o servidor Nitro necessário.

## O comando pull copiado não funciona

O comando usa o endereço de front-end público. Confirme se a CLI pode alcançar essa origem e confia em seu certificado TLS. Autentique a CLI separadamente quando necessário. Um login no navegador não cria um login CLI Docker.

Se você encontrar um problema de IU reproduzível, abra um [issue](https://github.com/crateview/registry-ui/issues) com a revisão afetada e detalhes de reprodução limpos. Use o [canal de segurança privada](https://github.com/crateview/registry-ui/security/policy) para vulnerabilidades.
## Monitoramento

Configuração desta versão: [Monitoramento](/pt-BR/docs/v0.4/monitoring).

Configuração desta versão — Monitoramento opcional, desativado por padrão. Visão geral abre o painel quando ativo; caso contrário /home redireciona para /explore. A ativação expõe dados globais a todos com acesso à interface, incluindo visitantes anônimos em registros públicos.
