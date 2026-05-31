---
title: "Self-Host Your First 5 Services with Docker Compose"
date: 2026-05-31
draft: false
tags: ["docker", "self-hosting", "homelab", "beginner", "privacy", "how-to"]
description: "Stop paying for services you can run yourself. Here are 5 genuinely useful self-hosted apps you can get running in an afternoon with Docker Compose."
---

Every month you're probably paying for at least one service you could run yourself - a password manager, cloud storage, a notes app, a photo backup tool. Self-hosting these isn't just cheaper; it means your data lives on hardware you control.

Docker Compose makes this accessible to anyone comfortable with a terminal. You don't need to be a developer. You just need a machine that stays on, a basic Linux install, and this guide.

---

## Before You Start

**What you need:**

- A Linux machine (a VM in Proxmox, an old PC, a mini PC - anything works)
- Docker and Docker Compose installed
- Basic comfort with SSH and a text editor

**Install Docker on Ubuntu/Debian:**

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

Verify it's working:

```bash
docker run hello-world
```

**File structure to keep things tidy:**

```cmd
~/homelab/
|--- vaultwarden/
|------- docker-compose.yml
|--- uptime-kuma/
|------ docker-compose.yml
|--- nextcloud/
|------- docker-compose.yml
...
```

Each service gets its own folder. When something breaks, you know exactly where to look.

---

## Service 1: Vaultwarden - Your Own Password Manager

**What it is:** A lightweight, self-hosted Bitwarden-compatible server. You use the official Bitwarden apps on all your devices - just pointed at your own server instead of Bitwarden's cloud.

**Why it matters:** Your passwords live on your hardware. No subscription needed.

```yaml
# ~/homelab/vaultwarden/docker-compose.yml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    volumes:
      - ./data:/data
    ports:
      - "8080:80"
    environment:
      - SIGNUPS_ALLOWED=true
    restart: unless-stopped
```

Run it:

```bash
cd ~/homelab/vaultwarden
docker compose up -d
```

Open `http://your-server-ip:8080`, create your account, then set `SIGNUPS_ALLOWED=false` in the environment to lock it down.

**Next step:** Once it's running, set up a reverse proxy with HTTPS (more on that below) - Vaultwarden requires HTTPS for the browser extension to work properly.

---

## Service 2: Uptime Kuma - Monitor Everything You Run

**What it is:** A slick, self-hosted uptime monitoring dashboard. Add your services, set check intervals, get notified when something goes down.

**Why it matters:** Once you're running multiple containers, you need to know when one silently dies.

```yaml
# ~/homelab/uptime-kuma/docker-compose.yml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```

Open `http://your-server-ip:3001`, create your admin account, and start adding monitors. Point it at all your other services. You can set up notifications via Telegram, Discord, email, or a dozen other methods.

This should be the second thing you install, right after Docker itself.

---

## Service 3: Nextcloud - Your Own Google Drive

**What it is:** A full self-hosted cloud storage and productivity suite. File sync, calendar, contacts, notes, and more - all running on your hardware.

**Why it matters:** Stop paying for Google One or iCloud storage for files you own.

```yaml
# ~/homelab/nextcloud/docker-compose.yml
services:
  nextcloud-db:
    image: mariadb:10.11
    container_name: nextcloud-db
    environment:
      - MYSQL_ROOT_PASSWORD=changeme
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=changeme
    volumes:
      - ./db:/var/lib/mysql
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    ports:
      - "8081:80"
    environment:
      - MYSQL_HOST=nextcloud-db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=changeme
    volumes:
      - ./data:/var/www/html
    depends_on:
      - nextcloud-db
    restart: unless-stopped
```

> **Change the passwords** before running this. Yes, even in a homelab.

Open `http://your-server-ip:8081` and complete the setup wizard. Install the Nextcloud desktop and mobile apps and point them at your server - file sync works exactly like Dropbox from that point.

---

## Service 4: Jellyfin - Your Own Netflix

**What it is:** A free, open-source media server. Point it at your movie and TV show folders and get a full streaming interface accessible from any browser, TV app, or mobile device.

**Why it matters:** Own your media library, stream it anywhere on your network (or remotely), no subscriptions.

```yaml
# ~/homelab/jellyfin/docker-compose.yml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /path/to/your/media:/media:ro
    ports:
      - "8096:8096"
    restart: unless-stopped
```

Replace `/path/to/your/media` with wherever your films and shows are stored. Open `http://your-server-ip:8096`, run through the setup wizard, and add your media libraries.

Jellyfin handles hardware transcoding if your server has a compatible GPU or Intel QuickSync - check the Jellyfin docs for the extra config lines needed to enable it.

---

## Service 5: Homepage - A Dashboard for Everything

**What it is:** A clean, customisable dashboard that shows all your self-hosted services in one place, with live status indicators, service info, and widgets.

**Why it matters:** Once you're running five or more services, you need a single place to see them all. Homepage is the best-looking option right now.

```yaml
# ~/homelab/homepage/docker-compose.yml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - "3000:3000"
    volumes:
      - ./config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped
```

The `docker.sock` mount lets Homepage auto-detect running containers and show their status. Edit the YAML files in `./config` to add your services, bookmarks, and widgets.

---

## Making It All Accessible: Nginx Proxy Manager

Visiting `192.168.1.100:8080`, `192.168.1.100:3001`, `192.168.1.100:8081` gets old fast. Nginx Proxy Manager gives you proper domain names and HTTPS for every service.

```yaml
# ~/homelab/nginx-proxy-manager/docker-compose.yml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    restart: unless-stopped
```

Open `http://your-server-ip:81`, log in with the default credentials (`admin@example.com` / `changeme` - change these immediately), and add proxy hosts.

With a domain pointing at your server, you can set up:

- `vault.yourdomain.com` - Vaultwarden
- `cloud.yourdomain.com` - Nextcloud
- `media.yourdomain.com` - Jellyfin

Nginx Proxy Manager handles Let's Encrypt SSL certificates automatically.

---

## Keeping Things Running

A few habits that will save you headaches:

**Update regularly.** Pull updated images monthly:

```bash
docker compose pull
docker compose up -d
```

**Back up your data folders.** Each service stores its data in the `./data` volume mount. Back those folders up to a NAS or external drive regularly.

**Check your logs when something breaks:**

```bash
docker logs container-name --tail 50
```

**Don't expose everything to the internet.** Use a VPN (Tailscale is excellent) or a Cloudflare Tunnel for remote access rather than opening ports on your router.

---

## What's Next?

Once you've got these five running, you're ready to explore:

- **Authentik** - SSO for all your services with a single login
- **Grafana + Prometheus** - metrics and dashboards for your whole stack
- **Immich** - self-hosted Google Photos replacement with mobile backup
- **Gitea** - your own private GitHub

The pattern is always the same: find a service, grab the `docker-compose.yml`, adapt it, and run it. The whole self-hosting ecosystem runs this way.

Your data, your hardware, your rules.

---

_Questions about any of these setups? Leave a comment - happy to help troubleshoot._
