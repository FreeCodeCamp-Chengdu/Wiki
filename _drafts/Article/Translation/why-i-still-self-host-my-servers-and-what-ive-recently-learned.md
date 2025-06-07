---
title: Christian Hollinger
date: 2024-08-25T00:00:00.000Z
authorURL: ""
originalURL: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/
translator: ""
reviewer: ""
---

# 为什么我仍然自托管服务器(以及最近学到的经验)
引言
除了电子邮件之外，我自己托管所有服务。我之前在[这里][20]、[这里][21]和[这里][22]写过相关内容。
总的来说，在家里，我运行着一个由 3 个节点组成的 Proxmox 集群，提供多项服务，为家庭网络提供支持。网络设备包括 Mikrotik 路由器、Mikrotik 交换机和 UniFi WiFi，此外还有一台外部 VPS。
本文主要讨论两个方面：为什么我仍然坚持这样做，以及最近从中学到了什么。可以将其视为一个简短的回顾，也是对读者探索相同领域的一种鼓励。
![heimdall.lan][88]
heimdall.lan [背景来自 HBO 宣传材料][23]

## 我的服务

我自托管：

-   `PiHole` 作为 DNS（冗余）
-   RouterOS（可能有点“自托管”，但它确实提供了 DHCP、VLAN、防火墙、DNS 路由等功能）
-   `UniFi controller` 作为 WiFi 控制器
-   `heimdall` 作为主页
-   `TrueNAS` 作为文件服务器（冗余）
-   `gitea` 作为本地 git 服务器
-   `wiki.js` 用于一般知识存储
-   `VS Code` 作为基于浏览器的编辑器
-   A `Ubuntu` 开发虚拟机用于一般代码杂耍（Debian 否则）
-   `mariadb` 用于数据库需求
-   `redis` 用于健忘数据库需求
-   `InfluxDB` 用于特定数据库需求（即“我打算重新回到的项目”）
-   `LibreNMS` 作为网络管理器和监控套件
-   `Calibre Web` 用于电子书和论文
-   `Komga` 用于漫画
-   `Jellyfin` 用于一般媒体
-   `Homebridge` 用于互联网的垃圾（tm）设备我不需要

此外，我还有一台外部 VPS，已经运行了 10 多年，托管了：

-   `nginx` 用于我的网站和这个博客
-   `firefoxsync-rs`，用于同步 Firefox（使用新的 `rust` 版本！）
-   `Nextcloud` 用于托管文件、日历和联系人

你可以想象，这确实是一项相当多的工作。我，尽管有普遍的看法，也有一个生活在外面的生活。事实上，我宁愿在周末出去（前提是天气不是 90 度）。这有时会与托管所有这些事情发生冲突。

对不熟悉的人来说:户外生活!(来自我们花园的示例图片) [拍摄:我][24]

那么...为什么要这样做呢?

## 为什么我要自托管

这篇文章并不完全是关于这个问题的,但我不喜欢标题党,所以我会切实回答标题中的问题:有两个原因。

首先,我喜欢独立 - 或者至少,尽可能独立。这和我们有备用电源的原因一样,也是为什么我会烤面包、保存食物,以及总体上像一个迫切想要喂饱 12 个孙子直到他们无法自行移动的祖母那样生活。这让我能够_相对_独立于你当地的`大公司`最近在搞的任何邪恶计划(_提示_:很可能是某种订阅服务)。

这基本上就是 Linux 和 Firefox 的论点 - 竞争是好事,自由也是。

如果这对你来说太抽象了,那么这篇文章_真正_要讲的是,它能教会你很多东西,这是我认为不言而喻的真理:学习是有益且有用的。

事实证明,强迫自己要么做一些你不是每天都做的事情,要么提高你偶尔做的事情的水平,或者仅仅是学习一些听起来有趣的东西,都能让你变得更好。这是个疯狂的概念,我知道。

在我的实际工作中,我不是系统管理员,而是一名软件工程师。我从事大型分布式数据处理系统的工作。我真的开始厌恶"数据工程师"这个头衔(这是另一个故事了),所以我不会用它。但是,在`$酷炫网络创业公司`领导数据基础设施工作的普通一天,我通常会做以下一项或多项工作:

