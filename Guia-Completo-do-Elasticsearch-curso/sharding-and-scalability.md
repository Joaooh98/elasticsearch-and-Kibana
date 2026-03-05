# Sharding and Scalability

## Ideia central

Elasticsearch escala horizontalmente com a ajuda de nos e shards.

Se um indice ficar grande demais para caber em um unico no, o Elasticsearch pode quebrar esse indice em partes menores. Cada parte e um `shard`.

Esse e o papel do sharding:

- aumentar a capacidade de armazenamento
- distribuir dados entre nos
- permitir que buscas sejam executadas em paralelo

## O que e sharding

Sharding e o processo de dividir um indice em varias partes menores.

Pontos importantes:

- o sharding acontece no nivel do indice
- nao acontece no nivel do cluster inteiro
- cada shard pertence a um indice especifico

Isso importa porque indices diferentes podem ter perfis completamente diferentes. Um indice pode ter centenas de documentos. Outro pode ter milhoes ou bilhoes.

Por isso, a configuracao de shards precisa ser pensada por indice, e nao por cluster de forma generica.

## O que e um shard

Um shard pode ser entendido como uma particao do indice.

Na pratica:

- cada shard fica alocado em um unico no
- cada shard funciona como uma unidade independente de armazenamento e busca
- internamente, cada shard e um indice do Apache Lucene

Ou seja:

- 1 indice Elasticsearch com 5 shards
- por baixo dos panos = 5 indices Lucene

## Por que shards existem

### 1. Escalar armazenamento

Imagine dois nos com 500 GB cada.

Se voce tiver um indice com 600 GB e ele tiver apenas 1 shard, esse shard nao cabe em nenhum dos nos.

Mas se esse mesmo indice tiver 2 shards de 300 GB:

- shard 0 pode ir para o no 1
- shard 1 pode ir para o no 2

Agora o indice cabe no cluster.

### 2. Escalar quantidade de documentos

Um shard tem limite de documentos. Em termos praticos, um shard pode armazenar pouco mais de 2 bilhoes de documentos.

Se um indice precisar crescer muito, usar varios shards permite distribuir esse volume.

### 3. Aumentar throughput de busca

Como os shards sao independentes, o Elasticsearch pode executar partes de uma busca em paralelo.

Isso significa:

- uma consulta pode atingir varios shards ao mesmo tempo
- se esses shards estiverem em nos diferentes, o cluster usa mais hardware
- a busca ganha mais throughput

O ganho de performance nao e automatico em qualquer cenario, mas a capacidade de paralelismo existe por causa dos shards.

## Shards nao precisam ficar um por no

Se um indice tiver 5 shards, isso nao significa que o cluster precisa de 5 nos.

Os shards podem ser distribuidos de varias formas:

- varios shards no mesmo no
- shards espalhados entre poucos nos
- shards espalhados entre muitos nos

O importante e lembrar:

- um shard individual sempre fica em um unico no
- mas varios shards podem coexistir no mesmo no

## Configuracao padrao

Desde o Elasticsearch 7, um indice novo e criado com:

- `1` shard primario por padrao

Historicamente, o padrao era `5` shards primarios. Isso causava muito over-sharding em clusters pequenos com indices pequenos.

## O que e over-sharding

Over-sharding acontece quando voce cria shards demais para a quantidade real de dados.

Problemas comuns:

- desperdicio de memoria
- mais sobrecarga de gerenciamento
- mais custo de busca e coordenacao
- cluster pequeno ficando pesado sem necessidade

Em outras palavras: shard demais tambem e problema.

## Posso mudar a quantidade de shards depois?

O numero de shards primarios nao e algo que voce altera livremente depois da criacao do indice.

Existem APIs como:

- `Split API`
- `Shrink API`

Mas a ideia principal desta aula e:

- escolher bem no inicio e mais simples
- para pequenos e medios indices, `1` shard costuma ser suficiente
- se voce espera muito crescimento, pode valer a pena criar com mais shards desde o inicio

## Como pensar na quantidade de shards

Nao existe formula magica.

Depende de fatores como:

- quantidade de nos
- capacidade de disco
- volume de documentos
- tamanho medio dos documentos
- quantidade de indices
- quantidade de consultas
- taxa de escrita

Regra pratica razoavel:

- pequenos e medios indices: normalmente `1` shard e suficiente
- indices que devem receber muitos milhoes de documentos: `5` shards pode ser um bom ponto de partida

Mas isso nao e regra fixa. E apenas um ponto inicial para raciocinar.

## Relacao com replicas

Shard primario e a particao principal.
Replica e uma copia desse shard.

