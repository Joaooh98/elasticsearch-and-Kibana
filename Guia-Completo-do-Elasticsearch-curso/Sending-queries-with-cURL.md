# Enviando Queries com cURL

## Objetivo

O Kibana Console e a forma mais pratica de testar queries no Elasticsearch, mas saber usar `cURL` continua sendo importante para:

- testar a API direto no terminal
- reproduzir chamadas fora do Kibana
- automatizar consultas em scripts
- entender melhor autenticacao, headers e corpo da requisicao

## Quando usar Kibana Console e quando usar cURL

### Kibana Console

Use quando quiser:

- escrever queries mais rapido
- aproveitar auto-complete
- ler respostas formatadas
- aprender a sintaxe das APIs do Elasticsearch

### cURL

Use quando quiser:

- testar o endpoint real do Elasticsearch
- fazer chamadas fora do navegador
- automatizar no shell
- depurar erro de conexao, autenticacao ou header

## Contexto da aula vs seu ambiente atual

A aula transcrita mostra o comportamento comum do Elasticsearch 8 com TLS habilitado, usando:

```bash
https://localhost:9200
```

No seu projeto atual isso e diferente. No seu `docker-compose.yml`, o HTTP SSL esta desativado:

- `xpack.security.enabled: "true"`
- `xpack.security.http.ssl.enabled: "false"`

Isso significa:

- a autenticacao continua ativa
- mas o endpoint local usa `http`
- no seu caso atual, os exemplos corretos sao com `http://localhost:9200`
- voce nao precisa de `--insecure` nem de `--cacert` nesse setup

Resumo:

- aula original: `https` + certificado
- seu ambiente local atual: `http` + usuario/senha

## Estrutura basica de um comando cURL

```bash
curl -X GET "http://localhost:9200" -u elastic:SUA_SENHA
```

Significado das partes:

- `curl`: cliente HTTP de terminal
- `-X GET`: verbo HTTP
- URL: endpoint do Elasticsearch
- `-u elastic:SUA_SENHA`: autenticacao basica

O `GET` pode ser omitido quando a chamada ja e naturalmente um `GET`.

## Primeira chamada

```bash
curl "http://localhost:9200" -u elastic:SUA_SENHA
```

Essa chamada retorna informacoes basicas do cluster, como:

- nome do no
- nome do cluster
- versao do Elasticsearch

Se quiser uma resposta mais legivel:

```bash
curl "http://localhost:9200?pretty" -u elastic:SUA_SENHA
```

## Autenticacao com `-u`

Existem duas formas principais.

### Informar apenas o usuario

```bash
curl -u elastic "http://localhost:9200"
```

O `cURL` vai pedir a senha depois.

Vantagem:

- a senha nao aparece no comando digitado

### Informar usuario e senha juntos

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200"
```

Vantagem:

- mais rapido para ambiente local

Desvantagem:

- a senha pode ficar visivel no terminal e no historico do shell

## Quando aparecem erros de TLS

Isso vale para instalacoes que usam `https`.

### Exemplo

```bash
curl -X GET "https://localhost:9200"
```

Voce pode receber algo como:

```text
curl: (60) SSL certificate problem: self signed certificate
```

Isso acontece porque o Elasticsearch local costuma gerar certificado autoassinado, e o `cURL` nao confia nele por padrao.

### Solucao rapida em ambiente local

```bash
curl --insecure -u elastic "https://localhost:9200"
```

O `--insecure` ignora a validacao do certificado.

### Solucao mais correta

```bash
curl --cacert config/certs/http_ca.crt -u elastic "https://localhost:9200"
```

Aqui voce fornece explicitamente o certificado da CA usada pelo Elasticsearch.

## Quando aparece `missing authentication credentials`

Exemplo:

```json
{
  "error": {
    "type": "security_exception",
    "reason": "missing authentication credentials for REST request [/]"
  },
  "status": 401
}
```

Isso significa que o cluster exige autenticacao e voce nao passou `-u`.

Correcao:

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200"
```

## Enviando corpo JSON com `-d`

Quando uma API precisa de body JSON, usamos `-d`.

Exemplo com a API de busca:

```bash
curl -X GET "http://localhost:9200/products/_search?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "match_all": {}
    }
  }'
```

Esse comando faz o seguinte:

- chama a API `_search`
- envia um JSON no corpo
- usa `match_all`, que tenta retornar todos os documentos do indice

## Por que `Content-Type` e obrigatorio

Quando voce usa `-d`, o `cURL` nao assume automaticamente que o corpo e JSON.

Por isso precisamos declarar:

