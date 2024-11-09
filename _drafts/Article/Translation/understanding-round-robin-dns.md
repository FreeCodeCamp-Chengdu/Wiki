---
title: 理解轮询 DNS
date: 2001-10-26T00:00:00.000Z
authorURL: ""
originalURL: https://blog.hyperknot.com/p/understanding-round-robin-dns
translator: ""
reviewer: ""
---

对于[OpenFreeMap][1]项目，我使用了轮询 DNS 背后的服务器。在本文中，我试图理解浏览器和 CDN 是如何选择使用哪一个服务器的。

通常情况下，当你从像 Digital Ocean 或 Hetzner 这样的 VPS 提供商托管网站时，你只需在 DNS 提供商的控制面板中添加一条 A 记录。

---

![图片1][2]

这意味着 rr-direct.hyperknot.com 将从 5.223.46.55 提供数据。

在轮询 DNS 中，你可以为同一个子域名指定多个服务器，如下所示：

![图片2][3]

这样可以在多个服务器之间分担负载，并且能够自动检测哪些服务器离线，从而选择在线的服务器。

这是一个非常简单而优雅的解决方案，避免了使用负载均衡器。它是免费的，而且可以在任何 DNS 提供商上使用，相比之下，负载均衡解决方案可能会非常昂贵（即使是在 Cloudflare 上，尽管它的其他服务定价合理）。

我开始着迷于它是如何工作的。表面上看一切都很优雅，但浏览器是如何决定连接哪个服务器的呢？

理论上，有一个叫做 Happy Eyeballs 的[RFC 8305][4]（也链接到[RFC 6724][5]）规定了客户端在连接前应如何对地址进行排序。

这确实超出了我的经验水平，但[这一部分][6]似乎最接近回答我的问题：

> 如果客户端是有状态的，并且有访问每个地址的路由的预期往返时间（RTTs）历史记录，它应该在规则 8 和 9 之间添加一个目标地址选择规则，优先选择 RTT 较低的地址。

根据我的理解，它基本上是：

-   检查服务器是否在线/离线
-   根据 ping 时间对在线服务器进行排序

让我们看看它在实践中是如何工作的。

我在世界各地创建了 3 个 VPS：一个在美国，一个在欧盟，一个在新加坡。我在 Cloudflare 中创建了 3 个代理和 3 个非代理 A 记录。


![图片3][8]

它们运行的 nginx 配置如下:

```nginx
server {
    server_name rr-direct.hyperknot.com rr-cf.hyperknot.com;

    # 这基本上是一个通配符块
    # 所以 /a/b/c 将返回相同的 color.png 文件
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

所以它们提供一个 color.png 文件(这是一个 1 像素的红/绿/蓝 PNG 文件)，以及它们的主机名(test-eu/us/sg)。

[我制作了一个 HTML 测试页面][9]，它用随机图像填充了一个 10x10 的网格。

服务器颜色如下：

-   美国 - 绿色
-   欧盟 - 蓝色
-   新加坡 - 红色

重要说明：我是从欧洲进行测试的；欧盟服务器离我比美国或尤其是新加坡的服务器要近得多。我应该看到蓝色方框！

### Chrome 的表现

Chrome 在所有位置之间有些随机选择，但一旦选定就会保持不变。它会在几个小时后重新评估选择。在这种情况下，它停留在新加坡服务器几个小时，尽管对我来说这是最慢的服务器。

![图片4][10]

另外，Chrome 在不使用 HTTP/2 时有一个有趣的行为：它有时会在两个服务器之间随机选择，创建如下模式。这里它在欧盟和美国之间随机选择。

![图片5][11]

### Firefox 的表现

行为与 Chrome 类似，在启动时随机选择一个位置然后保持不变。如果重启浏览器，它会选择一个不同的随机位置。

![图片6][12]

### Safari 的表现

令我最惊讶的是，Safari 总是正确地选择最近的服务器。即使服务器暂时离线，刷新几次后，它总能再次找到欧盟服务器！

![图片7][13]

### Curl 的表现

Curl 也能正确工作。第一次可能不会，但如果你运行命令两次，它总是会纠正到最近的服务器。

如果你在世界各地有多个 VPS，可以通过 ssh 尝试这个命令，看看选择了哪个：

```bash
curl https://rr-direct.hyperknot.com/server
test-us

