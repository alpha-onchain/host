REPLACE ALL example.com
set .env for prod
setup ssh access



# Host



## Setup with Raspberry imager
## Target
- Resources
  - Traefik as reverse proxy
  - Authelia as SSO / access gateway
  - Portainer
  - Uptime Kuma
  - Dozzle
  - protection through Authelia forward auth

- Production-oriented but still realistic for Raspberry Pi
  - persistent volumes
  - no insecure Traefik API
  - Docker provider with exposedByDefault=false
  - TLS via Let’s Encrypt
  - Authelia with file-based users for simplicity

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

## Run containers

```bash
docker compose -f stacks/proxy/docker-compose.yml up -d
docker compose -f stacks/portainer/docker-compose.yml up -d
docker compose -f stacks/ops/docker-compose.yml up -d
```


## First start

### Test locally
- Run `docker compose up -d`
- Check http://localhost:9000 (Portainer)
- Check http://localhost:3001 (Uptime Kuma)
- Check proxy routing

### On server
```bash
build docker compose

touch traefik/letsencrypt/acme.json
chmod 600 traefik/letsencrypt/acme.json
docker compose pull
docker compose up -d
```

## Notes

### Pense-bête
- This assumes your DNS records already point `traefik.example.com`, `portainer.example.com`, `status.example.com`, and `logs.example.com` to your Raspberry Pi, because Let’s Encrypt HTTP challenge on Traefik needs ports `80/443` reachable from the internet. Traefik’s ACME HTTP challenge and Docker-label routing are both officially supported.
- `exposedByDefault=false` is important so only explicitly labeled containers are published through Traefik. That is a good default for production.
- Dozzle and Portainer both need Docker socket access for their core purpose; that is normal for these tools, but it is also why protecting them behind HTTPS and auth is important. Dozzle’s docs explicitly mention proxy-based authentication support.
- Password admin for Authelia was generated with `docker run --rm authelia/authelia:latest authelia crypto hash generate argon2 --password 'CHANGE_ME'`
- Long random secrets & jwt was generated with `openssl rand -hex 64`
- backup strategy

<blockquote class="danger">⚠️ Do not publish extra ports for Portainer, Dozzle or any app.</blockquote>
<blockquote class="danger">⚠️ Always use images that suport arm64 (and not only amd64)</blockquote>
<blockquote class="danger">⚠️ Only Traefik should expose 80 & 443
</blockquote>

If we build custom apps, we must 
- `docker buildx create --use`
- `docker buildx build --platform linux/amd64,linux/arm64 -t newapp .`

### Peut-être plus tard
- Put this stack behind your router with only 80/443 forwarded.
- Keep Dozzle and Traefik dashboard behind basic auth at minimum.
- Change `policy: one_factor` to `policy: two_factor` and then enable TOTP/WebAuthn in Authelia
- put Portainer and Dozzle behind two_factor
- keep Uptime Kuma in one_factor
- add CrowdSec later if you expose this publicly
- After validation, pin versions more strictly:
  -  `traefik:v3.3.x`
  -  `portainer/portainer-ce:lts`
  -  `louislam/uptime-kuma:2.x`
  -  `amir20/dozzle:x.y.z`
   