```bash
-H "Content-Type: application/json"
```

Sem isso, o Elasticsearch pode responder com erro dizendo que o `Content-Type` nao e suportado.

## GET com body vs POST com body

O Elasticsearch aceita `GET` com body em varias APIs, inclusive `_search`. Mesmo assim, em alguns contextos `POST` pode ser mais conveniente.

Exemplo equivalente:

```bash
curl -X POST "http://localhost:9200/products/_search?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "match_all": {}
    }
  }'
```

## Exemplos uteis para praticar

### 1. Ver informacoes do cluster

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200?pretty"
```

### 2. Ver a saude do cluster

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200/_cluster/health?pretty"
```

### 3. Ver os nos

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/nodes?v"
```

### 4. Ver os indices

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/indices?v&expand_wildcards=all"
```

### 5. Ver shards e replicas pendentes

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/shards?v&h=index,shard,prirep,state,node,unassigned.reason"
```

### 6. Criar um indice

```bash
curl -X PUT "http://localhost:9200/produtos?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }'
```

### 7. Inserir um documento

```bash
curl -X POST "http://localhost:9200/produtos/_doc?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Notebook",
    "preco": 3500,
    "categoria": "eletronicos"
  }'
```

### 8. Buscar todos os documentos

```bash
curl -X GET "http://localhost:9200/produtos/_search?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "match_all": {}
    }
  }'
```

### 9. Buscar por termo exato

```bash
curl -X GET "http://localhost:9200/produtos/_search?pretty" \
  -u elastic:SUA_SENHA \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "term": {
        "categoria.keyword": "eletronicos"
      }
    }
  }'
```

### 10. Apagar um indice

```bash
curl -X DELETE "http://localhost:9200/produtos?pretty" \
  -u elastic:SUA_SENHA
```

## Erros comuns

### `Empty reply from server`

Normalmente acontece quando voce chama `http://` em um servidor que espera `https://`.

### `SSL certificate problem: self signed certificate`

O servidor esta usando TLS com certificado autoassinado e o `cURL` nao confia nele.

Saidas possiveis:

- usar `--insecure` para laboratorio
- usar `--cacert` apontando para o certificado correto

### `missing authentication credentials`

Voce esqueceu a autenticacao com `-u`.

### `Content-Type header [...] is not supported`

Voce enviou `-d`, mas esqueceu o header:

```bash
-H "Content-Type: application/json"
```

### `index_not_found_exception`

O indice informado ainda nao existe.

## Observacao para Windows

Em Linux e macOS, normalmente e mais simples escrever o JSON com aspas simples:

```bash
-d '{
  "query": {
    "match_all": {}
  }
}'
```

Em Windows, isso pode falhar dependendo do terminal. Nesse caso, use aspas duplas e escape o JSON:

```bash
curl -X GET "http://localhost:9200/produtos/_search?pretty" -u elastic:SUA_SENHA -H "Content-Type: application/json" -d "{\"query\":{\"match_all\":{}}}"
```

## Dicas praticas

- use `?pretty` quando quiser JSON formatado
- use `_cat` quando quiser leitura humana e tabular
- use variaveis de ambiente para evitar repetir URL e credenciais
- monte primeiro a query no Kibana Console e depois converta para `cURL`

## Exemplo com variaveis de ambiente

```bash
export ES_URL="http://localhost:9200"
export ES_USER="elastic"
export ES_PASS="SUA_SENHA"

curl -u "$ES_USER:$ES_PASS" "$ES_URL/_cluster/health?pretty"
```

## Exemplo com `jq`

Se tiver `jq` instalado, pode melhorar a leitura da resposta:

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200/_cluster/health" | jq
```

## Resumo direto

- Kibana Console e melhor para aprender e explorar
- `cURL` e melhor para terminal, script e automacao
- no seu ambiente atual, use `http://localhost:9200`
- passe autenticacao com `-u`
- quando enviar JSON com `-d`, inclua `-H "Content-Type: application/json"`

## Mini cheat sheet

```bash
curl -u elastic:SUA_SENHA "http://localhost:9200"
curl -u elastic:SUA_SENHA "http://localhost:9200/_cluster/health?pretty"
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/nodes?v"
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/indices?v&expand_wildcards=all"
curl -u elastic:SUA_SENHA "http://localhost:9200/_cat/shards?v&h=index,shard,prirep,state,node,unassigned.reason"
curl -X GET "http://localhost:9200/produtos/_search?pretty" -u elastic:SUA_SENHA -H "Content-Type: application/json" -d '{"query":{"match_all":{}}}'
```
