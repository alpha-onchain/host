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

## Notes
- This assumes your DNS records already point `traefik.example.com`, `portainer.example.com`, `status.example.com`, and `logs.example.com` to your Raspberry Pi, because Let’s Encrypt HTTP challenge on Traefik needs ports `80/443` reachable from the internet. Traefik’s ACME HTTP challenge and Docker-label routing are both officially supported.
- `exposedByDefault=false` is important so only explicitly labeled containers are published through Traefik. That is a good default for production.
- Dozzle and Portainer both need Docker socket access for their core purpose; that is normal for these tools, but it is also why protecting them behind HTTPS and auth is important. Dozzle’s docs explicitly mention proxy-based authentication support.