-   会议、办公时间、架构讨论,诸如此类
-   写带有有趣颜色的文本,这些天主要是`Python`、`Scala`、`go`、`typescript`和`sql`
-   处理实际的基础设施和工具:K8s、Terraform、Docker、VPCs、EC2s、nix 等有趣的东西
-   可能相关的是,盯着 AWS 成本浏览器或电子表格
-   照看一堆开源服务,管理更新等
-   保持项目管理内容合理更新和可展示
-   等等

我确实与系统管理员的工作有一些合理的重叠,主要是因为我支持开源软件,因此决定使用和[托管][25]各种开源工具作为我们数据栈的一部分。虽然大部分都很稳定,但偶尔确实需要一些基础的 Linux 技巧来让一切回到正轨。

## 理解复杂系统

话虽如此，我很少需要通过`ssh`登录到随机的 Linux 机器上去检查`nginx`。这并不是说这种情况_不会_发生 - 只是_不经常_发生。

而当它_确实_发生时，能够通过深入理解底层技术而不是仅停留在表面来分析复杂系统是非常有帮助的。

自托管所有这些服务 - 特别是当它们让你比仅仅复制粘贴命令更深入地探索技术细节时 - 会教会你很多有用的新知识。

让我举几个现实世界的例子 - 通常，基础的 Linux_管理_知识就能派上用场。获取这些知识最好的方式是要么日常使用 Linux（就我个人而言，现在使用的是 macOS）...要么自托管和自我管理服务器。

例如，知道设置[swapiness][26]很少有用，并在代码审查中指出这一点；或者知道修改[`LD_LIBRARY_PATH`][27]必然会带来麻烦。不过，意识到这两者_可能_在特殊情况下是有效的工具也很重要（对于后者，我们混合使用`poetry`和`nix`，你可以想象会发生什么）。

当然，维护分布式基础设施（如 Proxmox）或[从头开始编写一个][28]也有助于维护、设计和/或实现跨越多个概念的_其他_复杂分布式系统。

我在工作中的一个个人项目是一个[Apache Flink][29]管道（本身就是一个固有的分布式框架），使用 Scala 3 编写，通过[magnolia][30]和[avro4s][31]进行类型类推导，将[protobuf][32]消息通过[Kafka][34]转换为[Apache Iceberg][33]（它还做了更多事情，但这些是基础部分）。这个项目涉及了以下概念的深入理解，这些概念_不是_核心的"编程"任务，而是_额外_的内容（因为它是`scala`，还包括像 tagless final 这样的内容）：

VPC 和大量网络知识、Kubernetes、GRPC、exactly-once/at-least-once 语义、最终一致性、事务锁、一些[编译器构建][35]、protobuf 和模式演进、Kafka（或通用数据存储复制和多租户）、blob 存储、大量的指标和监控（因此也包括时序数据库，姑且这么说）等等。

![这是我们社区谷仓猫的照片，这样当我们讨论protobuf编译器时你就不会太无聊][90]

这是我们社区谷仓猫的照片，这样当我们讨论 protobuf 编译器时你就不会太无聊[拍摄：我][36]

当然，如果你是一个有经验的工程师，之前已经接触过分布式系统，这些主题可能都不会显得特别新颖或可怕 - 但关键是，其中很多内容都与自托管软件时遇到的问题有重叠 - 事实上，在我提到的 11 个左右的要点中，我敢说在我的自托管历程中至少接触过其中的 8 个。

最后，如上所述，在`work`中我也"拥有"（在这种情况下，这不是一个很糟糕的"企业"用语吗？）几个开源工具，我们在 Kubernetes 上自托管并进行内部使用，比如[Apache Superset][37]。这与本文主题的相似之处应该相当明显。:-)

## 过去 6 个月出现的问题

好吧,听起来很有用。但还记得我说过"这需要大量工作"吗?在过去大约 6 个月里,发生了以下这些事情(我相信这还不是完整的清单):

- 我们遇到了断电,而 UPS 本应负责为服务器和家庭网络提供备用电源,但它就是...不工作
- 因此,我们的网络(存在单点故障)在某台服务器宕机时就完全瘫痪了
- 我唯一外包运维的 VPS 服务器随机离线了将近一周
- 我的一台 Proxmox 主机每天晚上都会准时在 23:25 崩溃,然后在 23:40 重新上线

## 过去 6 个月学到(或回顾)的经验

那么,让我快速分享一下在这段时间内我做的一些事情,部分是为了解决上述问题,部分出于好奇,以及我从中学到和回顾的经验。

