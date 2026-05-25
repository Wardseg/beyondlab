---
title: "What is a firewall and why your home network needs one"
date: 2026-05-25
draft: false
tags: ["networking", "firewall", "opnsense", "security", "beginners"]
description: "Most home networks rely on a basic router with no real firewall. Here's what a firewall actually does, why it matters, and why I run OPNsense."
---

Most people's home network looks like this: an ISP-provided router sitting in
the corner, doing everything — routing, DHCP, DNS, NAT, and whatever passes for
a firewall in consumer networking gear.

It works. Until it doesn't.

If you're running a homelab, self-hosting services, or just care about what's
happening on your network, a proper firewall is one of the most valuable things
you can add to your setup. And with tools like OPNsense, it's more accessible
than ever.

## What does a firewall actually do?

At its core, a firewall inspects network traffic and decides what to allow and
what to block — based on rules you define.

Your ISP router has a basic firewall built in. It blocks unsolicited inbound
connections from the internet (which is why you need to set up port forwarding
to expose services). But that's about where it stops. It doesn't:

- Inspect traffic between devices on your network
- Let you segment devices into isolated groups (VLANs)
- Give you visibility into what's happening on your network
- Block outbound connections from compromised devices
- Let you control DNS at a network level

A proper firewall does all of this and more.

## Why your home network needs one

**Device segmentation** — do you have IoT devices on your network? Smart bulbs,
cameras, thermostats? These devices are notoriously poorly secured. A firewall
lets you put them on a separate VLAN with no access to your main network. If one
gets compromised, it can't touch your NAS or your PCs.

**Visibility** — most people have no idea what their devices are doing on the
network. A firewall with logging shows you every connection — which devices are
phoning home, what DNS queries are being made, where traffic is going.

**DNS control** — run your DNS resolver through the firewall and you can block
ads, trackers, and malicious domains for every device on your network, without
installing anything on individual devices.

**Controlled exposure** — if you're self-hosting services, a firewall lets you
control exactly what's accessible from where. Expose specific services on specific
ports to specific sources. Everything else gets dropped.

**Learning** — understanding how firewall rules work, how VLANs are configured,
and how traffic flows through a network is knowledge that transfers directly to
enterprise environments. Your homelab firewall is essentially the same technology
running in corporate networks, just at a smaller scale.

## ISP router vs dedicated firewall

Your ISP router is designed for simplicity, not control. It's a consumer device
with a simplified interface that hides most of what's happening underneath.

A dedicated firewall running something like OPNsense gives you:

| Feature                    | ISP Router     | OPNsense                 |
| -------------------------- | -------------- | ------------------------ |
| Stateful packet inspection | Basic          | Full                     |
| VLAN support               | Rarely         | Yes                      |
| DNS resolver               | Basic          | Full (Unbound)           |
| Traffic visibility         | Minimal        | Detailed logging         |
| Intrusion detection        | No             | Yes (Suricata)           |
| VPN server                 | Sometimes      | Yes (WireGuard, OpenVPN) |
| Granular rules             | No             | Yes                      |
| Updates                    | ISP controlled | You control              |

## Why I chose OPNsense

OPNsense is a free, open-source firewall and router platform based on FreeBSD.
It's what I run as a VM on Proxmox, and it handles all the routing and firewalling
for my network.

The alternatives worth knowing about:

**pfSense** — OPNsense's predecessor and the most well-known open-source firewall
platform. OPNsense was forked from pfSense in 2015. Both are excellent; OPNsense
has a more modern interface and more frequent security updates.

**VyOS** — more focused on routing than firewalling, better suited if you have
complex routing requirements.

**Mikrotik RouterOS** — commercial but cheap, very powerful, steep learning curve.

For a homelab, OPNsense is the most approachable option with the best combination
of features, community support, and documentation.

## Running OPNsense as a VM

One of the things I like most about my setup is that OPNsense runs as a virtual
machine inside Proxmox. This means:

- Easy snapshots before making risky config changes
- Simple backup and restore
- No dedicated hardware needed
- Can be migrated if I change hardware

The one consideration is that the network interfaces need to be passed through
correctly — OPNsense needs to see the physical NICs or virtual bridges to route
traffic properly. It's a bit more complex to set up than a dedicated appliance,
but the flexibility is worth it.

## Where to start

If you're new to firewalls and networking, OPNsense can feel overwhelming at
first. The interface has a lot of options.

The good news is you don't need to configure everything at once. A basic setup —
WAN interface, LAN interface, default rules — gets you a working firewall in
under an hour. You can add complexity gradually as you learn.

My next post walks through setting up OPNsense in a Proxmox VM from scratch —
including the network configuration that trips most people up the first time.

---

_Next up: Running OPNsense in a Proxmox VM — full setup guide._
