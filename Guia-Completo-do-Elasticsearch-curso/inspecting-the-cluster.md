# Inspecionando o Cluster

## Objetivo da inspeção

Inspecionar o cluster significa verificar se o Elasticsearch esta saudavel, quantos nos existem, como os shards foram distribuidos e se existe algum problema de alocacao.

Na pratica, esse tipo de consulta responde perguntas como:

- O cluster esta funcional?
- Existem shards faltando?
- Ha replicas pendentes?
- Esse `yellow` e normal ou indica problema?
- Em qual no cada shard foi alocado?

## 1. Verificando a saude geral do cluster

```http
GET /_cluster/health
```

Essa API retorna um resumo da saude do cluster.

### Principais status

- `green`: tudo esta alocado. Primarios e replicas estao ativos.
- `yellow`: os shards primarios estao ativos, mas uma ou mais replicas nao foram alocadas.
- `red`: pelo menos um shard primario nao foi alocado. Parte dos dados pode ficar indisponivel.

### Campos mais importantes da resposta

- `cluster_name`: nome do cluster.
- `status`: estado geral (`green`, `yellow`, `red`).
- `number_of_nodes`: quantidade total de nos no cluster.
- `number_of_data_nodes`: quantidade de nos de dados.
- `active_primary_shards`: quantos shards primarios estao ativos.
- `active_shards`: total de shards ativos, incluindo primarios e replicas.
- `unassigned_shards`: shards que existem na configuracao, mas nao conseguiram ser alocados.
- `unassigned_primary_shards`: shards primarios nao alocados. Se esse numero for maior que zero, o problema e serio.
- `active_shards_percent_as_number`: percentual total de shards ativos.

## 2. Entendendo por que o cluster fica `yellow`

O ponto central e este: um shard primario e sua replica nao podem ficar no mesmo no.

Isso existe para garantir redundancia real. Se o no cair, a replica deve sobreviver em outro no. Se primario e replica ficassem juntos, os dois seriam perdidos ao mesmo tempo.

Em um cluster local com apenas `1` no, o comportamento comum e:

- o primario e alocado normalmente
- a replica tenta ser alocada
- nao existe outro no disponivel
- a replica fica `UNASSIGNED`
- o cluster fica `yellow`

Ou seja: `yellow` em ambiente de estudo com um unico no normalmente nao significa falha grave. Significa apenas ausencia de um segundo no para receber as replicas.

## 3. Shards e replicas

### Shard primario

E a particao principal do indice. Todo indice precisa de shard primario para funcionar.

### Shard replica

E uma copia do shard primario. Serve para:

- aumentar a tolerancia a falhas
- melhorar disponibilidade
- distribuir leitura em clusters maiores

### Regra importante

- primario pode existir sozinho
- replica depende da existencia de outro no elegivel

Por isso um cluster com `1` no e indices com `number_of_replicas: 1` tende a ficar `yellow`.

## 4. Usando as APIs `_cat`

As APIs `_cat` sao feitas para leitura humana. Elas mostram informacoes em formato tabular, rapido de bater o olho.

Use `_cat` para diagnostico manual.
Use APIs JSON normais quando a resposta for consumida por aplicacao ou script.

## 5. Comandos `_cat` mais uteis

### Ver os nos do cluster

```http
GET /_cat/nodes?v
```

O parametro `v` mostra o cabecalho das colunas.

Esse comando ajuda a identificar:

- quantos nos existem
- nome do no
- IP
- uso de CPU
- memoria
- papel do no

### Ver os indices

```http
GET /_cat/indices?v&expand_wildcards=all
```

Esse comando mostra:

- nome do indice
- status
- quantidade de documentos
- quantidade de shards
- tamanho armazenado

O `expand_wildcards=all` faz a consulta incluir tambem indices ocultos e internos, como os do Kibana e do proprio Elasticsearch.

### Ver os shards e sua alocacao

```http
GET /_cat/shards?v=true&h=index,shard,prirep,state,node,unassigned.reason
```

Esse e um dos comandos mais importantes para entender o cluster.

#### Significado das colunas

- `index`: nome do indice.
- `shard`: numero do shard.
- `prirep`: indica se e `p` (primary) ou `r` (replica).
- `state`: estado atual do shard.
- `node`: no em que o shard esta alocado.
- `unassigned.reason`: motivo de o shard nao ter sido alocado.

#### Leitura pratica

Se voce encontrar algo assim:

```text
servicos_gov 0 p STARTED es01
servicos_gov 0 r UNASSIGNED CLUSTER_RECOVERED
```

Isso significa:

- o shard `0` primario do indice `servicos_gov` foi alocado com sucesso em `es01`
- a replica do mesmo shard nao conseguiu ser alocada
- como o cluster so tem um no, nao existe outro lugar para essa replica

## 6. Significado dos estados dos shards

- `STARTED`: shard ativo e funcionando.
- `INITIALIZING`: shard em processo de inicializacao.
- `RELOCATING`: shard sendo movido entre nos.
- `UNASSIGNED`: shard nao alocado em nenhum no.

Quando o shard `UNASSIGNED` for replica, o cluster pode ficar `yellow`.
Quando o shard `UNASSIGNED` for primario, o cluster pode ficar `red`.

## 7. Descobrindo a configuracao de replicas de um indice

```http
GET /servicos_gov/_settings?filter_path=*.settings.index.number_of_replicas
```

Esse comando serve para conferir quantas replicas o indice foi configurado para ter.

Se o resultado indicar `1`, mas o cluster tiver apenas um no, a replica ficara pendente.

## 8. Ajustando replicas em ambiente local

Se o ambiente for apenas de estudo e voce quiser o cluster `green`, pode reduzir replicas para `0`.

### Alterar um indice

```http
PUT /servicos_gov/_settings
{
  "index": {
    "number_of_replicas": 0
  }
}
```

### Alterar varios indices de estudo

```http
PUT /favorite_candy,meu-novo-indice,orcamento_publico,entidades_gov,final_wheat_data,stoks,servicos_gov/_settings
{
  "index": {
    "number_of_replicas": 0
  }
}
```

Depois disso, em um cluster de um no, a tendencia e o status mudar de `yellow` para `green`, desde que nao exista outro problema.

## 9. Roteiro rapido para diagnosticar o cluster

Quando quiser entender a situacao do cluster, siga esta sequencia:

### Passo 1. Ver a saude geral

```http
GET /_cluster/health
```

Pergunta respondida: o cluster esta `green`, `yellow` ou `red`?

### Passo 2. Ver onde esta o problema

```http
GET /_cat/shards?v=true&h=index,shard,prirep,state,node,unassigned.reason
```

Pergunta respondida: quais shards estao `UNASSIGNED` e se eles sao primarios ou replicas?

### Passo 3. Confirmar a configuracao do indice

```http
GET /servicos_gov/_settings?filter_path=*.settings.index.number_of_replicas
```

Pergunta respondida: o indice foi criado com replica?

### Passo 4. Ver os nos existentes

```http
GET /_cat/nodes?v
```

Pergunta respondida: ha nos suficientes para receber as replicas?

## 10. Resumo direto

- `/_cluster/health` mostra a saude geral do cluster.
- APIs `_cat` servem para inspeção rapida e leitura humana.
- `/_cat/shards` e a melhor forma de enxergar onde cada shard esta e por que algum ficou pendente.
- Cluster com `1` no e replicas configuradas costuma ficar `yellow`.
- Isso acontece porque a replica nao pode ficar no mesmo no do shard primario.

## Referencias

- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-nodes
- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-nodes-info