### 你可以自托管 VS Code

这是一个简单但很巧妙的发现:VS Code 的操作系统部分可以在浏览器中运行。这与 GitHub 的 codespaces 使用相同的概念,但是自托管:[https://github.com/coder/code-server][38]

这在 iPad 或 Mac(或 Windows)等需要 Linux 环境的设备上使用 VS Code 非常有用。我的运行在`Ubuntu`上。

![在iPhone上好用吗?呃...][91]

在 iPhone 上好用吗?呃... [拍摄:我][39]

我在思考如何让我那台价格不菲的 iPad Pro 更有用时偶然发现了这个。事实证明,你确实可以!我不是说你_应该_这么做,但你_可以_。

### UPS 电池会悄无声息地死亡,而且比你想象的更快

UPS 电池,至少像我这种简单的 1U Cyberpower 1500VA 消费级产品,只能用大约_3 年_。我的电池已经 4 年了,_完全_报废了。虽然我在概念上知道不是所有东西都像锂电池那样寿命长,但我_没想到_它们会这么快就达到"无法使用"的程度。

在经历了这里常见的断电和电压不稳后,我才痛苦地意识到这一点,网络会看似随机地断开。事实证明,没有冗余 DNS 的话,互联网就无法工作。

更换这些电池其实很简单(感谢 RefurbUPS)。现在,我实际上每月都会安排运行`powerst -test`,并会更频繁地测试其他模拟备用电池。

![pwrstat][92]

pwrstat [拍摄：我][40]

### 冗余 DNS 就是好 DNS

我们都知道 DNS 本质上是冗余的、全球分布的,并且最终一致的。但实际上我从未同时_托管_过 2 个 DNS 缓存。

我之前没这么做的原因是 - 我使用 Pi-Hole,它运行`dnsmasq`。我_还_把它用作 DHCP 服务器,以便查看哪台机器运行了哪些 DNS 查询。所以出于懒惰和便利的考虑,将 DHCP 从 Pi-Hole 界面移出是很麻烦的。

为了解决这个问题,我使用 RouterOS(Mikrotik)为每个设备和 VLAN/IP 池设置静态 IP,并重新启用了 DHCP 服务器。对于不使用 Pi-Hole 而使用公共 DNS 的非家庭 VLAN,我_已经_这样做了。

![VLAN][93]

VLAN [拍摄：我][41]

我也借此机会将 IP 范围映射到物理设备。

然后我使用该映射在 Pi-Hole 中将主机名映射到 IP:

![DNS映射][94]

DNS 映射 [拍摄：我][42]

之后,只需在 RouterOS 中配置两台 Pi-Hole 服务器,使它们都可用于客户端。这就是穷人版的高可用性!

由于我们不经常添加设备(而且我会将所有_需要_DNS 名称的服务添加到[heimdall][43]),通过查看 Pi-Hole 日志手动复制配置就足够了。

我已经把托管[orbital-sync][44]列入计划,以进一步简化这个过程。

在这一点上,你_也可以_通过针对 53 端口 UDP 流量的防火墙规则来强制使用这些 DNS 服务器(这在有孩子或工作环境中可能是个好主意)。RouterOS 可以轻松实现这一点。

### 树莓派可以运行 ARM 版 Proxmox,而普通 Proxmox 不行

通常情况下,Proxmox 只支持 x86 架构。

但是,这个 GitHub 上的[仓库][45]可以让 Proxmox 支持 ARM 架构。正因如此,我的小树莓派 5 现在成为了一个 Proxmox 节点,运行着 UniFi 和其中一个 Pi-Hole。

不推荐且不受支持?当然!就像通过 USB 设备使用`zfs`一样。但它就是能用(在合理范围内)!

![Promox节点][95]

Promox 节点 [拍摄:我][46]

我刚才提到了"穷人版的高可用性":有趣的是,[Linus Tech Tips][47]最近做了一个关于 Proxmox 正式版 H/A 虚拟机和故障转移的视频。我的 DNS 设置虽然不是那样,但_可以_做到,因为现在我有 3 个节点了,多亏了我这个改装过的 ARM 节点。就像他们说的:三个节点才能构成法定人数,宝贝!

_再补充一点_:我第一次接触"法定人数"这个概念是在早期的`hadoop`时代,那时候手动配置`zookeeper`、`hdfs`等并确保集群能形成法定人数是非常重要的。这肯定比"只需部署这个作业到 AWS"要复杂得多(我也不想再那样做了),但它确实让我学到了很多关于其中运作原理的知识!

### `zfs` + Proxmox 会吃掉内存并导致虚拟机被 OOM 杀死

你知道 OOM 杀死在`$currentYear`年还是"一个问题"吗?确实如此!我运行着带[SAS HBA 直通模式][48]的`TrueNAS Scale`,当 Proxmox 备份和设备备份同时运行时,我的机器偶尔会被 OOM 杀死。

说到这里 - 用这种配置运行 TrueNAS 也是不受支持/不推荐的,因为 TrueNAS 希望直接运行在物理机上。

总之,Proxmox 默认会使用高达 50%的内存作为`zfs`缓存,再加上虚拟机被配置为使用~80%的内存,在 RAM 有限的环境下 - 而且几乎全是从其他电脑收集来的旧内存条 - 这样运行效果并不好。

但是,原来你可以在 ZFS 内核设置中[配置][49]这个参数!

```bash
echo 'options zfs zfs_arc_max="8589934592"' >> /etc/modprobe.d/zfs.conf
update-initramfs -u
```

这样可以将`zfs`限制在 8 GiB 内存使用量。

### 随机崩溃之谜(是硬件问题吗?一直都是硬件问题。)

我的一台 Proxmox 主机每天晚上都会准时在 23:25 崩溃,然后在 23:40 重新上线。这是监控数据:

![TBD][96]

TBD [拍摄:我][50]

虽然它_确实_会自动重启,包括虚拟机,但当然无法解密`zfs`存储池。我只是在 Mac 的 TimeMachine 备份失败时才注意到这个问题!

好吧,查看日志发现:23:25 是另一个 Proxmox 节点将其虚拟机备份到这台服务器的时间,备份到一个单一的、非冗余的老硬盘上,其 SMART 值如下:

```plain
Error 4 occurred at disk power-on lifetime: 27469 hours (1144 days + 13 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  04 51 00 ff ff ff 0f

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  ea 00 00 00 00 00 a0 00   6d+03:38:11.728  FLUSH CACHE EXT
  61 00 08 ff ff ff 4f 00   6d+03:38:11.726  WRITE FPDMA QUEUED
  61 00 08 ff ff ff 4f 00   6d+03:38:11.726  WRITE FPDMA QUEUED
  61 00 08 ff ff ff 4f 00   6d+03:38:11.725  WRITE FPDMA QUEUED
  61 00 b0 ff ff ff 4f 00   6d+03:38:11.725  WRITE FPDMA QUEUED
```

...这导致了各种奇怪的死锁,使得机器无响应并被杀死。

![LibreNMS监控][97]

LibreNMS 监控 [拍摄:我][51]

更换硬盘后,问题解决。就是这么简单。

`大硬盘厂商`会告诉你_只能_使用闪亮的、新的、fancy 的、昂贵的硬盘,但地下室里装着服务器机架的人会告诉你,事实上,你完全可以重复使用那些古老的、各式各样的硬件来做一些不那么重要的备份,而且基本上~不会~几乎不会出问题!

### SNMP(v3)仍然很酷

我通过提醒来过活。我的待办清单上有一项已经存在一段时间了:"自动化网络映射和告警"。事实证明,这是一个已解决的问题,即使_不使用_现代的监控和追踪解决方案,如 Datadog(顺便说一下,我很喜欢它)、Jaeger、Prometheus 等。

上面那些关于硬盘衰退的图表?来自[LibreNMS][52]!

它使用简单网络管理协议([SNMP][53])。这是一个来自 80 年代的古老协议,_正好_可以做到这一点:监控你的设备。我之前一直没有去设置它,因为对现代人来说这是一个有点晦涩的协议。虽然我不允许入站流量(我们用[ngrok][54]来处理),但一个偏执的系统管理员才是好系统管理员。

> SNMP 依赖于安全字符串(或"团体字符串"),这些字符串授予对设备管理平面部分的访问权限。SNMP 的滥用可能允许未经授权的第三方访问网络设备。
> 
> 应该只使用 SNMPv3,因为 SNMPv3 具有认证和加密负载的能力。当使用 SNMPv1 或 SNMPv2 时,攻击者可能会嗅探网络流量以确定团体字符串。这种妥协可能会导致中间人攻击或重放攻击。
>
> ...
>
> 仅仅使用 SNMPv3 还不足以防止协议被滥用。更安全的方法是将 SNMPv3 与使用 SNMP 视图的管理信息库(MIB)白名单结合使用。
>
> [https://www.cisa.gov/news-events/alerts/2017/06/05/reducing-risk-snmp-abuse][55]

我在 LibreNMS 中添加了网络上所有支持的设备,使用最"安全"的协议版本 V3。

它可以免费完成一个很酷的、现代的、基于 SaaS 的追踪和监控服务所做的一切。好吧,也许不是_全部_ - 但它的深度令人愉悦。其他选项(不一定互斥)包括`zabbix`和`munin`,我之前都尝试过,但从未坚持使用。事实证明,这是一个深不可测的领域,而且不是我非常了解的领域。

![LibreNMS][98]
LibreNMS [由我提供][56]
但我_确实_喜欢图表和漂亮的仪表盘。
不要相信你的 VPS 供应商
我的 VPS 由 Contabo 托管,随机宕机了将近 4 天。你无法读到这个博客。
只有_通过不同 IP 的 VNC 才能工作。我能与自己从_2014_年就持续付费的服务器通信的唯一方式就是 VNC。_其他_什么都不行 - ping、SSH,什么都不行。而我为自己写出详细的支持工单并自己检查所有可能的问题而感到自豪 - 这次,问题_不是_出在 DNS 上。:-)
事件的简要时间线如下:首先,我提交工单描述问题,包括 DNS 配置的副本。
我会避免在此粘贴逐字逐句的互动内容,但他们的回应归结为
> 你的问题听起来像是没有设置 DNS 服务器的情况。
……后面是一些通过 resolv.conf 设置 Google DNS 服务器的步骤。
当然,DNS 是我首先检查的东西。见上文。我的回复包括了 DNS 配置的另一个副本(这次是截图),以及一个友好的提示,说我无法安装 apt 包,因为机器与他们数据中心之外的任何东西都断开了。
我试图通过做更多挖掘来提供更多见解。原来,我甚至无法直接 ping 通 IP(即,不涉及 DNS)。似乎我的 ISP 和他们的数据中心无法通信。

![外部网络路径说不][99]
外部网络路径说不 [由我提供][57]
还有一个有趣的观察结果是,VPS 无法与外部的任何东西通信。VNC 之所以有效,是因为它会通过同一数据中心的另一台机器隧道传输。
![Ping在内部也说不][100]
Ping 在内部也说不 [由我提供][58]
93.184.215.14 是[example.org][59]。这永远不会超时,因为我让它通宵运行。是的,考虑到简单的 ping 的局限性,我也用 netcat 做了所有这些调试。
更奇怪的是,arp 扫描给我返回了 207.244.240.1,这是他们在密苏里州的数据中心。虽然可以访问,但从外部到达那里充其量也是断断续续的:
![不稳定][101]
不稳定 [由我提供][60]
这里的一切都表明他们那边存在更大的网络问题,而我的服务器已经离线 48 小时了。
2 天后,我自己的选择已经用尽,机器突然重新上线了 - 据我所知,是在一个不同的数据中心。我没有做进一步的更改,而是着手迁移到 Hetzner(见下文)。我收到了这封电子邮件:
> 感谢您的耐心等待。我们检查了您的 VPS M SSD/它对 Ping 请求有反应,SSH 和 VNC 也都可以访问。因此,我们认为问题已经解决。请您再次检查您那边的情况。

……太棒了。我_什么都没做_(见上文 - 我想我也做不了什么!)然后它突然就工作了。
如果我_确实_搞砸了什么,我当然不会不承认 - 我在这篇文章和这个博客中已经多次这样做了,我_经常_搞砸系统管理员的事情 - 但这一次,我向你保证在这件事发生之前,我没有对这台机器做任何更改,在它神奇地被修复之前也没有。
我不指望从更便宜的 VPS 获得[99.999%][61]的可用性,他们只"[宣传][62]"(我轻描淡写地使用这个词 - 它隐藏在他们的条款和条件中)和[95%][63]的 SLA,这意味着他们允许每年超过 18 天的停机时间。
> (1) 提供商将确保对象存储基础设施、网页空间包、专用服务器、虚拟专用服务器和 VPS 的物理连接性以 95%的年平均比率可用。
不幸的是,这种_互动_,不一定是停机时间本身,实在太糟糕了,以至于我完全停止推荐 Contabo,并从那时起转移到 Hetzner - 当然,还有我自己的物理硬件,它有 99.995%的正常运行时间:
![正常运行时间][102]
正常运行时间 [由我提供][64]
实际上,从长远来看,它可能更接近 98%,但那涉及到我主动破坏东西。你可以自行判断。
必须快起来
所以,因为这个原因,我寻找替代方案。出于延迟原因,我想要一个便宜的美国 VPS,具有基本的舒适性。最终,经过多方考虑,我转移到了[Hetzner][65],另一家德国公司,至少在北美有一些数据中心,使用他们每月 7 美元左右的小型 CPX21 VPS。
让我非常惊讶的是,在基准测试中,该机器比我(名义上更强大的)Contabo VPS 还要快:

```bash
sysbench --threads=4 --time=30 --cpu-max-prime=20000 cpu run
```

难住我了
![VPS基准测试 1/2][103]
VPS 基准测试 1/2 [由我提供][66]
以及
![VPS基准测试 2/2][104]
VPS 基准测试 2/2 [由我提供][67]
是的,合成基准测试并不完美,但它们当然也不是完全没有意义。
同样,你可以自行判断这些数字的意义。
CIFS 仍然不快
不幸的是,服务器配备了 80 GB 的 SSD。那只相当于 29,000 张 3.5 英寸的软盘!请记住,这台服务器运行 Nextcloud,本质上需要大量存储空间。
我的 MacBook 使用 1.2/2 TB,我的 Proxmox 集群总容量约为 50 TB(公平地说,有很多冗余),甚至我的手机也使用了约 200GB。
我偶然发现了 Hetzner 的"[StorageBox][68]"。很酷的概念:基本上就是[rsync.net][69]所做的,但是是德国的。你基本上可以以非常低的价格获得一个简化的终端和一个缓慢的共享磁盘:约 4 美元/TB!不幸的是,没有美国的位置。
你可以通过 FTP、SFTP 或 SCP(又名 SSH)、WebDAV、各种备份工具如 borg 等访问这些磁盘。
总的来说是一个非常有前景的概念,但一旦你需要将所述磁盘连接到服务器以增加本地存储并在 Nextcloud 中可用,你的选择就有些受限,官方[文档][70]推荐使用 samba/cifs。[sshfs][71]是一个有效的替代方案,该项目处于维护模式,但我还是测试了它。我想从概念上讲,你可以让 WebDAV 工作。
无论如何,我尝试使用官方方法并进行了一些基准测试,因为它确实_感觉_很慢。
嗯,那是因为它确实很慢。这里有三个基准测试,两个使用 cifs(V3),分别使用严格和宽松的缓存,以及 sshfs。

```bash
SIZE="256M"
# Sequential
fio --name=job-r --rw=read --size=$SIZE --ioengine=libaio --iodepth=4 --bs=128K --direct=1
fio --name=job-w --rw=write --size=$SIZE --ioengine=libaio --iodepth=4 --bs=128k --direct=1
# Random
fio --name=job-randr --rw=randread --size=$SIZE --ioengine=libaio --iodepth=32 --bs=4K --direct=1 
fio --name=job-randw --rw=randwrite --size=$SIZE --ioengine=libaio --iodepth=32 --bs=4k --direct=1
```

![存储基准测试 1/2][105]
存储基准测试 1/2 [由我提供][72]
这是我用来测试的内容:
这也不是一个新发现,因为我不是第一个对此进行[基准测试][73]的人。
无论如何,这相当慢。缓慢的协议 cifs、缓慢的盒子_以及_跨大西洋的延迟相结合,使其不是特别有吸引力。
为了给你一些视角,这里是同样的图表,使用相同的基准测试在本地 Proxmox VM 上运行:
![存储基准测试 2/2][106]
存储基准测试 2/2 [由我提供][74]
Blob 存储、Blob 鱼和文件系统:都是"呃"
现在,甚至_不使用_"磁盘",这就是现在所有酷孩子都在做的事情。他们简单地抛弃了现代、快速、本地存储的几乎所有优势,包括高质量、经过测试、原子性、纠错(……)文件系统,你猜怎么着,没有_文件系统,而是称之为"对象"(或"blob")存储,并且没有获得这些功能,但获得了大量廉价的存储空间!
离题:既然"blob"这个词现在被索引了,在你为存储架构的失败而羞辱一条无助的鱼之前,请记住以下内容:
![可怜的鱼][107]
可怜的鱼 [由 Naila Latif 提供][75]
无论如何 - 尽管有缺点,但这并不是一个新概念。在专业领域,我会为开发人员机器、数据库等使用快速的 NVMe 驱动器,为不太密集的应用服务器使用常规 SSD,为慢速但便宜的数据存储和分布式查询引擎使用 blob 存储,并在需要时使用可选的缓存。在过去约 10 年的大多数数据平台项目中,我都基于 S3/GCS 等,并且通常存储了数百 TiB(如果不是 PiB)的数据。
然而 - 这些存储系统_不是_文件系统,试图假装 s3 和 zfs 相似,因为它们都存储文件(就像某种疯狂的、表面层次的🦆类型),这不会有好结果。
但是,考虑到我的访问模式 - 存储文件并偶尔手动访问它们,而不是在其上托管数据库,就像我使用基于慢速 HDD 的本地文件服务器一样,我仍然在寻找 blob 存储选项来仅存储我的 Nextcloud 数据。
你的选择基本上是大型提供商,如 AWS,其中 1TB 的费用大约为每月 20-24 美元,具体取决于提供商和地区。
实现相同协议(例如 s3)的第三方替代方案主要是[Wasabi][76]和[Backblaze][77]。

Wasabi 的最低收费为每月 6.99 美元(这是每 TB 的价格),并且_最少存储 90 天(即删除数据无济于事)。Backblaze 表面上更便宜,为每月 6 美元/TB,但如果超出平均每月存储数据的 3 倍,他们会对出口流量收取每 GB 0.01 美元的费用,而 Wasabi 包含了这部分费用。
考虑到在迁移期间我花了 35 美元的出口费用才将 500GB 的服务器备份从 S3 中取出,并在阅读了[这篇文章][78]后,我选择了 Wasabi 并将文件移动到那里:
![Wasabi][108]
Wasabi [由我提供][79]
然后使用 Nextcloud 的[外部存储][80]界面来挂载驱动器。
我能说什么呢,它...可以工作。它的行为就像 S3 一样:如果你需要列出大量文件或对大量小文件进行任何操作,那将是一个巨大的痛苦,而且_不会_很快。这就是为什么半官方建议使用[存储桶生命周期][81]策略来删除大量小文件的原因。
对于每 TB 6.99 美元,它工作得_还行_。在冷缓存(例如新设备)上列出文件慢得像糖蜜,搜索文件完全取决于你的索引(与本地 NVMe 存储上的 find 或 grep 相比)。
看到这个屏幕持续几秒钟是正常的:
![iOS上的Nextcloud][109]
iOS 上的 Nextcloud [由我提供][82]
公平地说,这实际上只是移动设备上的问题,因为无论如何我在 Mac 上已经下载了所有文件。
我可能还会尝试[rsync.net][83],它的价格为每月 12 美元/TB。
CrowdSec
最后但同样重要的是,你还记得[fail2ban][84]吗?你应该记得,因为它仍然是一个非常活跃的项目,可能应该作为简单入侵防御的默认推荐。
但是,在设置新服务器时,我认为值得检查一些较新的工具,即[CrowdSec]85),它将自己描述为

