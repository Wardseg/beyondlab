---
title: "LXC Containers vs. Docker vs. Full VMs in Proxmox: What I Actually Use and Why"
date: 2026-06-09
draft: false
tags: ["proxmox", "homelab", "docker", "how-to", "self-hosting"]
description: "LXC containers, Docker, full VMs — Proxmox supports all three. Here's how I decide which one to use for each service in my homelab, and why it matters."
---

One of the things that surprised me when I started using Proxmox seriously is that it doesn't just give you one way to run workloads — it gives you three: full virtual machines, LXC containers, and Docker (running inside one of the other two). Choosing between them isn't just a technical preference; it affects performance, maintenance overhead, and how much of your hardware you're burning for each service.

Here's how I think about it, and what I actually run on each.

## The Three Options

### Full Virtual Machines

A VM is a complete computer running on your hardware. Proxmox uses KVM (Kernel-based Virtual Machine) under the hood. The guest OS — whether that's Windows, Ubuntu, whatever — has no idea it's running inside a host. It gets virtual CPUs, virtual RAM, a virtual disk, virtual network cards.

The upside is total isolation. A VM can run any OS, has its own kernel, and can't accidentally interfere with the host. The downside is overhead: every VM runs a full OS, which means RAM usage adds up fast, and boot times are real.

### LXC Containers

LXC (Linux Containers) is different. Rather than virtualizing hardware, LXC containers share the host's Linux kernel. The container gets its own filesystem, its own network stack, its own process space — but not its own kernel. From inside the container, it looks and feels like a normal Linux system.

The result is something that boots in a second or two, uses a fraction of the RAM a VM would need, and has almost no performance overhead. But there are limits: LXC containers can only run Linux, and some applications that need kernel-level access don't work well in containers without extra configuration.

### Docker

Docker is an application containerization layer, not a hypervisor. In a Proxmox context, you're typically running Docker inside either a VM or an LXC container. Docker containers package an app and its dependencies into a portable image, which makes deploying services incredibly simple — pull an image, write a `docker-compose.yml`, run it.

The massive ecosystem of Docker images is a huge practical advantage. Almost every self-hosted service you'll find documented online has a Docker image available.

## How I Decide What Gets What

My rough mental model:

**Full VM when:**

- The service needs its own kernel (custom kernel modules, eBPF, etc.)
- I'm running a non-Linux OS (Windows VMs for testing)
- I want total isolation — e.g., OPNsense as my firewall
- The service is sensitive enough that I want a hard boundary from everything else

**LXC container when:**

- It's a Linux-only service with no exotic kernel requirements
- I want fast startup and minimal RAM overhead
- I'm running many small services and resources are tight
- The service is something stateless-ish that I don't mind rebuilding occasionally

**Docker (inside LXC or VM) when:**

- The service has a well-maintained Docker image and I want easy updates
- I'm running multiple related services together with Docker Compose
- I want the portability — being able to move a `docker-compose.yml` to another host trivially

## What I Actually Run

To make this concrete, here's how my current setup breaks down:

**Full VMs:**

- **OPNsense** — this is my firewall and handles all network traffic. It needs direct access to network interfaces (via PCIe passthrough for the NICs), and I want it fully isolated. A VM with snapshots makes upgrades much safer.
- **Windows test environment** — occasionally spin one up for compatibility testing. Nowhere else to run this.

**LXC Containers:**

- **Internal documentation** (MkDocs) — a lightweight static site server; LXC is perfect, uses maybe 50MB RAM
- **Custom monitoring dashboard** — one of my personal projects; runs happily as a container
- **Various small utilities** — anything that's a single Linux process with no special requirements

**Docker inside an LXC Container:**

- **Vaultwarden** (self-hosted Bitwarden) — Docker Compose, one YAML file, done
- **Uptime Kuma** — monitoring for my public-facing services
- **Jellyfin** — media server; the Docker image handles the complexity of libraries and dependencies
- **Homepage** — my homelab dashboard

The pattern you'll notice: I use Docker for anything with a well-supported image in the Docker Hub ecosystem. I use LXC for things where I want the control of a Linux system without Docker's abstraction. I use full VMs when isolation or OS requirements demand it.

## The RAM Math

This is where it gets practical. My DL380 G6 has 48GB of RAM. Here's roughly what different approaches cost:

A minimal Ubuntu Server VM uses about 500–800MB RAM just sitting idle. An LXC container running the same service might use 50–150MB. If you're running 10 services, that's potentially 5–8GB saved just by choosing LXC over VMs where possible.

Docker adds its own overhead, but it's relatively small once the Docker daemon is running. Running five Docker containers inside a single LXC container costs less than running five separate containers or VMs.

## The "Just Use Docker for Everything" Approach

A lot of homelab guides suggest this: one VM or one LXC container, Docker installed, everything running as Docker Compose stacks. It's simple, well-documented, and portable.

The downside is that you lose Proxmox's granular management benefits. With individual LXC containers, you can snapshot a specific service, set specific resource limits, and see exactly what each service is consuming. When everything is in one big Docker host, it's harder to isolate a misbehaving service or snapshot just one application.

I started with the "one big Docker VM" approach and gradually moved toward more granular separation as my homelab grew. Both work — it depends on how much management overhead you want to take on.

## Practical Tips

**LXC + Docker is a valid combo.** Running Docker inside an LXC container works well in Proxmox. You need to enable nesting in the LXC options (`features: nesting=1` in the config, or via the UI under Options → Features). Some guides will tell you this is unsupported — it works fine for homelab use.

**Give VMs and containers sensible names.** When you have 15 containers it becomes hard to track which is which. I prefix everything: `lxc-mkdocs`, `lxc-monitoring`, `vm-opnsense`, `vm-win11-test`.

**Use Proxmox's resource limits.** Every VM and container lets you set CPU and memory limits. Use them. An LXC container that goes rogue and eats all your RAM will take down everything else.

**Start smaller than you think.** You can always add resources later. Starting a new container with 4GB RAM when it only needs 256MB wastes resources you could use elsewhere.

## The Short Version

If you're new to Proxmox and trying to figure out where to start: run Docker inside an LXC container for most self-hosted services, and save full VMs for the things that genuinely need them (your firewall, anything non-Linux, anything that needs total isolation). You'll get the best of both worlds without burning your hardware resources unnecessarily.

---

_Curious how others split this — what's your Proxmox setup look like? LXC everywhere, Docker everywhere, or a mix? Let me know in the comments._
