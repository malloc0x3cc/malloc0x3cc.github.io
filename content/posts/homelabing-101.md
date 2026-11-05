---
title: "Homelabing 101"
date: 2026-11-05
draft: false
tags: ["server", "sysadmin"]
categories: ["personal-projects"]
---
### *The trees... The trees are speaking fed...*
![hackerman](/homelabing-101/hackerman.gif)

A few months ago, I finally got my own internet connection!\
*Context: My place doesn't have fiber access, so I had been "borrowing" the neighbour's Wi-Fi for a few years.*\
Now I have a 5G router with __UNLIMITED__ data. Pretty cool!\
Finally I can access the internet, put all my personal life online, maybe SSH into my machines, and that's it...\
Right?

Just kidding, let's spend an entire weekend breaking configurations on a homeserver until 4 AM.

---
### What's the point anyway?
![Blade Runner 2049](/homelabing-101/blade-runner.gif)

Let's face it: we're not living in 1998 anymore. Tech used to be cool back then, but now it's pretty bad.\
We own basically nothing, and we have collectively traded our privacy for convenience. We are living in what people 30 years ago would have called a *dystopia*.\
And don't get me started on the death of software optimization. Developers now act like you’re the problem, expecting you to buy workstation-grade hardware just to run their unoptimized garbage.

Sure, you could just give up and willingly hand over all your personal data to Big Tech, or simply reject modernity and return to monke...
![return to monke](/homelabing-101/monke.gif)

Luckily, a middle ground exists.

---
### Becoming more "tech self-sufficient".
![Stardew Valley](/homelabing-101/stardew-valley.png)

We all know that guy who used to live in a big city, and one day got so fed up that he sold everything to move to the countryside.\
He’s not totally isolated; he still enjoys modern conveniences and drives to town whenever he needs to. But by living farther away, **he bought his freedom**.

Maintaining a home server is the tech equivalent of that: instead of planting carrots and raising chickens, you host the services you actually need at home.

![Silicon Valley - Server Fire](/homelabing-101/silicon-valley-fire.gif)

Of course, *with great tech freedom comes great configuration* overhead, so at first, it might feel like a downgrade, you are now responsible for your backups, your security, your updates…\
And to be fair, some services can be notoriously tricky to self-host (emails for example), so you probably won't host *everything* under your roof.

Still, if you’re a tech nerd, this is a fun and rewarding DIY project that actually means something—and it might even save you a decent chunk of money.

---
### Behold: The mainframe
![Asus X751LJ](/homelabing-101/asus-X751LJ.jpg)

This is an Asus X751LJ I bought more than a decade ago. It was already underpowered back then (Intel Core i3-4005U @ 1.6GHz, 4GB of *soldered, non-upgradable* DDR3, Nvidia GT 920M) making it basically unusable as a modern computer, too bulky to bring around, ...\
This laptop is falling apart so badly that I was seriously considering handing it over to an Apple Store for recycling.\
In fact, it has died more times than I can count, yet it's still running today. Not because it's indestructible, but because I stubbornly refuse to let it die.\
Oh, and I had to rip out the battery because it almost caught fire once.
![spicy pillow](/homelabing-101/spicy-pillow.jpg)


A much better choice for a small, cheap, "powerful" but low-power server would be a Small Form Factor (SFF) **Lenovo ThinkCentre** or **Dell OptiPlex** paired with a 2-bay NAS for RAID 1 storage.\
But surprisingly, my Frankenstein setup gets the job done for now.

I upgraded its dying mechanical HDD for a 120GB SATA SSD I had lying around, installed Fedora Server and hooked up a 1TB HDD via USB 3.0.\
It's wired to a 5 Euro gigabit switch (TP-Link LS1005G) alongside my Mac Mini and my "gaming" computer, and the switch straight into the 5G router.\
From that point on, I never touched that computer again, everything is managed over SSH.

