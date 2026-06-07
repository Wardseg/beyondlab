---
title: "NAS for Your Homelab: Which Storage Setup Actually Makes Sense"
date: 2026-06-04
draft: false
tags: ["nas", "hardware", "storage", "homelab", "proxmox", "how-to"]
description: "Dedicated NAS appliance, DIY TrueNAS box, or a second-hand enterprise unit? A practical breakdown of homelab storage options — and what I actually run."
---

Storage is the part of a homelab that people tend to figure out twice. Once when they set it up, and once after something goes wrong.

I'm running a [Synology RS814](/posts/my-homelab-hardware-hp-dl380-g6-breakdown/) with 16TB attached to my Proxmox host over NFS. It works well — DSM is polished, the NFS share just appears in Proxmox's storage pool, and the drives have been running without complaint for years. But it's also a 2013 rackmount unit I got as part of a package deal, not something I'd necessarily recommend buying new in 2026.

So let's talk about what actually makes sense for homelab storage today.

---

## What a homelab NAS needs to do

Home NAS and homelab NAS are not the same thing. A home NAS sits in a cupboard and serves files to your TV and phone. A homelab NAS does that _and_:

- Acts as a storage backend for Proxmox VM disks (via NFS or iSCSI)
- Stores VM backups from Proxmox Backup Server or similar
- Runs containers or services of its own (if you want)
- Handles potentially heavy concurrent I/O from multiple VMs
- Ideally connects at 2.5GbE or faster, not 1GbE

The NFS/iSCSI angle is the big one. If you're running Proxmox and want shared storage — for live migration, for keeping VM disks off the main host, for centralizing backups — your NAS needs to handle that reliably under load. That rules out some of the cheaper ARM-based units and changes which specs actually matter.

---

## Three paths, each with real tradeoffs

### Path 1: NAS appliance (Synology / QNAP)

The appliance route means buying a purpose-built NAS box, dropping drives in, and using the vendor's OS. It's the lowest-friction approach and there's nothing wrong with it — I'm doing this, after all.

**Synology** remains the most polished experience. DSM's NFS and iSCSI support is solid, the web UI is genuinely good, and it integrates cleanly with Proxmox. The negative: newer Synology models have started pushing Synology-branded drives more aggressively, with DSM showing warnings when you use third-party drives. Not a dealbreaker, but worth knowing.

