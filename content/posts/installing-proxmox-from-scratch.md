---
title: "Installing Proxmox from scratch — step by step"
date: 2026-05-23
draft: false
tags: ["homelab", "proxmox", "virtualisation", "guide", "beginners"]
description: "A complete walkthrough for installing Proxmox VE on bare metal — from downloading the ISO to logging into the web interface for the first time."
---

Proxmox is one of those tools that looks more intimidating than it is. The
installation is straightforward, the web interface is clean, and within an hour
of starting you can have your first virtual machine running.

This guide walks through the full process from scratch.

## What you'll need

- A machine to install Proxmox on (dedicated hardware or a repurposed PC)
- A USB drive (at least 2GB) for the installer
- A network connection (wired preferred for the install)
- A separate machine to access the web interface from

Proxmox runs best on dedicated hardware — it's a Type 1 hypervisor designed to
be the only thing running on the machine. Don't install it alongside another OS.

## Step 1 — Download the Proxmox ISO

Go to [proxmox.com/downloads](https://www.proxmox.com/en/downloads) and download
the latest Proxmox VE ISO. At the time of writing that's Proxmox VE 9.x.

## Step 2 — Create a bootable USB drive

You'll need to write the ISO to a USB drive. The best tool for this is
[Rufus](https://rufus.ie) on Windows or [Balena Etcher](https://etcher.balena.io)
on any platform.

In Rufus:

1. Select your USB drive
2. Select the Proxmox ISO
3. Leave everything else as default
4. Click Start

This will erase everything on the USB drive, so make sure there's nothing
important on it.

## Step 3 — Boot from USB

Insert the USB drive into your server and boot from it. You may need to press
F11, F12, or Delete during startup to access the boot menu — this varies by
motherboard.

Select the USB drive from the boot menu.

## Step 4 — Run the Proxmox installer

The Proxmox installer boots into a graphical interface. Work through the steps:

**License agreement** — accept it and continue.

**Target disk** — select the drive you want to install Proxmox on. This will be
wiped completely. If you have multiple drives, choose carefully.

**Location and timezone** — set your country and timezone.

**Password and email** — set the root password for the Proxmox host. Use
something strong and store it in your password manager. The email is used for
system alerts.

**Network configuration** — this is the most important step:

- Select your network interface (usually the wired one)
- Set a static IP address for the Proxmox host — something like `192.168.1.10`
- Set the correct gateway (your router's IP, usually `192.168.1.1`)
- Set DNS — you can use `1.1.1.1` for now
- Set the hostname — something like `pve.yourdomain.local`

**Review and install** — check everything looks correct and click Install.

The installation takes 5-10 minutes. The machine will reboot when it's done.

## Step 5 — Access the web interface

Once the machine has rebooted, open a browser on another machine on the same
network and go to:

https://YOUR_PROXMOX_IP:8006

For example: `https://192.168.1.10:8006`

You'll get a certificate warning — this is expected since Proxmox uses a
self-signed certificate by default. Accept it and continue.

Log in with:

- Username: `root`
- Password: the password you set during installation
- Realm: `Linux PAM standard authentication`

You're in.

## Step 6 — Remove the subscription nag (optional)

Proxmox shows a popup on login reminding you to subscribe. It doesn't affect
functionality but it's annoying. To remove it, open the Shell from the
Proxmox web interface and run:

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{[[:space:]]+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" \
  /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && \
  systemctl restart pveproxy.service
```

Refresh the browser and the nag is gone.

## Step 7 — Add the community repository

By default Proxmox points to its enterprise repository which requires a
subscription. Switch to the community repository so you can update freely:

In the web interface go to:
**Node → Repositories → Add → No-Subscription**

Then disable the enterprise repository by selecting it and clicking Disable.

Now run an update:

```bash
apt update && apt dist-upgrade -y
```

## Step 8 — Create your first VM

Click **Create VM** in the top right of the web interface.

Work through the wizard:

- **General** — give it a name
- **OS** — upload or select an ISO (you'll need to upload one first via
  Storage → local → ISO Images → Upload)
- **System** — leave defaults for most use cases
- **Disks** — set the disk size you need
- **CPU** — start with 2 cores
- **Memory** — depends on the OS; 2GB is fine for most Linux VMs
- **Network** — leave as default (vmbr0)

Click Finish, then Start.

## What's next

With Proxmox running you have the foundation for everything else. Your next
steps are typically:

- Set up a network bridge for VM networking
- Create a few LXC containers for lightweight services
- Configure storage — add your NAS or additional drives
- Set up Proxmox Backup Server for snapshots and backups

---

_Next up: How I sourced enterprise hardware for cheap._
