# Estudo: Search Templates no Elasticsearch - Consultas por Cidade

## 1. Introdução

Este documento explora o uso de **Search Templates** no Elasticsearch, focando em consultas parametrizadas para buscar clientes por cidade em um índice de e-commerce.

## 2. Contexto do Índice

### 2.1 Estrutura do Índice: ecommerce_clientes_v1

O índice armazena informações de clientes com a seguinte estrutura relevante:
```json
"endereco": {
  "cidade": { 
    "type": "keyword", 
    "normalizer": "lowercase_normalizer" 
  }
}
```

**Ponto importante:** O campo `cidade` é do tipo `keyword` com normalização para minúsculas. Isso significa:
- Não é tokenizado (armazena valor exato)
- Normaliza automaticamente para minúsculas
- Remove acentos (asciifolding)
- Ideal para filtros exatos e agregações

### 2.2 Dados de Exemplo

O índice contém 10 clientes distribuídos em 5 cidades:
- Porto: 4 clientes (Bruno, Carla, Eduarda, Gabriela)
- Lisboa: 3 clientes (Ana, Daniel, Inês)
- Coimbra: 1 cliente (João)
- Braga: 1 cliente (Fábio)
- Viseu: 1 cliente (Hugo)

## 3. O que são Search Templates?

Search Templates são consultas parametrizadas e reutilizáveis no Elasticsearch. Funcionam como "funções" de busca que você define uma vez e executa múltiplas vezes com diferentes parâmetros.

### 3.1 Vantagens

- **Reutilização:** Escreve a query uma vez, usa múltiplas vezes
- **Manutenção:** Centraliza a lógica de busca
- **Segurança:** Evita injeção de código malicioso
- **Performance:** Templates são compilados e cacheados
- **Padronização:** Garante consistência nas buscas

### 3.2 Sintaxe Mustache

Os templates usam a linguagem **Mustache** para substituição de variáveis:
```mustache
{{variavel}}              // Substitui pelo valor
{{^variavel}}default{{/variavel}}  // Valor padrão se variável não existir
```

## 4. Template Completo: buscar_clientes_por_cidade

### 4.1 Criação do Template
```json
PUT _scripts/buscar_clientes_por_cidade
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "term": {
          "endereco.cidade": {
            "value": "{{cidade}}"
          }
        }
      },
      "from": "{{from}}{{^from}}0{{/from}}",
      "size": "{{size}}{{^size}}10{{/size}}",
      "sort": [
        {
          "{{sort_field}}{{^sort_field}}datas.criado_em{{/sort_field}}": {
            "order": "{{sort_order}}{{^sort_order}}desc{{/sort_order}}"
          }
        }
      ]
    }
  }
}
```

### 4.2 Anatomia do Template

#### Query Section
```json
"term": {
  "endereco.cidade": {
    "value": "{{cidade}}"
  }
}
```

- **term query:** Busca valor exato (perfeito para keywords)
- **{{cidade}}:** Parâmetro obrigatório que será substituído

#### Paginação
```json
"from": "{{from}}{{^from}}0{{/from}}",
"size": "{{size}}{{^size}}10{{/size}}"
```

- **from:** Offset inicial (padrão: 0)
- **size:** Quantidade de resultados (padrão: 10)
- **{{^from}}0{{/from}}:** Se `from` não for fornecido, usa 0

#### Ordenação Dinâmica
```json
"{{sort_field}}{{^sort_field}}datas.criado_em{{/sort_field}}": {
  "order": "{{sort_order}}{{^sort_order}}desc{{/sort_order}}"
}
```

- Campo de ordenação flexível (padrão: datas.criado_em)
- Ordem flexível (padrão: desc - mais recente primeiro)

### 4.3 Exemplos de Uso

#### Exemplo 1: Busca Básica
```json
POST ecommerce_clientes_v1/_search/template
{
  "id": "buscar_clientes_por_cidade",
  "params": {
    "cidade": "porto"
  }
}
```

**Resultado esperado:** 4 clientes do Porto, ordenados por data de criação (desc)

#### Exemplo 2: Com Paginação e Ordenação Customizada
```json
POST ecommerce_clientes_v1/_search/template
{
  "id": "buscar_clientes_por_cidade",
  "params": {
    "cidade": "lisboa",
    "from": 0,
    "size": 2,
    "sort_field": "stats.total_gasto",
    "sort_order": "desc"
  }
}
```

**Resultado esperado:** Primeiros 2 clientes de Lisboa com maior gasto total

**Análise dos dados:**
- Inês: 149.990 (1º)
- Ana: 24.990 (2º)
- Daniel: 0 (3º - não seria retornado devido ao size=2)

#### Exemplo 3: Buscar VIPs de uma Cidade
Embora o template atual não filtre por tags, você poderia combinar:
```json
POST ecommerce_clientes_v1/_search/template
{
  "id": "buscar_clientes_por_cidade",
  "params": {
    "cidade": "porto",
    "sort_field": "stats.total_pedidos",
    "sort_order": "desc"
  }
}
```

