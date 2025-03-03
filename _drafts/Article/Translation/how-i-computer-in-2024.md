---
title: How I Computer in 2024
date: 2024-07-31T00:00:00.000Z
authorURL: ""
originalURL: https://jnsgr.uk/2024/07/how-i-computer-in-2024/
translator: ""
reviewer: ""
---



<!-- more -->

# How I Computer in 2024

·2611 words·13 mins

Table of Contents

-   [Introduction]
-   [Hardware]
    -   [Desktop]
    -   [Server]
    -   [Laptop]
    -   [Phone]
-   [Connectivity & Security]
-   [Productivity Apps]
-   [Development]
-   [OS / Desktop]
-   [Server / Homelab]
-   [Summary]

## Introduction 

I’m always fascinated to see how people use their computers - which applications they choose, how they set up their desktop environments and even how their screens are laid out on their desk. I’ve learned some great tricks from friends and colleagues over the years, so I thought I’d write up how I use my machines in 2024.

The setup I’m using today has been quite static for a couple of years, with only minor adjustments. Each time I change something significant, I leave it for at least a couple of months to try and build muscle memory and see if I’m going to make the adjustment permanent.

[][17]

[![screenshot of an empty hyprland desktop showing waybar at the top](https://jnsgr.uk/2024/07/how-i-computer-in-2024/06_hu19d68ad61b5dede6fd1f1d19283c6ff2_18899216_660x0_resize_box_3.png)][18]

## Hardware 

### Desktop 

My main machine is a custom built desktop machine. It’s in a sombre looking, all black [beQuiet Silent Base 600][21] case. I’ve never been into RGB lights - I’m much more into good thermals and _silent_ operation. The full spec is as follows:

**CPU**: [AMD Ryzen 9 7950X][22]  
**GPU**: [AMD Radeon RX 7900XT][23]  
**RAM**: [G.SKILL Trident Z5 Neo RGB 64GB DDR5-6000][24]  
**PSU**: [Corsair HX1000i][25]  
**Disk**: [1TB SN850X][26] + [2TB SN850X][27]  
**Board**: [MSI MPG X670E CARBON WIFI][28]  
**Cooler**: [beQuiet Dark Rock Pro 5][29]  
**Case**: [beQuiet Silent Base 600][30] with 3x [beQuiet Silent Wings 4 PWM][31] fans  
**Keyboard**: [DURGOD Taurus K320 TKL][32] with Cherry MX Brown switches  
**Mouse**: [Razer Deathadder V2 Pro][33]  
**Monitor**: [57" Samsung G95NC Odessey Neo G9][34]  
**Camera**: [Sony ILME-FX3][35] / [FE 28-70mm F3.5-5.6][36] / [Elgato Cam Link 4K][37]  
**Speakers**: [Audioengine A2+][38]  
**Mic**: [RODE VideoMic GO II][39]  

On my desk, you’ll find a [57" Samsung G95NC Odessey Neo G9][40] monitor mounted on a [gas spring arm][41], which is the newest addition to my setup. For 5 years, I’d been running a pair of 27" [LG 27" UN850 4K][42] monitors mounted on a dual monitor arm and had toyed with the idea of moving to an ultra-wide for a while. The Samsung display is the first I have found that doesn’t compromise on resolution - it’s the same resolution as my two LG monitors combined, but on a single panel. I must admit that I’m quite surprised how much of a productivity booster it is _not_ having the split down the middle.



[![a photograph of my desk including a huge ultra-wide monitor](https://jnsgr.uk/2024/07/how-i-computer-in-2024/01_hu930cbab5a5dfad6c7189f290c5bed7dd_1798110_660x0_resize_box_3.png)][44]

At the time of writing, I work for [Canonical][45] which is an all remote company. The combination of the company itself and my role as VP Engineering means I spend a good portion of my day on video calls. In my opinion, investing in a solid AV setup is a service to your colleagues, particularly where your role involves managing people. I’m currently running a [Sony ILME-FX3][46] with the standard [FE 28-70mm F3.5-5.6][47] lens, hooked up to an [Elgato Cam Link 4K][48]. For audio, I use a [RODE VideoMic GO II][49] and a pair of [Audioengine A2+][50] speakers.

In the world of Linux desktops, I’ve found audio devices that present their own USB interface to be much less hassle. I completely disable the motherboard’s onboard sound, as well as the HDMI/DisplayPort sound outputs on my machine and leave just the USB audio interfaces from my mic and speakers enabled.

### Server [#][51]

I use the term “server” loosely… my homelab has gone through many iterations over the years, from all “on-prem”, to a mix of cloud services and devices, and back again.

My current setup is very modest, partly because my workstation is such a monster, meaning I can easily spin up multiple VMs/containers there when I want to experiment and not really impact the performance of the machine for more routine tasks.

Most of my services run on a single [Intel NUC6i7KYK][52]. This machine has an Intel i7-6770HQ CPU, 16GB RAM and a 512GB Samsung 970 Pro NVMe drive internally. It’s connected to a Samsung 840 EVO 4TB SATA drive by USB. I’m not much of a data-hoarder so I don’t require too much storage.

This machine is getting a bit tired and I’m thinking about replacing it with something a little more modern later this year.

### Laptop [#][53]

If I’m not at my desk, then I’m using my [Lenovo Z13 Gen 1][54]. I specified this machine with the AMD Ryzen 7 Pro 6860Z, 32GB RAM and a Hi-DPI display.

I can’t rate this machine highly enough. The build quality is a cut above even Lenovo’s normal standard - it feels very premium and much more in the style of Apple’s uni-body aluminium laptops. It’s got plenty of power, and the battery lasts most of the day under moderate usage.

I tend towards ultralight machines when I travel because I can always use my desktop machine remotely if I need more grunt (more on that later…), and I’m certainly not interested in trying to make dual integrated/discrete GPUs work properly.

### Phone 

I carry an [Apple iPhone 15 Pro][56]. I’ve been an iPhone user since around 2011 and likely won’t change any time soon. My family all use iPhones (and therefore FaceTime) and I like the particular trade-off of convenience/privacy that’s provided by Apple - however flawed that might be in absolute terms. The phone works great with my Airpods, the camera is better than I am at taking photos, and the battery life seems pretty good too.

I wrap the phone in a [Mous Limitless 5.0 Aramid Fibre][57] case to avoid too many oops moments!

I find it difficult to get too excited about phones these days, I see them more as a commodity.

## Connectivity & Security 

In 2021 I started using [Tailscale][59] in place of my hand-rolled Wireguard setup, and I haven’t looked back. It has to be one of my favourite pieces of technology ever. It runs on all of my things - desktops, laptops, servers, phones, tablets, etc.

I also recently took advantage of their [partnership with Mullvad][60]. I’ve used [Mullvad][61] as my default VPN provider when using untrusted networks for a few years - but using it through Tailscale means I can still access my tailnet while my internet traffic egresses through Mullvad without any extra configuration.

I use [NextDNS][62] as an alternative to running a [Pi-Hole][63] or similar. Tailscale have a nice [integration][64] which means that all the devices on my tailnet automatically get [DNS-over-HTTPS][65] without any additional configuration, as well as DNS-level ad-blocking.

I’ve got a couple of shared nodes in my tailnet - including one that my family can use as an exit node when they travel. As a result, I make quite extensive use of Tailscale [ACLs][66] to ensure people can only access what I want them to.

I’ve been a [1Password][67] user for more than a decade now and I think their products are fantastic. Their Linux app sets the bar for modern cross-platform applications in my view. I recently started using their secrets capability at the CLI - the ability to store a `.env` file with a bunch of benign secret references, and have the actual [secrets injected into the environment][68] or a [config file][69] is very handy.

I also have a small collection of [Yubikeys][70] with different connectors. One lives on my desk attached to my desktop, another lives in my pocket or otherwise on my person, and another is in a safe. They’re all NFC enabled so they work nicely with my mobile devices. I configure my Yubikeys with ed25519 [resident keys][71] for SSH, along with storing my GPG key (which rarely gets used these days…).

One of my favourite things about the Yubikey is their ability to store TOTP codes. It’s a bit of a pain when I onboard a new account having to add the new secret to each key, but the upside is I don’t have to work out how to update/transfer them all each time I get a new phone! It’s also handy on the desktop to be able to run `ykman oath accounts code <name>`.

## Productivity Apps 

A lot of my work is done in a browser. Canonical uses [Google Workspace][73] for emails, documents, slides, etc., so my default mode since joining has been to use Google Chrome for work things, and Firefox for personal things. I know that Firefox has [Account Containers][74] and other features that would help segregate the two, but I’ve found keeping my work and personal concerns in completely separate browsers to be useful.

After having run a [Nextcloud][75] server for several years on a Droplet ([using `docker-compose`][76]), I ultimately realised that I was _only_ using the file syncing capability, and wasn’t moving much data even then. I switched to using [Syncthing][77] to avoid the overhead of running a server instance, which works particularly well when combined with Tailscale.

All of my notes, both work and personal, live in [Obsidian][78]. When I first discovered Obsidian I fell for the classic trick of installing **all the extensions**, and have since paired that back. I went very deep with [Dataview][79], using it to collate actions from across my vault into various categories (meeting agendas, personal, reviews, etc.), but I found that as my vault grew the performance suffered quite a lot. A few months ago, I removed dataview, did some painful refactoring of my notes (lots of `sed`erry and `grep`pery!) and reverted to using Obsidian [queries][80].

I tried to get into [Zettelkasten][81] but found the maintenance a little… boring? I’ve ended up with a simple structure that I find really helps me in my day-to-day at work. Each day gets its own “Daily Note” which includes my agenda, linking to ongoing notes with the people or regular meetings I’m in. The daily notes are also a place for me to collate loose notes which might be searched later:


[![obsidian.md screenshot showing my daily note template](https://jnsgr.uk/2024/07/how-i-computer-in-2024/02_huf93b0c707cf46312a8fd9228e1b4cdb5_92067_660x0_resize_box_3.png)][83]

The agenda and the links are automatically generated using a small Go application I wrote - this application scrapes my Google Calendar, and according to some rules and the knowledge it has of my vault, generates the Markdown for the agenda and copies it to the clipboard. Each day, I sit down and type `agenda` at the command line, then paste into Obsidian. The notes for each person contain a running log of my notes with that person or group by date.

I use a few Obsidian plugins to help here - including [Templater][84], [QuickAdd][85], [Omnisearch][86] and [Linter][87]. The first two are particularly handy for quickly inserting common meeting agendas, sets of interview questions, playbooks, etc.

I moved away from tracking tasks in Obsidian, and started using [Todoist][88] late last year. Todoist is great - I like to keep running lists of tasks per person, so that when I next meet them in a 1:1 or otherwise, I have a quick reference of all the things I’m meant to speak with them about - and I can achieve that very easily with Todoist labels. The Obsidian integration means I can integrate the agenda with the meeting note for a specific person:



[![obsidian and todoist side-by-side showing the integration](https://jnsgr.uk/2024/07/how-i-computer-in-2024/03_hu961bfa5d6384c4efa7dfa90c87eca21a_361619_660x0_resize_box_3.png)][90]

## Development [#][91]

I’ve been a long-time user of [Alacritty][92] as a terminal emulator. I mostly use [Visual Studio Code][93] on the desktop - I like the community support for plugins, themes, etc. I’m also pretty handy in vim - I still have quite a snazzy [Neovim][94] setup which I use whenever I’m at the terminal. You can see my [neovim config][95] on Github - I don’t go too wild on plugins, but I’ve come to like [lightline][96], [telescope][97], and [nvim-tree-lua][98].



[![the alacritty terminal emulator showing a tmux session with neovim loaded](https://jnsgr.uk/2024/07/how-i-computer-in-2024/04_hu05bf6628177fb41deef80d4a55f42632_404501_660x0_resize_box_3.png)][100]

I mostly drive `git` from the command line, but I’ve recently taken to using [Sublime Merge][101] for complicated rebases, or where I want to stage lots of small hunks in files. I was a dedicated user of [Sublime Text][102] for some years, but felt like it lagged behind Visual Studio Code on features after a while - despite being somewhat addicted to how lightning fast Sublime Text felt in comparison.



[![visual studio code and sublime merge side-by-side](https://jnsgr.uk/2024/07/how-i-computer-in-2024/05_hu2d6d25863bc408587fe8518d526b1cf7_752958_660x0_resize_box_3.png)][104]

## OS / Desktop 

If you’ve read my blog before, it’ll be no surprise to you that I’m all-in on NixOS for all the things. I started that journey around 2 years ago and haven’t looked back. My journey on the Linux desktop has been quite varied over the years: my first ever Linux desktop experience was with [Knoppix][106] back in 2003. I then spent a few years dabbling with the various releases of Ubuntu before starting to use Linux on the desktop full-time in around 2014. From there I spent years on Arch Linux swapping between Plasma and GNOME about every 12 months.

I’ve become a fairly dedicated tiling window manager user, though I’ll admit that I bounced off it a few times before it stuck. When I made the switch to [Sway][107] in 2021, something clicked and I’ve not gone back from tiling since. I stuck with Sway in various configurations for quite a while, before moving to an almost identical looking setup based on [Hyprland][108] around 15 months ago. Hyprland seems nice - it’s mostly stable and I like the eye-candy.

Absolutely everything is themed with [Catppuccin Macchiato][109]. Not only do I love the theme, but I love how pervasive it is across all the apps/tools I use - and I’m a sucker for consistency!

You can see all the gory details of my Hyprland, waybar, rofi, mako, etc. [on Github][110].

[][111]

[![screenshot of a very busy hyprland desktop with editors, browsers, etc.](https://jnsgr.uk/2024/07/how-i-computer-in-2024/07_hu19d68ad61b5dede6fd1f1d19283c6ff2_2075377_660x0_resize_box_3.png)][112]

## Server / Homelab [#][113]

My server machine also runs NixOS, with a collection of media services and utilities. Where possible all of the services are run as “native” NixOS modules, with some running inside [systemd-nspawn][114] containers using the built-in language for [NixOS Containers][115]. If I’m experimenting with a new service, I sometimes run them in Docker to start with, especially if there isn’t already a NixOS module and I want to decide whether or not to invest the time in writing one!

I run [Caddy][116] as a reverse proxy ([recently switched][117] from Traefik). It can [talk directly to the Tailscale daemon][118] to issue LetsEncrypt certs for devices on your tailnet. This Caddy instance acts as a reverse proxy onto all the services running on the server, along with some other services on my home LAN, all over TLS. I tend to access each of these services through [Homepage][119] (which I previously [blogged about][120]):



[![my personal dashboard using gethomepage.dev](https://jnsgr.uk/2024/07/how-i-computer-in-2024/08_hu7f790721995065d374827ee567548c83_94865_660x0_resize_box_3.png)][122]

Each night, the contents of my iCloud Photos library is dumped using [icloud-photos-downloader][123] so that I have a local (and backed up) copy of my photos should anything untoward ever happen to my iCloud account.

It also runs a [Home Assistant][124] instance which runs my (in-progress!) custom integration for the underfloor heating and solar inverter in my house. I haven’t yet spent enough time with Home Assistant, but I plan to get it better set up over the coming months. I recently moved to a house with lots of “smart” devices, and I’d like to bring control of all the various devices into a single application. Once my experimenting is done, I’ll probably move the Home Assistant deployment to a dedicated low-power device.

This machine’s data is backed up nightly to [Borgbase][125]. As mentioned above, I use [Syncthing][126] to move files around, and I configure this server to act as a “receive only” target for all the directories that I sync. This means that my data is always in at least three places: on my desktop or laptop, on my server, and backed up to Borgbase. Sometimes I’ll ad-hoc access files that aren’t synced to a given machine using [Files][127], which is a nice looking, single PHP-file gallery for your files. I keep meaning to replace this with something that _isn’t PHP_, but I’ve yet to find a more compelling blend of simplicity and compelling user experience.

## Summary 

I don’t know how many other people are interested in how other people use their computers - but I hope you enjoyed the article. Feel free to reach out if you think I could be doing something better, or if you think you’ve got a killer app I might enjoy using!


Author

Jon Seager

Husband, father, leader, software engineer, geek.

---

[←→ Workstation VMs with LXD & Multipass 25 June 2024][133]


[3]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/
[21]: https://www.bequiet.com/en/case/1501
[22]: https://www.amd.com/en/products/processors/desktops/ryzen/7000-series/amd-ryzen-9-7950x.html
[23]: https://www.xfxforce.com/shop/xfx-speedster-merc310-7900xt
[24]: https://www.gskill.com/product/165/390/1665020865/F5-6000J3040G32GX2-TZ5NR
[25]: https://www.corsair.com/uk/en/p/psu/cp-9020259-uk/hx1000i-fully-modular-ultra-low-noise-platinum-atx-1000-watt-pc-power-supply-cp-9020259-uk
[26]: https://www.westerndigital.com/products/internal-drives/wd-black-sn850x-nvme-ssd?sku=WDS100T2X0E
[27]: https://www.westerndigital.com/products/internal-drives/wd-black-sn850x-nvme-ssd?sku=WDS200T2X0E
[28]: https://www.msi.com/Motherboard/MPG-X670E-CARBON-WIFI
[29]: https://www.bequiet.com/en/cpucooler/4466
[30]: https://www.bequiet.com/en/case/1501
[31]: https://www.bequiet.com/en/casefans/3703
[32]: https://www.durgod.com/product/k320-space-gray/
[33]: https://www.razer.com/ap-en/gaming-mice/razer-deathadder-v2-pro
[34]: https://www.samsung.com/uk/monitors/gaming/odyssey-neo-g9-g95nc-57-inch-240hz-curved-dual-uhd-ls57cg952nuxxu/
[35]: https://www.sony.co.uk/interchangeable-lens-cameras/products/ilme-fx3-body---kit
[36]: https://www.sony.co.uk/electronics/camera-lenses/sel2870
[37]: https://www.elgato.com/uk/en/p/cam-link-4k
[38]: https://audioengineeu.com/products/audioengine-a2-wireless-bluetooth-computer-speakers-60w-bluetooth-speaker-system-for-home-studio-gaming
[39]: https://rode.com/en/microphones/on-camera/videomic-go-ii
[40]: https://www.samsung.com/uk/monitors/gaming/odyssey-neo-g9-g95nc-57-inch-240hz-curved-dual-uhd-ls57cg952nuxxu/
[41]: https://www.amazon.co.uk/gp/product/B0B73XXDP5/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1
[42]: https://www.lg.com/us/monitors/lg-27un850-w-4k-uhd-led-monitor
[43]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/01.png
[44]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/01.png
[45]: https://canonical.com
[46]: https://www.sony.co.uk/interchangeable-lens-cameras/products/ilme-fx3-body---kit
[47]: https://www.sony.co.uk/electronics/camera-lenses/sel2870
[48]: https://www.elgato.com/uk/en/p/cam-link-4k
[49]: https://rode.com/en/microphones/on-camera/videomic-go-ii
[50]: https://audioengineeu.com/products/audioengine-a2-wireless-bluetooth-computer-speakers-60w-bluetooth-speaker-system-for-home-studio-gaming
[52]: https://ark.intel.com/content/www/us/en/ark/products/89187/intel-nuc-kit-nuc6i7kyk.html
[54]: https://www.lenovo.com/gb/en/p/laptops/thinkpad/thinkpadz/thinkpad-z13-%2813-inch-amd%29/len101t0036?srsltid=AfmBOor-8ic5yZrW3rlDXTTRwK8r05y-gjCpJK04fA4qtote0u2HZ7I6
[56]: https://www.apple.com/uk/iphone-15-pro/
[57]: https://uk.mous.co/products/limitless-5-0-magsafe-compatible-phone-case-aramid_fibre
[58]: #connectivity--security
[59]: https://tailscale.com/
[60]: https://tailscale.com/kb/1258/mullvad-exit-nodes
[61]: https://mullvad.net/en
[62]: https://nextdns.io/
[63]: https://pi-hole.net/
[64]: https://tailscale.com/kb/1218/nextdns
[65]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[66]: https://tailscale.com/kb/1018/acls
[67]: https://1password.com/
[68]: https://developer.1password.com/docs/cli/secrets-scripts
[69]: https://developer.1password.com/docs/cli/secrets-config-files
[70]: https://www.yubico.com/products/yubikey-5-overview/
[71]: https://developers.yubico.com/SSH/Securing_git_with_SSH_and_FIDO2.html
[72]: #productivity-apps
[73]: https://workspace.google.com/intl/en_uk/
[74]: https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/
[75]: https://nextcloud.com/
[76]: https://github.com/jnsgruk/nextcloud-docker-compose
[77]: https://syncthing.net/
[78]: https://obsidian.md/
[79]: https://blacksmithgu.github.io/obsidian-dataview/
[80]: https://publish.obsidian.md/tasks/Queries/About+Queries
[81]: https://zettelkasten.de/introduction/
[82]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/02.png
[83]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/02.png
[84]: https://github.com/SilentVoid13/Templater
[85]: https://github.com/chhoumann/quickadd
[86]: https://github.com/scambier/obsidian-omnisearch
[87]: https://github.com/platers/obsidian-linter
[88]: https://todoist.com/
[89]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/03.png
[90]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/03.png
[92]: https://alacritty.org/
[93]: https://code.visualstudio.com/
[94]: https://neovim.io/
[95]: https://github.com/jnsgruk/nixos-config/blob/main/home/common/shell/vim.nix
[96]: https://github.com/itchyny/lightline.vim
[97]: https://github.com/nvim-telescope/telescope.nvim
[98]: https://github.com/nvim-tree/nvim-tree.lua
[99]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/04.png
[100]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/04.png
[101]: https://www.sublimemerge.com/
[102]: https://www.sublimetext.com/
[103]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/05.png
[104]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/05.png
[106]: https://www.knopper.net/knoppix/index-en.html
[107]: https://swaywm.org/
[108]: https://hyprland.org/
[109]: https://github.com/catppuccin/catppuccin
[110]: https://github.com/jnsgruk/nixos-config
[111]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/07.png
[112]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/07.png
[114]: https://www.freedesktop.org/software/systemd/man/latest/systemd-nspawn.html
[115]: https://nixos.wiki/wiki/NixOS_Containers
[116]: https://caddyserver.com/
[117]: https://github.com/jnsgruk/nixos-config/commit/dffac1dc0635f377865bcdfc2349387d41fc965d
[118]: https://tailscale.com/kb/1190/caddy-certificates
[119]: https://gethomepage.dev/latest/
[120]: https://jnsgr.uk/2024/03/a-homelab-dashboard-for-nixos/
[121]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/08.png
[122]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/08.png
[123]: https://github.com/icloud-photos-downloader/icloud_photos_downloader
[124]: https://www.home-assistant.io/
[125]: https://www.borgbase.com/
[126]: https://syncthing.net/
[127]: https://www.files.gallery/
[129]: https://github.com/jnsgruk
[130]: https://hachyderm.io/@jnsgruk
[131]: https://linkedin.com/in/jnsgruk
[132]: https://t.me/jnsgruk
[133]: https://jnsgr.uk/2024/06/desktop-vms-lxd-multipass/