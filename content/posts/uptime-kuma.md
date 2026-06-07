---
title: "Uptime Kuma: Self-Hosted Monitoring That Actually Looks Good"
date: 2026-06-05
draft: false
tags: ["uptime-kuma", "monitoring", "self-hosting", "docker", "how-to"]
description: "Uptime Kuma is the easiest way to monitor your self-hosted services and get notified when something goes down. Here's how to set it up in Docker and make it useful."
---

Every homelab eventually has the same problem: services go down and you don't find out until you try to use them. You open Jellyfin on the TV and it won't load. You go to check Vaultwarden from your phone and it times out. You spent twenty minutes wondering if it's your phone, your network, or the service — and it was the service, and it's been down for three hours.

Uptime Kuma fixes that. It monitors your services, builds a status page, and notifies you the moment something stops responding. It's open source, self-hosted, and takes about five minutes to get running.

---

## What Uptime Kuma does

At its core, Uptime Kuma pings your services on a schedule and records whether they responded. If something stops responding, it sends you a notification. You get a dashboard showing uptime history for every service you're watching.

It supports more than simple HTTP checks. You can monitor:

- HTTP/HTTPS endpoints (checks status code and optionally response content)
- TCP ports
- Ping (ICMP)
- DNS records
- Docker containers
- Databases (MySQL, PostgreSQL, Redis, MongoDB)
- Steam game servers
- Custom keywords in HTTP responses

The notification integrations are extensive — Telegram, Discord, Slack, email, Gotify, ntfy, PagerDuty, and about sixty others. If you have a preferred way to get alerts, it probably works.

The status page feature deserves special mention. You can build a public-facing page that shows uptime for whichever services you choose, without exposing your dashboard. I use mine at a public URL via [Cloudflare Tunnel](/posts/cloudflare-tunnels-self-host-without-opening-a-port/).

---

## Installation with Docker

Docker is the cleanest way to run Uptime Kuma. The image is small, updates are easy, and it keeps everything contained.

Create a directory for persistent data:

```bash
mkdir -p ~/docker/uptime-kuma
```

Start the container:

```bash
docker run -d \
  --name uptime-kuma \
  --restart unless-stopped \
  -p 3001:3001 \
  -v ~/docker/uptime-kuma:/app/data \
  louislam/uptime-kuma:latest
```

That's it. Navigate to `http://your-server-ip:3001` and you'll see the first-time setup asking you to create an admin account.

### Docker Compose version

If you prefer Compose (I do), create `~/docker/uptime-kuma/docker-compose.yml`:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./data:/app/data
```

Then:

```bash
cd ~/docker/uptime-kuma
docker compose up -d
```

The `./data` directory will be created automatically and holds the SQLite database where Uptime Kuma stores everything.

---

## First-time setup

After creating your admin account, you land on the dashboard. Click **Add New Monitor** to add your first service.

**Monitor type:** Start with HTTP(s) for web services. TCP Port for services that aren't HTTP. Ping for basic reachability.

**Heartbeat interval:** How often Uptime Kuma checks the service. The default is 60 seconds. I drop this to 30 seconds for critical services and leave it at 60 for everything else. Don't go too low — some services will interpret rapid repeated requests as an attack.

**Retries before unhealthy:** How many consecutive failures before Uptime Kuma sends an alert. I set this to 2 — one failed check could be a blip, two in a row is a real problem.

**Accepted status codes:** By default, 2xx responses are "up." If your service returns something different (some return 3xx redirects), adjust this.

---

## Setting up notifications

Go to **Settings → Notifications** and add your preferred method.

The Telegram setup is the one I'd recommend for most people — it's instant, works on your phone, and the bot setup takes two minutes:

1. Message `@BotFather` on Telegram, create a bot, copy the token
2. Start a conversation with your bot, then get your chat ID from `https://api.telegram.org/bot<token>/getUpdates`
3. In Uptime Kuma, add a Telegram notification with your token and chat ID

For a simpler option, **ntfy** is excellent if you self-host it (or use the public server). Install the ntfy app, subscribe to a topic, configure Uptime Kuma to push to that topic. Instant push notifications, no accounts required.

Once you've configured a notification method, go back to each monitor and enable it under the notification settings for that monitor.

---

## Building a status page

**Dashboard → Status Page** — create a page, give it a name, and add the monitors you want to display publicly.

The status page shows:

- Current status (up/down/degraded)
- Uptime percentage over the last 24h/7d/30d/1y
- A ping graph
- Incident history if you've added any

You can make the page public (anyone can view it) or add a password. The page lives at `/status/your-page-slug` on whatever URL you're running Uptime Kuma on.

To make it accessible from outside your home, point a Cloudflare Tunnel at port 3001 and set up a public domain for it. If you want the dashboard itself to remain private but the status page to be public, that requires a bit more setup — either two separate hostnames, or NGINX in front doing path-based routing.

---

## Monitoring tips

**Monitor the thing users actually hit, not the internal service.** If you're running a reverse proxy in front of Jellyfin, monitor the reverse proxy URL — that's what fails when something goes wrong, and it tests the full stack.

**Use keyword monitoring for services that return 200 even when broken.** Some services return HTTP 200 with an error page. Add a keyword check to confirm specific content appears in the response.

**Group monitors with tags.** Uptime Kuma lets you add tags to monitors. Makes it easier to filter when you have twenty services.

**Don't monitor from the same machine if you can help it.** Uptime Kuma running on the same box as your services will report everything as down if that box goes down — which is exactly when you need it to work. Ideally run it on a separate machine. If that's not practical, consider a secondary monitoring setup on a cloud VPS, or use Uptime Kuma's push heartbeat feature alongside an external cron job.

---

## Updating

```bash
docker compose pull
docker compose up -d
```

Uptime Kuma updates frequently. The changelog is worth checking before major version jumps — it's an active project and the API occasionally changes between versions.

---

## What my setup looks like

I run Uptime Kuma in an LXC container on Proxmox, separate from most of my other services. It monitors:

- All my publicly accessible services (via their Cloudflare Tunnel URLs)
- Internal services via their LAN IPs
- My OPNsense firewall via ping
- A few external services I care about

Notifications go to Telegram. If I'm away from home and something goes down, I know within two minutes.

The status page is public — if you're trying to reach something I host and it's down, you can check there before wondering if it's your connection.

---

There are fancier monitoring solutions out there — Prometheus, Grafana, Zabbix. For a homelab, they're overkill. Uptime Kuma does what matters: it tells you when something broke, and it does it in a package that takes ten minutes to set up and zero minutes to maintain.

Start simple. Add more monitors than you think you need. You'll be surprised how often you're glad it's running.
