---
title: Home Lab Beginners guide (Hardware)
author: Hayden James
authorURL: https://linuxblog.io/about-me/
originalURL: https://linuxblog.io/home-lab-beginners-guide-hardware/
translator: ""
reviewer: ""
---

# [Home Lab Beginners guide (Hardware)][1]

<!-- more -->

**March 8, 2024** by **[Hayden James][2]**, in [Blog][3] [Linux][4]

Until recently, and for well over the past decade, my wife and I have been nomads. Moving from the Caribbean to Miami, New York, Las Vegas, Vancouver, and now back home. This has meant that for many of those years, my home office basically comprised of a few laptops and screens.

These days, we are more settled; my wife has completed her studies, works full-time, and my full-time job remains remote, via a home office [supporting Linux servers][5] and [hosting them][6].

After a short trip to Antigua, I had a brief meeting with a tech friend of mine, [Yves Ephraim][7]. He was an Engineer with [Cable & Wireless][8] for 20 years (until 2003). Yves left C&W shortly before I started there in 2004. He’s a champion of the internet and technological developments in Antigua & Barbuda.

We sat in his home office, where he told me a bit about his current projects, as well as his dabbling with [IPv6][9], web hosting, mail, [BGP4][10], among other things. Most of which he does via his very capable home lab.

Table of Contents

-   [What is a Home Lab?][11]
-   [May we inspire each other to great home labs][12]
-   [Home lab location, it’s all about location.][13]
-   [Network vs. Server Racks vs. Cabinets?][14]
-   [Recommended Home Lab Hardware][15]
    -   [To replace or not to replace: ISP Cable Modem][16]
    -   [Selecting Your Home Lab Rack][17]
    -   [Home Lab UPS (Uninterruptible Power Supply)][18]
    -   [Universal Home Lab Rack Shelf][19]
    -   [Rack Mount Home Lab Power supply][20]
    -   [Rack Mount Home Lab Cooling Fans][21]
    -   [Home Lab Routers and switches][22]
    -   [Home Lab Patch Panels and Network Cables][23]
    -   [Home Lab Servers][24]
-   [Summary and Conclusion][25]

## What is a Home Lab?

