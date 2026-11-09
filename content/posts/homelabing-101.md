---
title: "Homelabing 101"
date: 2026-11-05
draft: false
tags: ["server", "sysadmin"]
categories: ["personal-projects"]
---
### *The trees... The trees are speaking fed...*
<img src="/homelabing-101/hackerman.gif" alt="Hackerman" style="width: 75%;" />

A few months ago, I finally got my own internet connection!\
*Context: My place doesn't have fiber access, so I had been "borrowing" the neighbour's Wi-Fi for a few years.*\
Now I have a 5G router with __UNLIMITED__ data. Pretty cool!\
Finally I can access the internet, put all my personal life online, maybe SSH into my machines, and that's it...\
Right?

Just kidding, let's spend an entire weekend breaking configurations on a homeserver until 4 AM.

---
### What's the point anyway?
<img src="/homelabing-101/blade-runner.gif" alt="Blade Runner 2049" style="width: 100%;" />

Let's face it: we're not living in 1998 anymore. Tech used to be cool back then, but now it's pretty bad.\
We own basically nothing, and we have collectively traded our privacy for convenience. We are living in what people 30 years ago would have called a *dystopia*.\
And don't get me started on the death of software optimization. Developers now act like you’re the problem, expecting you to buy workstation-grade hardware just to run their unoptimized garbage.

Sure, you could just give up and willingly hand over all your personal data to Big Tech, or simply reject modernity and return to monke...
<img src="/homelabing-101/monke.gif" alt="Return to monke" style="width: 75%;" />

Luckily, a middle ground exists.

---
### Becoming more "tech self-sufficient".
<img src="/homelabing-101/stardew-valley.jpg" alt="Stardew Valley" style="width: 75%;" />

We all know that guy who used to live in a big city, and one day got so fed up that he sold everything to move to the countryside.\
He’s not totally isolated; he still enjoys modern conveniences and drives to town whenever he needs to. But by living farther away, **he bought his freedom**.

Maintaining a home server is the tech equivalent of that: instead of planting carrots and raising chickens, you host the services you actually need at home.

<img src="/homelabing-101/silicon-valley-fire.gif" alt="Silicon Valley - Computer on fire" style="width: 75%;" />

Of course, *with great tech freedom comes great configuration* overhead, so at first, it might feel like a downgrade, you are now responsible for your backups, your security, your updates…\
And to be fair, some services can be notoriously tricky to self-host (emails for example), so you probably won't host *everything* under your roof.

Still, if you’re a tech nerd, this is a fun and rewarding DIY project that actually means something—and it might even save you a decent chunk of money.

---
### Behold: The mainframe
<img src="/homelabing-101/asus-X751LJ.jpg" alt="Asus X751LJ a.k.a. 'The Mainframe'" style="width: 75%;" />

This is an Asus X751LJ I bought more than a decade ago. It was already underpowered back then (Intel Core i3-4005U @ 1.6GHz, 4GB of *soldered, non-upgradable* DDR3, Nvidia GT 920M) making it basically unusable as a modern computer, too bulky to bring around, ...\
This laptop is falling apart so badly that I was seriously considering handing it over to an Apple Store for recycling.\
This piece of crap died multiple times already, but as always I managed to bring it back to life.
<figure style="width: 45%;">
    <img src="/homelabing-101/spicy-pillow.jpg" alt="spicy pillow" />
    <figcaption>Oh, and I had to rip out the battery because it almost caught fire once</figcaption>
</figure>


A much better choice for a small, cheap, "powerful" but low-power server would be a Small Form Factor (SFF) **Lenovo ThinkCentre** or **Dell OptiPlex**, but surprisingly my Frankenstein setup gets the job done for now.

I upgraded its dying mechanical HDD for a 120GB SATA SSD I had lying around, installed Fedora Server and hooked up a 1TB HDD via USB 3.0.\
It's wired to a 5 Euro gigabit switch (TP-Link LS1005G) alongside my Mac Mini and my "gaming" computer (which runs Arch, btw), and the switch straight into the 5G router.\
From that point on, I never touched that computer again, everything is managed over SSH.

