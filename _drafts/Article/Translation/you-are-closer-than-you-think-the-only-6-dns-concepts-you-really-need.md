---
title: "你比想象中更接近：你真正需要的 6 个 DNS 概念"
date: 2024-07-04T00:00:00.000Z
author: Jonah
authorURL: https://jonahdevs.com/author/admin/
originalURL: https://jonahdevs.com/youre-closer-than-you-think-the-only-6-dns-concepts-you-really-need/
translator: ""
reviewer: ""
---

觉得 DNS 是个你永远无法完全理解的庞然大物？有个好消息：你可能比自己意识到的更接近掌握它了。事实上，你真正需要掌握的核心概念只有六个。剩下的呢？你可以把它们从你的大脑内存中卸载掉。让我们来看看这些基本要素，并向你展示你已经知道了多少——以及你需要学习多少才能自信地处理 80% 的 DNS 需求。

你可能想快速滚动浏览下一张表格……它有点吓人。

## DNS 功能和描述的速查表

|     | DNS 功能             | 描述                                                 |
| --- | -------------------- | ---------------------------------------------------- |
| 1   | A Records            | 将域名映射到 IPv4 地址                               |
| 2   | AAAA Records         | 将域名映射到 IPv6 地址                               |
| 3   | CNAME Records        | 为域名创建别名                                       |
| 4   | MX Records           | 指定用于处理电子邮件的邮件服务器                     |
| 5   | TXT Records          | 存储任意文本字符串，通常用于域名验证                 |
| 6   | NS Records           | 将域名或子域名委托给一组名称服务器                   |
| 7   | SOA Records          | 指定有关 DNS 区域的权威信息                          |
| 8   | PTR Records          | 反向 DNS 查询，将 IP 地址映射到域名                  |
| 9   | SRV Records          | 指定服务的位置（例如，VoIP、IM）                     |
| 10  | CAA Records          | 指定哪些证书颁发机构可以颁发 SSL/TLS 证书            |
| 11  | DKIM Records         | 电子邮件身份验证方法，用于检测伪造的发件人地址       |
| 12  | SPF Records          | 电子邮件身份验证方法，用于防止电子邮件欺骗           |
| 13  | DMARC Records        | 电子邮件身份验证、策略和报告协议                     |
| 14  | NAPTR Records        | 用于 ENUM 和 SIP 服务                                |
| 15  | SSHFP Records        | 存储 SSH 公共主机密钥指纹                            |
| 16  | DS Records           | 用于 DNSSEC 中以确保子域名的安全委托                 |
| 17  | TLSA Records         | 将 TLS/SSL 证书与 DANE 协议的域名相关联              |
| 18  | LOC Records          | 指定域名的地理位置                                   |
| 19  | HINFO Records        | 指定主机硬件和操作系统                               |
| 20  | RP Records           | 指定域名的责任人                                     |
| 21  | AFSDB Records        | 指定 AFS 单元数据库服务器的位置                      |
| 22  | Zone Transfers       | 配置主 DNS 服务器和辅助 DNS 服务器之间的区域传送设置 |
| 23  | TTL                  | 设置 DNS Records 的缓存持续时间                      |
| 24  | Reverse DNS Zones    | 配置用于 IP 到域名映射的反向查询区域                 |
| 25  | Subdomains           | 创建和管理子域名                                     |
| 26  | Wildcard DNS Records | 为子域名设置通配符 Records                           |
| 27  | DNS Forwarding       | 配置 DNS 服务器以将查询转发到其他 DNS 服务器         |
| 28  | DNSSEC               | 启用和配置 DNSSEC 以保护 DNS                         |
| 29  | GeoDNS               | 设置基于位置的 DNS 响应                              |
| 30  | Round Robin DNS      | 配置多个 A 或 AAAA Records 以实现负载均衡            |
| 31  | Dynamic DNS          | 允许自动更新 DNS Records                             |
| 32  | DNS Aliases          | 为同一主机创建备用名称                               |
| 33  | DNS Views            | 根据查询来源配置不同的 DNS 响应                      |

## DNS 功能描述

DNS 可能看起来像是一个充满神秘记录和晦涩配置的迷宫。但这里有一个不为人知的小秘密：大多数开发者在复杂化事情。你不需要成为一个 DNS 大师就能完成任务。实际上，理解六个关键概念就能覆盖你大约 80%的 DNS 需求。让我们来简单分解一下：

1. **A 记录** - 将域名映射到 IPv4 地址。
2. **CNAME 记录** - 为域名创建别名。
3. **MX 记录** - 指定处理电子邮件的邮件服务器。
4. **TXT 记录** - 存储文本字符串，通常用于域名验证。
5. **NS 记录** - 将一个域名委托给一组名称服务器。
6. **TTL** - 设置 DNS 记录的缓存时长。

A 记录：您域名的主页地址
你正在启动一个新网站。你的托管提供商告诉你你的服务器 IP 地址是 203.0.113.10。你会这样设置一个 A 记录：
```text
coolsite.com. IN A 203.0.113.10
```

在这种情况下，`203.0.113.10`实际上是你的网站在互联网上的 `IP住址`。当有人在浏览器中输入 `coolsite.com` 时，这个 A 记录告诉他们去这个特定的 IP 地址找到你的网站内容。

更深入一点：A 记录也可以用于子域映射。例如，你可以为 `blog.coolsite.com` 和 `shop.coolsite.com` 设置不同的 A 记录，每个都指向不同的 IP 地址。这允许灵活的托管安排，比如在一个服务器上托管你的主站点，在另一个服务器上托管你的博客。

CNAME：您域名的别名

