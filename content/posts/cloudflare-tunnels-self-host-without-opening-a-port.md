---
title: "Cloudflare Tunnels: Self-Host Without Opening a Single Port"
date: 2026-06-06
draft: false
tags: ["cloudflare", "networking", "self-hosting", "security", "how-to"]
description: "Cloudflare Tunnels let you expose self-hosted services to the internet without port forwarding, a public IP, or putting your home network at risk. Here's how they work and how to set one up."
---

Port forwarding has a bad reputation, and honestly, most of it is deserved. You open a hole in your router, expose your home IP to the internet, and spend the next week watching fail2ban block login attempts from IP ranges you've never heard of. There's a better way.

Cloudflare Tunnels let you expose self-hosted services to the internet without opening a single port on your router. Your home IP stays hidden. The traffic goes through Cloudflare's network. And setup takes about fifteen minutes.

I use them for everything on my homelab that faces the internet. This is how they work.

---

## How Cloudflare Tunnels actually work

The key insight is that the connection flows **outward** from your server, not inward from the internet.

Traditional reverse proxy setup:

```
Internet → your router (port 443 open) → nginx → your service
```

Cloudflare Tunnel setup:

```
Internet → Cloudflare edge → encrypted tunnel → cloudflared daemon → your service
```

Your server runs a small daemon called `cloudflared`. That daemon opens a persistent outbound connection to Cloudflare's edge network. When a request comes in for your subdomain, Cloudflare routes it through that tunnel to your service — without your router ever seeing inbound traffic on that port.

Your home IP is never exposed. Your router has nothing extra open. If someone tries to port-scan your home connection, they find nothing.

---

## Prerequisites

- A domain managed by Cloudflare (you need DNS control)
- A Linux machine running the service you want to expose (can be a container, a VM, or bare metal)
- A free Cloudflare account

That's it. No paid plan required. Cloudflare Tunnels are free.

---

## Setting up a tunnel

### Step 1: Install cloudflared

On Debian/Ubuntu:

```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
```

Verify it installed:

```bash
cloudflared --version
```

### Step 2: Authenticate

```bash
cloudflared tunnel login
```

This opens a browser window where you authorize Cloudflare to manage DNS for your domain. A certificate gets saved to `~/.cloudflared/cert.pem`.

### Step 3: Create the tunnel

```bash
cloudflared tunnel create my-homelab
```

This creates a tunnel with a UUID and generates a credentials file in `~/.cloudflared/`. Note the tunnel UUID — you'll need it.

### Step 4: Configure the tunnel

Create the config file at `~/.cloudflared/config.yml`:

```yaml
tunnel: <your-tunnel-uuid>
credentials-file: /home/youruser/.cloudflared/<uuid>.json

ingress:
  - hostname: service.yourdomain.com
    service: http://localhost:8096
  - hostname: another.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```

The ingress rules work top to bottom. Each hostname maps to a local service on a specific port. The final catch-all rule returns a 404 for anything that doesn't match.

A few things worth knowing about the `service` field:

- Use `http://` for unencrypted local services (fine — the traffic is local)
- Use `https://` if your local service runs with TLS
- You can point at other machines on your LAN: `http://192.168.1.50:8096`

### Step 5: Create the DNS records

```bash
cloudflared tunnel route dns my-homelab service.yourdomain.com
cloudflared tunnel route dns my-homelab another.yourdomain.com
```

This creates CNAME records in Cloudflare DNS pointing your subdomains at the tunnel. No manual DNS editing needed.

### Step 6: Run it

Test first:

```bash
cloudflared tunnel run my-homelab
```

If everything works, install it as a system service so it starts on boot:

```bash
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
```

---

## Access policies with Cloudflare Access

This is where things get genuinely good. Cloudflare Access lets you put an authentication layer in front of your services — before a request ever reaches your server.

Navigate to **Zero Trust → Access → Applications** in the Cloudflare dashboard and add an application for your subdomain. You can gate it behind:

- An email OTP (someone enters their email, Cloudflare sends a code)
- A Google/GitHub OAuth login
- A one-time PIN
- A specific list of allowed emails

This means you can expose a service publicly but require login — and that login lives entirely within Cloudflare's infrastructure. Your service doesn't need to implement authentication at all if you don't want it to.

I use this for services like Uptime Kuma and Karakeep that I want accessible from outside my home but don't need to be fully public.

---

## Practical tips

**Service doesn't need to be on the same machine.** The `cloudflared` daemon can forward to any IP on your LAN. Run the tunnel on one machine, point it at services running on others.

**Multiple services, one tunnel.** You don't need a separate tunnel per service. One tunnel with multiple ingress rules handles everything. Cleaner, less resource usage.

**The catch-all rule is required.** The last entry in your ingress config must be a catch-all. Without it, `cloudflared` will throw an error on startup.

**Check your service binding.** If your service is listening on `127.0.0.1` only, the tunnel needs to run on the same machine. If it's listening on `0.0.0.0`, you can point the tunnel from any machine on the LAN.

**Cloudflare sees your traffic.** This is worth being explicit about. Cloudflare terminates the TLS connection at their edge, which means they can see the decrypted traffic passing through. For most self-hosted services this is fine. For something like a password manager, consider whether you're comfortable with that, or run it behind a tunnel with end-to-end encryption.

---

## What I actually use this for

On my homelab, Cloudflare Tunnels expose:

- A public status page (Uptime Kuma) — more on that in [a separate post](/posts/uptime-kuma-self-hosted-monitoring/)
- Karakeep behind Cloudflare Access for personal use
- A few development projects when I need to share a preview

Everything else stays internal. The principle is: only expose what needs to be public, and gate everything that doesn't.

---

## Why not just use a VPN?

Wireguard or Tailscale are excellent for accessing your home network remotely. They're my preference for anything personal. But Cloudflare Tunnels solve a different problem: making a service genuinely public-facing, with a proper domain, without maintaining VPN clients on every device that might need access.

Both have their place. Tunnel for public-facing, VPN for personal access.

---

## Troubleshooting

**`ERR_TOO_MANY_REDIRECTS`** — Your service is running with HTTPS and Cloudflare is also enforcing HTTPS, creating a redirect loop. Either set your Cloudflare SSL/TLS mode to "Full" (not "Flexible"), or set your ingress rule to `https://localhost:port` and disable cert validation with `noTLSVerify: true` under the service config.

**Service unreachable after tunnel starts** — Check that the service is actually running with `ss -tlnp | grep <port>` and that you're pointing at the right port in your config.

**`config.yml` changes not taking effect** — Restart the service: `sudo systemctl restart cloudflared`. The daemon reads the config at startup.

---

Cloudflare Tunnels have become one of the first things I set up on any new self-hosted project. Once you stop worrying about port forwarding and dynamic IPs, the whole thing gets a lot simpler.
