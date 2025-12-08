# docker-compose.yml explicado

Este repositório usa Docker Compose para subir um Elasticsearch e um Kibana recentes (8.19.4, linha LTS 8.x) em modo de estudo/local. Abaixo explico cada campo e o que você mudaria em produção.

## Variáveis de ambiente e senhas
- `ELASTIC_PASSWORD: ${ELASTIC_PASSWORD:-changeme}`: senha do superusuário `elastic`. O Compose lê do seu shell; se não houver, cai no fallback `changeme` (não use em produção).
- `KIBANA_SYSTEM_PASSWORD: ${KIBANA_SYSTEM_PASSWORD:-changeme}`: senha do usuário interno `kibana_system`, que o Kibana usa para falar com o Elasticsearch. Mesmo esquema de fallback.
- Dica: exporte as variáveis antes de subir ou use um `.env` (não comitar) para evitar o fallback explícito no YAML.

## Serviço: elasticsearch
- `image: docker.elastic.co/elasticsearch/elasticsearch:8.19.4`: imagem oficial, linha LTS 8.19.x.
- `container_name: elasticsearch`: nome fixo do contêiner.
- `environment`:
  - `node.name`: identificador do nó (es01).
  - `cluster.name`: nome do cluster (elasticsearch).
  - `discovery.type: single-node`: força cluster de um nó; remova ao subir múltiplos nós.
  - `bootstrap.memory_lock: "true"`: pede para travar a memória e evitar swap (melhora estabilidade).
  - `xpack.security.enabled: "true"`: segurança/autenticação ligada (padrão nas versões novas).
  - `xpack.security.http.ssl.enabled: "false"` e `xpack.security.transport.ssl.enabled: "false"`: TLS desligado para facilitar uso local. Em produção, habilite e forneça certificados.
  - `ES_JAVA_OPTS: -Xmx1g -Xms1g`: heap da JVM ajustada para 1 GB. Ajuste conforme RAM.
  - `ELASTIC_PASSWORD`, `KIBANA_SYSTEM_PASSWORD`: senhas já descritas.
- `ulimits.memlock (soft/hard: -1)`: permite travar memória (requer suporte do host). Sem isso, o memlock pode não funcionar.
- `ports`:
  - `9200:9200`: API HTTP.
  - `9300:9300`: tráfego interno do cluster (necessário se tiver mais nós ou quer permitir conexões de transporte).
- `volumes: esdata:/usr/share/elasticsearch/data`: volume nomeado para persistir os dados, evitando problemas de permissão comuns em binds.
- `networks: elasticnet`: rede bridge compartilhada com o Kibana.

## Serviço: kibana
- `image: docker.elastic.co/kibana/kibana:8.19.4`: imagem oficial, mesma versão do ES.
- `container_name: kibana`: nome fixo do contêiner.
- `ports`:
  - `5601:5601`: UI do Kibana.
  - `9600:9600`: não é usado no dia a dia do Kibana; pode remover se não precisar expor porta extra.
- `environment`:
  - `SERVERNAME`: nome exibido e usado pelo Kibana (kibana).
  - `ELASTICSEARCH_HOSTS`: URL do ES dentro da rede Docker.
  - `ELASTICSEARCH_USERNAME: kibana_system`: usuário interno que o Kibana usa para autenticar no ES.
  - `ELASTICSEARCH_PASSWORD`: senha do `kibana_system` (mesma variável de cima).
  - `ES_JAVA_OPTS: -Xmx1g -Xms1g`: heap do Kibana (Node.js). Ajuste conforme RAM.
- `depends_on: elasticsearch`: garante que o serviço do ES seja iniciado antes do Kibana.
- `networks: elasticnet`: rede compartilhada.

## Seções globais
- `volumes: esdata: {}`: declara o volume nomeado usado pelo Elasticsearch.
- `networks: elasticnet: {}`: declara a rede bridge.

## Produção vs. estudo (o que mudar)
- Senhas: use valores fortes; nunca deixe fallbacks `changeme`. Guarde em `.env` fora do controle de versão ou use secrets/variáveis de ambiente do orquestrador.
- TLS: ligue `xpack.security.http.ssl.enabled` e `xpack.security.transport.ssl.enabled`, forneça certificados válidos e ajuste o Kibana para usar HTTPS.
- Multi-nó: remova `discovery.type: single-node`, defina `cluster.initial_master_nodes` e `discovery.seed_hosts`, crie serviços es02, es03 com volumes separados e portas de transporte acessíveis.
- Recursos: dimensione `ES_JAVA_OPTS` e alocação de CPU/RAM conforme seu host; memlock requer permissão do host.
- Portas: só exponha as necessárias; se não precisar de 9300 (ou 9600 do Kibana), remova o mapeamento.
- Dados: em produção, prefira volumes gerenciados (ou binds com permissões e backups definidos). O volume `esdata` já persiste o estado entre `up` e `down`.
- Observabilidade: configure logs e métricas (ex.: Filebeat/Metricbeat) se for além de estudo.
