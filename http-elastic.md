# Métodos HTTP no Elasticsearch

Guia completo sobre os métodos HTTP utilizados no Elasticsearch e Kibana Dev Tools.

---

## GET - Buscar/Consultar

**Para que serve:** Recuperar informações, buscar dados, consultar configurações.

**Como funciona:** Não altera nada, apenas lê dados.

### Exemplos

```json
# Buscar documentos
GET /meu-indice/_search
{
  "query": {
    "match": { "titulo": "portal" }
  }
}

# Ver configurações de um índice
GET /meu-indice/_settings

# Ver mapping
GET /meu-indice/_mapping

# Listar todos os índices
GET _cat/indices?v

# Buscar documento por ID
GET /meu-indice/_doc/123

# Verificar detalhes de um índice
GET /meu-indice
```

---

## POST - Criar/Executar

**Para que serve:** Criar documentos sem especificar ID, executar operações, fazer buscas complexas.

**Como funciona:** Cria novos recursos ou executa ações. O Elasticsearch gera ID automaticamente.

### Exemplos

```json
# Criar documento (ID automático)
POST /meu-indice/_doc
{
  "titulo": "Nova notícia",
  "conteudo": "Texto aqui"
}

# Busca (alternativa ao GET)
POST /meu-indice/_search
{
  "query": {
    "match_all": {}
  }
}

# Executar reindex
POST _reindex
{
  "source": { "index": "indice-velho" },
  "dest": { "index": "indice-novo" }
}

# Atualizar documento parcialmente
POST /meu-indice/_update/123
{
  "doc": {
    "visualizacoes": 100
  }
}

# Refresh do índice
POST /meu-indice/_refresh

# Bulk operations
POST _bulk
{ "index": { "_index": "meu-indice" } }
{ "campo": "valor" }

# Deletar por query
POST /meu-indice/_delete_by_query
{
  "query": {
    "match": { "ativo": false }
  }
}
```

---

## PUT - Criar/Substituir

**Para que serve:** Criar recursos com ID específico ou substituir completamente um recurso existente.

**Como funciona:** Cria se não existe, substitui totalmente se já existe.

### Exemplos

```json
# Criar índice
PUT /meu-novo-indice

# Criar índice com configurações
PUT /meu-indice
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "titulo": { "type": "text" },
      "data": { "type": "date" }
    }
  }
}

# Criar documento com ID específico
PUT /meu-indice/_doc/123
{
  "titulo": "Documento específico",
  "ativo": true
}

# Criar/atualizar mapping
PUT /meu-indice/_mapping
{
  "properties": {
    "novo_campo": { "type": "keyword" }
  }
}

# Criar template
PUT _index_template/meu-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1
    }
  }
}

# Criar alias
PUT /meu-indice/_alias/meu-alias

# Atualizar settings de um índice
PUT /meu-indice/_settings
{
  "index": {
    "number_of_replicas": 2
  }
}
```

---

## DELETE - Deletar

**Para que serve:** Remover índices, documentos, templates, aliases.

**Como funciona:** Remove permanentemente o recurso especificado.

### Exemplos

```json
# Deletar índice
DELETE /meu-indice

# Deletar múltiplos índices
DELETE /indice-1,indice-2,indice-3

# Deletar documento específico
DELETE /meu-indice/_doc/123

# Deletar por query (múltiplos documentos)
POST /meu-indice/_delete_by_query
{
  "query": {
    "match": { "ativo": false }
  }
}

# Deletar template
DELETE _index_template/meu-template

# Deletar alias
DELETE /meu-indice/_alias/meu-alias
```

---

## HEAD - Verificar existência

**Para que serve:** Verificar se um recurso existe sem retornar o corpo da resposta.

**Como funciona:** Retorna apenas o status HTTP (200 se existe, 404 se não existe).

### Exemplos

```json
# Verificar se índice existe
HEAD /meu-indice

# Verificar se documento existe
HEAD /meu-indice/_doc/123

# Verificar se template existe
HEAD _index_template/meu-template

# Verificar se alias existe
HEAD /meu-indice/_alias/meu-alias
```

**Resposta:** Apenas código HTTP, sem body. Útil para verificações rápidas e economia de banda.

---

## PATCH - Atualizar parcialmente

**Para que serve:** Atualizar apenas campos específicos sem substituir o documento inteiro.

**Como funciona:** Modifica apenas os campos informados, mantém o resto.

### Exemplos

```json
# Atualizar campos específicos
PATCH /meu-indice/_doc/123
{
  "doc": {
    "visualizacoes": 150,
    "ativo": false
  }
}
```

