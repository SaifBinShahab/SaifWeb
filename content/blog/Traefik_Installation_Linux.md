+++
title = "Installing Traefik on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Docker", "DevOps", "Homelab", "Server"]
+++
## Docker Setup
### Firewall limitations:

***Warning***

- Before you install Docker, make sure you consider the following security implications and firewall incompatibilities.

- If you use ufw or firewalld to manage firewall settings, be aware that when you expose container ports using Docker, these ports bypass your firewall rules. For more information, refer to Docker and ufw.
- Docker is only compatible with `iptables-nft` and `iptables-legacy`. Firewall rules created with `nft` are not supported on a system with Docker installed. Make sure that any firewall rulesets you use are created with `iptables` or `ip6tables`, and that you add them to the `DOCKER-USER` chain, see Packet filtering and firewalls.
&ensp;

### Intallation (Debian)
```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update apt database
sudo apt update

# Install docker and other goodies
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
&ensp;

Verify that the installation is successful by running the hello-world image:
```sh
sudo docker run hello-world
```
&ensp;

## Traefik
- Create a directory for traefik.

```sh
mkdir traefik
cd traefik
```
&ensp;

- Create a new bridge network to have docker's dns resolution. (Here named `docbridge`)
```sh
sudo docker network create docbridge
```
&ensp;

- Create a `docker-compose.yml` file with the following content:
```sh
vim docker-compose.yml
```
&ensp;

Put the following config:
```yml
---
services:

  traefik:
    image: "traefik:v3.2.1"
    container_name: "traefik"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./config/traefik.yml:/etc/traefik/traefik.yml:ro"
      - "./config/conf:/etc/traefik/conf:ro"
    networks:
      - docbridge
    restart: unless-stopped
  
networks:
  docbridge:
    external: true

```
&ensp;

- Create `traefik.yml` config file in the `config` directory for traefik docker container.
```sh
mkdir config

vim config/traefik.yml
```
&ensp;

Put the following config:
```yml
global:
  checkNewVersion: false
  sendAnonymousUsage: false
log:
  level: DEBUG
api:
  dashboard: true
  insecure: true
entryPoints:
  web:
    address: :80
  websecure:
    address: :443
providers:
  file:
    directory: "/etc/traefik/conf"
    watch: true

```
&ensp;

- Set Traefik Router rules
```sh
mkdir -p config/conf

vim config/conf/routes.yml
```
&ensp;

Put the following config
```yml
---
http:
  routers:
    nginx-http:
      rule: HostRegexp(`nginx`)
      service: service-1
      entryPoints: 
        - web
  services:
    service-1:
      loadBalancer:
        servers:
          - url: "http://nginx"

```
&ensp;

***Note: For "url" we have provided the nginx container name which is "nginx". That is because nginx in this set-up is running as a docker container. And the Docker bridge networks have internal DNS resolution built-in by Docker. That's why it is possible to point to nginx by simply using the container name, "nginx" but we have to provide the protocol here which is "http://". For the "url" We can also use "domain", "IP" and "IP:Port". For example, `http://127.0.0.1:8080` or `http://localhost:8000` or simply `https://example.com` and so on.***
&ensp;

## Setup demo docker containers (Nginx)
- Create a directory for nginx's docker container and a `docker-compose.yml` file.
```sh
mkdir nginx

vim nginx/docker-compose.yml
```
&ensp;

Put the following config inside the `docker-compose.yml`.
```yml
---
services:
  nginx:
    image: "nginx:latest"
    container_name: "nginx_demo"
    restart: unless-stopped
    networks:
      - docbridge

networks:
  docbridge:
    external: true

```
&ensp;

## Deploy docker containers
- We can deploy a docker container by going to the root directory of a container's docker files (where `docker-compose.yml` lives). We can run the container in detach mode by providing the `-d` flag. Otherwise, it will run in foreground and we can see the logs live.
```sh
cd nginx

sudo docker compose up -d
```
&ensp;

- Deploy the Traefik container and put it to detach mode:
```sh
cd traefik

sudo docker compose up -d
```
&ensp;

- If a docker container is running on foreground (without `-d`). Then we can put the container `down` with:
```sh
sudo docker compose down <container_name|container_ID|etc>

sudo docker compose down nginx
```

- Or, if a container is running in the background (with `-d`). Then we can put the container `down` by simply pressing `CTRL+C`.
&ensp;

Done!
