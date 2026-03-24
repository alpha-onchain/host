# host
## Setup with Raspberry imager
## Install Docker + Compose
```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```
## Build folder structure

```bash
mkdir /opt/homeserver/stacks/traefik
mkdir /opt/homeserver/stacks/portainer
mkdir /opt/homeserver/stacks/ops
mkdir /opt/homeserver/data/traefik
mkdir /opt/homeserver/data/portainer
mkdir /opt/homeserver/data/backups
```

## Build Docker stack
```bash
build docker compose

mkdir -p traefik/dynamic traefik/letsencrypt portainer/data uptime-kuma/data
touch traefik/letsencrypt/acme.json
chmod 600 traefik/letsencrypt/acme.json
docker compose pull
docker compose up -d
```
