---
title: "How to Start a Homelab in 2026 (Without Spending a Fortune)"
date: 2026-05-31
draft: false
tags: ["homelab", "proxmox", "beginner", "hardware", "self-hosting", "how-to"]
description: "A practical guide to building your first homelab in 2026 - what hardware to buy, what to skip, and how to get running without blowing your budget."
---

So you want a homelab. Maybe you've been lurking on r/homelab for months, staring at rack builds that cost more than a used car. Maybe you're an IT engineer at work and want a place to break things without a ticket being raised. Either way - good news: you don't need to spend a fortune to get started.

This guide is for people who want a _useful_ homelab, not a showpiece.

---

## What Is a Homelab, Really?

A homelab is any setup at home where you run servers, virtual machines, containers, or network gear to learn, experiment, or self-host services. It could be a repurposed old laptop or a full rack of enterprise gear. What matters is that it's _yours_ to break and fix.

Common reasons people run homelabs:

- Learn Proxmox, Docker, Kubernetes, or networking in a safe environment
- Self-host services like Nextcloud, Jellyfin, or Vaultwarden
- Practice for IT certifications (CCNA, VCP, etc.)
- Run home automation
- Get off subscription services and own your own data

---

## Step 1: Define What You Actually Want to Do

Before buying anything, answer this honestly: **what will you actually run?**

| Goal                           | What you need                      |
| ------------------------------ | ---------------------------------- |
| Learn Linux & Docker           | Any old PC with 8GB RAM            |
| Run VMs with Proxmox           | 16-32GB RAM minimum                |
| Home media server              | Lots of storage, modest CPU        |
| Network lab (VLANs, firewalls) | A managed switch + router hardware |
| All of the above               | Used enterprise server or mini PC  |

Don't buy a rack server "just in case." Start with what solves your current goal.

---

## Step 2: Choose Your Hardware

### The Budget Option - Used Mini PCs (~?100-250)

Mini PCs like the Beelink SER5, HP EliteDesk 800 G4 Mini, or Dell OptiPlex Micro are the sweet spot for beginners in 2026. They're:

- Silent enough for a home office
- Power-efficient (15-25W idle)
- Powerful enough to run Proxmox with 3-4 VMs
- Available in bulk from corporate fleet refreshes

Look for: **Intel i5 8th gen or newer, 16GB+ RAM, NVMe SSD**. The SSD makes a bigger difference than the CPU for most homelab workloads.

### The "I Want a Proper Lab" Option - Used Rack Servers (~?100-400)

HP ProLiant DL360/DL380 G8-G10 and Dell PowerEdge R720-R750 machines regularly appear on eBay and local IT resellers for very little. You get proper ECC RAM, hot-swap drives, IPMI/iLO remote management, and expansion headroom.

The trade-offs are real though: these machines are **loud**, **power-hungry**, and **physically large**. A DL380 G9 can idle at 100-150W. Over a year, that adds up on your electricity bill. Factor that in.

A good rule of thumb: if you're running it 24/7, power efficiency matters. If it's only on when you're using it, raw specs matter more.

### What to Skip

- **Raspberry Pi as your main server** - great for learning, terrible as a workhorse. The SD card will fail, the I/O is limited, and you'll outgrow it quickly.
- **NAS devices as compute** - NAS boxes (Synology, QNAP) are great for storage, not for running VMs.
- **10GbE networking on day one** - you don't need it. 1GbE handles everything until you're regularly moving large files across the network.

---

## Step 3: Install Proxmox

[Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview) is the go-to hypervisor for homelabs. It's free, open-source, and lets you run both VMs and containers (LXC) from a single web UI.

**Installation is straightforward:**

1. Download the Proxmox VE ISO from the official site
2. Flash it to a USB drive with Balena Etcher or Rufus
3. Boot your machine from USB and follow the installer
4. Access the web UI at `https://your-ip:8006`

When it's up, dismiss the "no valid subscription" warning (you don't need a paid licence for home use) and create your first VM.

**Good first VMs to spin up:**

- Ubuntu Server 24.04 LTS - the standard Linux learning environment
- A Docker host - one VM running Docker Compose handles most self-hosted services
- pfSense or OPNsense - if you want to get into network segmentation and VLANs

---

## Step 4: Set Up Basic Networking

You don't need a managed switch on day one, but you'll want one eventually. A cheap Cisco SG300 or TP-Link TL-SG108E (about ?30-50 used) gives you VLAN support for separating your lab traffic from your home network.

**Why VLANs matter:**
If something in your lab gets compromised or misconfigured, you don't want it able to talk to your personal devices. A simple lab VLAN keeps your homelab isolated.

Basic network layout that works well:

```
VLAN 1  - Home devices (phones, laptops, TV)
VLAN 10 - Homelab / servers
VLAN 20 - IoT devices (smart home, cameras)
```

OPNsense running as a VM on Proxmox can manage all of this - but that's a whole separate guide.

---

## Step 5: Run Your First Service

Once Proxmox is up and you have a Docker host VM, run something useful. Here's a minimal `docker-compose.yml` to get Uptime Kuma running - a simple status page for all your services:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    volumes:
      - ./data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```

Save that as `docker-compose.yml`, run `docker compose up -d`, then open `http://your-vm-ip:3001`. You're self-hosting.

From here, the path is clear: add more services, learn more tooling, and break things in a way that teaches you how to fix them.

---

## Common Mistakes to Avoid

**Overbuying hardware before you know what you need.** Start small. You can always add more.

**Skipping backups.** Whatever you run, back it up. A 3-2-1 strategy (3 copies, 2 media types, 1 offsite) sounds like overkill until you lose something.

**Running everything on one VM.** Separate concerns. Your media server doesn't need to be on the same VM as your network monitoring stack.

**Never documenting anything.** Future-you will forget why you configured something. Write it down - even a simple markdown file in a private Gitea repo is better than nothing.

---

## What's Next?

Once you have a Proxmox node running and a couple of services up, the natural next steps are:

- Set up a reverse proxy (Nginx Proxy Manager or Traefik) so you can reach services by name instead of IP and port
- Add a Cloudflare Tunnel for remote access without opening ports
- Try OPNsense for proper network segmentation
- Look at Ansible for automating your VM provisioning

The homelab rabbit hole is deep. The good news is that every mistake teaches you something you'd never learn in a vendor training course.

---

_Running a homelab yourself? Drop a comment below with what you're running - always curious what setups people have._
