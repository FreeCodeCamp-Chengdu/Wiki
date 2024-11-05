---
title: Thoughts while building
date: 2001-10-26T00:00:00.000Z
authorURL: ""
originalURL: https://blog.hyperknot.com/p/understanding-round-robin-dns
translator: ""
reviewer: ""
---

Share this post

<!-- more -->

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png)

#### Understanding Round Robin DNS

blog.hyperknot.com

Copy link

Facebook

Email

Note

Other

# Understanding Round Robin DNS

### In which I try to understand how browsers and Cloudflare choose which server to use

[![](https://substackcdn.com/image/fetch/w_80,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb4493a04-aa03-4863-92e4-6875f8db25fe_512x512.png)][1]

[Zsolt Ero][2]

Oct 26, 2024

16

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png)

#### Understanding Round Robin DNS

blog.hyperknot.com

Copy link

Facebook

Email

Note

Other

[

9

][3]

3

For [OpenFreeMap][4], I'm using servers behind Round Robin DNS. In this article, I'm trying to understand how browsers and CDNs select which one to use.

## What is Round Robin DNS?

Normally, when you are serving a website from a VPS like Digital Ocean or Hetzner, you add a single A record in your DNS provider's control panel.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F15591e83-689a-4821-8309-919e0528a434_768x140.png)



][5]

This means that rr-direct.hyperknot.com will serve data from 5.223.46.55.

In Round Robin DNS, you specify multiple servers for the same subdomain, like this.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0c97e110-0b2c-429b-b764-acb2331afa7e_792x268.png)



][6]

This allows you to share the load between multiple servers, as well as to automatically detect which servers are offline and choose the online ones.

It's an amazingly simple and elegant solution that avoids using Load Balancers. It's also free, and you can do it on any DNS provider, whereas Load Balancing solutions can get really expensive (even on Cloudflare, which is otherwise very reasonably priced).

## How does it work in theory?

I became obsessed with how it works. I mean, on the surface, everything is elegant, but how does a browser decide which server to connect to?

In theory, there is an [RFC 8305][7] called Happy Eyeballs (also linking to [RFC 6724][8]) about how the client should sort addresses before connecting.

Now, this is definitely above my experience level, but [this section][9] seems like the closest to answering my question:

> If the client is stateful and has a history of expected round-trip times (RTTs) for the routes to access each address, it SHOULD add a Destination Address Selection rule between rules 8 and 9 that prefers addresses with lower RTTs.

So in my understanding, it's basically:

-   Checking if servers are online/offline
    
-   Sort the online ones according to ping times
    

## In Practice

Let's see how it works in practice.

I created 3 VPSs around the world: one in the US, one in the EU, and one in Singapore. I made 3 proxied and 3 non-proxied A records in Cloudflare.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png)



][10]

They run nginx with a config like this:

```
server {
    server_name rr-direct.hyperknot.com rr-cf.hyperknot.com;

    # this is basically a wildcard block
    # so /a/b/c will return the same color.png file
    location / {
        root /data;
        rewrite ^ /color.png break;
    }

    location /server {
        alias /etc/hostname;
        default_type text/plain;
    }
}
```

So they serve a color.png, which is a 1px red/green/blue PNG file, as well as their hostname, which is test-eu/us/sg.

### Client Behavior When Servers Are Online

[I made a HTML test page][11], which fills a 10x10 grid with random images.

The server colors are the following:

-   US - green
    
-   EU - blue
    
-   SG - red
    

Important: I'm testing from Europe; the EU server is much closer to me than US or especially the SG one. I should be seeing blue boxes!

#### Chrome

Chrome selects somewhat randomly between all locations, but once selected, it sticks with it. It re-evaluates the selection after a few hours. In this case, it was stuck with SG for hours, even though it is by far the slowest server for me.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F82985913-6f42-4fd2-b009-61fedac5294f_958x570.png)



][12]

Also, an interesting behavior on Chrome was when not using HTTP/2: it can sometimes choose randomly between two servers, creating a pattern like this. Here it's choosing between EU and US randomly.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F36c86eaa-335c-4ceb-af52-e4db28fefcf4_958x570.png)



][13]

#### Firefox

Behaves similarly to Chrome, selects a location randomly on startup and then sticks with it. If I restart the browser, it picks a different random location.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67cfbcbe-87f8-4d83-a46e-a970b4f34fd7_958x570.png)



][14]

#### Safari

To my biggest surprise, Safari always selects the closest server correctly. Even if it goes offline for a while, after a few refreshes, it always finds the EU server again!

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa86d8b48-6660-4c76-b063-b10c73ec6fee_958x570.png)



][15]

