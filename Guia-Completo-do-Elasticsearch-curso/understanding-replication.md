# Understanding Replication

## Ideia central

Replicacao e o mecanismo que cria copias dos shards de um indice.

Essas copias existem para dois objetivos principais:

- aumentar disponibilidade
- evitar perda de dados em falha de no
- aumentar throughput de leitura e busca

Sem replicacao, se o no que armazena um shard falhar e nao existir outra copia, os dados daquele shard sao perdidos.

## O problema que a replicacao resolve

Quando voce aprendeu sharding, viu que um indice novo tem `1` shard primario por padrao.

Isso resolve a distribuicao dos dados, mas nao resolve tolerancia a falhas.

Se o disco ou o no que armazena esse shard quebrar:

- o shard some
- os dados daquele shard ficam indisponiveis
- pode haver perda de dados

Como falha de hardware e algo real em qualquer ambiente, precisamos de redundancia.

E exatamente esse o papel da replicacao.

## Como a replicacao funciona

Assim como o sharding, a replicacao e configurada no nivel do indice.

Ela funciona criando copias de cada shard primario do indice.

Essas copias sao chamadas de:

- replicas
- replica shards

O shard original passa a ser chamado de:

- primary shard

O conjunto formado pelo shard primario e suas replicas e chamado de:

- replication group

Em outras palavras:

- o indice e dividido em shards
- cada shard primario pode ter zero, uma ou mais replicas
- cada replica e uma copia completa do shard primario

Replica shard pode responder buscas do mesmo jeito que o shard primario.

## Configuracao padrao

Quando um indice e criado, o padrao e:

- `1` shard primario
- `1` replica por shard

Na pratica, isso significa que um indice com `1` shard primario tenta ter:

- `1` primary shard
- `1` replica shard

Se o indice tiver `2` shards primarios e `2` replicas por shard, entao existirao:

- `2` shards primarios
- `4` replica shards
- `2` replication groups

## Regra mais importante da replicacao

Uma replica nunca e armazenada no mesmo no que o shard primario correspondente.

Essa regra existe para garantir redundancia real.

Se primario e replica ficassem no mesmo no, uma unica falha derrubaria as duas copias ao mesmo tempo.

Por isso:

- o primario fica em um no
- a replica precisa ficar em outro no elegivel

## O que acontece em cluster com 1 no

Replicacao so faz sentido de verdade quando o cluster tem mais de um no.

Se o cluster tiver apenas `1` no:

- o shard primario pode ser alocado normalmente
- a replica nao pode ser colocada no mesmo no
- a replica fica `UNASSIGNED`
- o cluster tende a ficar `yellow`

Isso nao quer dizer que o indice esta quebrado.

Quer dizer apenas que:

- o indice esta funcional
- mas nao existe redundancia real
- se esse unico no falhar, voce pode perder dados

Em ambiente de desenvolvimento isso costuma ser aceitavel.
Em producao, normalmente nao e.

## O que acontece quando um novo no entra no cluster

Quando um segundo no e adicionado ao cluster, o Elasticsearch pode finalmente alocar as replicas pendentes.

Nesse momento:

- a replicacao passa a ter efeito real
- o cluster pode sair de `yellow` para `green`
- uma falha de no deixa de significar perda imediata de dados

Se houver mais replicas configuradas e mais nos disponiveis, o Elasticsearch pode espalhar ainda mais as copias.

## Quantas replicas usar

Nao existe um numero universal.

A decisao depende de:

- criticidade do sistema
- tolerancia a indisponibilidade
- possibilidade de restaurar dados de outra fonte
- numero de nos no cluster
- custo de armazenamento

Regra pratica razoavel:

- `1` replica costuma ser suficiente na maioria dos casos
- `2` replicas ou mais pode fazer sentido em sistemas criticos

Tambem existe uma regra estrutural importante:

- para ter protecao real, voce precisa de pelo menos `2` nos
- para aproveitar mais replicas, voce precisa de nos suficientes para distribui-las

## Replicacao vs snapshots

Replicacao e snapshot nao sao a mesma coisa.

### Replicacao

Replicacao protege os dados vivos do indice no momento atual.

Ela serve para:

- sobreviver a falha de no
- manter o indice disponivel
- continuar atendendo buscas mesmo apos perda de um no

### Snapshots

Snapshots sao backups em ponto no tempo.

Eles servem para:

- restaurar dados para um estado anterior
- recuperar um indice depois de erro operacional
- manter backup diario ou programado

Exemplo pratico:

- voce altera milhoes de documentos
- algo sai errado
- a replicacao nao ajuda, porque ela tambem replica o estado errado
- o snapshot permite voltar ao estado anterior

Resumo:

- replicacao protege continuidade e disponibilidade
- snapshot protege historico e rollback

## Replicacao tambem aumenta throughput

