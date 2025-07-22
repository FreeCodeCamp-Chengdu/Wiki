---
title: Tyblog
date: 2025-07-22T01:12:01.636Z
authorURL: ""
originalURL: https://blog.tjll.net/the-systemd-revolution-has-been-a-success/
translator: "luojiyin"
reviewer: ""
---

### [«][1]  systemd 的革命取得了彻底、完全、无可辩驳的成功

<!-- more -->

-   2025 年 7 月 8 日
-   1,782 字
-   预计阅读时间 7 分钟

![he-can-meme.png](https://blog.tjll.net/assets/images/he-can-meme.png)

图 1：2010 年代的 systemd 之战异常激烈，伤亡惨重。

时间回到 2013 年，我当时气得跳脚。

`systemd` 正在用二进制格式取代我的纯文本日志，还给 `init` 打了激素，它还在嘲笑我。Unix 哲学在呐喊：这就是 Linux（或者，正如许多人称呼的，GNU 加 Linux）的终结吗？

时间来到 2025 年，我在这里忏悔。不仅 systemd 是传统 `init` 的合格继任者，我还认为它值得为其所做的事情辩护——尤其是在它最初（甚至现在莫名其妙地）遭遇如此敌意的情况下。没有软件是完美的——除了 [TempleOS][2]——但我认为 systemd 基本上是一个成功的故事，证明了许多可怕的预测（包括我自己的）都是错误的。我错了！

#### `init`  旧石器时代

我希望我不需要再抱怨为什么过去的状况并不好——质量参差不齐的 init 脚本、混乱的依赖关系和极不统一的语义都令人沮丧。让我觉得不可思议的是，我作为一名全职软件工程师，竟然还经历过用定制 shell 脚本来编排进程管理的时代。“丢失”或无人管理的进程、用 `S99` 这类目录来排序依赖的怪异方式，以及各种 `/etc/init.d` 脚本的不同接口，这些都是真实存在的问题。

![dino.png](https://blog.tjll.net/assets/images/dino.png)

图 2：/etc/init.d，总会找到办法

在 **LINUX INIT WARS** 期间，你或许能写出一份 upstart、s6 或 OpenRC 的 init 脚本，问题不算太多。但即便如此，你还是要支持各种服务管理配置格式，而它们的行为又略有不同。我为这些不同的 init 系统都写过服务！体验并不美好！

许多传统服务管理的缺陷，事后看来更加明显。原始的 `init` 主要是处理和/或回收孤儿进程，而基于 systemd 的 PID 1 则在沙箱和依赖管理方面有了很大提升。我们还没谈到定时器、套接字或挂载点呢。

#### 我废弃了你妈

我们没必要详细重述我们是怎么走到今天的。但"怎么走到这一步"正是我认为 systemd 最终成功的原因之一。

要知道，旧的 init 系统管理进程的两种主要方式——前台或 fork——在 systemd 里都得到了完全支持。现代 systemd 提供了更细致的"我已就绪"信号（通过 `Type=notify`），而不仅仅是"进程是否存活"，这种向后兼容性极大地弥合了遗留鸿沟。systemd 的作者甚至写了[生成器代码来帮助迁移旧服务][3]。

我不认为 ini 风格的配置格式是万能的（我喜欢 [Dhall][4]），但这也是 systemd 作者向系统管理员伸出的橄榄枝：它不需要图灵完备的配置格式或领域专用语言。你通常能看懂它的意思，也知道怎么改：

Systemd

-   用于高亮内置项的字体。
-   用于高亮关键字的字体。
-   用于高亮类型和类名的字体。

```plain
[Service]
```

[默认很重要][5]，配置语言也很重要。我很欣赏 systemd 选择了一个显而易见的方案。

我还可以举其他例子，但我想强调的是，systemd 有意选择了

-   向后兼容，
-   简单的配置范式，
-   并主动支持和帮助用户迁移。

并不是每个开源项目都会明确采取措施支持旧路径的弃用。Lennart，你真是个好人。

#### 信任这个进程

我不只是觉得 systemd 是我们现在更酷的"老爸"，能把以前烦人的事做得更好，systemd 还带来了许多全新的好东西。

###### 谁来关心纯文本？

![log-logs.png](https://blog.tjll.net/assets/images/log-logs.png)

图 3：日志的日志在有条不紊地记录

`journald` 来了。过去的我也讨厌它。大家对 journald 的主要抱怨是它的日志文件不是纯文本。我会怀念吗？有一点吧。我骨子里还是个 Linux 老顽固，喜欢用 `awk` 处理一切。

但我 _真的_ 很喜欢有一个地方能收集所有 `stdout` 和 `stderr`！你有没有在写日志到 journal 时用过自定义字段？我会给一些服务加上 `NOTIFY_SLACK=1`，监听实验室的日志流，把这些事件转发到 Slack 频道，这样更容易看到我想要的日志，非常棒！

此外，把日志轮转和磁盘空间的责任交给 journald 也很方便。它能感知文件系统空间，我基本再也不用猜测轮转频率了[1][6]。你知道你的日志文件是二进制格式而不是纯文本的部分原因，是因为 journald 会自动压缩它们吗？这个默认选择可能为整个计算领域节省了数十亿 GB 的空间。

我们依然可以实时查看日志，依然可以把日志流转发到其他服务器，服务现在也能可靠地信任它们的输出会在运行时被捕获。这些都是纯粹的好事。

###### 定时器（Time-r Out）

![clock-wall.png](https://blog.tjll.net/assets/images/clock-wall.png)

我还记得在大学工作时调试 `cron` 脚本：是不是 `$PATH` 配错了？要不要在某处 `echo $USER`？为什么默认输出会发到 _邮件队列_ 里？？？

如果要评选"最容易理解的替代品"，systemd 的定时器系统可能是最佳候选。每个 Linuxer 看到一串星号就能自豪地知道 `0 0 * * *` 是什么意思，但我们都知道 `OnCalendar=daily` 更容易理解。`OnCalendar=minutely` 是个词吗？语法警察可能不认，但你大概能猜到它的意思！

我可以写一整篇博客来讲我有多喜欢 systemd 的定时器，这里就列个清单吧：

-   `Persistent=true` 是个好工具，能确保你不会错过定时任务的执行。
-   `systemctl list-timers` 是查看机器上所有定时任务的极佳方式。
-   `OnCalendar=` 和 `OnActiveSec=` 的调度灵活性既强大又易懂。

###### 套接字激活

光是这一点，就极大地优化了系统。`nix-daemon` 就很好地利用了这一点，通过"懒加载"只在需要时运行：当你不在构建时，守护进程会停止，但只要你需要，`nix-daemon.socket` 就会启动 `nix-daemon.service`。这太棒了！

一如既往，systemd 甚至提供了 `systemd-socket-proxyd` 可执行文件，帮助那些还不支持原生协议的服务实现套接字激活。我用这个技巧让像 Minecraft 服务器这样重量级的守护进程按需启动：我不用改动原始守护进程，`systemd-socket-proxyd` 就能让我用套接字激活按需运行它。

###### 一把"单元"

![unit-hand.png](https://blog.tjll.net/assets/images/unit-hand.png)

当你把各种单元类型——`service`、`path`、`timer`、`mount`、`socket` 等——组合起来时，你几乎可以把系统变成一个状态机。我在 NixOS 上就这么做过，这是一种强大的方式来建模相互依赖的服务管理。

像挂载点这样的系统配置用 `mount` 单元表达，可以让依赖网络挂载的守护进程正确排序。用 `path` 单元可以很容易地在文件变更时触发服务重启。`service` 单元可用的选项多得令人咋舌，几乎能满足你能想到的所有需求。说真的——你知道 `ConditionVirtualization=` 可以用来判断你是在 AWS 还是 Docker 环境下运行，从而决定是否运行某个单元吗？太神奇了。

###### 安全性

如果你写过不少 `.service` 单元，你就知道可用的加固选项多得惊人。已经有很多优秀的博客介绍这些选项，这里就不赘述了。

我个人的问题是**记不住这些选项**。你知道 systemd 还专门做了工具来帮你记吗？**每一个选项**都解释了你可以用来加固守护进程的运维安全好处，而且大多数情况下都很容易添加，不会破坏功能。这些都是利用能力机制的好方法。


```shell
systemd-analyze security polkit.service
```

```text
  NAME                                                        DESCRIPTION                                                                                         EXPOSURE
```

#### 黑粉酱与千禧年的恐怖

![pointing-at-systemd.png](https://blog.tjll.net/assets/images/pointing-at-systemd.png)

我写这篇文章的部分原因是我总能碰到[这样的讨论串][7]：

> 我以前以为 systemd 之所以成为默认并被大多数发行版采用，是因为它易用，而且一套工具包涵盖了很多功能，我能理解这种吸引力。但自从换到 artix openrc 后，我就不明白为什么他们要用 systemd，明明 openrc 作为 init 系统和服务管理都更好，systemd 套件的其他组件也都能被替换，为什么要这样做？

天哪。好吧，我尊重 `stvpidcvnt111111` 有权发表自己的看法，但我们不能让这种像平庸的屁一样的言论飘进像计算基础设施这样关键的领域。请把你的臭气**带走**。

我不打算和稻草人争论，但其实我还是要说：

> systemd 做得太多了。

你有没有想过，仅仅"回收旧进程 ID"对一个安全、健壮的 init 守护进程来说还**不够**？也许它还应该保护系统的其他部分，跟踪所需服务的存活状态？

> systemd 做得很烂

如果我看到这种观点，只能认为对方不是做软件工程的。只要你用过 `systemctl` 或 `journalctl`，就会有完全不同的体验。我甚至**从没听说过** systemd 在核心职责（启动、停止、管理守护进程）上失败过。

> systemd 太臃肿，什么都想做

以现代 systemd 的功能之多，我很惊讶它没有更多漏洞（是的，我知道 systemd 有 CVE）。我没有确切数据，但我很好奇"systemd 本身导致的漏洞"与"systemd 提供的服务沙箱机制阻止的漏洞"之间的差距。能轻松把可执行文件放进非特权环境是个很棒的特性。整个行业因为 systemd 降低了进程沙箱的门槛，几乎可以肯定变得更安全了。

没错，`systemd-boot` 和 `systemd-networkd` 做的是不同的事情。坦率地说，作为运维人员，得益于 systemd-* 项目的高质量软件，我的生活明显**更好了**，而且它们的配置方式也很统一。我也在底层集成过 systemd 的 API，所以这种听起来很吓人的"扩张"其实并不封闭。API 都在那儿！你可以用！

我一贯更喜欢用 systemd 相关的替代品，比如 `systemd-resolved` 和 `systemd-networkd`，因为它们最终更容易配置和使用。

> red hat 想通过 systemd 控制 linux 生态

![alex-jones.jpg](https://blog.tjll.net/assets/images/alex-jones.jpg)

这是真的。我简直不敢相信我们，**SYSTEMD 全球光明会**，居然被曝光了。

## 脚注

我知道 logrotate 能做很多智能的事情。但配置 journald 只需要"输出到 stdout，搞定"。

---

[人力资源协调问题][9]

---

[1]: https://blog.tjll.net/human-resources-misalignment/
[2]: https://en.wikipedia.org/wiki/TempleOS
[3]: https://www.freedesktop.org/software/systemd/man/latest/systemd-sysv-generator.html
[4]: https://dhall-lang.org/
[5]: https://mattinouye.com/defaults-matter
[7]: https://old.reddit.com/r/artixlinux/comments/1lc382o/why_is_systemd_the_default/
[9]: https://blog.tjll.net/human-resources-misalignment/ "Previous post: The Human Resources Alignment Problem"