> 一个免费、现代和协作的行为检测引擎,与全球 IP 信誉网络相结合。它基于 fail2ban 的理念,但兼容 IPV6 且速度快 60 倍(Go vs Python)(……)
CrowdSec 的优点在于,顾名思义,威胁检测模式正在与他们的社区(匿名)共享,这(据称)产生了更好的威胁检测效果。它还有一个插件[中心][86],为各种工具提供"保镖"。
它还附带了一个整洁的命令行界面,可以为你提供它正在做什么的统计信息:
例如,它会告诉我它因 crowdsecurity/CVE-2019-18935(一个针对[此][87] CVE 的规则)禁止了 33 个 IP。
总的来说,我是它的粉丝 - 它无疑感觉像是 fail2ban 的自然演变(即使我怀疑它们可以共存),只要背后的公司在某个时候不会利用其用户群,我就会坚持使用它。
结论
如果你是一名软件工程师,我建议你自己托管一些东西。通过强制接触那些在日常工作中不太可能遇到的问题,你可以学到很多东西,这本身就是一种好处。更好的是,我相信你最终会在日常工作中至少使用其中一些东西,前提是你从事的是与后端有点关系的工作。
通过自己托管东西,你还可以获得合理程度的自主权 - 或者至少是一些对冲 - 来对抗你的整个生活都是永久租用订阅的企业梦想。我认为这很好。