#### curl  

Curl also works correctly. First time it might not, but if you run the command twice, it always corrects to the nearest server.

If you have multiple VPSs around the world, try this command via ssh and see which one gets selected:

```
curl https://rr-direct.hyperknot.com/server
test-us

curl https://rr-direct.hyperknot.com/server
test-eu
```

#### Cloudflare

Cloudflare picks a random location based on your client IP and then sticks with it. (It behaves like client\_ip\_hash modulo server\_num or similar.)

As you have seen in the images, the right-side rectangle is always green. On my home IP, no matter what I do, Cloudflare goes to the US server. Curl shows the same.

```
curl https://rr-cf.hyperknot.com/server
test-us
```

Now, if I go to my mobile hotspot, it always connects to the EU server.

If I log into some VPSes and run the same curl command, I can see this behavior across the world. Every VPS gets connected to a totally random location around the world, but always the same.

```
curl https://rr-cf.hyperknot.com/server
test-sg
```

### Client Behavior with Partially Offline Servers

So what happens when one of the servers is offline? Say I stop the US server:

```
service nginx stop
```

#### Chrome

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4261f746-b85e-4407-bf6e-d01c5962e7bb_958x570.png)



][16]

#### Firefox

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb15577b7-7255-4c5f-bb86-0a66ed463bce_958x570.png)



][17]

#### Safari

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F03015118-9864-4c4b-ab35-9e04b42b08f3_958x570.png)



][18]

#### curl

```
curl https://rr-direct.hyperknot.com/server
test-eu
```

As you can see, all clients correctly detect it and choose an alternative server.

Actually, they do this fallback so well that if I turn off the server while they are loading, they correct within < 1 sec! Here is an animation of the 50x50 version of the same grid, on Safari:

#### Cloudflare

And what about Cloudflare? As you can see in the screenshots above, Cloudflare does **not** detect an offline server. It keeps accessing the server it decided for your IP, regardless of whether it's online or offline.

If the server is offline, you are served offline. In curl, it returns:

```
curl https://rr-cf.hyperknot.com/server
error code: 521
```

I've been trying to understand what this behavior is, and I highly suspect it's a **bug** in their network. One reference I found in their [documentation][19] is this part:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F86323993-bbaa-4d0b-b21c-72c2b44a2fcc_1388x622.png)



][20]

Based on this documentation - and by common sense as well - I believe Cloudflare should also behave like browsers and curl do.

### Cloudflare Wish List

1.  At the very least, offline servers should be detected.
    
2.  Moreover, it would also be really nice if the server with the lowest latency were selected, like in Safari!
    

I mean, currently, if you have one server in the US and one in New Zealand, exactly 50% of your US users will be served from New Zealand, which makes no sense. Also, for Safari users, it's actually slower compared to not using Cloudflare!

---

There is a [HN discussion now][21], where both the CEO and the CTO of Cloudflare replied!

Note 1: I've tried my best to understand articles **[1][22]**, **[2][23]**, **[3][24]** which Matthew Prince **[pointed out to me][25]** on X. As I understand, they are referring to Cloudflare servers as "servers", not users' servers behind A records. Also, I couldn't find anything related to Round Robin DNS.

Note 2: If you have any idea **how I can keep running** this experiment without paying for 3 VPS-es around the world, please comment below. I'd love to keep it running. Is there a serverless platform that supports HTTPS and Round Robin DNS?

If you enjoyed this post, feel free to subscribe to this blog

16

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png)

#### Understanding Round Robin DNS

blog.hyperknot.com

Copy link

Facebook

Email

Note

Other

[

9

][26]

3

#### Discussion about this post

Comments

Restacks

