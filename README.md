REPLACE ALL example.com
set .env for prod
setup ssh access



# Host

## Generate SSH key to pull code from Github

- Generate key : `ssh-keygen -t ed25519 -C "me@nadda.net"`
- Check key (private & public) : `ls ~/.ssh`
- Start ssh agent to use this key at each git request : `eval "$(ssh-agent -s)"`
- Add key to ssh agent `ssh-add ~/.ssh/<<id_XXXXXXX>>`
- Copy public key `cat ~/.ssh/<<id_XXXXXXX>>.pub`
- Copy public key to github https://github.com/settings/keys
  1. Click "New SSH key"
  1. Title: e.g. raspberry-pi or laptop
  1. Paste your key
  1. Click Add SSH key

## Setup with Raspberry imager
## Target
- Resources
  - Traefik as reverse proxy
  - Authentik as SSO / access gateway
  - Portainer
  - Uptime Kuma
  - Dozzle
  - protection through Authentik forward auth

- Production-oriented but still realistic for Raspberry Pi
  - persistent volumes
  - no insecure Traefik API
  - Docker provider with exposedByDefault=false
  - TLS via Let’s Encrypt
  - Authentik with Google login and forward auth

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
- Configure portainer connexion : https://integrations.goauthentik.io/hypervisors-orchestrators/portainer/
- This assumes your DNS records already point `traefik.example.com`, `portainer.example.com`, `status.example.com`, and `logs.example.com` to your Raspberry Pi, because Let’s Encrypt HTTP challenge on Traefik needs ports `80/443` reachable from the internet. Traefik’s ACME HTTP challenge and Docker-label routing are both officially supported.
- `exposedByDefault=false` is important so only explicitly labeled containers are published through Traefik. That is a good default for production.
- Dozzle and Portainer both need Docker socket access for their core purpose; that is normal for these tools, but it is also why protecting them behind HTTPS and auth is important. Dozzle’s docs explicitly mention proxy-based authentication support.
- Authentik bootstrap password is set with `AUTHENTIK_BOOTSTRAP_PASSWORD` and is only read on the first startup
- Authentik secret key can be generated with `openssl rand -base64 60 | tr -d '\n'`
- Authentik PostgreSQL password can be generated with `openssl rand -base64 36 | tr -d '\n'`
- Google SSO setup (Google Cloud + Authentik):
  1. In Google Cloud Console, create an OAuth client (Web application).
  2. Set the redirect URI to `https://auth-lab.nadda.net/source/oauth/callback/google/`.
  3. In Authentik admin, go to Directory -> Federation and social login -> OAuth Sources -> Create -> Google.
  4. Use values from `.env`: `OIDC_GOOGLE_CLIENT_ID` and `OIDC_GOOGLE_CLIENT_SECRET`.
  5. In the source, enable display on login page and use scopes `openid profile email`.
  6. Test at `https://auth-lab.nadda.net/if/flow/default-authentication-flow/` and click Continue with Google.
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
- Configure Google as a social login source in Authentik
- Create proxy providers and applications for Dozzle, Uptime Kuma, and AOC Publisher
- Raise assurance later with MFA policies in Authentik
- add CrowdSec later if you expose this publicly
- After validation, pin versions more strictly:
  -  `traefik:v3.3.x`
  -  `portainer/portainer-ce:lts`
  -  `louislam/uptime-kuma:2.x`
  -  `amir20/dozzle:x.y.z`

# debug
- Check traefik logs : `docker logs traefik --tail 20`
- First Authentik setup is on `https://auth-lab.nadda.net/if/flow/initial-setup/` unless you rely on the bootstrap admin password
- If portainer has a timeout at conf, just restart the container : `docker restart portainer`