[![ghcr.io release](https://img.shields.io/github/v/release/santimar/traefik-home?label=latest%20version&style=for-the-badge)](https://github.com/santimar/traefik-home/pkgs/container/traefik-home/versions)

# Traefik Home
![preview](/doc/preview.jpg)

This tool will create an homepage for a quick access to the domains where containers are exposed via [traefik proxy](https://traefik.io/traefik/) using v2 configuration. (for old v1 configuration see [here](https://github.com/lobre/traefik-home))

Domains are automatically retrieved reading traefik labels and only http(s) routers are supported.

**Note**: currently only ARMv7 architecture is supported

## Why this tool

Treafik is a reverse proxy that reads label on the docker compose file and automatically set up itself, so that you can access a service on a container with the specified hostname.
Even though Traefik provides a dashboard that allow you to see services that are proxied, you still have to remember (or save somewhere) all the hostnames in order to access your services.
This tool creates an Home page where all hostnames all listed, for a easy access.

It uses [docker-gen](https://github.com/jwilder/docker-gen) to monitor docker configuration changes and render an web page that will be served using `nginx`
This way changes are immedialtely reflected whenever something gets updated.

## Usage

For a quick preview you can just run the image with the following docker run command:

```
docker run --name traefik-home \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    --label traefik.enable=true \
    --label traefik.http.routers.traefik-home.rule="Host(`home.example.com`)" \
    --label traefik.http.services.traefik-home.loadbalancer.server.port="80" \
    ghcr.io/santimar/traefik-home:latest
```

Wait for the service to be online, then go to `home.example.com` and enjoy the view.

For long runs however, the docker-compose file is a better approach.

## Docker compose
```yaml
version: '3'

services:
  traefik-home:
    image: ghcr.io/santimar/traefik-home:latest  # or use a specific tag version
    container_name: traefik-home
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    labels:
      - traefik.enable="true"
      - traefik.http.routers.traefik-home.rule="Host(`home.example.com`)"
      - traefik.http.services.traefik-home.loadbalancer.server.port="80"

  other-service:
    image: ...
    labels:
      # see below for more info about exposed containers configuration
      - traefik-home.icon: "https://url/of/an/icon.png"
      - traefik-home.alias: "Alias"
    
```

## Labels to configure Home

The `traefik-home` container can be configured using the following optional labels.

| Label  | Description | Default
| --- | --- | --- |
| traefik-home.show-footer | Wheter to show footer on the home page | true |
| traefik-home.show-status-dot | Wheter to show green/red status dot near the container name | true |

## Labels to configure containers

Home will use the following `traefik` labels to generate the HTML page.

| Label  | Description |
| --- | --- |
| traefik.http.routers.\<service\>.rule | Domain used by Home to generate the link to the service |
| traefik.http.routers.\<service\>.entrypoints | Only `web` or `websecure` entrypoints are shown |
| traefik.home | Only explicitly enabled container are shown on the homepage |

On each exposed container, the following optional labels can be added to provide a personalized configuration.

| Label  | Description |
| --- | --- |
| traefik-home.hide="true" | Do not show this container in the home page |
| traefik-home.icon="https://url/of/icon"  | URL of an image that will be used as icon for the container. If this label is not used, a icon with the container's initials will be used |
| traefik-home.alias="alias"  | If used, the alias will be shown instead of the container name |

## Update

When a new release is available, just:
```
docker pull ghcr.io/santimar/traefik-home:latest
```

and then 

```
docker-compose up -d traefik-home 
```