For a homelab specifically, the [QNAP TS-464](https://www.amazon.com/dp/B0BQ5TWCL8/?tag=beyondlab-20) is the more interesting pick. Intel Celeron N5105, 8GB RAM standard (upgradeable to 16GB), dual 2.5GbE, M.2 slots for SSD cache, and a PCIe Gen 3 slot you can use for a 10GbE card later. That PCIe slot matters — it's the difference between hitting a networking ceiling at 300MB/s and being able to push closer to 1GB/s when you need it. QNAP's QTS software is more cluttered than DSM but it's capable, and the container support (via Container Station) is genuinely useful if you want to run services on the NAS itself.

For rackmount, QNAP also makes the [TS-464U](https://www.amazon.com/dp/B0B992N9LL/?tag=beyondlab-20) — same internals as the TS-464 but 1U form factor. If you're already running rack-mounted gear like I am, keeping the NAS in the rack as well is cleaner than having a desktop unit sitting somewhere else.

**When appliances make sense:** You want something that works without configuration overhead. You're not looking to learn ZFS internals. You value long-term support and a polished management interface over maximum flexibility.

### Path 2: DIY NAS running TrueNAS Scale or Unraid

This is where the homelab community tends to land eventually. You take a machine — often repurposed desktop hardware or a Mini-ITX build — and run TrueNAS Scale or Unraid on it.

The appeal is real: you get ZFS natively, more control over the storage stack, and you can reuse hardware you already have. TrueNAS Scale in particular integrates tightly with Proxmox — you can run it as a VM on Proxmox and pass HBA cards through to it, or run it on dedicated hardware and present storage via NFS/iSCSI/SMB.

The tradeoffs:

- **ZFS wants ECC RAM.** Not strictly required, but the argument for it is strong — ZFS uses RAM aggressively for caching (ARC), and bit-flips in memory can cause data corruption that ZFS can't detect. ECC RAM costs more and limits your platform options.
- **ZFS wants RAM, period.** 16GB minimum is a comfortable starting point; 32GB+ if you're doing serious caching. This isn't a platform you spin up on 4GB.
- **Unraid is simpler but costs money.** Unraid's license model (one-time fee, ~$60–$130 depending on tier) is reasonable but it's not free. TrueNAS Scale is free.

For the hardware, a used workstation or server with an HBA card is a common approach. An [LSI HBA in IT mode](https://www.amazon.com/dp/B00HAQU28M/?tag=beyondlab-20) passes drives directly through to TrueNAS without the controller interfering, which is what you want for ZFS. Pair it with a machine that has ECC support — used Xeon workstations like the HP Z440 or Dell Precision T5810 are popular choices and go for reasonable money on the secondhand market.

**When DIY makes sense:** You want to learn storage properly. You already have hardware to repurpose. You want ZFS features (snapshots, send/receive replication, scrubbing) and are willing to invest time in the setup.

### Path 3: Enterprise secondhand

This is how I ended up with the RS814 — it came alongside the server rather than as a deliberate purchase — but buying enterprise NAS hardware used is a legitimate strategy.

Used Synology rackmount units (RS1221+, RS1621xs+) show up secondhand at prices well below retail. Same with QNAP's enterprise line. The caveat is support: enterprise NAS units often run older versions of their respective OS, and you need to verify that the firmware is still receiving updates before committing.

The honest version of this advice: secondhand enterprise NAS is great if you know what you're getting. If you're not sure, stick to a new appliance or a DIY build where you control the components.

---

## The drives question

Regardless of which box you buy, the drives matter more than most people expect.

**Don't use desktop drives in a NAS.** Desktop drives use aggressive error recovery routines that can cause a RAID controller to drop the drive from the array while it tries to recover a sector. NAS drives are tuned differently — they time out faster and let the RAID/ZFS layer handle the recovery instead.

The two NAS drive families worth buying:

**Seagate IronWolf** — [4TB](https://www.amazon.com/dp/B07H289S79/?tag=beyondlab-20) is a solid entry point, [8TB](https://www.amazon.com/dp/B084ZV4DXB/?tag=beyondlab-20) if you want more capacity per bay. CMR recording (not SMR), NAS-rated workload, and IronWolf Health Management integrates with Synology DSM and QNAP QTS directly. Five-year warranty on the Pro variant if you want that.

**WD Red Plus** — the direct alternative to IronWolf. Make sure it's the _Plus_ variant; the regular WD Red went through a period of using SMR recording which caused real problems in RAID arrays. The Plus line is CMR and fine.

For a homelab specifically, I'd size up on drives rather than bay count if you can — 2× 8TB in a 2-bay unit is more useful than 4× 2TB in a 4-bay unit, and you leave yourself expansion room.

---

## Connecting your NAS to Proxmox

Once you have a NAS running, adding it to Proxmox as storage takes about five minutes.

In Proxmox, go to **Datacenter → Storage → Add → NFS**:

- **ID:** pick a name (e.g., `nas-nfs`)
- **Server:** your NAS IP
- **Export:** the NFS share path from your NAS config
- **Content:** tick what you want — Disk image, Container, Backup, ISO, etc.

For better performance, enable the NFS async option on the NAS side (check your NFS export settings in DSM or QTS). Also make sure the connection between your Proxmox host and NAS is on a dedicated VLAN or at minimum a dedicated NIC — mixing NAS traffic with VM traffic on the same interface causes latency spikes.

If you're using iSCSI instead of NFS, the setup is slightly more involved but gives you better performance for VM disk I/O because iSCSI presents a block device rather than a file system.

---

## What I'd actually buy today

If I were setting up a fresh homelab with a budget for dedicated NAS hardware:

For a **desktop/tower homelab**, the [QNAP TS-464](https://www.amazon.com/dp/B0BQ5TWCL8/?tag=beyondlab-20) with 2–4× [Seagate IronWolf 8TB](https://www.amazon.com/dp/B084ZV4DXB/?tag=beyondlab-20) drives. PCIe slot for 10GbE later, dual 2.5GbE today, Intel Quick Sync for transcoding if needed, enough RAM to run containers alongside storage duties.

For a **rack-mounted homelab**, the [QNAP TS-464U](https://www.amazon.com/dp/B0B992N9LL/?tag=beyondlab-20) with the same drive pairing. Same internals as the desktop version, 1U form factor, fits cleanly in the rack alongside the server.

For **maximum control and ZFS**, a DIY build running TrueNAS Scale on repurposed Xeon workstation hardware with an LSI HBA. More setup time, more learning, more reward.

My RS814 will keep doing its job until something breaks or I run out of capacity. When that day comes, I'll probably go DIY — the ZFS pull is real and the hardware cost is lower than it used to be. But if you want something solid without the configuration overhead, the QNAP appliance route is where I'd point you.

---

## One thing that doesn't get said enough

Your NAS backup situation needs to exist from day one, not "once I get around to it."

A NAS running RAID is not a backup. RAID protects you from a single drive failure. It does not protect you from ransomware, accidental deletion, fire, flood, or a misconfigured `rm -rf`. The 3-2-1 rule applies: 3 copies of your data, on 2 different media, with 1 offsite.

Synology's Hyper Backup and QNAP's Hybrid Backup Sync can both push encrypted snapshots to Backblaze B2 or similar object storage for a few dollars a month. Set it up before you put anything on the NAS that you care about.

---

_Disclosure: Some links in this post are affiliate links. If you buy through them, I get a small commission at no extra cost to you._
