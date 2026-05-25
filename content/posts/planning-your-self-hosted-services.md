---
title: "Planning your self-hosted services — what to run and why"
date: 2026-05-21
draft: false
tags: ["homelab", "self-hosted", "services", "beginner", "subscriptions"]
description: "Before you start spinning up containers, it helps to know what you actually want to run. Here's how to think about planning your self-hosted stack — and the resource that changed how I approach it."
---

I'll be honest — I didn't plan my self-hosted stack.

I started with one service, liked the feeling of owning my own data, and just kept
adding things. Vaultwarden, then MkDocs, then more. Recently I added Karakeep.
Before that something else. Each one solving a problem, replacing a subscription,
or just satisfying curiosity.

It works — but looking back, a bit of upfront planning would have saved me time,
avoided some redundancy, and helped me think more clearly about what I actually
needed versus what just seemed interesting at the time.

If you're starting out, this post is the planning guide I wish I'd had.

## Start with the why

Before you look at any software, ask yourself why you want to self-host.

The reasons matter because they point you toward different categories of services:

**Cutting subscriptions** — If your motivation is getting off the subscription
treadmill, focus on direct replacements for things you're already paying for.
Cloud storage, password managers, photo backups, media streaming. Every service
you replace is money back in your pocket every month.

**Privacy and data ownership** — If you don't like your data sitting on someone
else's servers, focus on services that handle sensitive information. Notes,
documents, calendars, contacts, passwords.

**Learning** — If you want to build skills, pick services that challenge you
technically. A reverse proxy, a monitoring stack, a Git server. The complexity
is the point.

**All of the above** — which is where most people end up, honestly.

My starting point was subscriptions. But curiosity took over quickly and the
learning became just as motivating.

## The resource you need to bookmark

Before you decide what to run, spend some time with
[Awesome Self-Hosted](https://github.com/awesome-selfhosted/awesome-selfhosted).

It's a curated list of hundreds of self-hosted applications, organised by
category — everything from password managers and photo galleries to media
servers, note-taking apps, and monitoring tools. It's the most comprehensive
overview of what's possible and it's kept up to date by the community.

Don't try to read it all at once. Use it as a reference. When you have a
specific need — "I want to replace Google Photos" — go to the Photo Galleries
section and compare your options.

## Think in categories, not individual apps

A useful way to approach your stack is to think in categories rather than
jumping straight to specific apps:

**Replace your cloud storage**

- [Nextcloud](https://nextcloud.com) — the most feature-complete option.
  Files, calendar, contacts, notes, and more in one platform.
- [Seafile](https://www.seafile.com) — faster and simpler if you just need
  file sync.

**Replace your password manager**

- [Vaultwarden](https://github.com/dani-garcia/vaultwarden) — a lightweight,
  self-hosted implementation of the Bitwarden API. This is what I run.
  Compatible with all official Bitwarden clients.

**Replace Google Photos**

- [Immich](https://immich.app) — the best self-hosted photo and video backup
  solution right now. Fast, beautiful, and actively developed.

**Replace your media subscriptions**

- [Jellyfin](https://jellyfin.org) — free, open source media server.
  Stream your own movies, TV shows, and music anywhere.

**Bookmark and read-later**

- [Karakeep](https://karakeep.app) — bookmark everything with a touch of AI.
  This is what I'm currently running and genuinely enjoying.
- [Wallabag](https://wallabag.org) — save articles to read later, without
  the tracking.

**Notes and knowledge base**

- [Obsidian](https://obsidian.md) — not self-hosted itself, but pairs well
  with a self-hosted sync backend.
- [Joplin](https://joplinapp.org) — open source note-taking with
  self-hosted sync support.
- [Silverbullet](https://silverbullet.md) — a newer option worth watching.

**Monitoring your stack**

- [Uptime Kuma](https://github.com/louislam/uptime-kuma) — simple, beautiful
  uptime monitoring for your services.
- [Grafana](https://grafana.com) + [Prometheus](https://prometheus.io) —
  more powerful monitoring if you want metrics and dashboards.

## What I'm building next

My stack is still growing. I'm currently working on two projects that aren't
quite ready to talk about yet — a note-taking app with AI integration, and a
custom dashboard to monitor all my services. Both born from the same itch:
the existing tools are good, but I wanted something that fits exactly how I
work.

That's the thing about self-hosting. It doesn't stop at running other people's
software. Eventually you start building your own.

## Practical advice before you start

**Don't run everything at once.** Pick two or three services that solve a real
problem for you. Get them stable, understand how they work, then add more.

**Think about maintenance.** Every service you run needs occasional updates,
backups, and troubleshooting. A leaner stack is easier to maintain than a
sprawling one.

**Check resource requirements.** Some services are lightweight (Vaultwarden
uses almost no RAM). Others are hungry (Nextcloud, Immich). Know what your
hardware can handle.

**Have a backup strategy from day one.** If you're replacing Google Drive with
Nextcloud, your data is only as safe as your backup setup. Don't skip this step.

## The honest summary

Self-hosting is one of the most satisfying things you can do as someone who
cares about technology. Owning your stack, controlling your data, and replacing
subscriptions one by one — it compounds over time in ways that are hard to
explain until you've done it.

But it starts with knowing what you want to build. Take twenty minutes with
[Awesome Self-Hosted](https://github.com/awesome-selfhosted/awesome-selfhosted),
think about the subscriptions you resent most, and start there.

---

_Next up: Why I chose Proxmox over ESXi and Hyper-V._
