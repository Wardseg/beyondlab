---
title: "Running OPNsense in a Proxmox VM - full setup guide"
date: 2026-05-26
draft: false
tags: ["networking", "opnsense", "proxmox", "firewall", "guide"]
description: "A complete walkthrough for setting up OPNsense as a virtual machine inside Proxmox — including network configuration, initial setup, and first rules."
---

Running OPNsense as a VM inside Proxmox is one of the best decisions I made
when building my homelab. You get a full-featured firewall with the added
benefits of snapshots, easy backups, and no dedicated hardware required.

The setup is a bit more involved than installing on bare metal — mostly because
of how network interfaces need to be configured in Proxmox. But once you
understand the network topology, it clicks quickly.

This guide walks through the full process.

## The network topology

Before touching anything, understand what we're building:

Internet → Physical NIC (WAN) → OPNsense VM → Virtual Bridge (LAN) → Your devices

Proxmox needs two network interfaces for OPNsense:

- **WAN interface** — connected to your physical NIC that faces your ISP router
  or modem. OPNsense will get its upstream IP here.
- **LAN interface** — a virtual bridge (vmbr) that your other VMs and LXC
  containers connect to. OPNsense acts as the gateway for this network.

If you only have one physical NIC, you can use VLANs to separate WAN and LAN
traffic — but two NICs is simpler and recommended for a first setup.

## Step 1 — Configure network bridges in Proxmox

In the Proxmox web interface go to:
**Node → Network → Create → Linux Bridge**

Create two bridges:

**vmbr0 — WAN bridge**

- Bridge ports: your physical WAN NIC (e.g. `enp1s0`)
- Leave CIDR and gateway empty — OPNsense will handle this

**vmbr1 — LAN bridge**

- Bridge ports: leave empty (this is an internal virtual bridge)
- Set a CIDR if you want Proxmox itself on this network (e.g. `192.168.1.1/24`)

Click Apply Configuration when done.

## Step 2 — Download the OPNsense ISO

Go to [opnsense.org/download](https://opnsense.org/download/) and download
the latest DVD ISO image. At the time of writing that's OPNsense 24.x.

Upload it to Proxmox:
**Storage → local → ISO Images → Upload**

## Step 3 — Create the OPNsense VM

Click **Create VM** in the Proxmox web interface and work through the wizard:

**General**

- Name: `opnsense`
- VM ID: leave as default

**OS**

- Select the OPNsense ISO you uploaded
- Guest OS type: Other

**System**

- Leave defaults

**Disks**

- Bus: VirtIO SCSI
- Disk size: 16GB is plenty for OPNsense itself

**CPU**

- Cores: 2
- Type: host (better performance)

**Memory**

- 2048MB (2GB) minimum — 4GB recommended if you plan to run IDS/IPS

**Network — first interface (WAN)**

- Bridge: vmbr0
- Model: VirtIO

Click **Add** to add a second network interface:

**Network — second interface (LAN)**

- Bridge: vmbr1
- Model: VirtIO

Click Finish. Don't start the VM yet.

## Step 4 — Install OPNsense

Start the VM and open the Console in Proxmox.

The OPNsense installer boots. When prompted:

- Log in with username `installer` and password `opnsense`
- Select **Install (ZFS)** or **Install (UFS)** — UFS is simpler for a VM
- Select your virtual disk
- Confirm and proceed

The installation takes a few minutes. When complete, remove the ISO
(VM → Hardware → CD/DVD → Edit → No media) and reboot.

## Step 5 — Initial console configuration

After reboot OPNsense boots to a console menu. You need to assign interfaces
before you can access the web interface.

**Option 1 — Assign interfaces**

Select option `1` from the menu.

- Do you want to configure LAGGs? → No
- Do you want to configure VLANs? → No (for now)
- WAN interface → select the interface connected to vmbr0 (usually `vtnet0`)
- LAN interface → select the interface connected to vmbr1 (usually `vtnet1`)
- Confirm the assignment

**Option 2 — Set interface IP addresses**

Select option `2` from the menu.

Configure LAN:

- Select LAN interface
- IPv4: `192.168.1.1`
- Subnet: `24`
- No IPv6 for now
- Enable DHCP server on LAN: Yes
- DHCP range: `192.168.1.100` to `192.168.1.200`

## Step 6 — Access the web interface

From a machine connected to your LAN (or from a VM on vmbr1), open a browser
and go to:

`https://192.168.1.1`

Log in with:

- Username: `root`
- Password: `opnsense`

You're in. The setup wizard will walk you through basic configuration.

## Step 7 — Run the setup wizard

The setup wizard covers:

**General information**

- Hostname: `opnsense`
- Domain: `home.arpa` or your local domain
- Primary DNS: `1.1.1.1` (you can change this later to Unbound)

**Time server**

- Leave defaults

**WAN interface**

- If your ISP uses DHCP (most do): select DHCP
- If you're using a static IP: configure accordingly

**LAN interface**

- Confirm the IP you set earlier (`192.168.1.1`)

**Set root password**

- Change from the default immediately

Click Reload when done.

## Step 8 — Essential first steps after setup

**Update OPNsense**

Go to **System → Firmware → Updates** and install all available updates.
Always do this before anything else.

**Enable SSH (optional)**

Go to **System → Settings → Administration** and enable SSH if you want
terminal access. Restrict it to LAN only.

**Review firewall rules**

Go to **Firewall → Rules → LAN**. By default OPNsense allows all outbound
traffic from LAN — this is fine to start with. You'll tighten rules as you
learn more.

**Set up DNS with Unbound**

Go to **Services → Unbound DNS → General** and enable Unbound. This gives
you a local DNS resolver with DNSSEC support — much better than forwarding
everything to your ISP's DNS.

## Snapshot before you go further

Now that you have a working OPNsense installation, take a snapshot in Proxmox
before making any further changes:

In Proxmox: **opnsense VM → Snapshots → Take Snapshot**

Name it `fresh-install`. This gives you a clean restore point if anything
goes wrong while experimenting with rules and settings.

## What's next

With OPNsense running you have the foundation for a properly segmented,
controlled network. The next steps are:

- Set up VLANs to separate IoT, trusted, and lab traffic
- Configure DNS-over-TLS with Unbound and Cloudflare
- Set up Kea DHCP for proper IP management
- Add an IDS/IPS with Suricata

---

_Next up: VLANs explained simply — and why I use them._
