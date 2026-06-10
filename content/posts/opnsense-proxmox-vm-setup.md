---
title: "How I Set Up OPNsense as a VM in Proxmox (And Why You Should Too)"
date: 2026-06-07
draft: false
tags: ["how-to", "proxmox", "opnsense", "networking", "firewall"]
description: "Running OPNsense as a Proxmox VM gave me full firewall control without extra hardware. Here's exactly how I set it up and why it was worth every minute."
---

My home network used to be whatever my ISP router decided it should be. No VLANs, no proper firewall rules, no visibility into what was actually happening. When I started running Proxmox on the HP DL380 G6, one of the first things I wanted to fix was the network layer — and OPNsense running as a VM turned out to be exactly the right move.

This post covers why I went down this route, what you need to make it work, and a step-by-step walkthrough of the setup.

## Why Run a Firewall as a VM?

The obvious question: isn't it a bit weird to run your firewall _inside_ the network it's supposed to be protecting?

Sort of — but in practice it works well, as long as you understand the trade-off. If Proxmox goes down, so does your firewall. For a homelab, that's usually fine. You're not running production infrastructure. And the benefits are real:

- **No extra hardware.** A dedicated firewall appliance costs money and takes up a slot. OPNsense in a VM uses resources you're already paying for.
- **Snapshots.** Before a major OPNsense upgrade, take a snapshot. If something breaks, you're back in 30 seconds.
- **Flexibility.** You can run multiple virtual NICs, experiment with VLANs, and reconfigure the network without touching a single cable.

The key hardware requirement is having at least two physical NICs — one for WAN (your modem/ISP connection) and one for LAN. You'll pass these through to the OPNsense VM.

## What You Need

- A Proxmox VE host with two physical network interfaces
- The OPNsense ISO (download from [opnsense.org](https://opnsense.org/download/))
- At least 2GB RAM and 2 vCPUs allocated to the VM (I use 4GB and 2 vCPUs)
- About 16GB of disk space for the VM

## Step 1: Upload the OPNsense ISO

In the Proxmox web UI, go to your storage (I use `local`) → **ISO Images** → **Upload**. Upload the OPNsense `.iso` file you downloaded.

Alternatively, if your Proxmox host has internet access, you can use the **Download from URL** option and paste the direct link.

## Step 2: Create the VM

Click **Create VM** in the top right. Walk through the wizard:

- **General**: Give it a name like `opnsense-fw` and a VM ID (I use 100 to keep it easy to remember)
- **OS**: Select your uploaded ISO; Guest OS type is **Other**
- **System**: Leave defaults (SeaBIOS is fine for OPNsense)
- **Disks**: 16GB is plenty; VirtIO SCSI controller recommended
- **CPU**: 2 cores minimum
- **Memory**: 2048 MB minimum, I use 4096 MB
- **Network**: Add your **LAN** NIC here (VirtIO driver); we'll add WAN after

After creating the VM, go to its **Hardware** tab and click **Add → Network Device**. Add your second NIC (the WAN-facing one). Now the VM has two network interfaces.

## Step 3: Configure NIC Passthrough (or Linux Bridge)

There are two approaches here:

**PCIe Passthrough** gives OPNsense direct access to the physical NIC — best performance, but requires IOMMU enabled in your BIOS and Proxmox config. On the DL380 G6 this works well once you enable VT-d in the BIOS.

**Linux Bridge** (the simpler option) creates a virtual bridge in Proxmox that connects the physical NIC to the VM. Less overhead than you'd think, and much easier to set up. This is what I'd recommend starting with.

For the Linux bridge approach, go to **Proxmox node → Network → Create → Linux Bridge**:

- Bridge port: your WAN physical NIC (e.g., `eno1`)
- No IP address on this bridge — OPNsense will handle it

Do the same for your LAN NIC.

Then make sure your OPNsense VM's two network devices are attached to these two bridges respectively.

## Step 4: Install OPNsense

Start the VM and open the console. OPNsense boots from the ISO. Log in with `installer` / `opnsense` and follow the prompts. It's a straightforward install — select your target disk, choose the default options, and let it run.

When it reboots, it'll ask you to assign interfaces. This is important:

- The **WAN** interface should be the one connected to your modem/ISP bridge
- The **LAN** interface is what everything else on your network will connect to

OPNsense will start its DHCP server on LAN automatically. Default LAN IP is `192.168.1.1`.

## Step 5: Initial Configuration

Connect a device to your LAN and navigate to `https://192.168.1.1`. Default credentials are `root` / `opnsense`.

Run through the setup wizard:

1. Set your hostname and DNS servers (I use `1.1.1.1` and `9.9.9.9`)
2. Configure WAN — usually DHCP if your modem handles the PPPoE, or PPPoE directly if you want OPNsense to dial out
3. Set a new admin password

At this point you have a working firewall. Everything on LAN can reach the internet; nothing from WAN can reach LAN unprompted.

## What I Did Next

Once the basics were running, I went further:

**VLANs** — I created separate VLANs for trusted devices, IoT gear, and a lab/homelab segment. This keeps my Proxmox management interface isolated from the network my smart plugs are on. OPNsense handles inter-VLAN routing and firewall rules between them.

**Unbound DNS** — OPNsense has a built-in DNS resolver. I use it to block ad domains network-wide and to resolve local hostnames (so I can reach services by name instead of IP).

**Suricata IDS** — Optional, but OPNsense has Suricata built in. I have it running in detection-only mode on the WAN interface, mostly out of curiosity about what's hitting the edge.

**Automatic backups** — OPNsense can export its config as an XML file. I have a cron job that pulls this to a folder on my Synology RS814 weekly.

## The Gotchas

A few things that tripped me up:

- **Console access matters.** If you misconfigure the LAN interface, you'll lose web UI access. Keep the Proxmox console open so you can fix things via CLI.
- **VirtIO drivers.** Use VirtIO for network adapters in the VM — the performance difference over e1000 is noticeable when you're moving a lot of traffic.
- **Snapshots before upgrades.** OPNsense updates are usually smooth, but I've had one upgrade scramble a plugin config. Always snapshot first.
- **IOMMU for passthrough.** If you want true PCIe passthrough rather than Linux bridges, make sure `intel_iommu=on` (or `amd_iommu=on`) is in your Proxmox GRUB config.

## Is It Worth It?

Absolutely. Running OPNsense as a Proxmox VM costs nothing extra if you already have the hardware, and it turns your homelab network into something you actually understand and control. The combination of proper firewall rules, DNS control, and VLANs makes a bigger difference than almost any other homelab upgrade I've made.

If you're already running Proxmox and your network is still "whatever the ISP router does," this is the next thing to set up.

---

_Have questions about the setup or want to know more about VLAN configuration? Drop a comment below._
