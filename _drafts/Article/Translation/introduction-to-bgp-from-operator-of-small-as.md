---
title: An Introduction to BGP... from the operator of a small AS
date: 2023-07-14T05:05:37.000Z
authorURL: ""
originalURL: https://quantum5.ca/2023/07/14/introduction-to-bgp-from-operator-of-small-as/
translator: ""
reviewer: ""
---

# An Introduction to BGP... from the operator of a small AS

<!-- more -->

[BGP][1]

12 minutes

Quantum

[qt.ax/bgp][2]

[Border Gateway Protocol][3] (often abbreviated BGP) is a critical protocol that makes the modern Internet possible, yet remains one of its most poorly understood parts even among its long-time users. At the same time, it has played a significant role in several high-profile outages on the Internet. As someone who has been running my “own piece of the Internet”—[AS200351][4]—for half a year now, I think the time has come to write a piece explaining exactly what BGP is, what AS200351 is, and how the Internet truly functions behind the scenes. We’ll start with the basics.

To understand BGP, we must first understand why it is called the “Internet” in the first place. To simplify greatly, the Internet is called that because it’s an _inter_connected network of _net_works (more precisely, [autonomous systems][5], or ASes) glued together by BGP. Since this sounds like a nonsensical sequence of words, let’s dive a bit deeper.

## What is a network?

Let’s start by considering a very simple home network belonging to Alice without considering the interconnected aspect. For the sake of simplicity, we’ll use the more familiar IPv4[1][6] addresses from the 192.168.0.0/16 range.

**A quick note on notation:** IPv4 addresses consist of four bytes (8 bits each) for a total of 32 bits. Each byte is conventionally expressed in decimal, separated by dots. The /16 refers to all IPv4 addresses that share the first 16 bits (or the first two decimal numbers in the dotted notation) as 192.168.0.0, i.e. every address that starts with 192.168. This is called the [CIDR notation][7].

Here’s Alice’s home network:

![Simple network](/assets/bgp/simple-site-122138c86761d2238169e60ec103c1a7f0defd46f99091819511b0b447f884d38de44308d3c43746818ad7f997b9ddb0f9942dffb2c16d42c2c26595807349cc.svg)

In the diagram, there is a router at 192.168.1.1, which currently does nothing since it’s not connected to any other network, and two computers—192.168.1.2 and 192.168.1.3, all connected to a [switch][8]. We refer to this network as 192.168.1.0/24 in CIDR notation, containing addresses 192.168.1.x.

Note that in a common home setup, the router and switch are usually bundled into a single appliance, but these are logically two distinct functions. A switch essentially connects all the devices attached to it together in such a way that all devices can talk directly to each other (technically called a single broadcast domain). A router, on the other hand, connects distinct broadcast domains together—without requiring all involved devices to see each other directly. Alternatively, in the words of ChatGPT[2][9]: “A network switch is like a special box that helps devices in the same house talk to each other, while a router is like a magic door that helps different houses talk to each other.” (I am not sure how helpful this actually is.)

In the world of networking, such a network might be referred to as a _site_, since this is a network existing in only one location.

## Connecting sites

Now, what if Alice’s uncle wants a network in his home too and to be able to talk to Alice’s home network? He can get a similar setup, with a router at 192.168.2.1 for his router, etc. to create the network 192.168.2.0/24.

The obvious question is: how would we connect 192.168.1.0/24 and 192.168.2.0/24 together? Well, this is where the routers come in. Let’s connect the routers together on a separate network, then give the router in Alice’s home the external address 192.168.0.1 and the router in her uncle’s house 192.168.0.2:

![Two sites connected](/assets/bgp/simple-as-65980e97358045a14a90518c714f0301a24938c47dfb6139b4988eb4319fdeaa866c88d70eee9f8f3d39ca3d91374502a9e623d44cdb713c38c4676eafd26faa.svg)

Now, we can tell 192.168.0.1 that all packets destined for 192.168.2.0/24 should be sent to 192.168.0.2. This is called a _route_. Then, if 192.168.1.2 wants to talk to 192.168.2.3, it can send packets to the router 192.168.1.1, which is also known as 192.168.0.1, to forward the packet to 192.168.0.2, who is also known as 192.168.2.1, to send it to the final destination of 192.168.2.3. Of course, to receive a reply, we also need to tell 192.168.0.2 that all packets destined for 192.168.1.0/24 should be sent to 192.168.0.1. Explicitly defining routes this way is called _static routing_.

In this setup, only the routers need to be aware of each other. For the computers on each network, all they have to know is that packets going to destinations that are not in the same network should go to the router, who knows what to do with them.

In the world of BGP, a group of sites run by the same entity might be called an “autonomous system” or an AS. We’ll define this term formally later.

## Connecting ASes

