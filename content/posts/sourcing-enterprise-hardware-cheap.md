---
title: "How I sourced enterprise hardware for cheap"
date: 2026-05-24
draft: false
tags: ["homelab", "hardware", "budget", "second-hand", "server"]
description: "Enterprise hardware is surprisingly affordable if you know where to look and what to check. Here's how to find great deals without getting burned."
---

One of the biggest surprises when I got into homelabbing was just how cheap
enterprise hardware can be.

Servers that cost €15,000 new, sold for €50-150 on the second-hand market a
decade later. Network switches that ran the backbone of a company's infrastructure,
available for less than a pizza. 10-gigabit networking gear that was out of reach
for home users a few years ago, now completely affordable second-hand.

The reason is simple: businesses refresh their hardware on a predictable cycle —
typically every 3-5 years. When they do, the old gear gets sold off, donated, or
scrapped. Most of it still has years of useful life left.

Here's how to find it and how to evaluate it before you buy.

## Where to look

**eBay** is the most reliable global source for second-hand IT hardware. Search
specifically — include the exact model number, generation, and any specs you need.
Always check sold listings (filter by Completed Items) to understand real market
value before bidding or buying.

**Local marketplaces** — Facebook Marketplace, Craigslist, Gumtree, and local
equivalents vary by country but are worth checking regularly. Local sellers
often price lower because they want to avoid shipping hassle. You can inspect
before buying and walk away if something looks wrong.

**IT liquidation auctions** — companies like GovPlanet, BidSpotter, and various
regional auction houses regularly sell off decommissioned IT equipment. The deals
can be excellent but you're often buying untested gear in bulk. Good for
experienced buyers, riskier for beginners.

**Friends and colleagues** — my DL380 G6 came from a friend who wasn't using it
anymore. Never underestimate your network. IT professionals regularly have old
gear sitting in garages or storage rooms they'd happily give away or sell cheaply.

## What to look for

**Server generation matters** — newer generations are generally more power
efficient. An HP DL380 G6 (2009) draws significantly more power than a G9 or G10.
Factor running costs into your evaluation — a server that costs €80 but draws
200W 24/7 will cost you more in electricity over a year than the hardware itself.

**Single socket vs dual socket** — dual socket servers give you more cores but
also more power draw. For a homelab running a handful of VMs, a single socket
machine is usually plenty.

**RAM** — enterprise servers often come with a decent amount of RAM already
installed. Check what's included and what the maximum supported is. DDR3
registered ECC RAM is extremely cheap second-hand and easy to upgrade.

**Drive bays** — check how many drive bays are included and whether caddies
(the sleds drives sit in) are included. Missing caddies are a common gotcha —
they're often sold separately and can add up in cost.

**Rails and accessories** — rack rails are frequently missing from second-hand
servers. If you're rack mounting, confirm rails are included or budget for them
separately.

## What to check before buying

**Drive health** — if the server comes with drives, check SMART data before
trusting them with anything important. I use a USB dock alongside
[CrystalDiskInfo](https://crystalmark.info/en/software/crystaldiskinfo/) on
Windows or `smartctl` on Linux. Key indicators:

- Reallocated Sectors Count should be 0
- Power On Hours gives you context on drive age
- Any pending or uncorrectable sectors are red flags

**POST test** — ask the seller to confirm the server POSTs (powers on and reaches
BIOS) before you buy. A photo or video of the machine posting is a reasonable
request for any significant purchase.

**Noise and power draw** — enterprise servers are not quiet. They're designed for
data centres with proper cooling, not spare rooms. If noise is a concern, research
the specific model before buying. Some generations are significantly louder than
others, and some have fan mods available to reduce noise.

**iDRAC / iLO access** — most enterprise servers have a remote management
interface (Dell calls it iDRAC, HP calls it iLO). Confirm it's accessible and
ideally factory reset before buying — you don't want someone else's credentials
on your management interface.

## My approach

When I'm evaluating a potential purchase I run through a simple checklist:

1. What will this actually run? Does the spec match the use case?
2. What has this model sold for recently on eBay?
3. What's the estimated annual power cost at my electricity rate?
4. Are there known issues with this model I should know about?
5. Does it come with drives, caddies, rails?
6. Can the seller confirm it posts?

Most bad purchases come from skipping one of these steps.

## The sweet spots right now

Based on current second-hand market prices, these categories offer excellent
value:

**HP DL360/DL380 G8/G9** — a generation newer than mine, more power efficient,
still very cheap. Great all-round homelab servers.

**Dell PowerEdge R620/R630** — similar value proposition to the HP equivalents.
Well documented, easy to find parts for.

**Mini PCs (refurbished)** — if noise and power consumption matter, refurbished
[HP EliteDesk](https://www.amazon.com/s?k=hp+elitedesk+800+g4+mini&tag=beyondlab-20)
or [Dell OptiPlex Micro](https://www.amazon.com/s?k=dell+optiplex+7060+micro&tag=beyondlab-20)
units are excellent value. Quiet, efficient, and capable of running a small
Proxmox cluster.

**Managed switches** — Cisco Catalyst and HP ProCurve switches from enterprise
refresh cycles are remarkably cheap and give you proper VLAN support, LACP, and
management features you'd pay a premium for on consumer gear.

## The bottom line

The second-hand enterprise hardware market is one of the best-kept secrets in
homelabbing. With patience, a bit of research, and a clear idea of what you
actually need, you can build a capable lab for a fraction of the cost of buying
new.

The key is knowing what to look for, what to check, and what to walk away from.

---

_Next up: What is a firewall and why your home network needs one._
