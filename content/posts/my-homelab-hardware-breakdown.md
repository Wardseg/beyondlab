---
title: "My homelab hardware: the HP DL380 G6 breakdown"
date: 2026-05-20
draft: false
tags: ["homelab", "hardware", "proxmox", "dl380", "server"]
description: "A look at the hardware running my homelab — an HP DL380 G6 I got for free, its specs, its tradeoffs, and what I'd buy if I had to start over."
---

Every homelab starts somewhere. Mine started with a conversation with a friend in bar who
had a server sitting in his garage collecting dust and no interest in running it
anymore.

"You want it?"

That's how I ended up with an HP DL380 G6 — and honestly, it's been both one of
the best and most humbling pieces of kit I've ever worked with.

## The specs

This is what's running everything:

| Component  | Spec                                      |
| ---------- | ----------------------------------------- |
| CPU        | 8 x Intel Xeon E5520 @ 2.27GHz (1 Socket) |
| RAM        | 36 GiB                                    |
| Storage    | 1TB (internal) + 16TB Synology RS814 NAS  |
| Hypervisor | Proxmox VE 9.1                            |
| Kernel     | Linux 6.17.13-2-pve                       |
| Boot Mode  | Legacy BIOS                               |

At idle it's only using about 3% CPU and 24% RAM — which tells you just how
much headroom there is for running virtual machines and containers. For a homelab
workload it's genuinely overkill, in the best possible way.

The 1TB internal storage handles the Proxmox installation and VM disks. The 16TB
comes from a Synology RS814 — a 1U rack-mount NAS that fits neatly alongside the
DL380 and handles everything else: backups, media, documents, and anything that
needs bulk storage. The RS814 runs Synology DSM and has been rock solid — quiet,
efficient, and completely set-and-forget compared to the server itself.

## What it's actually like to live with

Here's the honest part.

The DL380 G6 is a **2U rack server** built for a data centre, not a spare room.
It is large, heavy, and it will remind you of its presence constantly.

**Noise** — it is loud. Not "bit of a hum" loud. Jet engine on startup loud,
settling to a persistent fan roar that you learn to tune out if it's in a
separate room. In a living space it would be genuinely unpleasant.

**Power draw** — it is not efficient by modern standards. This is a machine from
2009 designed when power consumption was less of a concern. Running it 24/7 has
a real cost on your electricity bill that's worth factoring in before you commit.

**Size and weight** — it needs rack space or a very sturdy shelf. Moving it alone
is a workout.

None of this was a dealbreaker for me because I got it for free. But if I'd had
to pay for it and account for ongoing power costs, the calculus would look
very different.

## What I'd buy if I were starting from scratch

This is the question I get asked most often when people see the setup.

Honestly? I wouldn't buy a rack server at all.

I'd look seriously at **mini PCs** — something like the
[Beelink SER5](https://www.amazon.com/dp/B0BV5GKBK7/?tag=beyondlab-20).
Small, quiet, power efficient, and cheap enough that you can buy two or three
and set them up as a **[Proxmox](https://www.proxmox.com) cluster**. You get
high availability, redundancy, and a setup that won't overheat a small room or
double your electricity bill.

The tradeoffs are less raw RAM and fewer PCIe slots — but for most homelab
workloads that doesn't matter. You're running services, not rendering video.

If you want something in between — more power than a mini PC, less bulk than a
rack server — small form factor machines like the
[HP EliteDesk](https://www.amazon.com/s?k=hp+elitedesk+mini&tag=beyondlab-20) or
[Dell OptiPlex](https://www.amazon.com/s?k=dell+optiplex+micro&tag=beyondlab-20)
are worth looking at. Common, cheap second-hand, and very capable with a RAM upgrade.

## What actually runs on it

Right now the DL380 is running [Proxmox VE](https://www.proxmox.com) with a
handful of VMs and LXC containers:

- **OPNsense** — the firewall and network brain
- **MkDocs** — internal documentation
- Various LXC containers for self-hosted services

Current load is minimal — 3% CPU, 24% RAM — which leaves plenty of room to keep
adding services without worrying about resources.

## The verdict

The HP DL380 G6 is a capable, reliable machine that taught me a lot. Getting it
for free made every tradeoff worth it. If I had to pay market rate and cover
ongoing power costs, I'd make a different choice.

But that's the thing about homelabs — the best hardware is the hardware you
actually have access to. Start with what you've got, learn from it, and upgrade
when the limitations actually start to hurt.

Mine haven't yet.

---

_This post contains affiliate links. If you purchase through them I may earn a
small commission at no extra cost to you._

_Next up: Why I chose Proxmox over ESXi and Hyper-V._
