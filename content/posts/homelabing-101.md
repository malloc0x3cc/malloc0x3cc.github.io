---
title: "Homelabing 101"
date: 2026-10-31
draft: true
tags: ["server", "sysadmin"]
categories: ["personal-projects"]
---
### *The trees... The trees are speaking fed...*
![Hackerman](https://media1.tenor.com/m/TbTe1Nc6j34AAAAC/hacker-hackerman.gif)

A few months ago, I finally got my own internet connection !\
*Context: My place doesn't have fiber access, so I had been "borrowing" the neighbour's Wi-Fi for a few years.*\
Now I have a 5G router with __UNLIMITED__ data. Pretty cool !\
Finally I can acces the internet, put all my personal life online, maybe SSH into my machines, and that's it...\
Right ?

---
### What's the point anyway ?
![Blade Runner 2049](https://giffiles.alphacoders.com/207/207124.gif)

Let's face it: we're not living in 1998 anymore. Tech used to be cool back then, but now it sucks.\
We own basically nothing, and we have collectively traded our privacy for convinience. We are living in what people 30 years ago would have called a *distopia*.\
Sure, you could just give up and willingly hand over all your personal data to Big Tech, or simply reject modernity and return to monke...

Luckily, a middle ground exists: **Becomming more "tech self-sufficient"**.

![Stardew Valley](https://images5.alphacoders.com/782/782779.png)

We all know that guy who used to live in a big city, and one day got so fed up that he sold everything to move to the countryside.\
He’s not totally isolated; he still enjoys modern conveniences and drives to town whenever he needs to. But by living farther away, **he bought his freedom**.\
Maintaining a home server is the tech equivalent of that: instead of planting carrots and raising chickens, you host the services you actually need at home.

![Silicon Valley - Server Fire](https://c.tenor.com/b2ZhtePJIHQAAAAM/silicon-valley-fire.gif)

Of course, *with great tech freedom comes great configuration* overhead, so at first, it might feel like a downgrade, you are now responsible for your backups, your security, your updates…\
And to be fair, some services can be notoriously tricky to self-host (emails for example), so you probably won't host *everything* under your roof.

Still, if you’re a tech nerd, this is a fun and rewarding DIY project that actually means something—and it might even save you a decent chunk of money.

---
### Behold: The mainframe
![Asus X751LJ](http://iweb.al/uploads/2020-11-01/c6cdaf9939d56489f5cc5d7c3a51eeb7.jpg)

This is an Asus X751LJ I bought more than a decade ago, it was already underpowered at the time (Intel Core i3-4005U @1.6GHz, 4GB of *soldered, non upgradable* DDR3, Nvidia GT920M) so it's basically unusable as a modern computer, too big/heavy to bring around, and I had to remove the battery because it nearly caught fire once.

A much better choice for a small, cheap, low power consuption server would be a Small Form Factor/Mini **Lenovo ThinkCentre** or **Dell Optiplex** + a 2-bay NAS for RAID 1 storage.\
But surprisingly, even this setup can do the job for now.

I upgraded it's dying mechanical HDD for a 120GB SATA SSD I had lying around, installed Fedora Server and plugged a 1TB HDD via USB 3.\
It's connected to a 5 Euro gigabit switch (TP-Link LS1005G) along my Mac Mini and my "gaming" computer, the switch is plugged into the router.\
From that point, I never touched that computer again, everything is done via SSH.

---
### What works, what doesn't
First of all, the external HDD is SMB shared, remote access via Tailscale. Everything else is a Docker container\
The main ones are:
- **Adguard Home**: DHCP Server, DNS Blocklists
- **Unbound**: DNS Resolver (Quad9 are leaving France, Mullvad stopped their public DNS, I still have Digitale-Gesellschaft's DNS as fallback)
- **Vaultwarden**: Self-hosted Bitwarden, OTP and everything for free
- **Navidrome**: Basically Spotify for all my personal music, with transcoding to save mobile data
- **qBittorrent**: To seed *mostly* Linux ISOs
- **Dozzle** (Logs), **Glance** (Dashboard), **Maddy** (SMTP), ...

I do have *way more* things running as experiments mostly, but nothing worth mentionning.\
Even though it makes sense because everything is headless/terminal only, I was plesently surprised to see all the services I can comfortably run on such low specs, and I still have a lot of room for more actually !

Maddy is only here just in case one of my services need to end an important email (password recovery mostly).

No ports needed to be opened of my router, and again, Tailscale takes care of private remote access (It's even possible to replace Tailscale by self hosting Headscale).