---
### What works (A.K.A. What's not currently broken)
![Let's all love Lain](/homelabing-101/lain.gif)

First off, the external drive is shared over SMB, with remote access handled via Tailscale. Everything else runs inside Docker containers.\
Here are the main ones:
- **Adguard Home**: Local DHCP Server and network-wide DNS ad-blocking (Disclaimer: Doesn't work for Youtube, you still need uBlock Origin for that)
- **Unbound**: Recursive DNS resolver (Mullvad partnered with Quad9, and Quad9 is pulling out of France due to censorship court orders; I keep Digitale Gesellschaft as an upstream fallback) 
- **Vaultwarden**: Self-hosted Bitwarden (TOTP and premium features unlocked for free)
- **Navidrome**: Music streaming using my personal library, with on-the-fly transcoding to spare my mobile data plan
- **qBittorrent**: To seed "Linux ISOs"
- **Dozzle** (Container logs), **Glance** (Dashboard), **Maddy** (SMTP, just in case a container needs to send out a critical email, like password recovery), ...

I have a bunch of other containers running mostly as experiments, but nothing critical enough to mention here.\
Even though it makes sense given that everything runs headless without a GUI, I was pleasantly surprised by how many services I can comfortably host on such humble specs (20 containers right now, and I still have plenty of headroom left).\
Zero port forwarding on my router. Tailscale takes care of secure remote mesh access without exposing attack surfaces to the public web (and down the road, it can even be replaced with a self-hosted Headscale instance).

The best part of repurposing an old mobile dual-core chip? It barely pulls anything from the wall at idle. It's dead silent, draws less power than an LED lightbulb, and stays cooler without a battery cooking itself inside the chassis.

![90s kid on a computer](/homelabing-101/90s-computer-kid.png)

While this isn't a tutorial, pro tip for laptop homelabbers: don't forget to set HandleLidSwitch rules to `ignore` in `logind.conf`, unless you enjoy wondering why your entire infrastructure went dark the moment you shut the lid.
```bash
# /etc/systemd/logind.conf.d/lid.conf
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

At some point, you will realize that maintaining a single, monolithic `compose.yml` gets messy fast. That's why having a master compose file using Docker's native `include` syntax alongside modular subfolders is a game changer:

```yaml
# ~/docker/compose.yml
name: server
include:
  - core/compose.yml
  - network/compose.yml
  - media/compose.yml
```
Coupled with a clean directory layout:
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
This keeps each stack isolated and easy to maintain while still letting you control the entire fleet with a simple `docker compose up -d` from the root directory.

---
### The compromises (and why it's not production-ready)
![Anime girl on a computer](/homelabing-101/anime-computer.gif)

Let’s not kid ourselves: this is a fun, practical toy, not a high-availability tier-3 datacenter. 

There are obvious bottlenecks:
1. **Zero redundancy:** A single external USB HDD means that if the drive dies, everything not backed up vanishes. USB cables can also get nudged loose.
2. **Power instability:** Since the battery had to be removed, a 0.5-second power flicker cuts the whole server instantly. No UPS (yet).
3. **Bandwidth asymmetry:** 5G is shockingly fast on downloads, but the upload speed and latency can vary wildly depending on weather and cell tower congestion.

Note: Cellular 5G connections live behind strict carrier-grade NAT (CGNAT), meaning inbound port-forwarding is dead on arrival anyway. Tailscale's WireGuard hole-punching isn't just a privacy layer here, it's the only reason this setup is reachable from outside.

Is it reliable enough for everyday personal use? Surprisingly, yes.\
Would I host critical client infrastructure on it? Absolutely not.

And of course, money is, *once again*, the biggest limit. I would **LOVE** to host LLMs and other types of AI, or build a Docker Swarm cluster just for the love of the game, but it is simply not feasible... At least for now...

---
### What's next?
![Yasuo Iwakura](/homelabing-101/yasuo.gif)

This project started as an afternoon experiment to revive e-waste, but homelabbing is a slippery slope.\
I still have a lot of services to try, a lot of cheap hardware to find on [Leboncoin](https://leboncoin.fr), and of course a lot of stuff to break.

If you have an old laptop collecting dust in a closet, don't throw it away. Wipe the drive, install a headless Linux distro, throw Docker on it, and reclaim a small piece of your digital sovereignty.

Who knows? Maybe this little experiment could evolve into a cool F2F network one day. Though the real challenge with F2F isn't the tech: it's finding friends autistic enough to actually care about that stuff.\
Good luck with that.