---
### What works (A.K.A. What's not currently broken)
<img src="/homelabing-101/lain.gif" alt="Let's all love Lain" style="width: 75%;" />

First off, the external drive is shared over SMB, with remote access handled via Tailscale. Everything else runs inside Docker containers.\
Here are the main ones:
- **Adguard Home**: Local DHCP Server and network-wide DNS ad-blocking (Disclaimer: Doesn't work for Youtube, you still need uBlock Origin for that)
- **Unbound**: Recursive DNS resolver (Mullvad partnered with Quad9, and Quad9 is pulling out of France due to censorship court orders; I keep Digitale Gesellschaft as an upstream fallback) 
- **Vaultwarden**: Self-hosted Bitwarden (TOTP and premium features unlocked for free)
- **Navidrome**: Music streaming using my personal library, with on-the-fly transcoding to spare my mobile data plan
- **qBittorrent**: To seed "Linux ISOs"
- **Dozzle** (Container logs), **Glance** (Dashboard), **Maddy** (SMTP, just in case a container needs to send out a critical email, like password recovery), ...

I have a bunch of other containers running mostly as experiments, but nothing worth mentioning here.\
Even though it makes sense given that everything runs headless without a GUI, I was pleasantly surprised by how many services I can comfortably host on this old laptop (20 containers right now, and I still have plenty of headroom left).\
Zero port forwarding on my router. Tailscale takes care of secure remote mesh access without exposing attack surfaces to the public web (and it can even be replaced with a self-hosted Headscale instance).

Power consuption is comparable to a modern lightbulb at idle and it's dead silent.

<img src="/homelabing-101/90s-computer-kid.jpg" alt="90s kid on a computer" style="width: 75%;" />

While this isn't a Docker tutorial, it's worth mentionning I realized at some point that maintaining a single `compose.yml`  for everything gets messy. That's why I have a master compose file using Docker's `include` syntax alongside subfolders:

```yaml
# ~/docker/compose.yml
name: server
include:
  - core/compose.yml
  - network/compose.yml
  - media/compose.yml
```
```text
~/docker
├── compose.yml
├── core/
│   ├── compose.yml
│   └── ...
├── media/
│   ├── compose.yml
│   └── ...
└── network/
    ├── compose.yml
    └── ...
```
This keeps each stack isolated and easy to maintain while still letting you control everything with `docker compose up -d` from the root directory.

---
### The compromises (and why it's not production-ready)
<img src="/homelabing-101/anime-computer.gif" alt="Anime girl on a computer" style="width: 75%;" />

While this is a fun, practical toy, it's not a datacenter. 

There are obvious bottlenecks to this setup, justy to name a few:
1. Zero data redundancy (easy fix: a DAS or a NAS)
2. Power instability (easy fix: a UPS)
3. Bandwidth asymmetry (easy fix: moving to a place with fiber)

Note: 5G is fast on downloads, but the upload speed and latency can vary wildly depending on weather, congestion, or God's plans maybe.\
Cellular 5G connections live behind Carrier-Grade NAT (CGNAT), so port-forwarding is usually not a thing. Tailscale is the only reason this setup is reachable from outside.

It is reliable enough for everyday personal use, but it's simply too risky to host critical stuff in here.\
Get a VPS for that.

And of course, money is, *once again*, the biggest limit. I would **LOVE** to host LLMs and other types of AI, or build a Docker Swarm cluster just for the love of the game, but it is simply not realistic for now...

---
### What's next?
<img src="/homelabing-101/yasuo.gif" alt="Yasuo IWAKURA" style="width: 75%;" />

This project started as an afternoon experiment to revive e-waste, but homelabbing is addictive.\
I still have a lot of services to try, a lot of cheap hardware to find on [Leboncoin](https://leboncoin.fr), and of course a lot of configs to break.

If you have an old laptop collecting dust in a closet, try to use it for this kind of projects instead of throwing it away.

Who knows? Maybe this little experiment could evolve into a cool F2F network one day. Though the real challenge with F2F is finding friends autistic enough to actually care about that stuff.\
Good luck with that.
