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

1 A Records Map domain names to IPv4 addresses
2 CNAME Records Create aliases for domain names
3 MX Records Specify mail servers for handling email
4 TXT Records Store text strings, often used for domain verification
5 NS Records Delegate a domain to name servers
6 TTL Set caching duration for DNS records
Look, I get it. DNS can seem like a labyrinth of cryptic records and arcane configurations. But here’s the dirty little secret: most devs are massively overcomplicating things. You don’t need to be a DNS guru to get shit done. In fact, understanding just six key concepts will cover about 80% of your DNS needs. Let’s break it down, shall we?

A Records: Your Domain’s Home Address
You’re launching a new website. Your hosting provider tells you your server’s IP address is 203.0.113.10. You’d set up an A record like this:
coolsite.com. IN A 203.0.113.1

In this case, 203.0.113.10 is the actual “house address” where your website lives on the internet. When someone types coolsite.com into their browser, this A record tells them to go to this specific IP address to find your website’s content.

Diving a bit deeper: A records can also be used for subdomain mapping. For instance, you could have separate A records for “blog.coolsite.com” and “shop.coolsite.com”, each pointing to different IP addresses. This allows for flexible hosting arrangements, like having your main site on one server and your blog on another.

CNAME: Your Domain’s Nickname Generator
You want both www.coolsite.com and coolsite.com to lead to the same place. You’d set up a CNAME record like: www.coolsite.com. IN CNAME coolsite.com.

Here, you’re saying that www.coolsite.com is just another name for coolsite.com. It’s like having a “doing business as” name – both lead to the same website, giving users flexibility in how they type your URL.

For the curious: CNAME records can be chained, but be careful – this can increase DNS lookup times. Also, the target of a CNAME must always be a domain name, never an IP address. This makes CNAMEs great for services that might change their IP addresses, as you only need to update the target domain’s A record.

MX Records: Directing Your Digital Mail
MX records are all about email. They tell the world which servers handle the email for your domain. Without these, your “contact@coolsite.com” address is about as useful as a chocolate teapot. Set them up right, and your email flows like a dream.

Example MX Records Usage
You’re using Google Workspace for email. You’d set up MX records like:
coolsite.com. IN MX 1 ASPMX.L.GOOGLE.COM.
coolsite.com. IN MX 5 ALT1.ASPMX.L.GOOGLE.COM.

These records are telling email servers that when someone sends an email to @coolsite.com, it should be delivered to Google’s email servers. The numbers (1 and 5) indicate priority – emails will try the first server before falling back to the second.

Going further: MX records have a priority value. You can set up multiple MX records with different priorities, creating a fallback system. If the primary mail server is down, the next one in line takes over. This adds redundancy to your email system, ensuring you never miss important messages.

TXT Records: The Swiss Army Knife of DNS
TXT records are the wild cards of DNS. Need to verify domain ownership? TXT record. Setting up email authentication? TXT record. They’re basically a place to stash any text-based information about your domain. Handy little buggers.

Example TXT Record
You need to verify your domain for Google Search Console. They ask you to add a TXT record like:
coolsite.com. IN TXT “google-site-verification=randomstringhere”

This record is like a secret handshake with Google. By adding this specific text to your domain’s DNS, you’re proving to Google that you have control over the domain, allowing you to use their Search Console tools for your site.

Digging deeper: TXT records are crucial for modern email security protocols like SPF, DKIM, and DMARC. These help prevent email spoofing and improve deliverability. TXT records can also be used for human-readable notes about the domain, though this is less common in practice.

NS Records: The Internet’s Yellow Pages
NS Record Example

NS (Name Server) records are crucial for the hierarchical structure of DNS. They tell the internet which servers are authoritative for your domain’s DNS information.

Example NS Record Usage:
You’re using Cloudflare for DNS. You’d set up NS records like:
coolsite.com. IN NS dana.ns.cloudflare.com.
coolsite.com. IN NS rick.ns.cloudflare.com.

By setting these NS records, you’re delegating responsibility for your domain’s DNS to Cloudflare’s name servers. It’s like telling the internet, ‘For information about coolsite.com, ask these Cloudflare servers.’

Going deeper: NS records are part of a larger system of delegation in DNS. When you change NS records, you’re changing who controls your domain’s DNS settings. This change needs to be reflected at your domain registrar to take full effect, which is why NS changes can take time to propagate fully across the internet.

TTL: The Expiry Date on Your DNS Cache
Time To Live (TTL) is how long DNS information is cached. Set it low, and changes propagate quickly but increase server load. Set it high, and you’ll wait longer for changes to take effect, but it’s easier on the servers. It’s all about finding that Goldilocks zone.

TTL Record Example
You’re planning to change your website’s IP address soon. You’d lower the TTL on your A record to 300 seconds (5 minutes):
coolsite.com. IN A 203.0.113.10 300

The 300 here means that DNS servers should only remember this information for 5 minutes before checking for updates. Before making significant DNS changes, it’s common practice to lower your TTL. This ensures that when you make the change, it propagates quickly across the internet. After the change has taken effect, you can increase the TTL back to a higher value for efficiency.

For the nerds: TTL is specified in seconds. A common strategy is to lower your TTL a day or two before making significant DNS changes. This ensures that when you make the change, it propagates quickly. After the change, you can increase the TTL back to a higher value for efficiency.

And there you have it, folks. Six simple concepts that’ll cover most of your DNS needs. Sure, there’s more to learn if you want to go deep – AAAA records for IPv6, DKIM for email authentication, DNSSEC for the paranoid – but honestly? You can probably offload that stuff from your brain for now.

Focus on these six, and you’ll handle 80% of your DNS tasks without breaking a sweat. When you do run into something more complex, you can always look it up. After all, isn’t that what we do for most of our jobs anyway?

So stop stressing about DNS and get back to building cool shit. That’s what we’re here for, right?