**Nota:** No Elasticsearch, PATCH é menos comum. Normalmente usa-se `POST /indice/_update/id` para atualizações parciais.

---

## Resumo Comparativo

| Método | Uso principal | Cria se não existe? | Substitui se existe? | Idempotente? |
|--------|---------------|---------------------|----------------------|--------------|
| **GET** | Buscar/Consultar | Não | Não | Sim |
| **POST** | Criar (ID auto) / Executar | Sim | Adiciona novo | Não |
| **PUT** | Criar (ID manual) / Substituir | Sim | Sim (total) | Sim |
| **DELETE** | Remover | Não | Remove | Sim |
| **HEAD** | Verificar existência | Não | Não | Sim |
| **PATCH** | Atualizar parcial | Não | Sim (parcial) | Não |

---

## Diferenças Importantes

### POST vs PUT para documentos

```json
# POST - ID automático (cria sempre novo documento)
POST /indice/_doc
{ "campo": "valor" }
# Resposta: "_id": "auto-gerado-xyz123"

# PUT - ID manual (substitui se existir)
PUT /indice/_doc/123
{ "campo": "valor" }
# Resposta: "_id": "123"
```

### POST _update vs PUT (documento completo)

```json
# POST _update - atualiza só campos especificados
POST /indice/_update/123
{
  "doc": { 
    "visualizacoes": 100 
  }
}
# Resultado: Outros campos permanecem intactos

# PUT - substitui documento inteiro
PUT /indice/_doc/123
{
  "visualizacoes": 100
}
# Resultado: Outros campos são REMOVIDOS!
```

### GET vs POST para buscas

```json
# GET - busca simples
GET /indice/_search?q=titulo:portal

# POST - busca complexa com body
POST /indice/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "titulo": "portal" } }
      ],
      "filter": [
        { "range": { "data": { "gte": "2024-01-01" } } }
      ]
    }
  }
}
```

---

## Boas Práticas

### 1. Use GET para leitura
```json
# Correto
GET /indice/_search

# Evite (funciona, mas semanticamente incorreto)
POST /indice/_search  # use apenas se body for muito grande
```

### 2. Use POST para criar sem ID
```json
# Correto
POST /indice/_doc
{ "titulo": "novo" }
```

### 3. Use PUT para criar com ID específico
```json
# Correto
PUT /indice/_doc/123
{ "titulo": "específico" }
```

### 4. Use DELETE com cuidado
```json
# Perigoso - deleta tudo
DELETE /meu-indice

# Mais seguro - deletar documentos específicos
POST /meu-indice/_delete_by_query
{
  "query": {
    "term": { "_id": "123" }
  }
}
```

### 5. Use HEAD para verificações rápidas
```json
# Eficiente
HEAD /meu-indice

# Menos eficiente (retorna todo o body)
GET /meu-indice
```

---

## Casos de Uso no Portal Único Conector CES

### Buscar notícias
```json
GET /noticias-portal/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "conteudo": "governo" } }
      ],
      "filter": [
        { "term": { "ativo": true } }
      ]
    }
  }
}
```

### Criar nova notícia
```json
POST /noticias-portal/_doc
{
  "titulo": "Nova funcionalidade",
  "conteudo": "Descrição...",
  "autor": "João Carlos",
  "data_publicacao": "2024-01-19",
  "ativo": true
}
```

### Atualizar visualizações
```json
POST /noticias-portal/_update/123
{
  "script": {
    "source": "ctx._source.visualizacoes += params.count",
    "params": { "count": 1 }
  }
}
```

### Verificar se índice existe
```json
HEAD /noticias-portal
```

### Reindexar dados
```json
POST _reindex
{
  "source": {
    "index": "noticias-portal-old"
  },
  "dest": {
    "index": "noticias-portal-new"
  }
}
```

---

## Status HTTP Comuns

| Código | Significado | Quando ocorre |
|--------|-------------|---------------|
| 200 | OK | Operação bem-sucedida |
| 201 | Created | Recurso criado com sucesso |
| 204 | No Content | Sucesso sem retorno (comum com HEAD) |
| 400 | Bad Request | Sintaxe inválida na requisição |
| 404 | Not Found | Recurso não encontrado |
| 409 | Conflict | Conflito de versão |
| 429 | Too Many Requests | Rate limit excedido |
| 500 | Internal Server Error | Erro no servidor |

---

## Referências

- [Elasticsearch REST APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)
- [HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [Kibana Dev Tools](https://www.elastic.co/guide/en/kibana/current/console-kibana.html)

---

**Criado por:** João Carlos  
**Data:** 19 de Janeiro de 2024  
**Projeto:** portal-unico-conector-ces-api