[20]: https://chollinger.com/blog/2019/04/building-a-home-server/
[21]: https://chollinger.com/blog/2023/04/migrating-a-home-server-to-proxmox-truenas-and-zfs-or-how-to-make-your-home-network-really-complicated-for-no-good-reason/
[23]: https://www.hbo.com/curb-your-enthusiasm
[25]: https://ngrok.com/blog-post/how-ngrok-uses-dagster-to-run-our-data-platform
[26]: https://www.howtogeek.com/449691/what-is-swapiness-on-linux-and-how-to-change-it/
[27]: https://www.hpc.dtu.dk/?page_id=1180
[28]: https://github.com/chollinger93/bridgefour
[29]: https://flink.apache.org/
[30]: https://github.com/softwaremill/magnolia
[31]: https://github.com/sksamuel/avro4s
[32]: https://scalapb.github.io/
[33]: https://iceberg.apache.org/
[34]: https://kafka.apache.org/
[35]: https://github.com/scalapb/ScalaPB/pull/1674
[37]: https://superset.apache.org/
[38]: https://github.com/coder/code-server
[43]: https://github.com/linuxserver/Heimdall
[44]: https://github.com/mattwebbio/orbital-sync
[45]: https://github.com/jiangcuo/Proxmox-Port
[47]: https://www.youtube.com/watch?v=hNrr0aJgxig
[48]: /blog/2023/10/moving-a-proxmox-host-with-a-sas-hba-as-pci-passthrough-for-zfs--truenas/
[49]: https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Module%20Parameters.html#zfs-arc-max
[52]: https://www.librenms.org/
[53]: https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol?useskin=vector
[54]: https://ngrok.com/
[55]: https://www.cisa.gov/news-events/alerts/2017/06/05/reducing-risk-snmp-abuse
[59]: http://example.org/
[61]: https://uptime.is/99.999
[62]: https://contabo.com/en-us/legal/terms-and-conditions/
[63]: https://uptime.is/95
[65]: https://www.hetzner.com/
[68]: https://www.hetzner.com/storage/storage-box/
[70]: https://docs.hetzner.com/robot/storage-box/access/access-samba-cifs/
[71]: https://github.com/libfuse/sshfs
[73]: https://blog.ja-ke.tech/2019/08/27/nas-performance-sshfs-nfs-smb.html
[75]: https://nailalatif002.medium.com/on-ugliness-of-blobfish-culpable-ignorance-and-gods-guilt-14fd3b226908
[76]: https://wasabi.com/
[77]: https://www.backblaze.com/
[78]: https://help.nextcloud.com/t/high-aws-s3-costs-due-to-nextcloud-requests/68687/4
[80]: https://docs.nextcloud.com/server/29/admin_manual/configuration_files/external_storage_configuration_gui.html
[81]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html
[82]: https://chollinger.com
[83]: https://rsync.net/
[84]: https://github.com/fail2ban/fail2ban
[85]: https://github.com/crowdsecurity/crowdsec
[86]: https://app.crowdsec.net/hub?q=next&page=1
[87]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-18935
[88]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825124525084.png
[89]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825135445550.png
[90]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825131026074.png
[91]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825131634407.png
[92]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240824101120003.png
[93]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607213853924.png
[94]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607214429699.png
[95]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607214732963.png
[96]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607215257458.png
[97]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607215539705.png
[98]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240824103028884.png
[99]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607221106538.png
[100]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607221115352.png
[101]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240607221122749.png
[102]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825104016231.png
[103]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/GPGtDkkXoAAW9XD.png
[104]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/GPGtEuXXYAAdzJ4.png
[105]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825111920987.png
[106]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825134453828.png
[107]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/blob.jpeg
[108]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825112653613.png
[109]: https://chollinger.com/blog/2024/08/why-i-still-self-host-my-servers-and-what-ive-recently-learned/assets/image-20240825114612035.png