Replicacao nao serve apenas para disponibilidade.

Ela tambem pode aumentar a capacidade de leitura de um indice.

Isso acontece porque replica shard tambem pode responder consultas.

Se um indice tiver:

- `1` shard primario
- `2` replicas

entao o Elasticsearch pode distribuir buscas entre:

- o shard primario
- replica 1
- replica 2

Assim, varias buscas podem rodar em paralelo sobre copias do mesmo conteudo.

Isso pode aumentar o throughput de leitura, especialmente quando:

- o indice recebe muitas consultas
- os nos ainda tem CPU e memoria disponiveis
- o gargalo e leitura, nao escrita

Mas esse ganho nao e automatico em qualquer ambiente.

Limitacoes importantes:

- replica consome disco, porque e copia completa
- se os nos ja estiverem saturados, o ganho pode ser pequeno
- adicionar replicas sem hardware suficiente nao resolve tudo

## Exemplo mental de uso

Imagine um indice `products` com:

- `1` shard primario
- `1` replica

Se o cluster tiver `2` nos, o primario pode ficar em um no e a replica em outro.

Beneficios:

- se um no cair, ainda existe uma copia
- as buscas podem ser distribuidas entre os dois shards

Se voce aumentar para `2` replicas, podera ter ainda mais throughput de busca, desde que existam nos e recursos suficientes para isso.

## O que observar no Kibana

Para enxergar a replicacao na pratica, vale observar:

- saude do cluster
- status do indice
- estado dos shards
- configuracao de replicas

## Consultas para executar no Kibana

As consultas abaixo ajudam a visualizar a replicacao no seu ambiente.

## 1. Criar um indice com configuracao padrao

```http
PUT /pages
```

Como nao ha `settings` explicitas, o indice sera criado com:

- `1` shard primario
- `1` replica

## 2. Ver a saude do cluster

```http
GET /_cluster/health?pretty
```

Em um cluster com apenas `1` no, o esperado e ver o status `yellow`.

Isso acontece porque a replica ainda nao encontrou um segundo no para ser alocada.

## 3. Listar os indices

```http
GET /_cat/indices?v&h=health,status,index,pri,rep,docs.count,store.size
```

Observe principalmente:

- `pri`: numero de shards primarios
- `rep`: numero de replicas configuradas
- `health`: estado do indice

Se `pages` aparecer com `rep = 1` em um cluster de 1 no, isso ajuda a explicar o `yellow`.

## 4. Ver os shards do indice

```http
GET /_cat/shards/pages?v&h=index,shard,prirep,state,node,unassigned.reason
```

Leitura pratica esperada:

- o shard com `prirep = p` deve estar `STARTED`
- o shard com `prirep = r` tende a ficar `UNASSIGNED`

Significado:

- `p` = primary shard
- `r` = replica shard

## 5. Ver a configuracao do indice

```http
GET /pages/_settings?filter_path=*.settings.index.number_of_shards,*.settings.index.number_of_replicas
```

Essa consulta confirma quantos shards e replicas o indice recebeu.

## 6. Ajustar replicas em ambiente local

Se o objetivo for apenas estudo em um cluster de `1` no, voce pode reduzir replicas para `0` e evitar `yellow`.

```http
PUT /pages/_settings
{
  "index": {
    "number_of_replicas": 0
  }
}
```

Depois disso, o cluster tende a ficar `green`, desde que nao exista outro problema.

## 7. Aumentar replicas novamente

```http
PUT /pages/_settings
{
  "index": {
    "number_of_replicas": 1
  }
}
```

Se o cluster continuar com apenas `1` no, a replica voltara a ficar pendente.

## 8. Remover o indice de teste

```http
DELETE /pages
```

## Kibana e `auto_expand_replicas`

Alguns indices internos do Kibana podem aparecer com:

- `1` shard
- `0` replicas

Isso pode parecer estranho a primeira vista, mas existe um detalhe importante.

Esses indices podem usar a configuracao:

```text
auto_expand_replicas = 0-1
```

Esse valor significa:

- com `1` no, o indice fica com `0` replicas
- com mais de `1` no, o indice sobe para `1` replica

Ou seja, a quantidade de replicas pode ser ajustada dinamicamente conforme o numero de nos do cluster.

## Resumo direto

- replicacao cria copias dos shards primarios
- cada copia e uma replica shard
- primario + replicas formam um replication group
- o padrao de um indice novo e `1` shard primario e `1` replica
- replica nunca fica no mesmo no que seu primario
- em cluster de `1` no, replicas ficam `UNASSIGNED`
- por isso o cluster costuma ficar `yellow`
- replicacao aumenta disponibilidade e tolerancia a falhas
- replicacao tambem pode aumentar throughput de leitura
- snapshots nao substituem replicacao
- snapshots servem para backup e rollback
- para producao, normalmente voce quer pelo menos `2` nos
