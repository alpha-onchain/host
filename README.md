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
mkdir /opt/homeserver/stacks/proxy
mkdir /opt/homeserver/stacks/portainer
mkdir /opt/homeserver/stacks/ops
mkdir /opt/homeserver/data/proxy
mkdir /opt/homeserver/data/portainer
mkdir /opt/homeserver/data/backups
```

## Build Docker stack
