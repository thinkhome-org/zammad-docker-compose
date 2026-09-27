# ThinkHome: Zammad with existing Cloudflare and Elasticsearch stacks

Deploy this repository in Portainer on the same Docker host as the running Cloudflare and Elasticsearch stacks.

Compose path: `docker-compose.yml`

Additional paths, in order:

1. `scenarios/add-external-network-to-nginx.yml`
2. `scenarios/disable-elasticsearch-service.yml`
3. `scenarios/thinkhome-external-elasticsearch.yml`

The existing stacks must have created bridge networks named `cloudflare` and `elasticsearch`. The nginx scenario attaches the bundled Zammad nginx to `cloudflare` and removes its published host port; it does not add another proxy. The Elasticsearch scenario disables the bundled search service. The ThinkHome scenario attaches Zammad application processes to the external search network.

Set these Environment variables in Portainer, alongside your existing PostgreSQL and other settings:

```env
ZAMMAD_NGINX_EXTERNAL_NETWORK=cloudflare
NGINX_SERVER_SCHEME=https
ZAMMAD_HTTP_TYPE=https
ZAMMAD_FQDN=servis.thinkhome.org
ELASTICSEARCH_SCHEMA=https
ELASTICSEARCH_HOST=elasticsearch
ELASTICSEARCH_PORT=9200
ELASTICSEARCH_USER=elastic
ELASTICSEARCH_PASS=<actual Elasticsearch password>
ELASTICSEARCH_NAMESPACE=zammad
```

The existing Elasticsearch uses HTTPS with a custom CA. On a new Zammad installation, first deploy with `ELASTICSEARCH_ENABLED=false`. Copy only the public certificate from the Elasticsearch container at `/usr/share/elasticsearch/config/certs/ca/ca.crt`; import it in Zammad at Admin → Settings → Security → SSL Certificates. Then set `ELASTICSEARCH_ENABLED=true` and redeploy Zammad. Do not copy private keys or disable TLS verification. For an existing instance, start indexing from Zammad administration after connection succeeds.

Set the existing Cloudflare Tunnel public hostname `servis.thinkhome.org` to HTTP origin `http://zammad-nginx:8080`. Keep the tunnel token in the Cloudflare stack. Do not commit passwords to this repository.

Confirm the actual Elasticsearch password on the initialized cluster; changing `ELASTIC_PASSWORD` in its Compose environment alone does not reset an existing password.
