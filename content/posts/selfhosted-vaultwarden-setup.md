---
title: "Self-Hosted Password Management: Why I Switched to Vaultwarden"
date: 2026-06-10
draft: false
tags: ["how-to", "self-hosting", "docker", "security", "vaultwarden"]
description: "I got tired of paying for a password manager I didn't fully trust. Vaultwarden gives me everything Bitwarden offers — running on my own hardware, for free."
---

For years I used a commercial password manager. It worked fine. Then the price went up, there was a security incident that made the news, and I found myself wondering why I was paying monthly for something I could run myself on hardware I already owned.

That's how I ended up on Vaultwarden. It's been running in my homelab for months now, it's never gone down, and I've not thought about it since the initial setup. This post covers why I made the switch, how Vaultwarden works, and exactly how to set it up.

## What Is Vaultwarden?

Vaultwarden is an unofficial, open-source implementation of the Bitwarden server API. It's written in Rust, runs in a single Docker container, and is compatible with all the official Bitwarden clients — the browser extensions, the mobile apps, the desktop apps, all of it.

The important distinction: you're not running a fork of Bitwarden. You're running a compatible server that speaks the same protocol, paired with the official Bitwarden clients. This means you get the polished client experience without relying on Bitwarden's cloud infrastructure.

Vaultwarden was previously called Bitwarden_RS, in case you see that name in older documentation.

## Why Self-Host Your Password Manager?

Fair question — your passwords are arguably the most sensitive data you have. Is it wise to run this yourself?

My thinking:

**Control over your data.** When you use a cloud password manager, your encrypted vault sits on someone else's server. When that company has a breach (and several have), you're dependent on their security practices. With Vaultwarden, the encrypted vault lives on your hardware.

**No subscription.** Bitwarden's paid plan costs around $10/year, which is cheap. But it's another recurring charge, and self-hosting eliminates it.

**Features without upselling.** Bitwarden's free cloud tier has limits — certain features (like some two-factor options, reports) are paid-only. Vaultwarden unlocks all of these for free since you control the server.

**The trade-off is real though.** You're now responsible for backups, uptime, and keeping the software updated. If your homelab goes down, you can't access your passwords until it's back. I mitigate this by keeping an offline export as a backup, and by making sure Vaultwarden runs on a container that starts automatically.

## What You Need

- A Linux host running Docker (I run Vaultwarden in an LXC container on Proxmox with Docker installed)
- A domain name with HTTPS — Vaultwarden requires a secure connection for the official clients to connect. I use Cloudflare Tunnels to expose it securely without opening ports.
- Basic comfort with Docker Compose

## Setting Up Vaultwarden

### 1. Create the Docker Compose file

Create a directory for Vaultwarden and a `docker-compose.yml`:

```bash
mkdir -p ~/vaultwarden && cd ~/vaultwarden
nano docker-compose.yml
```

```yaml
version: "3"

  vaultwarden:
services:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    volumes:
      - ./vw-data:/data
    environment:
      - DOMAIN=https://vault.yourdomain.com
      - SIGNUPS_ALLOWED=true
      - ADMIN_TOKEN=your_secure_admin_token_here
    ports:
      - "8080:80"
```

A few things to customise:

- `DOMAIN` — set this to the URL you'll access Vaultwarden from
- `ADMIN_TOKEN` — generate a secure random string (`openssl rand -base64 48` works well). This protects the admin panel at `/admin`
- `SIGNUPS_ALLOWED=true` — set this to `false` after you've created your account, so no one else can register on your instance

### 2. Start the container

```bash
docker compose up -d
```

Vaultwarden is now running on port 8080. You can verify it's working by navigating to `http://your-server-ip:8080`.

### 3. Set up HTTPS access

The official Bitwarden clients refuse to connect to a non-HTTPS server. You have a few options:

**Cloudflare Tunnels** — my preferred approach. Install `cloudflared` on your host, create a tunnel pointing to `localhost:8080`, and Cloudflare handles the HTTPS certificate and proxying. No ports need to be opened on your router. This is what I use, and I've written a full guide on [setting up Cloudflare Tunnels](/posts/cloudflare-tunnels-self-host-without-opening-a-port).

**Reverse proxy with Let's Encrypt** — use Nginx Proxy Manager or Caddy in front of Vaultwarden with automatic SSL certificate renewal. More moving parts but gives you full control.

**Tailscale** — if you only need access from your own devices, Tailscale gives you a private network with HTTPS. Vaultwarden won't be publicly accessible, which is actually more secure.

### 4. Create your account and lock down signups

Navigate to your Vaultwarden URL, click **Create Account**, and set up your master password. Make it strong — this is the one password you actually need to remember.

Once your account is created, go back to your `docker-compose.yml` and change `SIGNUPS_ALLOWED` to `false`, then restart the container:

```bash
docker compose down && docker compose up -d
```

### 5. Connect the Bitwarden clients

In the official Bitwarden browser extension or mobile app, before logging in, click the **Settings** gear icon and change the **Server URL** to your Vaultwarden domain (e.g., `https://vault.yourdomain.com`). Then log in with the account you just created.

That's it. The clients work identically to the cloud version.

## The Admin Panel

Navigate to `https://your-vaultwarden-url/admin` and enter your admin token. From here you can:

- See all registered users
- Send organisation invites
- Configure email (SMTP) for notifications
- Enable or disable features
- Check diagnostics

Setting up SMTP is worth doing — it lets Vaultwarden send you verification emails and alerts, and is needed if you want to share items with other users.

## Backups

This is critical. Your password vault is only as safe as your backup strategy.

The entire Vaultwarden data directory (`./vw-data` in the setup above) is what you need to back up. It contains:

- `db.sqlite3` — the main database with your encrypted vault
- `attachments/` — any file attachments
- `config.json` — your admin configuration

I back this up to my Synology RS814 NAS nightly using a simple rsync cron job. I also do a manual export from the Bitwarden client (vault → Export → encrypted JSON) every month and keep that offline.

If you lose this data, you lose your passwords. Back it up.

## Keeping It Updated

Vaultwarden releases updates regularly. To update:

```bash
cd ~/vaultwarden
docker compose pull
docker compose down && docker compose up -d
```

I do this roughly monthly. It takes about 30 seconds and has never caused issues.

## Is It Actually Reliable?

In my experience, yes. Vaultwarden is a single Rust binary that uses SQLite for storage. It's not complex infrastructure. It starts fast, uses very little RAM (around 10–20MB at idle), and just runs.

The main reliability risk is self-inflicted: if your homelab goes down, you lose access to your passwords from outside your network. My mitigations:

- Offline export stored securely for emergencies
- Vaultwarden container set to `restart: unless-stopped` so it comes back after a reboot
- Uptime Kuma monitoring it and alerting me if it goes down

## Final Thoughts

Vaultwarden is one of those self-hosted services that I set up once, forgot about, and just works. It's not the most complex thing you can run in a homelab, but it's arguably one of the most practically useful — you interact with your password manager dozens of times a day.

If you're paying for a cloud password manager and you already have a homelab, the switch is worth making. The setup takes an afternoon, the ongoing maintenance is minimal, and you end up with something you actually control.

---

_Running Vaultwarden already, or have questions about the setup? Leave a comment below — happy to help with specific configurations._