![My Home Lab - 2020](https://static.linuxblog.io/wp-content/uploads/2020/03/my_homelab-868x868.jpg "My Home Lab - 2020")  
_My current Home lab in a 12u rack as of May 2020_

Think of a home lab as a place where you can fail in the privacy of your own home. As Thomas A. Edison said: _“I have not failed. I’ve just found 10,000 ways that won’t work.”_ I consider myself an expert at failure. But seriously, I would like to fail a lot more, and a home lab will create endless opportunities for me to fail. Of course, all the while seeking success, but you got that already.

In general, a lab is a place where you can safely perform experiments. Most of you reading this article are techies and sysadmins. As you know, trying out new things on production equipment never ends well. Shush… it’s OK, I know, I know, you didn’t think that one command would take everything offline. This risk is the reason why we build ourselves a sandbox environment to dabble, test, and fail in, all from the comfort of our own homes.

To date, I host my test labs on servers in North America and Europe. I’m creating a customized network and server home lab; to fill those areas where I would like to become more familiar. I will also be using my home lab for remote backups, [network monitoring and alerting of remote servers][26], and wired UAP APs, among other things.

## May we inspire each other to great home labs

As I told Yves, our chat had me revisiting the idea. In fact, I think my exact words were, “Yves, I see your home lab, and I’ll raise you mine.” As always, I’m certain there’s a lot more to learn through discussion. In light of this fact, I wanted to share this journey with you, starting with the hardware selection process for my home lab.

Of course, over the years, I’ve had equipment lying around, but that stops today. Well, sort of. So, if you have ever been interested in building a home lab or if you already have one, then [let’s mingle, share and dabble together][27].

Where do we begin? Not to sound cliché, but let’s start from the ground up, or wall forward. What I’m trying to say is, we need storage space, physical storage space, for our new home lab.

## Home lab location, it’s all about location.

Location, Location, Location. It’s all about location! Please excuse the lack of originality – I’m a sysadmin, not a writer. I’ll try my best to get you to the end of this article.

Location is critical for several reasons. The choice between the home office, living room, closet, attic, basement, or garage depends on a range of important variables. These include room temperature and ventilation, workable space around your equipment, ease and distance of network cable runs, foot traffic, 24-hour ease of access, power, noise levels from your home lab, and more.

Here’s a quick list of Pros vs. Cons I’ve compiled to get us thinking about all the possible home lab locations. Choose wisely:

**Home office**

-   Pros: Proximity to work area/desk/devices, fewer cable runs, and you can watch lights flash all day.
-   Cons: no home office or you already spend too much time in your home office.

**Living room**

-   Pros: Usually cool, lots of space/setup options, blinky-blinky sci-fi movie nights, and counts as family time.
-   Cons: divorce, foot traffic, could get damaged, or damaged during the divorce.

**Closet**

-   Pros: Easily accessible, stealthy, and you get to say: “Look at what I have hiding in my closet.”
-   Cons: Poor ventilation (excess heat), lack of space, and one less closet = unhappy wife.

**Basement**

-   Pros: Usually cooler temps, and volunteering to do laundry (maybe con?).
-   Cons: don’t have a basement, flooding, spiders, or no access when injured.

**Attic**

-   Pros: Less noise, easier cable runs.
-   Cons: can get hot depending on where you live, roof leaks, humidity/condensation, and creepy at night.

**Garage**

-   Pros: Less noise in the house, completely out of sight, wife won’t even notice.
-   Cons: A Bug’s Life _in_ your lab, excess heat (if no AC), dust, could require longer cables, or gets wrecked when parking the car.

For my lab, I’m going to build it out in my home office. I have laptops, a desktop, a server, and other devices in my office, so that this location will require shorter cable runs. When selecting your location, consider the coolest area of the room/space, avoid direct sunlight and consider reserving space for future expansion.

It’s wise if you draw/sketch out a home network diagram or [use network design software][28]. Keep in mind your floor plan and how you will accomplish the cable runs. For example, most of the homes on this island have 8″ concrete interior walls. Think about these things before, not after. In other words, map everything out, make notes and create lists.

## Network vs. Server Racks vs. Cabinets?

![home lab rack vs cabinet](https://static.linuxblog.io/wp-content/uploads/2020/02/home_lab_rack_vs_cabinet-868x399.jpg "home lab rack vs cabinet")

Next, we need to decide how we will store the equipment (modems, routers, switches, servers, patch panels, UPS systems, power strips, cooling fans, etc.).

Network cabinets and racks are often confused with S_erver_ cabinets and racks. Routers, switches, patch panels, and the like are usually much shallower than servers. As such, Network Cabinets and Racks are usually not as deep as Server Cabinets and Racks. Also, networking devices often produce less heat than servers. You will find some network cabinets will have glass doors that may not leave enough ventilation for servers.

After deciding the depth and ventilation requirements for your home lab, there are a couple of other things to consider. A cabinet is an enclosed space with door(s) and/or removable sides, whereas a rack is a semi or fully open (4 sides open) frame. To help you decide whether to use a cabinet or rack, consider the following:

-   If you are **installing large, heavy servers**, then the extra stability of cabinets or four-sided racks should be considered.
-   If you **need frequent access** to the sides or rear of equipment, then an open rack or cabinet with removable sides would work well.
-   If your **equipment requires extra cooling**, an enclosed cabinet will need more attention to cooling and ventilation.
-   If the **room is prone to dust**, the extra protection of a cabinet will go a long way in keeping it out of your equipment.
-   If you are **installing in a general living area** frequented by house guests, consider an enclosed cabinet that can often look neater in appearance when locked. However, a well-maintained open rack with tidy power and network cable runs can look just as neat!
-   If **restricted access/security** is required, many enclosed cabinets often offer lock and key access control for better security.

## Recommended Home Lab Hardware

_**UPDATE:** I’ve updated the article. Some of the devices mentioned have been discontinued or are not readily accessible anymore. I’ve updated with upgrades I’ve made and other recommended hardware and accessories. It’s been nearly four years since my original homelab build._

Now that you have already measured your equipment’s maximum depth and considered all the above advice, it’s time to buy your first piece of hardware. Or not. You could have probably skipped to this section if you prefer more em’, creative methods for hosting your home lab. However, most of the above advice applies even if you plan to set up your home lab in your existing living room cabinet, on a bookshelf, under your desk, or in a cardboard box. No, seriously…

![cardboard box home lab](https://static.linuxblog.io/wp-content/uploads/2020/02/cardboard-box-home-lab-868x564.jpg)  
_[Image][29]: Fancy cardboard box cabinet._

**NOTE:** You can get most of the equipment listed below via Craigslist, Universities, and other local second-hand options in some countries. Prices on this page reflect Amazon’s current pricing, which is subject to change. As an Amazon Associate member, I’ve included paid links to items, for which I’m compensated. For all other items not listed on Amazon, if the links become broken over time, please let me know, as it always seems to happen often, which is another benefit of using Amazon’s links. Let’s move on to the recommended home lab hardware.

### To replace or not to replace: ISP Cable Modem

![replace cable modem](https://static.linuxblog.io/wp-content/uploads/2020/02/replace_cable_modem-868x196.jpg "replace cable modem")

Before we delve a bit deeper into my home lab recommendations, if you are a beginner to switching and routing, then a lot of the hardware below may seem to be a bit of an overkill. You can get your feet wet by merely replacing your ISP’s cable modem (ISP or Internet Service Provider), you know, that thing with all the flashing lights that you frequently have to unplug to reset to regain internet access. Yup, that one, the mini heater in your living room.

Most US ISPs charge monthly rental for that thing. Buying your own could make some financial sense. As well as giving better performance, reliability, and security. Bet you were already sold with the financial bit eh? But it gets better; most of these modems provided by ISPs are usually of poor quality, used, and lack features. Lastly, your ISP’s Modem/Router combos often cannot independently update firmware versions as new exploits emerge. This means you are at the mercy of your ISP to push new patched firmware.

If you have a great ISP and this does not apply to you, stick with your modem. The only reason to replace it in such circumstances will be to have access to additional network features and create a home lab where you can test away! I replaced the modem/router combo from my ISP. I went with the [Motorola MB7420 16×4 Cable Modem][30]. _Update: 3 years later, there are much faster models, for example, the [Motorola MB7621 32×8][31]._

If you would like to keep things simple, then go with a [Modem + Wi-Fi router combo][32]. Otherwise, continue if you would like to build out a home lab to manage Firewall, NAS storage, VPN, VLANs, PoE (Power over Ethernet), Virtualization, remote network infrastructure monitoring/alerting, wired APs, web servers, backup servers, the list goes on.

_**Update:** This modem, similar to most I’ve used, gives off quite a bit of heat. When deciding placement, I would suggest toward the top or leave space below and above. That said, I love this modem! Unlike the ISP’s modem, it shows the quality of your connection, including Freq. (MHz), Pwr (dBmV), SNR (dB), corrected and uncorrected, and even allows manual downstream frequency setting if needed by your ISP._  

### Selecting Your Home Lab Rack

A rack _unit_ (abbreviated _U_ or _RU_) is a unit of measure defined as 1 3⁄4 inches (or 44.45 mm). It’s the unit of measurement for the height of 19-inch and 23-inch rack frames and the equipment’s height. The height of the frame/equipment is expressed as multiples of rack units. A typical full-size rack is 42U high. Racks and cabinets are then listed as 1U, 2U, 3U, 4U, and so on.

Here’s an example below of a 6U rack (labeled):

![6u rack example](https://static.linuxblog.io/wp-content/uploads/2020/02/6u-rack-2-868x583.jpg "6u rack example")

-   [StarTech 12U Wall Mount Rack – 13.5″ Deep / 19″ wide / 125lbs Capacity][33] – $158 ([specs][34]) – During my search for a wall-mounted rack, I almost purchased the 6U version of this rack, then at the last minute, I switched to the 12U version for possible future expansion.
-   [Samson SRK-12 Universal Equipment Rack Stand][35] – $230 ([specs][36]) – This was my first choice; this is an audio equipment rack that is fully compatible with shallow-mount equipment. One advantage is that it’s supported on wheels, so you don’t have to fix it to a wall or floor. However, I’m 6′ 2″, and I want to mount my rack about 4′ off the ground, so as you will see, I have also purchased a 2U cooling fan (intake), which I will place at or to the bottom of my setup. If your room is air-conditioned 24/7 and you don’t mind the low-to-floor setup, this may be the rack for you. Here’s a [great example of how clean this Rack can look when set up][37].
-   [NavePoint 9U Basic IT Wall Mount Network Rack Locking Glass Door][38] – $235 ([specs][39]) – This is a very beautiful-looking Network Cabinet. The issue for me is that I only use the AC in my office in the summer, and I don’t want the heat ever to be an issue; thus, for my home setup, an enclosed cabinet wasn’t right for me.

_**Update:** This is a very solid wall-mount rack. I can confirm that what you see is what you get. No issues to report. It does exactly what it’s supposed to._

### Home Lab UPS (Uninterruptible Power Supply)

![CyberPower OR500LCDRM1U](https://static.linuxblog.io/wp-content/uploads/2020/02/CyberPower-OR500LCDRM1U-868x99.jpg "CyberPower OR500LCDRM1U")

At home, we have a backup generator that runs on propane. It can run for up to a week, which is useful in the tropics. Other than hurricane season, power cuts are extremely rare here. So far, for 2020, we have had one outage due to a lightning strike on a nearby electrical post.

It takes a minute or so to failover to the backup generator. So the rack mount UPS solution I’m looking for is something that will support between 100W and 200W for a couple of minutes. Long enough to fail over to the backup gen or to shut down all the equipment cleanly. In the future, I may expand this with larger batteries, but with no way of returning anything I purchase, I’m a bit wary of investing in a larger backup battery only to find out it’s a dud. I will test for a year first, and so here’s what I decided on:

-   [CyberPower OR500LCDRM1U UPS, 500VA/300W, 6 Outlets, AVR, 1U Rackmount][40] – $220 ([specs][41]) – A small 1U rackmount UPS coming in at 20lbs, which should give me around 10 to 15 minutes of runtime.

Ultimately, you will have to decide whether you even require an uninterruptible power supply at home. From experience, UPS batteries are almost always a disappointment, and this is largely because of the cost vs. return. If you really need to have a UPS, you could spend thousands just to have a long enough runtime. If you work from home or host critical services hosted from home – you probably shouldn’t – then it may be worthwhile. My advice, get a UPS that provides just enough runtime for everything to shut down cleanly and only use the UPS for the most essential and efficient devices in your home lab. With all things considered, I’ll have to come back to this section before recommending any other UPS models.

_**Update:** Surprisingly, the battery performance of this UPS so far is above my expectations. I ran it down to around 50%, which gave me just over 30 mins of run time, including powering [2 PoE APs][42] and a [27″ monitor][43]. As mentioned later in this article, pay attention to your power consumption. This was something I considered, and it’s paid off thus far. The APs for example, max power consumption is just 6.5W per AP. The max load is about 20% to 30% of UPS capacity._ 

### Universal Home Lab Rack Shelf

![AC Infinity Vented Cantilever 1U Universal Rack Shelf](https://static.linuxblog.io/wp-content/uploads/2020/02/AC-Infinity-Vented-Cantilever-1U-Universal-Rack-Shelf-1-868x308.jpg "AC Infinity Vented Cantilever 1U Universal Rack Shelf")

Did you notice that the CyberPower 1U UPS does not have rack mounts? Of course, you could rest it on top of other equipment; however, I wouldn’t recommend that because we should keep the batteries as cool as possible and leave space between equipment to reduce the amount of trapped heat. Lastly, for this setup, it will look more professional if placed on a ventilated 1U rack shelf:

-   [AC Infinity Vented Cantilever 1U Universal Rack Shelf, 10″ Deep, for 19”][44] – $30 ([specs][45]) – These shelves are available from 6″ to 16″ deep. The backup battery is 9.5″. Pay attention to the depth of your rack/cabinet and equipment before deciding what you need.

_**Update:** Very sturdy, great for keeping equipment cool and cable management thanks to the vents and slots._ 

### Rack Mount Home Lab Power supply

![ADJ Products AC POWER STRIP PC 100A](https://static.linuxblog.io/wp-content/uploads/2020/02/ADJ-Products-AC-POWER-STRIP-PC-100A-868x89.jpg "ADJ Products AC POWER STRIP PC 100A")

The aforementioned CyberPower UPS comes with six outlets – four backup /w surge + two surges only. Currently, under my desk, I’m using seven outlets, and this is before ordering a replacement ISP modem, router, switch, servers and does not consider any future expansion. Right above the UPS, I will be mounting the following:

-   [ADJ Products AC POWER STRIP (PC-100A)][46] – $30 ([specs][47] / [alternative][48]) – will plug this into the 1U CyberPower UPS.

_**Update:** This is a very convenient piece of hardware. It plugs directly into the UPS and allows easy on/off switching of other equipment and devices._ 

### Rack Mount Home Lab Cooling Fans

![AC Infinity CLOUDPLATE T7-N](https://static.linuxblog.io/wp-content/uploads/2020/02/AC-Infinity-CLOUDPLATE-T7-N-868x269.jpg "AC Infinity CLOUDPLATE T7-N")

Maintaining an optimal temperature for rack-mounted equipment prevents overheating, ensures consistent performance, and extends their lifespan. The plan is to mount an intake fan at the bottom rack unit position (remember hot air rises). After testing, I will decide whether I’ll need an exhaust fan at the very top of the rack.

-   [AC Infinity CLOUDPLATE T7-N, Rack Mount Fan Panel 2U, Intake for 19” Racks][49] – $130 ([specs][50]) – an intelligent cooling fan system providing quiet high airflow cooling.  The system is housed in a solid aluminum 2U case that’s less than 4 inches deep. It includes a thermostat for automatic fan speed during varying temperatures. A max airflow rating of 220 CFM will provide useful cooling to server, network, and other equipment.
-   [AC Infinity CLOUDPLATE T1-N, Rack Mount Fan Panel 1U, Intake for 19” Racks][51] – $150 ([specs][52]) – A more compact 1U design that moves less air (up to 60 CFM) if you are low on free rack units.
-   [AC Infinity CLOUDPLATE T2, Rack Mount Fan Panel 1U, Top Exhaust for 19″ racks][53] – $150 ([specs][54]) – This 1U top exhaust cooling fan can move up to 300 CFM. At 13.5″ deep it will be either a tight squeeze or will need modification for my rack. So I have not ordered this until I can examine it closer.

_**Update:** At the lowest fan speed, this cools rack temps by about 2 to 3 degrees Fahrenheit. I’ll update my thoughts again in the summer._

**_Update 2:_** As of September 2022, two of the four fans have failed. I’ve removed it from the rack and will try to clean it with compressed air. Over the past 2 years of use, they are filled with dust! I still recommend buying these units, but I also recommend cleaning them out more often than I have!

_**Update 3:** As of September 2023 – 3 of the 4 fans have failed again._ 

### Home Lab Routers and switches

![Ubiquiti Networks EdgeRouter ER-10X](https://static.linuxblog.io/wp-content/uploads/2020/02/Ubiquiti-Networks-EdgeRouter-ER-10X-868x304.jpg "Ubiquiti Networks EdgeRouter ER-10X")  
After many weeks of research, online comparisons, reading reviews, watching YouTube videos, and countless hours spent on [/r/homelab][55], I can safely say that the following three companies’ devices will fully cover all of your routing and switching needs: [Cisco][56] > [Ubiquiti][57] > [TP-Link][58]. Sorry TP, nothing personal you are just 3rd, but not last; is that better?

For my setup, I’m going more small-business than a home office as far as equipment goes. I want to be able to have future access to additional features. The following were my selections, followed by the runner-ups.

**Routers**

-   ****Multi-WAN Load balancing:**** [Peplink Balance 20X \[CAT 7\] | Futureproof Gigabit Dual WAN Router][59] – $500 – Simultaneous Dual-Band (2.4GHz / 5GHz), Wi-Fi 5 1x WAN Port, 4 x LAN Port, 1x Embedded Cellular Modem with Redundant SIM Slots, 1x USB Interface 1 Gbps Router Throughput.
-   **Small Biz features:** [Ubiquiti EdgeRouter ER-10X, 10 Port Gigabit Router with PoE Flexibility][60] – $220 ([specs][61]) – (10) Gigabit RJ45 Ports, PoE Passthrough on Port 10, Dual-Core, 880 MHz, MIPS1004Kc Processor, 512 MB DDR3 RAM, 512 MB NAND Flash Storage, Internal Switch, Serial Console Port.
-   **Home lab Value:** [TP-Link ER7206 Multi-WAN VPN Router][62] – $150 (specs) – 1 Gigabit SFP WAN Port + 1 Gigabit WAN Port + 2 Gigabit WAN/LAN Ports plus1 Gigabit LAN Port. Supports up to 100× LAN-to-LAN IPsec, 50× OpenVPN, 50× L2TP, and 50× PPTP VPN connections.

**Switches**

-   **Enterprise features:** [Ubiquiti EdgeSwitch ES-10XP, Managed 10-Port Gigabit Switch with PoE][63] – $215 ([specs][64]) – (8) Gigabit RJ45 Ports, (2) SFP ports, 24V passive PoE output on all RJ45 ports, 10 Gbps Total Non-Blocking Throughput, 20 Gbps Switching Capacity.
-   **Home lab Value:** [TP-Link 8-Port Gigabit Ethernet Easy Smart Switch (TL-SG108E)][65] – $40 ([specs][66]) Unmanaged Pro, Plug and Play, Shielded Ports.

There’s also [Netgear][67], [pfSense][68], and many, many other options, but you want me to complete this article, don’t you? Of course, this means that you will still have to research the recommendations on your own, as not all of my picks may fit your requirements.

_**Update**: Just what I needed. The router runs modified Debian. How cool! It’s packed with enterprise features. I’m excited to learn full command line management._ 

### Home Lab Patch Panels and Network Cables

![16 port patch panel](https://static.linuxblog.io/wp-content/uploads/2020/02/16-port-patch-panel-2-868x234.jpg)

In hindsight, I should have gone with the [24 port patch panel][69]. Instead, I went with [this 16 port patch panel][70] because, well, I liked how centered and less busy it looked. I know. I don’t want to hear it. But look at that thing! If it isn’t already obvious by the 16 ports, it’s the image just above.

The fastest internet plan available from my ISP is 160Mbps down / 30Mbps up. For my home lab, I won’t be needing CAT8 or CAT7 anytime soon, and I don’t need high LAN traffic. So I purchased [250′ of regular CAT6 reel][71] and [some different lengths Cat6][72] as I’m ok with the 1000Mbps limit. Yeah, I also like my coffee black. If you want smoother network traffic, then, by all means, go for at least [CAT6a][73]. However, make sure you go with a [shielded patch panel][74]. Going CAT7 cable or Cat6a is a waste of money if your RJ45 connectors and patch panels don’t support it. “Keep it simple stupid!” No, I’m not talking to you; that was me speaking to myself while selecting the patch panel and cables.

_**Update:** High quality! Easy to punch down. All my patch connections worked on the first try._

### Home Lab Servers

![Thinkcentre as a server](https://static.linuxblog.io/wp-content/uploads/2020/02/thinkcentre-as-a-server-868x175.jpg "Thinkcentre as a server")

For this article’s purpose, I won’t recommend any specific servers because this will vary a ton depending on what you will be hosting. NAS storage, VMs, web servers, backup servers, mail servers, ad blockers, and all the rest. For my requirements, I purchased a [ThinkCentre M73][75] and [ThinkCentre M715q][76]; both used off eBay. As far as full-fledged servers go, some good options are [Dell][77], [HP,][78] [Cisco][79], and [Lenovo][80]. For home labs, these are often best purchased _used_ from Craigslist or eBay. You can use Wikipedia to search the specs of older generations, for example, [Dell PowerEdge model history][81].

_**Update:** AMD has raised the bar. I’m most impressed with the CPU performance of the M715q. They both run quiet and cool, with Ubuntu Server and Windows 10._ 

## Summary and Conclusion

![homelab](https://static.linuxblog.io/wp-content/uploads/2020/02/homelab-868x376.jpg "homelab")  
_[Image][82]: your setup, should be your setup. As traditional or as custom as you’d like!_

When writing this guide, I tried my best to convey my passion, thoughts, and considerations throughout the selection process. It’s important that you find your passion when it comes to building your home lab. What will you use it for? How will it benefit you and others?

If you have a plan, and if your lab will be something you enjoy learning, failing, and succeeding with, it will pay for itself in one way or another. That said, be careful with the power consumption! You don’t want to end up with… _mortgage of $1,100, a power bill of $1,200, the feeling when walking into your home lab… NOT priceless._

Plan with room to expand, and feel free to start small. You don’t have to start with a rack, cabinet, patch panels, and all the other gear. Get your feet wet, replace your ISP modem, then build from there. Avoid making rash decisions on hardware; instead, sleep on it. Then, post a question or two in the **[new community forums][83]**, to get valuable feedback from other techies and sysadmins. You will find that the more you research, it will open up additional ways to get things just right or very wrong. More importantly, have fun! Share your home lab goals with your spouse, colleagues, kids, and that neighbor across the street who still only waves after five years.

I hope this article was useful and inspiring and opens a dialogue between us. Feel free to [contact me][84]. I love hearing, supporting, and learning from you guys!

**_Update 1 – March 10th, 2020:_**

As of today, I’ve almost completed the setup of my new Home lab, and it’s now fully functional. I’m pleased with all of the items purchased. For this update, I’ve added to the article some highlights, tips, and findings below each hardware item in _italics_.

One thing I missed in the original article and Home lab setup was the coaxial cable. The existing cable turned out to be about three feet shorter than required. Tomorrow, I’ll replace it with a longer cable. As you can see, I’ve also replaced the ISP modem with the Motorola model listed below.

Unfortunately, the [Ubiquiti EdgeRouter and EdgeSwitch][85] turned out to be about an inch too wide. I failed to consider this because I measured their combined width and then compared that measurement to the rack’s interior width instead of the width of the rack shelf. Oops! Now, so that the switch doesn’t remain sitting on the router, I’ve just ordered two [Ubiquiti rack mounts][86]. Better cooling and aesthetics, but also the rack mounts will allow positioning the router and switch directly below the 16 port patch panel pictured below.  Check back in a week or two for another update.

![homelab update March 2020](https://static.linuxblog.io/wp-content/uploads/2020/02/homelab_update-868x670.jpg "homelab update March 2020")  
_My Home lab  – a work in progress (everything pictured is listed below)_

_**Update 2 – March 25th, 2020: I Will follow up with details and feedback shortly. Here’s an updated status photo:**_

![homelab update 2](https://static.linuxblog.io/wp-content/uploads/2020/03/homelab_update2-868x868.jpeg)

_**Update 3 – April 19th, 2020.**_

Over the past few weeks, I’ve added a few things.

-   A [Netgear 4G LTE modem][87] (and [antenna][88]) for internet auto fail-over from ISP1 (cable internet) to ISP2 (4G internet).
-   [1U Blanking Panels][89] to improve the efficiency of the cooling fans.
-   The previous two vented 1U blanks are now at the top of the rack. It will be used to mount a monitor. (see to-do’s below)
-   Quick access [1u mess cover][90] to prevent accidental power switch toggling.
-   [Rackmount kits][91] for the [E][92][dgeRouter ER-10X][93] and [EdgeSwitch ES-10XP][94].
-   Cleaned up all wires and cables. Labeled network cables but didn’t label the patch panel, of course!
-   Added VLANs, restricted bandwidth of guest devices as well as some other home devices, hardened firewall.
-   Installed Zabbix server for additional monitoring of 150+ remote servers.
-   Added a domain to static IP, which has a couple of public-facing subdomains.
-   Removed or smoked out some of the logos and text for a cleaner look. (aka. OCD)
-   Added [x2 UAPs][95] and [x2 beacons][96].
-   [Outlet savers][97] so I could make use of all available power outlets.
-   Added push notification for when ISP1 goes down since switching to backup ISP2 is completely seamless.

To-do’s:

-   Add a separate 21″ monitor to display Zabbix hosts/dashboard.
-   Dabble with [pfSense][98]. Reset everything to defaults and reconfigure with lessons learned.
-   Add indoor PoE surveillance cameras (currently only outdoor cams).
-   Set up KVM desktop switch to be able to switch easier between the two servers and desktop.
-   Check rack temperatures from June to August to determine if the setup is as cooling efficiently as planned.

Latest home lab photo:

![Home Lab #homelab](https://static.linuxblog.io/wp-content/uploads/2020/03/homelab.jpg "Home Lab #homelab")  
![Home Lab cables](https://static.linuxblog.io/wp-content/uploads/2020/03/homelab_cable_runs-868x338.jpg "Home Lab cables")

_**Update 4 – May 1st, 2020:**_

-   Added a separate 21″ monitor to display remote hosts/dashboard using Raspberry Pi and Zabbix.

![May 2020 home lab](https://static.linuxblog.io/wp-content/uploads/2020/03/may_2020_home_lab.jpeg "May 2020 home lab")

_**Update 5 – June 7th, 2020:**_

-   Added keyboard and mouse for walk-by access to Zabbix UI.
-   Swapped out mesh security cover with tinted plexiglass.
-   At this point, my home lab is complete. Is it ever? :)

![home lab June 2020](https://static.linuxblog.io/wp-content/uploads/2020/03/home-lab.jpg)

_**Update – August 19th, 2021:**_

A month ago a friend asked if I could set up something similar for their home. They wanted a simple whole-home VPN setup. Pictured below is the result. All together uses less than 40 watts via a 700VA UPS.

-   [StarTech.com 6U Wall Mount Network Equipment Rack – 14 Inch Deep][99]
-   [AC Infinity Vented Cantilever 1U Universal Rack Shelf, 10″ Deep][100]
-   [C2G 12-Port Patch Panel][101]
-   [VCE UL Listed CAT6 RJ45 Keystone Jack Inline Couplers][102]
-   [Vilros – Raspberry Pi 4 2GB Basic Kit][103] – running [Unifi Controller][104]
-   [Brume (GL-MV1000)][105] – [Edge Gateway + VPN (wireguard)][106]
-   [Edgeswitch 10xp][107]
-   [StarTech.com 8 Outlet Horizontal 1U Rack Mount PDU Power Strip][108]
-   [CyberPower SL700U Standby UPS System, 700VA/370W, 8 Outlets, 2 USBs][109]

![Whole Home VPN - Homelab](https://static.linuxblog.io/wp-content/uploads/2021/08/whole-home-VPN-homelab.jpg "Whole Home VPN - Homelab")

_**Update – October 7th, 2022:** Added update on the AC Infinity CLOUDPLATE T7-N, Rack Mount Fan Panel 2U (Intake). Make sure to clean the dust from these units every couple of months. It took 2 years without any maintenance for 2 of the 4 fans to become disabled. I will update if they work again once cleaned._ 

_**Update – December 13th, 2022:  
**__We moved to a new home in February. So I removed the home lab – cables and all. Finally had the time to set up. I’ve made very small changes because I really like it as is._

_**Changes:**_

-   _Mounted at a more ergonomic height.  
    _
-   _Flipped some devices around (for easier access to unplugging/power buttons)_
-   _Replaced the EdgeRouter with a new [Peplink Balance 20x][110] for ISP [load balanced using the “fastest response” method][111]._

_[![Peplink Balance 20x Router Review](https://static.linuxblog.io/wp-content/uploads/2023/02/Balance-20x-blog-article-image-868x580.png "Peplink Balance 20x Router Review")][112]_

_**Update: May 3rd, 2023:**_

-   _Purchased a [1500VA Cyberpower UPS][113] and o__ffloaded 3 PC monitors and a gaming PC. So that the in-rack 1u UPS only connected to the servers and network devices. This allows for up to 3 hours of runtime on battery if needed._ 

**Update: Sept 24th, 2023:**

-   _Added information above on failure of the AC Infinity CLOUDPLATE T7-N, Rack Mount intake fans._ 
-   _Added a link to my [Peplink Balance 20x Router Review][114]._

_Published: Feb 25th, 2020 | Last updated: March 8th, 2024_

**Additional related articles:** 

-   [5 Network Devices for work-from-home and Small Business][115].
-   [Finding Linux Compatible Printers][116].
-   [Home Lab inspiration – A letter from a reader.][117]
-   [After 10 Yrs of Linux, I Switched to Windows. What next?][118]

[1]: https://linuxblog.io/home-lab-beginners-guide-hardware/ "Home Lab Beginners guide (Hardware)"
[2]: https://linuxblog.io/about-me/ "Posts by Hayden James"
[3]: https://linuxblog.io/category/articles/
[4]: https://linuxblog.io/category/linux/
[5]: https://linuxblog.io/web-server-services/
[6]: https://stacklinux.com/
[7]: http://www.yvesephraim.com/
[8]: https://www.cwnetworks.com/
[9]: https://www.google.com/intl/en/ipv6/statistics.html
[10]: https://www.bgp4.as/
[11]: #What_is_a_Home_Lab "What is a Home Lab?"
[12]: #May_we_inspire_each_other_to_great_home_labs "May we inspire each other to great home labs"
[13]: #Home_lab_location_its_all_about_location "Home lab location, it’s all about location."
[14]: #Network_vs_Server_Racks_vs_Cabinets "Network vs. Server Racks vs. Cabinets?"
[15]: #Recommended_Home_Lab_Hardware "Recommended Home Lab Hardware"
[16]: #To_replace_or_not_to_replace_ISP_Cable_Modem "To replace or not to replace: ISP Cable Modem"
[17]: #Selecting_Your_Home_Lab_Rack "Selecting Your Home Lab Rack"
[18]: #Home_Lab_UPS_Uninterruptible_Power_Supply "Home Lab UPS (Uninterruptible Power Supply)"
[19]: #Universal_Home_Lab_Rack_Shelf "Universal Home Lab Rack Shelf"
[20]: #Rack_Mount_Home_Lab_Power_supply "Rack Mount Home Lab Power supply"
[21]: #Rack_Mount_Home_Lab_Cooling_Fans "Rack Mount Home Lab Cooling Fans"
[22]: #Home_Lab_Routers_and_switches "Home Lab Routers and switches"
[23]: #Home_Lab_Patch_Panels_and_Network_Cables "Home Lab Patch Panels and Network Cables"
[24]: #Home_Lab_Servers "Home Lab Servers"
[25]: #Summary_and_Conclusion "Summary and Conclusion"
[26]: https://linuxblog.io/20-top-server-monitoring-application-performance-monitoring-apm-solutions/
[27]: https://linuxcommunity.io/t/how-to-build-a-homelab/49
[28]: https://www.smartdraw.com/network-diagram/how-to-draw-network-diagrams.htm
[29]: https://www.reddit.com/r/homelab/comments/ep9bqy/i_see_your_fancy_rack_and_raise_you_a_stylish_rack/
[30]: https://amzn.to/32lbV1h
[31]: https://amzn.to/3ZwNtrn
[32]: https://amzn.to/3a5Lmzx
[33]: https://amzn.to/39XLcdz
[34]: https://www.startech.com/Server-Management/Racks/12u-wall-mount-rack~WALLMNT12
[35]: https://amzn.to/2STvDy3
[36]: http://www.samsontech.com/samson/products/accessories/racks/srk8/
[37]: https://imgur.com/BAGlEvA
[38]: https://amzn.to/3974q0c
[39]: https://navepoint.com/navepoint-9u-450mm-depth-wallmount-networking-cabinet-assembled-basic-series/
[40]: https://amzn.to/38QM7wi
[41]: https://www.cyberpowersystems.com/product/ups/smart-app-lcd/or500lcdrm1u/
[42]: https://amzn.to/2vdi1EO
[43]: https://amzn.to/3T8OXnf
[44]: https://amzn.to/2SVmwga
[45]: https://www.acinfinity.com/rack-accessories/rack-shelves/vented-cantilever-1u-rack-shelf-10/
[46]: https://amzn.to/2SWNG6p
[47]: https://www.adj.com/pc-100a
[48]: https://amzn.to/3c4nbDx
[49]: https://amzn.to/2T65qes
[50]: https://www.acinfinity.com/component-cooling/rack-fan-systems/cloudplate-t7-n-quiet-rack-cooling-fan-system-2u-intake/
[51]: https://amzn.to/2T65qes
[52]: https://www.acinfinity.com/component-cooling/rack-fan-systems/cloudplate-t7-n-quiet-rack-cooling-fan-system-2u-intake/
[53]: https://amzn.to/38VfUUt
[54]: https://www.acinfinity.com/component-cooling/rack-fan-systems/cloudplate-t2-quiet-rack-cooling-fan-system-1u-top-exhaust/
[55]: https://www.reddit.com/r/homelab/
[56]: https://www.cisco.com/c/en_my/solutions/small-business/products/routers-switches.html
[57]: https://store.ui.com/collections/routing-switching
[58]: https://www.tp-link.com/en/business-networking/
[59]: https://amzn.to/46qVSPa
[60]: https://amzn.to/38WhPIE
[61]: https://www.ui.com/edgemax/edgerouter-10x/
[62]: https://amzn.to/46vx1Ki
[63]: https://amzn.to/2HPDuGi
[64]: https://www.ui.com/edgemax/edgeswitch-10xp/
[65]: https://amzn.to/3a4v0Yi
[66]: https://www.tp-link.com/en/business-networking/easy-smart-switch/tl-sg108e/
[67]: https://www.netgear.com/home/products/networking/switches/
[68]: https://www.pfsense.org/
[69]: https://amzn.to/2Tflgn9
[70]: https://amzn.to/2Vh9lYA
[71]: https://amzn.to/2PldEy5
[72]: https://amzn.to/37XehEK
[73]: https://blog.tripplite.com/cat6-vs-cat6a-ethernet-cables-everything-you-need-to-know
[74]: https://amzn.to/2w07tc3
[75]: https://www.lenovo.com/us/en/desktops/thinkcentre/m-series-tiny/m73/
[76]: https://www.lenovo.com/us/en/desktops-and-all-in-ones/thinkcentre/m-series-tiny/ThinkCentre-M715q-Tiny/p/11TC1MT715Q
[77]: https://www.dell.com/en-us/work/shop/servers/cp/servers-storage-networking
[78]: https://www.hpe.com/us/en/servers.html
[79]: https://www.cisco.com/c/en/us/products/servers-unified-computing/index.html
[80]: https://www.lenovo.com/gb/en/data-center/servers/c/servers
[81]: https://en.wikipedia.org/wiki/List_of_Dell_PowerEdge_Servers
[82]: https://imgur.com/gallery/i8YdrnU
[83]: https://linuxcommunity.io/t/how-to-build-a-homelab/49
[84]: https://linuxcommunity.io/
[85]: https://www.ui.com/products/#edgemax
[86]: https://amzn.to/2PZPFVu
[87]: https://amzn.to/2wNWCTg
[88]: https://amzn.to/2XORKIE
[89]: https://amzn.to/2XPIC6v
[90]: https://amzn.to/2XLWejd
[91]: https://amzn.to/3bpqZOU
[92]: https://amzn.to/38WhPIE
[93]: https://amzn.to/38WhPIE
[94]: https://amzn.to/2HPDuGi
[95]: https://amzn.to/3bn1YEc
[96]: https://amzn.to/3anxuAq
[97]: https://amzn.to/2VFwkez
[98]: https://www.pfsense.org/
[99]: https://amzn.to/3z7IejK
[100]: https://amzn.to/2UvZ6S7
[101]: https://amzn.to/3z7I2kw
[102]: https://amzn.to/3y2E6QT
[103]: https://amzn.to/3gjDVuq
[104]: https://www.ui.com/
[105]: https://amzn.to/3mgx8W1
[106]: https://linuxblog.io/5-network-devices-for-work-from-home-and-small-business/
[107]: https://www.ui.com/edgemax/edgeswitch-10xp/
[108]: https://amzn.to/3y2nHf1
[109]: https://amzn.to/3gh8gK0
[110]: https://www.peplink.com/products/balance-20x/
[111]: https://www.peplink.com/technology/load-balancing-algorithms/
[112]: https://linuxblog.io/peplink-balance-20x-router-review/
[113]: https://amzn.to/44lKNPd
[114]: https://linuxblog.io/peplink-balance-20x-router-review/
[115]: https://linuxblog.io/5-network-devices-for-work-from-home-and-small-business/ "5 Network Devices for work-from-home and Small Business"
[116]: https://linuxblog.io/finding-linux-compatible-printers/
[117]: https://linuxblog.io/home-lab-inspiration/ "Home Lab inspiration – A letter from a reader"
[118]: https://linuxblog.io/10-yrs-of-linux-switched-to-windows-what-next/ "After 10 Yrs of Linux, I Switched to Windows. What next?"