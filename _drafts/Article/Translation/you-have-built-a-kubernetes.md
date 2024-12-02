---
title: Dear friend, you have built a Kubernetes
date: 2024-11-23T00:00:00.000Z
authorURL: ""
originalURL: https://www.macchaffee.com/blog/2024/you-have-built-a-kubernetes/
translator: ""
reviewer: ""
---

[Mac's Tech Blog][1]

<!-- more -->

[![RSS Feed](https://www.macchaffee.com/blog/icons/rss.svg)][2]

# Dear friend, you have built a Kubernetes

_This post will make more sense if you first read [Dear Sir, You Have Built a Compiler][3]._

Dear friend,

I am afraid to inform you that you have built a Kubernetes. I know you wanted to "choose boring tech" to just run some containers. You said that "Kubernetes is overkill" and "it's just way too complex for a simple task" and yet, six months later, you have pile of shell scripts that do not work—breaking every time there's a slight shift in the winds of production.

Surely, switching to Docker Compose will be the end of your woes; at least that way, someone else maintains a standard config file format for you to use. But wait, Compose is still [not a holistic solution][4]. "Do I really need a separate solution for deployment, rolling updates, rollbacks, and scaling?" you ask yourself? Surely not. Your app is so simple; a backend, a reverse proxy, postgres, and a job runner. So you march on, and add another few sections to your `deploy.sh` script, certain, that this will be the last of what you need to do to maintain this pile of hacks.

Ah, but wait! Inevitably, you find a reason to expand to a second server. While it's true [a single server can go a long way][5] many things can force this decision, such as the need for special hardware, high availability, or the speed of light. Tired, you parameterize your deploy script and configure firewall rules, distracted from the crucial features you should be working on and shipping. One of your team members suggests connecting the servers with Tailscale: an overlay network with service discovery. After that you will know, yes know, that there is no tougher complexity hurdle to clear than the networking.

Except if you quit or go on vacation, who will maintain this custom pile of shell scripts? The untested tarball handling? The inscrutable iptables rules? Who will know about those undocumented sysctl edits you made on the VM? So you add everything to Ansible, so that you may treat your VM as immutable and version-controlled. Certain, of course, that because you’re not using Kubernetes, it is going to be way easier to maintain than a Kubernetes cluster. What glorious engineering, you say to yourself.

In the last leg of your journey to avoid building a Kubernetes, your manager tells you that your app needs to programmatically spawn other containers. Spawning containers, of course, requires you to mount the Docker socket in your web app, which is wildly insecure. Not my problem, your manager says. So you write a separate service that exposes a safe subset of the Docker API to your web app. Done at last, you say to yourself, without having to build a Kubernetes.

A standard config format, a deployment method, an overlay network, service discovery, immutable nodes, and an API server. Dear friend, you have built a Kubernetes.

_Addressed to,_

_Those who wanted to avoid Kubernetes._

---

_PS: I don't mean to imply that you can never roll your own deployment method that fits your needs better than Kubernetes, or that nothing will ever be better than Kubernetes. I just want to caution you, my friend, to make sure you understand the problems Kubernetes solves before dismissing it as overly-complex._

[Home][6]

This work is licensed under [CC BY-SA 4.0][7]

[1]: https://www.macchaffee.com/blog
[2]: https://www.macchaffee.com/blog/atom.xml
[3]: https://rachit.pl/post/you-have-built-a-compiler/
[4]: https://www.macchaffee.com/blog/2024/docker-compose/
[5]: https://news.ycombinator.com/item?id=41340751
[6]: /
[7]: https://creativecommons.org/licenses/by-sa/4.0/