curl https://rr-direct.hyperknot.com/server
test-eu
```

### Cloudflare 的表现

Cloudflare 根据你的客户端 IP 随机选择一个位置然后保持不变。(它的行为类似于 client_ip_hash 模除以 server_num 或类似方式。)

如你在图片中所见，右侧矩形始终是绿色的。在我的家庭 IP 上，无论我做什么，Cloudflare 都会去美国服务器。Curl 显示相同的结果。

```bash
curl https://rr-cf.hyperknot.com/server
test-us
```

现在，如果我切换到移动热点，它总是连接到欧盟服务器。

如果我登录到一些 VPS 并运行相同的 curl 命令，我可以在全世界看到这种行为。每个 VPS 都连接到世界上完全随机的位置，但总是相同的。

```bash
curl https://rr-cf.hyperknot.com/server
test-sg
```

### 当服务器离线时会发生什么？

假设我停止美国服务器：

```bash
service nginx stop
```

![图片8][14]

![图片9][15]

![图片10][16]

```bash
curl https://rr-direct.hyperknot.com/server
test-eu
```

如你所见，所有客户端都能正确检测到这一点并选择替代服务器。

实际上，它们做这种故障转移做得如此之好，以至于如果我在它们加载时关闭服务器，它们会在 1 秒内纠正！这是同一网格 50x50 版本在 Safari 上的动画：

那 Cloudflare 呢？如上面的截图所示，Cloudflare**不会**检测到离线服务器。它会继续访问它为你的 IP 决定的服务器，不管它是否在线。

如果服务器离线，你就会被服务离线。在 curl 中，它返回：

```bash
curl https://rr-cf.hyperknot.com/server
error code: 521
```

我一直在试图理解这种行为是什么，我强烈怀疑这是他们网络中的一个**bug**。我在他们的[文档](https://developers.cloudflare.com/fundamentals/basic-tasks/protect-your-origin-server/#zero-downtime-failover)中找到的一个参考是这部分：

![图片11][17]

基于这个文档 - 也是出于常识 - 我认为 Cloudflare 也应该像浏览器和 curl 那样表现。

1. 至少，应该能检测到离线服务器。
2. 而且，如果能像 Safari 那样选择延迟最低的服务器就更好了！

我的意思是，目前如果你在美国有一个服务器，在新西兰有一个服务器，恰好 50%的美国用户会从新西兰得到服务，这毫无意义。而且，对 Safari 用户来说，它实际上比不使用 Cloudflare 还要慢！

现在有一个[HN 讨论][18]，Cloudflare 的 CEO 和 CTO 都回复了！

注 1：我已尽最大努力理解 Matthew Prince 在 X 上[指出给我](https://x.com/eastdakota/status/1850103009826554285)的文章**[1][20]**、**[2][21]**、**[3][22]**。据我理解，他们提到的"服务器"指的是 Cloudflare 服务器，而不是 A 记录背后的用户服务器。此外，我找不到任何与轮询 DNS 相关的内容。

注 2：如果你有任何想法关于**如何继续运行**这个实验而不用支付世界各地 3 个 VPS 的费用，请在下面评论。我很想继续运行它。有支持 HTTPS 和轮询 DNS 的无服务器平台吗？

## 相关链接
- [OpenFreeMap 项目][1]
- [RFC 8305 - Happy Eyeballs Version 2][4]
- [RFC 6724 - Default Address Selection][5]

  
[1]: https://openfreemap.org/
[2]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F15591e83-689a-4821-8309-919e0528a434_768x140.png
[3]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0c97e110-0b2c-429b-b764-acb2331afa7e_792x268.png
[4]: https://datatracker.ietf.org/doc/html/rfc8305
[5]: https://datatracker.ietf.org/doc/html/rfc6724#section-6
[6]: https://news.ycombinator.com/item?id=41955912
[7]: https://x.com/eastdakota/status/1850103009826554285
[8]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13e623d8-a8c8-44e9-ad80-0e61b53323f6_1186x516.png
[9]: https://assets.openfreemap.com/share/2024-10/rr.html
[10]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F82985913-6f42-4fd2-b009-61fedac5294f_958x570.png
[11]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F86323993-bbaa-4d0b-b21c-72c2b44a2fcc_1388x622.png
[12]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F36c86eaa-335c-4ceb-af52-e4db28fefcf4_958x570.png
[13]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa86d8b48-6660-4c76-b063-b10c73ec6fee_958x570.png
[14]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4261f746-b85e-4407-bf6e-d01c5962e7bb_958x570.png
[15]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb15577b7-7255-4c5f-bb86-0a66ed463bce_958x570.png
[16]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F03015118-9864-4c4b-ab35-9e04b42b08f3_958x570.png
[17]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F86323993-bbaa-4d0b-b21c-72c2b44a2fcc_1388x622.png
[18]: https://news.ycombinator.com/item?id=41955912
[19]: https://x.com/eastdakota/status/1850103009826554285
[20]: https://t.co/MefximeFqU
[21]: https://t.co/IlYL4Emgz7
[22]: https://t.co/GKE4mdUiNH