<table class="comment-content"><tbody><tr><td class="comment-head"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><div class="user-head"><a href="https://substack.com/profile/76382232-josh-enders"><div class="profile-img-wrap"><picture><source type="image/webp" srcset="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9e5ff4b0-9188-4328-8324-a2a7cbf204b5_144x144.png"><img src="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9e5ff4b0-9188-4328-8324-a2a7cbf204b5_144x144.png" sizes="100vw" alt="" width="66" height="66" class="_img_16u6n_1 pencraft pc-reset"></picture></div></a></div></div></td><td class="comment-rest"><div class="comment-meta"><span class="commenter-name"><div class="pencraft pc-display-flex pc-gap-4 pc-alignItems-center pc-reset pc-display-inline-flex"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><a href="https://substack.com/profile/76382232-josh-enders"><div class="pencraft pc-reset _color-pub-primary-text_q8zsn_187 _line-height-20_q8zsn_80 _font-text_q8zsn_106 _size-13_q8zsn_39 _weight-semibold_q8zsn_148 _reset_q8zsn_1">Josh Enders</div></a></div></div></span><span class="comment-publication-name-separator">·</span><span class="edited-indicator"><em>edited Oct 26</em></span></div><div class="comment-body"><p><span>Load balancing via DNS is entirely dependent on the behavior of caching DNS resolvers. Clients are beholden to how answers are sorted and it’s rarely fair. Even with a zero second TTL, the TTL of answers is often ignored. The situation is even worse with a TTL, as the answers are rarely re-resolved after the expiration. The JVM, for example, is notorious for defaulting to ignoring TTL entirely ruining round-robin load abounding via DNS. That’s not to say that it can’t be defective but its limitations should be well understood.</span></p><div role="button" class="show-all-toggle"><div class="show-all-toggle-label">Expand full comment</div></div></div><div class="pencraft pc-display-flex pc-gap-16 pc-paddingTop-8 pc-justifyContent-flex-start pc-alignItems-center pc-reset comment-actions _withShareButton_mhaex_5"><span class="pencraft pc-reset _decoration-hover-underline_q8zsn_290 _reset_q8zsn_1"><a class="pencraft pc-reset _link_mhaex_1 _link_uqs75_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle"><path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"></path></svg><div class="pencraft pc-reset _color-pub-secondary-text_q8zsn_190 _line-height-20_q8zsn_80 _font-meta_q8zsn_115 _size-11_q8zsn_31 _weight-medium_q8zsn_145 _transform-uppercase_q8zsn_234 _reset_q8zsn_1 _meta_q8zsn_434">Reply</div></div></a></span><span class="pencraft pc-reset _decoration-hover-underline_q8zsn_290 _reset_q8zsn_1"><a class="pencraft pc-reset _link_mhaex_1 _link_uqs75_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-share"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path><polyline points="16 6 12 2 8 6"></polyline><line x1="12" x2="12" y1="2" y2="15"></line></svg><div class="pencraft pc-reset _color-pub-secondary-text_q8zsn_190 _line-height-20_q8zsn_80 _font-meta_q8zsn_115 _size-11_q8zsn_31 _weight-medium_q8zsn_145 _transform-uppercase_q8zsn_234 _reset_q8zsn_1 _meta_q8zsn_434">Share</div></div></a></span><button tabindex="0" type="button" id="trigger38607" aria-expanded="false" aria-haspopup="dialog" aria-controls="dialog38608" arialabel="View more" class="pencraft pc-reset _iconButton_1r1ly_154 _iconButtonBase_1r1ly_154 _buttonBase_1r1ly_1 _buttonOldColors_1r1ly_65 _priority_secondary-theme_1r1ly_259 _fill_empty_1r1ly_320"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-ellipsis"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg></button></div></td></tr></tbody></table>

