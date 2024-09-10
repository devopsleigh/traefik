# Traefik

Traefic is a reverse proxy that provides certificates via Let's Encrypt and integrates with Docker.

## Getting started

1. Create a Docker network named proxy and start the Traefik container.
   ```bash
   docker network create -d bridge proxy
   ```
2. Review the compose.yml for context and a .env file to set secrets.
   ```config
   CF_API_EMAIL=
   CF_DNS_API_TOKEN=
   TRAEFIK_DASHBOARD_CREDS=
   MY_DOMAIN=
   LETSENCRYPT_EMAIL=
   ```

## Usage

### DNS

Each service will need a DNS CNAME to point to the Traefik host.

### Proxy for other hosts

Rename `custom.yml.example` to `custom.yml` and edit to configure services that this Traefik instance will proxy (another VM or a NAS, for example).

### Docker

There are a few configurations to be made to allow Traefik to proxy Docker containers.

1. On your DNS provider, create a CNAME for the new service and set the target to the host running Traefik.
2. In the compose.yml file for each container you want to proxy, append the block below.
   ```compose
       networks:
         proxy:
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.${SERVICE_NAME}.entrypoints=http"
         - "traefik.http.routers.${SERVICE_NAME}.rule=Host(`${SERVICE_NAME}.${MY_DOMAIN}`)"
         - "traefik.http.routers.${SERVICE_NAME}.middlewares=default-whitelist@file"
         - "traefik.http.middlewares.${SERVICE_NAME}-https-redirect.redirectscheme.scheme=https"
         - "traefik.http.routers.${SERVICE_NAME}.middlewares=${SERVICE_NAME}-https-redirect"
         - "traefik.http.routers.${SERVICE_NAME}-secure.entrypoints=https"
         - "traefik.http.routers.${SERVICE_NAME}-secure.rule=Host(`${SERVICE_NAME}.${MY_DOMAIN}`)"
         - "traefik.http.routers.${SERVICE_NAME}-secure.tls=true"
         - "traefik.http.routers.${SERVICE_NAME}-secure.service=${SERVICE_NAME}"
         - "traefik.http.services.${SERVICE_NAME}.loadbalancer.server.port=${SERVICE_INTERNAL_PORT}"
         - "traefik.docker.network=proxy"
   networks:
     proxy:
     external: true
   ```
3. Create a .env file to set the variables per host.
   ```config
   SERVICE_NAME=service1
   SERVICE_INTERNAL_PORT=the container's internal port where Traefik will redirect traffic
   MY_DOMAIN=my.domain
   ```
