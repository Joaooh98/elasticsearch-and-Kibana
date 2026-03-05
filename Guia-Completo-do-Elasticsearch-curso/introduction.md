# Aula 1 - Elasticsearch vs OpenSearch

**Data:** 26/01/2025  
**Status:** Concluído

## Contexto Histórico

**2015**
- Outubro: Amazon Elasticsearch Service lançado

**2019**
- Março: Open Distro for Elasticsearch lançado
- Maio: Elastic torna features de segurança gratuitas

**2021**
- Janeiro: Elastic anuncia mudanças de licença (de Apache 2.0 para SSPL/Elastic License)
- Janeiro: AWS anuncia fork do Elasticsearch e Kibana
- Abril: AWS anuncia OpenSearch
- Setembro: Elastic processa AWS por trademark infringement e false advertising

**2022**
- Fevereiro: Elastic e AWS chegam a acordo no processo

## Diferenças de Licença

**Elasticsearch (após v7.10)**
- Apache License 2.0 (versões 6.0 até 7.10)
- SSPL & Elastic License (v7.11+, v8.0+)
- Adicional: features proprietárias (Security, Monitoring, SQL, Graph, Alerting, Machine Learning, Reporting)

**OpenSearch**
- Apache License 2.0 (mantém open source)
- Fork feito a partir do Elasticsearch 7.10

## Core APIs Compartilhadas

Ambos mantêm compatibilidade nas APIs essenciais:
- Search Queries
- Indexing Documents
- Aggregations
- Index Management
- Basic Security

## Features Exclusivas

**Elasticsearch**
- SIEM
- Observability
- Enterprise Search
- Machine Learning
- (features proprietárias da Elastic License)

**OpenSearch**
- Security Analytics
- Observability
- Machine Learning
- (mantém foco open source)

## Managed Services Disponíveis

**Elasticsearch**
- Elastic Cloud (oficial)
- Bonsai

**OpenSearch**
- Amazon Web Services (AWS OpenSearch)
- DigitalOcean
- Aiven
- Instaclustr
- Exoscale
- Alibaba Cloud

**Ambos**
- Google Cloud
- Microsoft Azure

## Conexão com Trabalho Atual

No portal-unico-conector-ces-api usamos Elasticsearch com Elastic License. Importante saber:
- Estamos na família 8.x (provavelmente 8.0+)
- Temos acesso às features de segurança básicas (gratuitas desde 2019)
- Não temos acesso às features proprietárias avançadas (ML, SIEM, etc) a menos que projeto tenha licença comercial

## Pontos de Atenção

1. **Migração é possível mas trabalhosa**: OpenSearch e Elasticsearch divergem mais a cada release
2. **APIs core são compatíveis**: queries, indexação, aggregations funcionam igual nos dois
3. **Decisão de licença da Elastic afetou o ecossistema**: criou fork permanente
4. **AWS oferece OpenSearch como alternativa open source**: importante para quem quer evitar vendor lock-in

## Para Investigar

- Quais features proprietárias o projeto atual usa (se usa)?
- Há planos de migração para OpenSearch ou continuamos com Elasticsearch?
- Verificar versão exata do Elasticsearch que usamos no portal-unico

## Resumo em 3 Linhas

Elasticsearch mudou de licença em 2021 (Apache 2.0 para SSPL/Elastic License), causando fork do AWS que criou OpenSearch mantendo Apache 2.0. Ambos compartilham core APIs mas divergem em features avançadas. Nossa stack usa Elasticsearch com acesso às features básicas gratuitas.