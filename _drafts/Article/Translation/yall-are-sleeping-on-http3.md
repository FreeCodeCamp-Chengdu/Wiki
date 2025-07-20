---
title: Y’all are sleeping on HTTP/3
date: 2025-07-20T14:55:59.365Z
authorURL: ""
originalURL: https://kmcd.dev/posts/yall-are-sleeping-on-http3/
translator: ""
reviewer: ""
---

# [Y’all are sleeping on HTTP/3][1]

<!-- more -->

[][2]11 min 2,220 words 2024-08-06[][3][][4]

[article][5] [http][6] [http2][7] [http3][8] [web][9] [webdev][10]

## Wake up call

This post is about HTTP/3 and QUIC. If you don’t know what that is, there are [many][11], [many][12], [many][13], [many][14], [many][15], [many][16], [many][17] good resources that will get you up to speed. I’m writing this post to enlighten people on what has been happening in the last few years.

**[All major browsers][18]** support HTTP/3 now.

**[Most major cloud providers][19]** support HTTP/3 now.

**[Most major load balancers][20]** support HTTP/3 now.

In a short few years **[over 30% of web traffic is served with HTTP/3][21]**.

_**HTTP/3 isn’t the future. It’s the present.**_

Every time I mention HTTP/3 there’s always someone who pops up who’s completely unaware that it even exists, thinks that it’s some minor change or thinks that supporting HTTP/3 is cargo cult behavior. Every web developer should probably know that HTTP/3 exists because HTTP/3 is a giant change. HTTP/3 abandons TCP in favor of a channel-aware UDP-based protocol called QUIC. To me, _**this feels important!**_ This feels like people need to be talking about it, doing more experiments around QUIC, and writing more tooling, security analysis and benchmarks. QUIC has the potential to dethrone TCP as the reliable layer 4 protocol. This is a big deal.

Why should you care about HTTP/3 if you’re not a big nerd who likes learning about network protocols? Because it promises to bring a lot of advantages: faster page loads, smoother video streaming, and more resilient connections. Plus, I think web developers need to know about the foundations of their profession. And this foundation has been changing. Rapidly.

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/thinking.png)

## What’s wrong with TCP?

### Multiple streams: One Connection

There are issues with multiplexing multiple streams onto one TCP-backed connection. This is because TCP treats all data on a connection as one big stream. If one part of that stream gets delayed or lost, everything else gets stuck behind it, like a traffic jam on a single-lane road. This is called ‘[head-of-line blocking][22],’ and it can significantly slow down web page loading, especially when multiple resources (like images, scripts, and stylesheets) are being requested simultaneously.

### It isn’t good in dynamic network environments

As mentioned in the last section, TCP connections are slow to set up because of the number of round trips required. In environments that change often, causing your web clients to switch networks (and client IP addresses), the clients will have to re-establish an entirely new connection each time you switch networks. Think about the large pause that happens when you switch wifi networks while using video chat. That’s exactly this issue. TCP was not designed to handle this situation without that pause. In the following section, I’m going to tell you how QUIC (and HTTP/3) handles this situation in a much more reliable and smooth way. This is extremely relevant today in a world where mobile phones can choose to swap between multiple wifi and mobile networks.

### Why can’t I use wifi and mobile at the same time?

This _is_ actually possible with TCP, using a feature called [multipath TCP][23], but the rollout has been slow and difficult for TCP. QUIC’s built-in support for connection migration and 0-RTT resumption offers a smoother and more efficient solution to this problem, potentially enabling true multipath connectivity in the future.

## Enter: QUIC and HTTP/3

### Faster Connections

First off, QUIC requires far fewer round trips to set up an encrypted connection. Instead of 3 needed for TLS+HTTP/2, QUIC just has one.

### Zero Round Trip Time (0-RTT) Resumption

[QUIC connections can be resumed][24], even if the client’s IP address changes. This allows connections to be re-established with _**zero**_ round trips, so when you are on a conference call and switch wifi networks your HTTP/3 connection can resume instantaneously. While there have been security concerns around this feature, such as the potential for replay attacks, these have been largely mitigated through measures like rotating keys and client IDs.