Nesta aula, o foco e escalabilidade via sharding, mas vale lembrar:

- shards ajudam a dividir o indice
- replicas ajudam em disponibilidade e leitura

No seu ambiente local com 1 no, use `number_of_replicas: 0` nos exemplos, para evitar cluster `yellow`.

## O que observar no Kibana

Quando voce roda:

```http
GET /_cat/indices?v
```

uma das colunas exibidas e `pri`.

Essa coluna representa a quantidade de shards primarios do indice.

Se um indice mostrar:

- `pri = 1`

isso significa que ele foi criado com 1 shard primario.

## Consultas para executar no Kibana

As consultas abaixo ajudam a enxergar sharding na pratica.

## 1. Ver indices e quantidade de shards primarios

```http
GET /_cat/indices?v&h=health,status,index,pri,rep,docs.count,store.size
```

Use essa consulta para observar:

- `pri`: numero de shards primarios
- `rep`: numero de replicas
- `store.size`: tamanho total do indice

## 2. Ver configuracao de shards de um indice especifico

Troque `produtos` pelo nome do indice que quiser inspecionar.

```http
GET /produtos/_settings?filter_path=*.settings.index.number_of_shards,*.settings.index.number_of_replicas
```

## 3. Criar um indice com 1 shard

```http
PUT /demo_shards_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

## 4. Criar um indice com 5 shards

```http
PUT /demo_shards_5
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 0
  }
}
```

Esses dois indices servem para comparar o comportamento de configuracoes diferentes.

## 5. Confirmar a configuracao dos dois indices

```http
GET /demo_shards_1,demo_shards_5/_settings?filter_path=*.settings.index.number_of_shards,*.settings.index.number_of_replicas
```

## 6. Ver os shards alocados

```http
GET /_cat/shards/demo_shards_1,demo_shards_5?v&h=index,shard,prirep,state,node
```

Aqui voce consegue ver:

- quantos shards cada indice realmente tem
- qual shard esta em qual no
- que todos os shards estao no mesmo no no seu cluster local de 1 no

## 7. Indexar alguns documentos de exemplo

```http
POST /demo_shards_1/_doc
{
  "nome": "produto 1",
  "categoria": "teste",
  "preco": 10
}
```

```http
POST /demo_shards_5/_doc
{
  "nome": "produto 1",
  "categoria": "teste",
  "preco": 10
}
```

```http
POST /demo_shards_5/_doc
{
  "nome": "produto 2",
  "categoria": "teste",
  "preco": 20
}
```

## 8. Contar documentos

```http
GET /demo_shards_1/_count
```

```http
GET /demo_shards_5/_count
```

## 9. Executar busca e observar o bloco `_shards`

```http
GET /demo_shards_1/_search?filter_path=took,_shards,hits.total
{
  "query": {
    "match_all": {}
  }
}
```

```http
GET /demo_shards_5/_search?filter_path=took,_shards,hits.total
{
  "query": {
    "match_all": {}
  }
}
```

Observe a resposta em `_shards`.

Voce tende a ver algo como:

- no indice com 1 shard, `_shards.total` proximo de `1`
- no indice com 5 shards, `_shards.total` proximo de `5`

Isso ajuda a visualizar que a busca esta sendo executada sobre mais particoes.

## 10. Ver saude do cluster depois dos indices de teste

```http
GET /_cluster/health?pretty
```

Como os exemplos usam `number_of_replicas: 0`, o esperado no seu ambiente de 1 no e evitar replicas pendentes nesses indices de laboratorio.

## 11. Remover os indices de teste

```http
DELETE /demo_shards_1,demo_shards_5
```

## Roteiro de estudo sugerido

Se quiser absorver o tema de forma pratica, siga esta ordem:

1. Rode `GET /_cat/indices?v&h=health,status,index,pri,rep,docs.count,store.size`
2. Crie `demo_shards_1`
3. Crie `demo_shards_5`
4. Rode `GET /_cat/shards/demo_shards_1,demo_shards_5?v&h=index,shard,prirep,state,node`
5. Indexe alguns documentos
6. Rode `_search` nos dois indices e compare o bloco `_shards`
7. Apague os indices de teste

## Resumo direto

- sharding divide um indice em partes menores
- cada parte e um shard
- shard e criado no nivel do indice
- cada shard fica em um unico no
- shards permitem escalar armazenamento e throughput
- desde o Elasticsearch 7, o padrao e `1` shard primario por indice
- shard demais pode causar over-sharding
- para pequenos e medios indices, `1` shard costuma bastar
- para indices que vao crescer muito, vale pensar em mais shards no momento da criacao
