```json
PUT ecommerce_clientes_v1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "normalizer": {
        "lowercase_normalizer": {
          "type": "custom",
          "filter": ["lowercase", "asciifolding"]
        }
      },
      "analyzer": {
        "pt_search": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  }
}


GET ecommerce_clientes_v1/_mapping

PUT ecommerce_clientes_v1/_mapping
{
  "dynamic": "strict",
  "properties": {
    "cliente_id": { "type": "keyword" },

    "nome": {
      "type": "text",
      "analyzer": "pt_search",
      "fields": {
        "keyword": {
          "type": "keyword",
          "normalizer": "lowercase_normalizer"
        }
      }
    },

    "email": {
      "type": "keyword",
      "normalizer": "lowercase_normalizer"
    },

    "telefone": { "type": "keyword" },

    "documento": {
      "properties": {
        "tipo": { "type": "keyword" },
        "numero": { "type": "keyword" }
      }
    },

    "endereco": {
      "properties": {
        "rua": { "type": "text", "analyzer": "pt_search" },
        "numero": { "type": "keyword" },
        "bairro": { "type": "text", "analyzer": "pt_search" },
        "cidade": { "type": "keyword", "normalizer": "lowercase_normalizer" },
        "distrito": { "type": "keyword", "normalizer": "lowercase_normalizer" },
        "codigo_postal": { "type": "keyword" },
        "pais": { "type": "keyword" }
      }
    },

    "marketing_opt_in": { "type": "boolean" },
    "status": { "type": "keyword" },

    "datas": {
      "properties": {
        "criado_em": { "type": "date" },
        "atualizado_em": { "type": "date" },
        "ultimo_login": { "type": "date" }
      }
    },

    "stats": {
      "properties": {
        "total_pedidos": { "type": "integer" },
        "total_gasto": { "type": "scaled_float", "scaling_factor": 100 },
        "ticket_medio": { "type": "scaled_float", "scaling_factor": 100 }
      }
    },

    "tags": {
      "type": "keyword",
      "normalizer": "lowercase_normalizer"
    }
  }
}


POST ecommerce_clientes_v1/_bulk
{"index":{"_id":"C0001"}}
{"cliente_id":"C0001","nome":"João Carlos Paiva","email":"joao.paiva@email.com","telefone":"+351912345678","documento":{"tipo":"NIF","numero":"245678901"},"endereco":{"rua":"Rua da Sofia","numero":"120","bairro":"Baixa","cidade":"Coimbra","distrito":"Coimbra","codigo_postal":"3000-389","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-09-10T10:15:00Z","atualizado_em":"2026-01-18T09:00:00Z","ultimo_login":"2026-01-19T20:10:00Z"},"stats":{"total_pedidos":12,"total_gasto":124990,"ticket_medio":10416},"tags":["vip","newsletter"]}
{"index":{"_id":"C0002"}}
{"cliente_id":"C0002","nome":"Ana Martins","email":"ana.martins@email.com","telefone":"+351913111222","documento":{"tipo":"NIF","numero":"289001234"},"endereco":{"rua":"Avenida da Liberdade","numero":"45","bairro":"Santo António","cidade":"Lisboa","distrito":"Lisboa","codigo_postal":"1250-096","pais":"PT"},"marketing_opt_in":false,"status":"ATIVO","datas":{"criado_em":"2025-08-01T12:10:00Z","atualizado_em":"2026-01-12T15:40:00Z","ultimo_login":"2026-01-15T08:22:00Z"},"stats":{"total_pedidos":3,"total_gasto":24990,"ticket_medio":8330},"tags":["new"]}
{"index":{"_id":"C0003"}}
{"cliente_id":"C0003","nome":"Bruno Silva","email":"bruno.silva@email.com","telefone":"+351914333444","documento":{"tipo":"NIF","numero":"201334455"},"endereco":{"rua":"Rua de Santa Catarina","numero":"300","bairro":"Bonfim","cidade":"Porto","distrito":"Porto","codigo_postal":"4000-447","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-10-21T09:30:00Z","atualizado_em":"2026-01-05T10:00:00Z","ultimo_login":"2026-01-20T09:01:00Z"},"stats":{"total_pedidos":7,"total_gasto":78900,"ticket_medio":11271},"tags":["newsletter"]}
{"index":{"_id":"C0004"}}
{"cliente_id":"C0004","nome":"Carla Sousa","email":"carla.sousa@email.com","telefone":"+351915555666","documento":{"tipo":"NIF","numero":"237556677"},"endereco":{"rua":"Rua Miguel Bombarda","numero":"18","bairro":"Cedofeita","cidade":"Porto","distrito":"Porto","codigo_postal":"4050-377","pais":"PT"},"marketing_opt_in":true,"status":"SUSPENSO","datas":{"criado_em":"2025-06-02T18:20:00Z","atualizado_em":"2025-12-30T11:12:00Z","ultimo_login":"2025-12-15T14:30:00Z"},"stats":{"total_pedidos":1,"total_gasto":4990,"ticket_medio":4990},"tags":["chargeback"]}
{"index":{"_id":"C0005"}}
{"cliente_id":"C0005","nome":"Daniel Oliveira","email":"daniel.oliveira@email.com","telefone":"+351916777888","documento":{"tipo":"NIF","numero":"298776655"},"endereco":{"rua":"Rua de São Bento","numero":"9","bairro":"Estrela","cidade":"Lisboa","distrito":"Lisboa","codigo_postal":"1200-821","pais":"PT"},"marketing_opt_in":false,"status":"ATIVO","datas":{"criado_em":"2025-11-03T07:05:00Z","atualizado_em":"2026-01-02T12:00:00Z","ultimo_login":"2026-01-02T12:01:00Z"},"stats":{"total_pedidos":0,"total_gasto":0,"ticket_medio":0},"tags":["lead"]}
{"index":{"_id":"C0006"}}
{"cliente_id":"C0006","nome":"Eduarda Fernandes","email":"eduarda.fernandes@email.com","telefone":"+351917999000","documento":{"tipo":"NIF","numero":"223344556"},"endereco":{"rua":"Rua do Almada","numero":"210","bairro":"Centro","cidade":"Porto","distrito":"Porto","codigo_postal":"4050-038","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-07-19T16:45:00Z","atualizado_em":"2026-01-19T22:10:00Z","ultimo_login":"2026-01-19T22:10:00Z"},"stats":{"total_pedidos":22,"total_gasto":209990,"ticket_medio":9545},"tags":["vip","recorrente"]}
{"index":{"_id":"C0007"}}
{"cliente_id":"C0007","nome":"Fábio Costa","email":"fabio.costa@email.com","telefone":"+351918121314","documento":{"tipo":"NIF","numero":"210987654"},"endereco":{"rua":"Avenida 25 de Abril","numero":"500","bairro":"Centro","cidade":"Braga","distrito":"Braga","codigo_postal":"4710-123","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-12-01T20:00:00Z","atualizado_em":"2026-01-10T09:20:00Z","ultimo_login":"2026-01-11T10:11:00Z"},"stats":{"total_pedidos":4,"total_gasto":45990,"ticket_medio":11497},"tags":["newsletter"]}
{"index":{"_id":"C0008"}}
{"cliente_id":"C0008","nome":"Gabriela Almeida","email":"gabriela.almeida@email.com","telefone":"+351919151617","documento":{"tipo":"NIF","numero":"256789123"},"endereco":{"rua":"Rua de Camões","numero":"77","bairro":"Santo Ildefonso","cidade":"Porto","distrito":"Porto","codigo_postal":"4000-144","pais":"PT"},"marketing_opt_in":false,"status":"INATIVO","datas":{"criado_em":"2025-03-12T11:11:00Z","atualizado_em":"2025-09-30T11:11:00Z","ultimo_login":"2025-08-20T17:00:00Z"},"stats":{"total_pedidos":9,"total_gasto":9990,"ticket_medio":1110},"tags":["low_spend"]}
{"index":{"_id":"C0009"}}
{"cliente_id":"C0009","nome":"Hugo Ribeiro","email":"hugo.ribeiro@email.com","telefone":"+351910202030","documento":{"tipo":"NIF","numero":"299887766"},"endereco":{"rua":"Rua Direita","numero":"10","bairro":"Sé","cidade":"Viseu","distrito":"Viseu","codigo_postal":"3500-112","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-09-25T08:00:00Z","atualizado_em":"2026-01-16T19:00:00Z","ultimo_login":"2026-01-17T09:45:00Z"},"stats":{"total_pedidos":2,"total_gasto":15990,"ticket_medio":7995},"tags":["new"]}
{"index":{"_id":"C0010"}}
{"cliente_id":"C0010","nome":"Inês Rodrigues","email":"ines.rodrigues@email.com","telefone":"+351911223344","documento":{"tipo":"NIF","numero":"234560987"},"endereco":{"rua":"Rua do Ouro","numero":"1","bairro":"Baixa","cidade":"Lisboa","distrito":"Lisboa","codigo_postal":"1100-061","pais":"PT"},"marketing_opt_in":true,"status":"ATIVO","datas":{"criado_em":"2025-05-05T05:05:00Z","atualizado_em":"2026-01-20T12:00:00Z","ultimo_login":"2026-01-20T12:00:00Z"},"stats":{"total_pedidos":15,"total_gasto":149990,"ticket_medio":9999},"tags":["vip","upsell"]}


POST ecommerce_clientes_v1/_refresh


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

POST ecommerce_clientes_v1/_search/template
{
  "id": "buscar_clientes_por_cidade",
  "params": {
    "cidade": "porto"
  }
}

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