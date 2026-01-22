Componentes da Arquitetura
1. Fontes de Dados (Esquerda)
Página Web (navegador):

Representa logs de aplicações web
Dados de acesso, erros, eventos

Banco de Dados (cilindro azul):

Logs de banco de dados
Queries, transações, erros
Métricas de performance

2. Logstash (Centro - servidor com ícones)
O que faz:

Coleta dados de múltiplas fontes
Processa e transforma os dados
Enriquece informações
Filtra e organiza

Ícones visíveis:

Cadeado: segurança e criptografia
Arquivo: parsing de logs

ruby# 

```
Exemplo de pipeline Logstash
input {
  file {
    path => "/var/log/app.log"
  }
  jdbc {
    jdbc_connection_string => "jdbc:postgresql://localhost/db"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```
### 3. Elasticsearch (Centro-direita - logo amarelo/preto/azul)

**O que faz:**
- **Armazena** os dados processados
- **Indexa** para busca rápida
- **Pesquisa** em tempo real
- Motor de busca distribuído

**Características:**
- Armazena dados em formato JSON
- Busca full-text super rápida
- Escalável horizontalmente

### 4. Kibana (Direita - logo rosa/preto/azul)

**O que faz:**
- **Visualiza** os dados do Elasticsearch
- **Dashboards** interativos
- **Análises** em tempo real
- Interface web de gerenciamento

**Funcionalidades:**
- Gráficos e tabelas
- Mapas
- Alertas
- Machine Learning (com licença)

### 5. Beats (Topo - logos coloridos)

**Filebeat (amarelo/preto/azul):**
- Coleta logs de arquivos
- Leve e eficiente
- Envia para Logstash ou Elasticsearch

**Metricbeat (laranja/preto/azul):**
- Coleta métricas do sistema
- CPU, memória, disco, rede
- Métricas de serviços

## Fluxo de Dados

```
┌─────────────┐
│ Fontes      │
│ - Web       │
│ - Database  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Beats       │ ◄─── Coleta leve de dados
│ - Filebeat  │
│ - Metricbeat│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Logstash    │ ◄─── Processa e transforma
│ - Parse     │
│ - Filter    │
│ - Enrich    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Elasticsearch│ ◄─── Armazena e indexa
│ - Index     │
│ - Search    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Kibana     │ ◄─── Visualiza
│ - Dashboard │
│ - Analytics │
└─────────────┘
```

Exemplo Prático com E-commerce
Aplicando ao seu contexto de clientes:
yaml# Beats coleta eventos
Cliente faz login → Filebeat captura log

# Logstash processa
```
filter {
  if [event] == "login" {
    mutate {
      add_field => { "ultimo_login" => "%{@timestamp}" }
    }
  }
}
```
# Elasticsearch armazena
```
POST /ecommerce_eventos/_doc
{
  "cliente_id": "C0001",
  "evento": "login",
  "timestamp": "2026-01-22T10:00:00Z"
}
```
# Kibana visualiza
Dashboard mostra:
- Logins por hora
- Clientes ativos
- Mapa de acessos por cidade



## Alternativas de Arquitetura

### Arquitetura Simples

Beats → Elasticsearch → Kibana
(Sem Logstash - processamento no Beats)
### Arquitetura Completa
Múltiplas Fontes → Beats → Logstash → Elasticsearch → Kibana
(Com processamento pesado no Logstash)

### Arquitetura com Kafka (Alta Volume)
Beats → Kafka → Logstash → Elasticsearch → Kibana
(Buffer para alto volume de dados)
Casos de Uso Reais
1. Monitoramento de Aplicação Web

Filebeat coleta logs do Apache/Nginx
Logstash parseia e enriquece
Elasticsearch armazena
Kibana mostra dashboard de erros

2. Métricas de Infraestrutura

Metricbeat coleta CPU, memória
Envia direto para Elasticsearch
Kibana alerta quando CPU > 80%

3. Análise de Clientes (seu caso)

Aplicação envia eventos para Logstash
Logstash processa e envia para Elasticsearch
Kibana mostra comportamento de compra

Resumo
A imagem mostra:

Pipeline completo de dados do Elastic Stack
Fluxo bidirecional entre componentes
Diferentes fontes de dados (web, database)
Processamento intermediário (Logstash)
Armazenamento (Elasticsearch)
Visualização (Kibana)
Coleta leve (Beats)

Propósito:
Centralizar logs, métricas e dados de diferentes fontes em um único lugar para análise, monitoramento e visualização em tempo real.