I mentioned earlier how the rollout of multipath for TCP hasn’t been super successful. With the features to support 0-RTT, support for multipath is essentially built-in, so you may see more concurrent usage of wifi, ethernet and mobile networks in the future.

### Multiplexing

In contrast to TCP+HTTP/2, QUIC ensures that packet loss or delays on one stream don’t impact others. This is because QUIC manages ordering guarantees at the individual stream level, not for the entire connection, effectively [eliminating head-of-line blocking][25].

### Improved Congestion Control

QUIC’s [more responsive congestion control][26] leads to faster recovery from packet loss.

### “But I heard UDP is unreliable”

It’s a common misconception that UDP is inherently unreliable compared to TCP. TCP packets are just as unreliable, but they transparently get retried by TCP client and server implementations. In other words, both TCP and UDP packets can be lost or corrupted during transmission; the difference lies in how they handle those errors. While it is true that UDP doesn’t offer the same built-in guarantees as TCP, QUIC implements the same guarantees on top of UDP.

Think of it like this: TCP is like a delivery service that meticulously tracks every package and resends any that go missing. UDP, on the other hand, is more like a bulk mailer, sending out a flood of letters and hoping most of them arrive. However, QUIC builds its own tracking system on top of UDP, ensuring reliable delivery of data.

This means QUIC can enjoy UDP’s speed and flexibility without sacrificing reliability. In fact, in some cases, QUIC can even outperform TCP in terms of packet loss recovery, thanks to its advanced congestion control mechanisms.

So, while the “UDP is unreliable” mantra might have been true in the past, protocols built on UDP can be even more reliable than TCP.

## Let’s see how far we’ve come

Okay, so I mentioned how available HTTP/3 is a few times now. But let’s look at the specifics: browser support, cloud support and support when self-hosting.

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/tech-elite.png)

### Web Browsers

Let’s start with web browsers. HTTP/3 support doesn’t matter if browsers don’t support the technology. [CanIUse][27] shows the support for each browser and it looks very good for all major browsers:

-   [Chrome][28]
-   [Firefox][29]
-   [Edge][30]
-   Opera

The only caveat is that Safari only enables HTTP/3 for a portion of users, but Apple will come around eventually.

### Cloud Providers

Next, if you want to host websites with HTTP/3 it would be awesome if cloud providers supported HTTP/3. Well, you’re in luck. Most major cloud providers do support HTTP/3, including:

-   [Cloudflare][31]
-   [Google Cloud CDN and Load Balancer][32]
-   [AWS CloudFront][33]
-   [Akamai CDN][34]
-   [Azure Application Gateway][35] (not widely available as it’s in Private Preview)
-   [CDN77][36]
-   [Fastly CDN][37]

### Load Balancers

But what if you’re setting up your own infrastructure, foregoing the cloud? What are the available options?

-   [nginx][38]
-   [Envoy][39]
-   [Caddy][40]
-   [LiteSpeed][41]
-   [H2O][42]
-   [traefik][43]
-   [haproxy][44]

### Is HTTP/2 already dying?

Take a look at the comparative usage of HTTP/2 and HTTP/3 over the last few years:

[Source: w3techs.com][45]

From this graph, you can see that in a short few years, HTTP/3 usage is rapidly approaching the same usage as HTTP/2. We had “peak HTTP/2” in 2021 and maybe next year we will see HTTP/3 overtake HTTP/2 to be the de-facto standard for new web deployments.

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/reaper.png)

I will be honest though, other sources, such as [Cloudflare][46] and [Internet Society Pulse][47], present a slightly different picture, with HTTP/2 still handling a significant portion of traffic.

### Why don’t web developers know about this?

