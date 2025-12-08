Use this repo’s `docker-compose.yml` to run the latest Elastic Stack LTS 8.x (Elasticsearch + Kibana 8.19.4).

Prereqs
- [Docker](https://docs.docker.com/desktop/linux/) or [Docker Desktop](https://docs.docker.com/desktop/)
- [Docker Compose](https://docs.docker.com/compose/)

One-time setup (set your own strong passwords)
```sh
export ELASTIC_PASSWORD="changeme"          # password for the elastic superuser
export KIBANA_SYSTEM_PASSWORD="changeme"    # password Kibana uses to talk to Elasticsearch
```

Start the stack
```sh
docker-compose up -d
```

URLs (auth required)
- http://localhost:5601/ (Kibana, user: `elastic`, pass: `$ELASTIC_PASSWORD`)
- http://localhost:9200/ (Elasticsearch API, same user/pass)

Logs
```sh
docker-compose logs elasticsearch
docker-compose logs kibana
```

Stop / remove
```sh
docker-compose stop        # stop containers
docker-compose down        # stop and remove containers + network (keeps data volume)
```

Notes
- Security stays enabled; TLS is off for local dev. Do not reuse these passwords in production.
- If you truly want to disable auth for a throwaway dev run, set `xpack.security.enabled: "false"` in `docker-compose.yml` (not recommended for anything else).