Now, consider Bob who has a similar setup for his family (naturally, with a different set of IP addresses). Now, what if Alice and Bob want to connect their networks (autonomous systems) together so that they can all talk to each other? Once again, they can set up static routes on their routers so they know how to find each other’s prefixes.

However, there is a problem here—let’s consider what happens if Alice’s other uncle wants to add a network. Alice can easily change the static routing tables of all routers she manages, but what about Bob? Alice would have to tell Bob to add the new route to _his_ routing tables on _all his routers_. This is sort of annoying. Now imagine if it’s not just Alice and Bob, but ten people, or worse, a hundred people or a thousand people—Alice would have to tell them all to change their routing table so that they know how to reach her new prefix. This is a _huge_ pain and clearly doesn’t scale.

## Enter BGP

The early days of the Internet worked like the example above. Naturally, it ran into the same issues. Every time an autonomous system, which represented universities or similar entities, added a new IP prefix, _everyone else_ needs to be notified to update their routes. Soon, it became clear that something automatic was needed.

Thus, two engineers sat down at a restaurant and came up with a new protocol on the backs of three ketchup-stained napkins (or two, depending on who you ask[3][10]). The result was the “three-napkin protocol”, more formally known as Border Gateway Protocol or BGP.

With BGP, Alice’s routers can talk to Bob’s routers and announce to each other the destinations that they can reach. The receiving side can add all the routes received from the other routers over BGP to their own routing tables so that the attached computers can reach those destinations. Each BGP connection is called a _session_, and each end of the session is called a _peer_. When two routers are connected to each other, they are said to be peered.

## What is an autonomous system?

Now, I’ve been using the term “autonomous system” quite a bit, but haven’t really defined it. The formal definition is something like “a group of IP networks run by one or more network operators with a single clearly defined routing policy,”[4][11] but that sounds like a bunch of meaningless words.

It might be more useful to think of an autonomous system as a bunch of networks controlled by some entity that bothered to get a globally unique number assigned. That number is called an ASN (autonomous system number). The entity in question is most often some kind of an ISP (Internet Service Provider), but it could be anyone. For example, I run [my own autonomous system][12] that was assigned the number 200351. Typically, autonomous systems are referred to ASxx, where xx is the ASN.

Some examples of autonomous systems include:

-   AS13335: Cloudflare, Inc.;
-   AS15169: Google LLC;
-   AS577: Bell Canada (a Canadian residential ISP);
-   AS1299: Arelion (a large ISP that connects other ISPs); and
-   AS200351: Guanzhong Chen (that’s me!).

Of course, there are many more, totalling over 100 000100\\,000 as of 2023. These autonomous systems can interconnect with each other, and together they constitute the Internet.

## What’s the Internet?

At this point, “autonomous systems connected to each other by BGP” is a lot more meaningful. However, there’s a bit more to it than that. As you might imagine, it’s not practical to connect all ASes together directly, since there are over 100 000100\\,000 of them.

A key advantage of BGP is that not every network needs to be peered directly. For example, if Alice and Bob are peered, and Bob and Carol are peered, then Alice and Carol don’t need to be peered to be able to connect to each other. Instead, Bob can announce Alice’s routes to Carol and Carol’s routes to Alice. Then, Alice and Carol can communicate through Bob.

This aspect is really how BGP holds the Internet together—your ISP (Internet Service Provider) doesn’t need to peer with every other AS in the world. Instead, as long as there is a series of other ISPs willing to connect your ISP to any given destination, your ISP can reach all of the Internet. In BGP terminology, the intermediate ISP is said to offer “transit”, and we’ll dive into this in the future.

As you can see, the Internet is really an interconnected network of autonomous systems, glued together by BGP.

## How does BGP work?

Now, it would be crazy to end a post about BGP after only talking about what BGP does and not how it works, so let’s do a quick overview without diving too deeply into the weeds.

BGP peers talk to each other over [TCP][13] port 179. Typically, both peers listen on the port and try connecting to the other, and it doesn’t matter which one is the server and which one is the client, as long as a connection is established.

The BGP peers can, in fact, belong to the same AS. When used this way, it’s called _Internal_ BGP or iBGP, as opposed to _External_ BGP or eBGP when used between two different ASes. In the case of iBGP, this can solve the problem of Alice managing all her sites without relying on static routing, to use our example above.

Logically, each BGP peer can perform two very simple operations: _announcing_ a route and _withdrawing_ a route. If a BGP connection is lost, each peer will deem the other peer to have withdrawn all their routes.

Each route includes the following basic information:

-   Prefix: the IP prefix this route is meant to reach;
-   Next hop: the IP address to which the peer should forward the packets destined to the prefix;
-   AS path: the chain of ASNs through which the final destination is reached; and
-   Communities: extra tags on the route.

(There are obviously more, but these are the most important.)