Well, simply put, none of this really intersects with them in any meaningful way. You can’t even tell what version of HTTP your browser is making from Javascript. No, [not even with the Fetch API][48]. I recently looked into this because I wanted to print out the version of HTTP being used when loading my website in [a tool that I made][49] but I had to resort to [adding headers to the response via Cloudflare][50] to get that information. Because of all of this, I’m certain that there’s a large number of web developers who have no idea that their website is using HTTP/3.

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/the-same.png)

If you want to know which version of HTTP your browser is using when loading your website, most browser inspector interfaces can show this by going to the network tab, right-clicking the headers and adding “Protocol” to the list of columns to show. It also usually shows the protocol when you look at the details of each request.

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/inspector.png)

## Challenges Ahead

There are two main areas to focus on with QUIC: adding more tooling and language support for the protocol.

### Tooling/Language Support

Even though browsers and load balancers have good support for QUIC, most programming languages don’t support HTTP/3 because QUIC presents a vastly different way to communicate. Without kernel support or widely supported bindings, adding QUIC to a language is a bit like re-implementing TCP in the language. TCP is usually relatively easy to add because OS kernels typically implement TCP for you and provide bindings. As far as I know, that isn’t the case for QUIC.

### Will QUIC stay in userspace?

From what I’ve seen, there’s no well-supported kernel module for QUIC. There exist some projects that do [add a kernel module for Linux][51] but it doesn’t seem like it’s heavily used yet. TCP has benefitted a lot from being backed by OS kernels, which gives it performance optimizations not available in user space but that strength is also a weakness. TCP is now hard to change and adapt. It appears like QUIC has evolved much quicker because of its user-space implementations, but should it stay there? [Some have reported][52] that it’s possible to match the performance of TCP, even when running in userspace. So does QUIC need the performance improvements that are possible in the kernel? Should there be [a hybrid approach][53] that could give a balance of flexibility and performance advantages? To me, those are open questions and I’m curious how this will unfold over time.

### Naysayers

Since QUIC does present a lot of changes, there are people, [mostly network engineers][54], who do not like the quick adoption of QUIC. Here are some of the complaints:

-   Most web traffic is a bloated mess anyway, and the additional few milliseconds it takes to stand up a TCP connection doesn’t account for a large amount of bloat in modern web applications.
-   QUIC makes it more difficult for network administrators to monitor and inspect traffic.
-   [QUIC induces more CPU overhead][55], as it doesn’t have decades of optimization that TCP has.
-   QUIC is a relatively new protocol, and there may still be undiscovered bugs or vulnerabilities, especially with so many languages/platforms re-implementing the protocol.

Further, many argue that the benefits just don’t outweigh the risks. I don’t think I agree. I think these network engineers are discounting just how much traffic comes from mobile or dynamic (sketchy) wifi environments. However, I am surprised at how quickly and the method by which QUIC has been rolled out. Should we pump the brakes or is it too late?