<table class="comment-content"><tbody><tr><td class="comment-head"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><div class="user-head"><a href="https://substack.com/profile/58235475-shu"><div class="profile-img-wrap"><picture><source type="image/webp" srcset="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe09195c2-42d2-411d-976e-6d231f2d04f7_144x144.png"><img src="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe09195c2-42d2-411d-976e-6d231f2d04f7_144x144.png" sizes="100vw" alt="" width="66" height="66" class="_img_16u6n_1 pencraft pc-reset"></picture></div></a></div></div></td><td class="comment-rest"><div class="comment-meta"><span class="commenter-name"><div class="pencraft pc-display-flex pc-gap-4 pc-alignItems-center pc-reset pc-display-inline-flex"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><a href="https://substack.com/profile/58235475-shu"><div class="pencraft pc-reset _color-pub-primary-text_q8zsn_187 _line-height-20_q8zsn_80 _font-text_q8zsn_106 _size-13_q8zsn_39 _weight-semibold_q8zsn_148 _reset_q8zsn_1">Shu</div></a></div></div></span><a href="https://blog.hyperknot.com/p/understanding-round-robin-dns/comment/74511356" rel="nofollow" native="" class="comment-timestamp">Oct 28</a></div><div class="comment-body"><p><span>Hey, thanks for such an insightful post. You can get free arm based servers from Oracle. It all depends on availability though. Attaching link below:</span></p><p><span><a href="https://www.oracle.com/cloud/free/" target="_blank" rel="nofollow ugc noopener" class="linkified">https://www.oracle.com/cloud/free/</a></span></p><div role="button" class="show-all-toggle"><div class="show-all-toggle-label">Expand full comment</div></div></div><div class="pencraft pc-display-flex pc-gap-16 pc-paddingTop-8 pc-justifyContent-flex-start pc-alignItems-center pc-reset comment-actions _withShareButton_mhaex_5"><span class="pencraft pc-reset _decoration-hover-underline_q8zsn_290 _reset_q8zsn_1"><a class="pencraft pc-reset _link_mhaex_1 _link_uqs75_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle"><path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"></path></svg><div class="pencraft pc-reset _color-pub-secondary-text_q8zsn_190 _line-height-20_q8zsn_80 _font-meta_q8zsn_115 _size-11_q8zsn_31 _weight-medium_q8zsn_145 _transform-uppercase_q8zsn_234 _reset_q8zsn_1 _meta_q8zsn_434">Reply</div></div></a></span><span class="pencraft pc-reset _decoration-hover-underline_q8zsn_290 _reset_q8zsn_1"><a class="pencraft pc-reset _link_mhaex_1 _link_uqs75_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-share"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path><polyline points="16 6 12 2 8 6"></polyline><line x1="12" x2="12" y1="2" y2="15"></line></svg><div class="pencraft pc-reset _color-pub-secondary-text_q8zsn_190 _line-height-20_q8zsn_80 _font-meta_q8zsn_115 _size-11_q8zsn_31 _weight-medium_q8zsn_145 _transform-uppercase_q8zsn_234 _reset_q8zsn_1 _meta_q8zsn_434">Share</div></div></a></span><button tabindex="0" type="button" id="trigger38609" aria-expanded="false" aria-haspopup="dialog" aria-controls="dialog38610" arialabel="View more" class="pencraft pc-reset _iconButton_1r1ly_154 _iconButtonBase_1r1ly_154 _buttonBase_1r1ly_1 _buttonOldColors_1r1ly_65 _priority_secondary-theme_1r1ly_259 _fill_empty_1r1ly_320"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-ellipsis"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg></button></div></td></tr></tbody></table>

[1 reply by Zsolt Ero][33]

[7 more comments...][34]

Top

Latest

Discussions

No posts

Ready for more?

[1]: https://substack.com/profile/121308039-zsolt-ero
[2]: https://substack.com/@hyperknot
[3]: https://blog.hyperknot.com/p/understanding-round-robin-dns/comments
[4]: https://openfreemap.org/
[5]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F15591e83-689a-4821-8309-919e0528a434_768x140.png
[6]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0c97e110-0b2c-429b-b764-acb2331afa7e_792x268.png
[7]: https://datatracker.ietf.org/doc/html/rfc8305
[8]: https://datatracker.ietf.org/doc/html/rfc6724#section-6
[9]: https://datatracker.ietf.org/doc/html/rfc8305#section-4
[10]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png
[11]: https://assets.openfreemap.com/share/2024-10/rr.html
[12]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F82985913-6f42-4fd2-b009-61fedac5294f_958x570.png
[13]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F36c86eaa-335c-4ceb-af52-e4db28fefcf4_958x570.png
[14]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67cfbcbe-87f8-4d83-a46e-a970b4f34fd7_958x570.png
[15]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa86d8b48-6660-4c76-b063-b10c73ec6fee_958x570.png
[16]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4261f746-b85e-4407-bf6e-d01c5962e7bb_958x570.png
[17]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb15577b7-7255-4c5f-bb86-0a66ed463bce_958x570.png
[18]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F03015118-9864-4c4b-ab35-9e04b42b08f3_958x570.png
[19]: https://developers.cloudflare.com/fundamentals/basic-tasks/protect-your-origin-server/#zero-downtime-failover
[20]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F86323993-bbaa-4d0b-b21c-72c2b44a2fcc_1388x622.png
[21]: https://news.ycombinator.com/item?id=41955912
[22]: https://t.co/MefximeFqU
[23]: https://t.co/IlYL4Emgz7
[24]: https://t.co/GKE4mdUiNH
[25]: https://x.com/eastdakota/status/1850103009826554285
[26]: https://blog.hyperknot.com/p/understanding-round-robin-dns/comments
[27]: https://substack.com/profile/76382232-josh-enders
[28]: https://substack.com/profile/76382232-josh-enders
[29]: https://substack.com/profile/58235475-shu
[30]: https://substack.com/profile/58235475-shu
[31]: https://blog.hyperknot.com/p/understanding-round-robin-dns/comment/74511356
[32]: https://www.oracle.com/cloud/free/
[33]: https://blog.hyperknot.com/p/understanding-round-robin-dns/comment/74511356
[34]: https://blog.hyperknot.com/p/understanding-round-robin-dns/comments