## 5. Template Simplificado: buscar_cidade_simples

### 5.1 Criação
```json
PUT _scripts/buscar_cidade_simples
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "term": {
          "endereco.cidade": "{{cidade}}"
        }
      }
    }
  }
}
```

### 5.2 Quando Usar?

**Use o template simplificado quando:**
- Não precisa de paginação customizada
- Não precisa de ordenação específica
- Quer simplicidade e velocidade
- Buscas rápidas e diretas

**Use o template completo quando:**
- Precisa controlar paginação
- Precisa ordenar por diferentes campos
- Quer flexibilidade total

## 6. Conceitos Importantes

### 6.1 Term Query vs Match Query

**Term Query** (usado nos templates):
```json
{ "term": { "endereco.cidade": "porto" } }
```
- Busca valor EXATO
- Usa o valor indexado sem análise
- Perfeito para keywords
- Case-sensitive (mas nosso normalizer resolve isso)

**Match Query** (alternativa):
```json
{ "match": { "nome": "João Silva" } }
```
- Analisa o texto antes de buscar
- Para campos text
- Tokeniza e busca termos

### 6.2 Por que "porto" em Minúsculas?

O campo cidade usa `lowercase_normalizer`:
```json
"normalizer": "lowercase_normalizer": {
  "filter": ["lowercase", "asciifolding"]
}
```

Isso significa que:
- "Porto" é armazenado como "porto"
- "PORTO" é armazenado como "porto"
- "Pôrto" é armazenado como "porto" (remove acentos)

**Portanto, sempre busque em minúsculas!**

### 6.3 Scaled Float

O campo `total_gasto` é `scaled_float` com `scaling_factor: 100`:
```json
"total_gasto": 124990  // Representa 1.249,90 EUR
```

Isso economiza espaço armazenando como inteiro multiplicado por 100.

## 7. Operações de Manutenção

### 7.1 Visualizar Template
```json
GET _scripts/buscar_clientes_por_cidade
```

### 7.2 Deletar Template
```json
DELETE _scripts/buscar_clientes_por_cidade
```

### 7.3 Atualizar Template
Use o mesmo comando PUT para sobrescrever:
```json
PUT _scripts/buscar_clientes_por_cidade
{
  // nova definição
}
```

### 7.4 Listar Todos os Templates
```json
GET _cluster/state/metadata?filter_path=**.stored_scripts
```

## 8. Boas Práticas

### 8.1 Nomenclatura
- Use nomes descritivos: `buscar_clientes_por_cidade` em vez de `template1`
- Prefixe por domínio: `clientes_`, `pedidos_`, etc.
- Use snake_case para consistência

### 8.2 Validação
Sempre teste o template antes de usar em produção:
```json
POST _render/template
{
  "source": "{ \"query\": { \"term\": { \"endereco.cidade\": \"{{cidade}}\" }}}",
  "params": {
    "cidade": "porto"
  }
}
```

### 8.3 Valores Padrão
Sempre defina valores padrão sensatos:
- from: 0
- size: 10 (evita retornar milhares de docs)
- sort: campo relevante ao contexto

### 8.4 Segurança
- Valide parâmetros no backend antes de enviar
- Não permita SQL/NoSQL injection
- Limite o `size` máximo (ex: não permitir size > 100)

## 9. Exercícios Práticos

### Exercício 1: Buscar Clientes VIP
Modifique o template para aceitar um parâmetro `tag`:
```json
"query": {
  "bool": {
    "must": [
      { "term": { "endereco.cidade": "{{cidade}}" }},
      { "term": { "tags": "{{tag}}" }}
    ]
  }
}
```

### Exercício 2: Filtro por Status
Adicione filtro opcional de status (ATIVO, SUSPENSO, INATIVO).

### Exercício 3: Range de Valores
Busque clientes por cidade que gastaram entre X e Y:
```json
{
  "range": {
    "stats.total_gasto": {
      "gte": "{{gasto_minimo}}",
      "lte": "{{gasto_maximo}}"
    }
  }
}
```

## 10. Comparação: Template vs Query Direta

### Query Direta
```json
POST ecommerce_clientes_v1/_search
{
  "query": {
    "term": { "endereco.cidade": "porto" }
  },
  "from": 0,
  "size": 10
}
```

**Desvantagens:**
- Repete código
- Difícil manutenção
- Sem versionamento central
- Mais verboso

### Template
```json
POST ecommerce_clientes_v1/_search/template
{
  "id": "buscar_clientes_por_cidade",
  "params": { "cidade": "porto" }
}
```

**Vantagens:**
- Código limpo
- Manutenção centralizada
- Versionamento fácil
- Reutilização

## 11. Conclusão

Search Templates são ferramentas poderosas para:
- Padronizar buscas complexas
- Melhorar manutenibilidade
- Aumentar segurança
- Facilitar evolução do código

O template `buscar_clientes_por_cidade` demonstra conceitos fundamentais:
- Parametrização com Mustache
- Valores padrão
- Flexibilidade de ordenação
- Paginação controlada
