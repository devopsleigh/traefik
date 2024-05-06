# Traefik

Traefic is a reverse proxy that provides certificates via Let's Encrypt and integrates with Docker.

## Usage

A few configurations have to be made to allow Traefik to proxy Docker containers.

1. On your DNS provider, create a CNAME for the new service and set the target to the host running Traefik.
2. Create a Docker network named proxy and start the Traefik container.
   ```bash
   docker network create -d bridge proxy
   docker compose up -d --force-crecreate
   ```
3. In the compose.yml file for each container you want to proxy, append the block below.
   ```compose
       networks:
         proxy:
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.${SERVICE_NAME}.entrypoints=http"
         - "traefik.http.routers.${SERVICE_NAME}.rule=Host(`${SERVICE_NAME}.${HOMEPAGE_VAR_DOMAIN}`)"
         - "traefik.http.routers.${SERVICE_NAME}.middlewares=default-whitelist@file"
         - "traefik.http.middlewares.${SERVICE_NAME}-https-redirect.redirectscheme.scheme=https"
         - "traefik.http.routers.${SERVICE_NAME}.middlewares=${SERVICE_NAME}-https-redirect"
         - "traefik.http.routers.${SERVICE_NAME}-secure.entrypoints=https"
         - "traefik.http.routers.${SERVICE_NAME}-secure.rule=Host(`${SERVICE_NAME}.${HOMEPAGE_VAR_DOMAIN}`)"
         - "traefik.http.routers.${SERVICE_NAME}-secure.tls=true"
         - "traefik.http.routers.${SERVICE_NAME}-secure.service=${SERVICE_NAME}"
         - "traefik.http.services.${SERVICE_NAME}.loadbalancer.server.port=${SERVICE_INTERNAL_PORT}"
         - "traefik.docker.network=proxy"
   networks:
     proxy:
     external: true
   ```
4. Create a .env file to set the variables per host.
   ```config
   SERVICE_NAME=service1
   SERVICE_INTERNAL_PORT=the container's internal port where Traefik will redirect traffic
   ```