你希望 `www.coolsite.com` 和 `coolsite.com` 都能指向同一个地方。你会这样设置一个`CNAME`记录：`www.coolsite.com.` 在的 coolsite.com 的 CNAME。

在这里，你是在说 www.coolsite.com 只是 coolsite.com 的另一个名字。这就像拥有一个“商业名称”一样 —— 两者都指向同一个网站，给用户在输入你的 URL 时提供灵活性。

对于好奇的人来说：CNAME 记录可以被链式连接，但要小心 —— 这会增加 DNS 查找时间。另外，CNAME 的目标必须始终是一个域名，永远不是一个 IP 地址。这使得 CNAME 非常适合那些可能会更改 IP 地址的服务，因为你只需要更新目标域的 A 记录。

MX 记录：指向您的电子邮件

MX 记录全都关于电子邮件。它们告诉世界哪些服务器处理你的域名的电子邮件。没有这些，你的`contact@coolsite.com`地址就像巧克力茶壶一样没什么用。设置正确后，你的电子邮件就会像梦一样流畅。

MX 记录示例用法

你在使用 Google Workspace 处理电子邮件。你会这样设置 MX 记录：
```text
coolsite.com. IN MX 1 ASPMX.L.GOOGLE.COM.
coolsite.com. IN MX 5 ALT1.ASPMX.L.GOOGLE.COM.
```

这些记录告诉电子邮件服务器，当有人发送电子邮件到`@coolsite.com`时，应该将邮件投递到 Google 的电子邮件服务器。数字（1 和 5）表示优先级,邮件会先尝试第一个服务器，如果第一个服务器不可用，再回退到第二个服务器。

更进一步：MX 记录有一个优先级值。你可以设置多个具有不同优先级的 MX 记录，创建一个回退系统。如果主邮件服务器宕机，排在下一位的服务器将接管。这为你的电子邮件系统增加了冗余性，确保你不会错过重要信息。

TXT 记录：DNS 的瑞士军刀
TXT 记录是 DNS 的万能卡。需要验证域名所有权？TXT 记录。设置电子邮件认证？TXT 记录。它们基本上是存放关于你的域的任何基于文本的信息的地方。非常方便的小工具。

TXT 记录示例
你需要为 Google 搜索控制台验证你的域名。他们要求你添加一个 TXT 记录，如下所示：
```text
coolsite.com. IN TXT “google-site-verification=randomstringhere”
```

这个记录就像是与 Google 的秘密握手。通过向你的域名的 DNS 添加这段特定的文本，你向 Google 证明了你对该域名的控制权，从而允许你使用他们的搜索控制台工具为你的网站服务。

深入挖掘：TXT 记录对于现代电子邮件安全协议如 SPF、DKIM 和 DMARC 至关重要。这些协议有助于防止电子邮件伪造并提高邮件的可投递性。TXT 记录还可以用于关于域的人类可读的注释，尽管这在实践中不太常见。

NS 记录：互联网的黄页

NS 记录示例

NS（名称服务器）记录对于 DNS 的层次结构至关重要。它们告诉互联网哪些服务器对你的域的 DNS 信息具有权威性。
NS 记录示例用法：
你正在使用 Cloudflare 来提供 DNS 服务。你会这样设置 NS 记录：
```text
coolsite.com. IN NS dana.ns.cloudflare.com.
coolsite.com. IN NS rick.ns.cloudflare.com.
```
通过设置这些 NS 记录，你就将你的域的 DNS 管理责任委托给了 Cloudflare 的名称服务器。这就像是告诉互联网：“要获取 coolsite.com 的信息，请询问这些 Cloudflare 服务器。”

更深入地了解：NS 记录是 DNS 中更大的委托系统的一部分。当你更改 NS 记录时，你正在改变谁控制你的域的 DNS 设置。这个变化需要在你的域名注册商那里得到反映，才能完全生效，这就是为什么 NS 记录的更改可能需要一段时间才能在整个互联网中完全传播的原因。

TTL：你的 DNS 缓存的有效期限
生存时间（TTL）是 DNS 信息被缓存的时长。设置得低，更改传播得快，但会增加服务器负载。设置得高，你需要等待更长的时间来使更改生效，但对服务器来说压力较小。关键在于找到那个“刚刚好”的区域。

TTL 记录示例

你计划不久后更改你的网站 IP 地址。你会将 A 记录的 TTL 降低到 300 秒（5 分钟）：
```text
coolsite.com. IN A 203.0.113.10 300
```

这里的 300 意味着 DNS 服务器应该只记住这个信息 5 分钟，然后再检查更新。在进行重大 DNS 更改之前，降低 TTL 是常见的做法。这确保了当你进行更改时，它能快速地在互联网上传播。更改生效后，你可以将 TTL 增加回较高的值以提高效率。

给技术爱好者：TTL 是以秒为单位指定的。一个常见的策略是在进行重大 DNS 更改的前一两天降低你的 TTL。这确保了当你进行更改时，它能快速传播。更改后，你可以将 TTL 增加回较高的值以提高效率。

就是这样，朋友们。这六个简单的概念将涵盖你大部分的 DNS 需求。当然，如果你想深入了解，还有更多内容可以学习——AAAA 记录用于 IPv6，DKIM 用于电子邮件验证，DNSSEC 用于那些偏执狂——但老实说？现在你大概可以不用在脑海中存储那些东西。

专注于这六个概念，你就能轻松应对 80%的 DNS 任务，不会有任何压力。当你遇到更复杂的问题时，你总是可以查阅资料。毕竟，这不就是我们在大多数工作中所做的事情吗？

所以，别再为 DNS 感到压力，回去继续做酷炫的东西吧。这不就是我们在这里的原因吗？