![](https://kmcd.dev/posts/yall-are-sleeping-on-http3/angry-neteng.png)

## The Future is QUIC

HTTP/3 and QUIC are more than just incremental improvements; they represent a fundamental shift in how we build the web. By overcoming the limitations of TCP, QUIC offers a faster, more reliable, and more secure foundation for modern Internet communication.

The benefits are clear: faster page loads, smoother video streaming, more responsive applications, and seamless transitions between networks. This translates to a better user experience, increased engagement, and ultimately, a more vibrant and dynamic internet.

But QUIC’s potential extends far beyond the web. Its ability to handle real-time communication, coupled with its built-in security features, makes it an attractive option for applications like [online gaming][56], [video conferencing][57], and more. There are already some big projects built on QUIC:

-   [SSH3: SSH using QUIC][58]
-   [Hysteria: Censorship-resistant proxy][59]
-   [Cloudflare Tunnels][60]
-   [Media over QUIC][61]
-   [WebTransport (based on HTTP/3) was decided on in gRPC-Web][62] to support bidirectional streams

While the transition to HTTP/3 is well underway, it’s far from complete. There are still challenges to overcome, such as improving tooling, expanding language support, and ensuring compatibility with existing infrastructure. And it remains to be seen if HTTP/3 is ready for mass adoption.

However, the momentum is undeniable. With major browsers, cloud providers, and load balancers embracing QUIC, the future is bright. As developers and engineers continue to innovate and build on this foundation, we can expect to see even more exciting possibilities emerge.

So, what are you waiting for? If you’re a web developer or tech enthusiast, now is the time to dive into QUIC and start exploring its potential. Experiment with new applications, build innovative tools and share your findings with the community. I’ve started doing this by [trying HTTP/3 with gRPC][63]. The future of the internet is being written in QUIC, and you can be a part of it.

If you want to learn more, here are a lot of resources that I used to research for this article:

-   [HTTP/3 Spec (RFC 9114)][64]
-   [QUIC Spec (RFC 9000)][65]
-   [QUIC matches TCP’s efficiency, says our research. | Fastly][66]
-   [QUIC in Linux Kernel][67]
-   [Chromium’s document on QUIC][68]
-   [HTTP/3 Protocol — Performance and Security Implications][69]
-   [HTTP/3 explained][70]
-   [Can I Use HTTP/3][71]
-   [HTTP/3 — Challenges to security and possible response][72]
-   [The Challenges Ahead for HTTP/3][73]
-   [HTTP/3: Shiny New Thing, or More Issues?][74]
-   [W3 Techs - Historical yearly trends in the usage statistics of site elements for websites][75]
-   [Why HTTP/3 is eating the world][76]
-   [How QUIC helps you seamlessly connect to different networks][77]
-   [net: In-kernel QUIC implementation with Userspace handshake][78]

### See Also

-   [gRPC Over HTTP/3][79]
-   [HTTP/1.0 From Scratch][80]
-   [HTTP/0.9 From Scratch][81]
-   [What version of HTTP are you using?][82]
-   [Go to a random page][83][][84]

[← HTTP/1.0 From Scratch][85] [HTTP/0.9 From Scratch →][86]

[1]: https://kmcd.dev/posts/yall-are-sleeping-on-http3/
[2]: /me/
[3]: https://github.com/sudorandom/kmcd.dev/commits/main/content/posts/2024/yall-are-sleeping-on-http3/index.md
[4]: https://kmcd.dev/posts/yall-are-sleeping-on-http3/
[5]: https://kmcd.dev/categories/article/
[6]: https://kmcd.dev/tags/http/
[7]: https://kmcd.dev/tags/http2/
[8]: https://kmcd.dev/tags/http3/
[9]: https://kmcd.dev/tags/web/
[10]: https://kmcd.dev/tags/webdev/
[11]: https://www.cloudflare.com/learning/performance/what-is-http3/
[12]: https://http.dev/3
[13]: https://datatracker.ietf.org/doc/html/rfc9114
[14]: https://nordvpn.com/blog/what-is-quic-protocol/
[15]: https://blog.cloudflare.com/the-road-to-quic
[16]: https://datatracker.ietf.org/doc/rfc9000/
[17]: https://www.debugbear.com/blog/http3-quic-protocol-guide
[18]: https://caniuse.com/http3
[19]: https://kmcd.dev/posts/yall-are-sleeping-on-http3/#cloud-providers
[20]: https://kmcd.dev/posts/yall-are-sleeping-on-http3/#load-balancers
[21]: https://w3techs.com/technologies/details/ce-http3
[22]: https://http3-explained.haxx.se/en/why-quic/why-tcphol
[23]: https://www.multipath-tcp.org/
[24]: https://blog.apnic.net/2023/10/02/how-quic-helps-you-seamlessly-connect-to-different-networks/
[25]: https://http.dev/3#the-quic-protocol-and-limitations-with-tcp
[26]: https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp
[27]: https://caniuse.com/http3
[28]: https://www.chromium.org/quic/
[29]: https://hacks.mozilla.org/2021/04/quic-and-http-3-support-now-in-firefox-nightly-and-beta/
[30]: https://techcommunity.microsoft.com/t5/discussions/how-to-enable-http3/m-p/2265230
[31]: https://developers.cloudflare.com/speed/optimization/protocol/http3/
[32]: https://cloud.google.com/blog/products/networking/cloud-cdn-and-load-balancing-support-http3
[33]: https://aws.amazon.com/about-aws/whats-new/2022/08/amazon-cloudfront-supports-http-3-quic/
[34]: https://techdocs.akamai.com/property-mgr/docs/http3-support
[35]: https://techcommunity.microsoft.com/t5/azure-networking-blog/quic-based-http-3-with-application-gateway-feature-information/ba-p/3913972
[36]: https://www.cdn77.com/blog/gquic-support-road-to-http3
[37]: https://docs.fastly.com/en/guides/enabling-http3-for-fastly-services
[38]: https://nginx.org/en/docs/quic.html
[39]: https://gateway.envoyproxy.io/latest/tasks/traffic/http3/
[40]: https://caddyserver.com/docs/caddyfile/options#section-global-options
[41]: https://docs.litespeedtech.com/lsws/cp/cpanel/quic-http3/
[42]: https://h2o.examp1e.net/configure/http3_directives.html
[43]: https://doc.traefik.io/traefik/routing/entrypoints/#http3
[44]: https://www.haproxy.com/blog/how-to-enable-quic-load-balancing-on-haproxy
[45]: https://w3techs.com/technologies/history_overview/site_element/all/y
[46]: https://blog.cloudflare.com/http3-usage-one-year-on
[47]: https://pulse.internetsociety.org/technologies
[48]: https://developer.mozilla.org/en-US/docs/Web/API/Response
[49]: https://kmcd.dev/http/
[50]: https://developers.cloudflare.com/ruleset-engine/rules-language/fields/
[51]: https://github.com/lxin/quic
[52]: https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency/
[53]: https://lwn.net/Articles/965134/
[54]: https://www.reddit.com/r/networking/comments/148qz1f/why_is_there_a_general_hostility_to_quic_by/
[55]: https://news.ycombinator.com/item?id=39169357
[56]: https://daposto.medium.com/quic-for-gamenetworking-46cf23936228
[57]: https://quic.video/blog/replacing-webrtc/
[58]: https://github.com/francoismichel/ssh3
[59]: https://github.com/apernet/hysteria
[60]: https://blog.cloudflare.com/getting-cloudflare-tunnels-to-connect-to-the-cloudflare-network-with-quic
[61]: https://quic.video/
[62]: https://github.com/grpc/grpc-web/blob/master/doc/roadmap.md
[63]: https://kmcd.dev/posts/grpc-over-http3/
[64]: https://datatracker.ietf.org/doc/html/rfc9114
[65]: https://datatracker.ietf.org/doc/rfc9000/
[66]: https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency/
[67]: https://github.com/lxin/quic
[68]: https://www.chromium.org/quic/
[69]: https://codilime.com/blog/http3-protocol/
[70]: https://http3-explained.haxx.se/en
[71]: https://caniuse.com/http3
[72]: https://sanjeev41924.medium.com/http-3-challenges-to-security-and-possible-response-e81f429506e0
[73]: https://pulse.internetsociety.org/blog/the-challenges-ahead-for-http-3
[74]: https://medium.com/tech-internals-conf/http-3-shiny-new-thing-or-more-issues-6e4fe14e52ea
[75]: https://w3techs.com/technologies/history_overview/site_element/all/y
[76]: https://blog.apnic.net/2023/09/25/why-http-3-is-eating-the-world/
[77]: https://blog.apnic.net/2023/10/02/how-quic-helps-you-seamlessly-connect-to-different-networks/
[78]: https://lwn.net/Articles/965134/
[79]: /posts/grpc-over-http3/
[80]: /posts/http1.0-from-scratch/
[81]: /posts/http0.9-from-scratch/
[82]: /posts/http-tool/
[83]: /random/
[84]: /random/
[85]: https://kmcd.dev/posts/http1.0-from-scratch/
[86]: https://kmcd.dev/posts/http0.9-from-scratch/