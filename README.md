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
mkdir /opt/homeserver/stacks/uptime-kuma
mkdir /opt/homeserver/data/traefik
mkdir /opt/homeserver/data/portainer
mkdir /opt/homeserver/data/uptime-kuma
mkdir /opt/homeserver/data/backups
```

## Build Docker stack
```bash
build docker compose

touch traefik/letsencrypt/acme.json
chmod 600 traefik/letsencrypt/acme.json
docker compose pull
docker compose up -d
```