As you can see, BGP is a very simple protocol, perhaps too simple—it doesn’t really have any built-in security mechanisms[5][14], for example. The complexity doesn’t come from BGP itself, but arises from the network it is used to build. The more complex the network, the fancier the routing logic, the more routes there are, resulting in the huge and complex network that is the Internet.

## Why does BGP cause outages?

From all the information here, we can now see why BGP is involved in so many Internet outages. BGP tells the routers of the Internet how to connect the whole thing together, and with one mistake, things could stop working.

There are really two main failure modes with plain BGP:

1.  Mistakenly announcing routes. Basically, a peer that’s not supposed to announce a route announces it anyways, whether by accident (route leak) or maliciously ([BGP hijack][15]). For example, in 2008, Pakistan tried to block YouTube by announcing routes to YouTube internally that led nowhere, but accidentally leaked it to the whole Internet, bringing down YouTube. Malicious attackers can also do this to impersonate websites or capture traffic.
2.  Mistakenly withdrawing routes. For example, in the [2021 Facebook outage][16], Facebook accidentally withdrew the routes to its DNS server, the result being that no one on the Internet could reach their DNS servers and obtain the IP addresses for `facebook.com` et al. Naturally, this brought down Facebook.

## Next time?

That concludes our basic introduction to BGP. [Next time][17], I’ll explain a bit more about autonomous systems and how they are organized into different tiers. This should give you a better understanding of how the Internet is organized and how relationships between ASes work.

## Notes

1.  IPv4 is the legacy version of the Internet Protocol, and the addresses look like 192.0.2.123. This is still the most common version and familiar to most people, hence why I am using it in the examples here. However, it’s beginning to show its age—the 2322^{32} total addresses are running out. IPv6 is the new version with 21282^{128} addresses, a mind-bogglingly large number. Its addresses look like 2001:db8::123 or 2001:db8:1234:5678:90ab:cdef:dead:beef. Due to IPv4 address scarcity, AS200351 is IPv6-only. [↩][18]
    
2.  Specifically, when GPT-3.5 is asked to explain the difference between routers and switches to a 5-year-old. [↩][19]
    
3.  The Internet seems pretty evenly divided on this. I’ve seen [a picture][20] that purports to be of the three napkins in question, so I am going with three. [↩][21]
    
4.  This is the [definition given by RIPE NCC][22] (_Réseaux IP Européens_ Network Coordination Centre), the [regional Internet registry][23] for Europe and Western Asia. We’ll learn more about them when we dive deeper into ASes. [↩][24]
    
5.  This is a pretty big problem, actually. There are solutions such as [RPKI][25] and [ASPA][26] that are aimed at fixing security issues, but adoption leaves a lot to be desired. The Internet runs on trust and lots and lots of route filters. [↩][27]
    

Please enable JavaScript to view the [comments powered by Disqus.][31]

[][32]

[1]: /category/bgp/
[2]: https://qt.ax/bgp "Border Gateway Protocol"
[3]: https://en.wikipedia.org/wiki/Border_Gateway_Protocol
[4]: https://as200351.net
[5]: https://en.wikipedia.org/wiki/Autonomous_system_(Internet)
[6]: #fn:1
[7]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation
[8]: https://en.wikipedia.org/wiki/Network_switch
[9]: #fn:2
[10]: #fn:3
[11]: #fn:4
[12]: https://as200351.net
[13]: https://en.wikipedia.org/wiki/Transmission_Control_Protocol
[14]: #fn:5
[15]: https://en.wikipedia.org/wiki/BGP_hijacking
[16]: https://en.wikipedia.org/wiki/2021_Facebook_outage
[17]: /2023/07/19/diving-into-autonomous-systems/
[18]: #fnref:1
[19]: #fnref:2
[20]: /assets/bgp/napkins-90c13247921d7171f9eee3008d900532bca3423e2bdb772589ce086c161f947d45672f0e71038994bcd8d6783a753303b7d128fd71e1526133cd0163431c6017.png
[21]: #fnref:3
[22]: https://www.ripe.net/publications/docs/ripe-679#Definition
[23]: https://en.wikipedia.org/wiki/Regional_Internet_registry
[24]: #fnref:4
[25]: https://en.wikipedia.org/wiki/Resource_Public_Key_Infrastructure
[26]: https://datatracker.ietf.org/doc/html/draft-ietf-sidrops-aspa-verification
[27]: #fnref:5
[28]: /2024/10/31/implementing-aspa-validation-in-bird2-filter-language/
[29]: /2024/06/23/on-inter-rir-transfer-as200351-from-ripe-ncc-to-arin/
[30]: /2023/12/21/bgp-route-selection-high-availability-anycast/
[31]: https://disqus.com/?ref_noscript
[32]: /2023/07/14/introduction-to-bgp-from-operator-of-small-as/