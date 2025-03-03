---
title: 我在2024年的计算机使用方式
date: 2024-07-31T00:00:00.000Z
authorURL: ""
originalURL: https://jnsgr.uk/2024/07/how-i-computer-in-2024/
translator: ""
reviewer: ""
---



<!-- more -->

# 我在 2024 年的计算机使用方式

·2611 字·13 分钟阅读时间

目录

-   [介绍]
-   [硬件]
    -   [台式机]
    -   [服务器]
    -   [笔记本电脑]
    -   [手机]
-   [连接与安全]
-   [生产力应用]
-   [开发]
-   [操作系统/桌面环境]
-   [服务器/家庭实验室]
-   [总结]

## 介绍

我一直对人们如何使用他们的计算机感到着迷——他们选择哪些应用程序，如何设置他们的桌面环境，甚至他们的屏幕在桌面上是如何布局的。多年来，我从朋友和同事那里学到了一些很棒的小技巧，所以我想写下我在 2024 年是如何使用我的机器的。

我目前使用的设置在过去几年中一直相当稳定，只有一些小的调整。每次我做出重大改变时，我都会至少保留几个月，以尝试建立肌肉记忆，并看看我是否会永久采用这种调整。


![screenshot of an empty hyprland desktop showing waybar at the top][134]

## Hardware 

### Desktop 

我的主力机器是一台定制的台式机。它装在一个低调的全黑色 [beQuiet Silent Base 600][21] 机箱中。我从来不喜欢 RGB 灯效——我更注重良好的散热和**静音**运行。具体配置如下：

**CPU**: [AMD Ryzen 9 7950X][22]  
**GPU**: [AMD Radeon RX 7900XT][23]  
**内存**: [G.SKILL Trident Z5 Neo RGB 64GB DDR5-6000][24]  
**电源**: [Corsair HX1000i][25]  
**硬盘**: [1TB SN850X][26] + [2TB SN850X][27]  
**主板**: [MSI MPG X670E CARBON WIFI][28]  
**散热器**: [beQuiet Dark Rock Pro 5][29]  
**机箱**: [beQuiet Silent Base 600][30]，配备 3 个 [beQuiet Silent Wings 4 PWM][31] 风扇  
**键盘**: [DURGOD Taurus K320 TKL][32]，Cherry MX 茶轴  
**鼠标**: [Razer Deathadder V2 Pro][33]  
**显示器**: [57 英寸三星 G95NC Odessey Neo G9][34]  
**摄像头**: [Sony ILME-FX3][35] / [FE 28-70mm F3.5-5.6][36] / [Elgato Cam Link 4K][37]  
**音箱**: [Audioengine A2+][38]  
**麦克风**: [RODE VideoMic GO II][39]  

在我的桌面上，你会看到一个 [57 英寸三星 G95NC Odessey Neo G9][40] 显示器，安装在一个 [气弹簧支架][41] 上，这是我最新的设备升级。过去 5 年，我一直使用一对 27 英寸的 [LG 27" UN850 4K][42] 显示器，安装在双显示器支架上，并一直考虑换成超宽屏显示器。三星显示器是我找到的第一款在分辨率上不妥协的显示器——它的分辨率与我之前的两台 LG 显示器相同，但集成在一块面板上。我必须承认，没有中间的分割线确实大大提升了我的工作效率。



![一张我的桌面的照片，包括一个巨大的超宽显示器][135]

在撰写本文时，我在 [Canonical][45] 工作，这是一家完全远程的公司。公司性质加上我作为工程副总裁的角色，意味着我每天有相当一部分时间都在视频会议中度过。在我看来，投资一套可靠的音视频设备是对同事的一种服务，尤其是当你的角色涉及管理他人时。我目前使用 [Sony ILME-FX3][46] 相机，搭配标准的 [FE 28-70mm F3.5-5.6][47] 镜头，并通过 [Elgato Cam Link 4K][48] 连接。音频方面，我使用 [RODE VideoMic GO II][49] 麦克风和一对 [Audioengine A2+][50] 音箱。

在 Linux 桌面环境中，我发现那些自带 USB 接口的音频设备使用起来更加方便。我完全禁用了主板上的板载声卡以及 HDMI/DisplayPort 音频输出，只保留了麦克风和音箱的 USB 音频接口。

### Server 

我使用"服务器"这个词比较宽松……我的家庭实验室在过去的几年里经历了多次迭代，从完全"本地化"到混合使用云服务和设备，然后又回到了本地。

我目前的设置非常简约，部分原因是我的工作站性能非常强大，这意味着我可以轻松地在上面启动多个虚拟机/容器进行实验，而不会影响机器在常规任务中的性能。

我的大部分服务都运行在一台 [Intel NUC6i7KYK][52] 上。这台机器配备了 Intel i7-6770HQ CPU、16GB 内存和一块 512GB 的三星 970 Pro NVMe 硬盘。它通过 USB 连接了一块三星 840 EVO 4TB SATA 硬盘。我不是一个数据囤积者，所以不需要太多存储空间。

这台机器已经有些老旧了，我考虑在今年晚些时候换一台更现代的机器。

### Laptop 

如果我不在办公桌前，那么我会使用我的 [Lenovo Z13 Gen 1][54]。我选择了 AMD Ryzen 7 Pro 6860Z 处理器、32GB 内存和高分辨率显示屏的配置。

我对这台机器的评价非常高。它的做工甚至超过了联想通常的标准——感觉非常高端，更像是苹果的一体式铝制笔记本电脑风格。它拥有足够的性能，在中等使用强度下，电池可以持续一整天。

我倾向于在旅行时使用超轻薄的机器，因为如果我需要更多的性能，我随时可以远程使用我的台式机（稍后会详细介绍……），而且我绝对不想尝试让集成/独立双显卡正常工作。

### Phone 

我随身携带一部 [Apple iPhone 15 Pro][56]。我从 2011 年左右开始使用 iPhone，可能短期内不会更换。我的家人都使用 iPhone（因此也使用 FaceTime），我喜欢苹果在便利性和隐私之间提供的特定权衡——尽管从绝对意义上来说，这可能存在缺陷。这款手机与我的 AirPods 配合得很好，相机比我更擅长拍照，电池续航时间也相当不错。

我给手机套上了 [Mous Limitless 5.0 Aramid Fibre][57] 保护壳，以避免太多意外时刻！

我发现现在很难对手机感到太兴奋，我更多地将其视为一种商品。

## Connectivity & Security 

2021 年，我开始使用 [Tailscale][59] 替代我手工搭建的 Wireguard 设置，并且再也没有回头。它必须是我有史以来最喜欢的技术之一。它运行在我所有的设备上——台式机、笔记本电脑、服务器、手机、平板电脑等。

我最近还利用了他们的 [与 Mullvad 的合作][60]。我在使用不受信任的网络时，已经使用 [Mullvad][61] 作为默认的 VPN 提供商几年了——但通过 Tailscale 使用它意味着我仍然可以访问我的 tailnet，同时我的互联网流量通过 Mullvad 出口，而无需任何额外配置。

我使用 [NextDNS][62] 作为运行 [Pi-Hole][63] 或类似工具的替代方案。Tailscale 有一个很好的 [集成][64]，这意味着我 tailnet 上的所有设备都会自动获得 [DNS-over-HTTPS][65]，而无需任何额外配置，同时还提供 DNS 级别的广告拦截。

我的 tailnet 中有几个共享节点——包括一个我的家人在旅行时可以使用的出口节点。因此，我广泛使用 Tailscale [ACL][66] 来确保人们只能访问我希望他们访问的内容。

我已经使用 [1Password][67] 超过十年了，我认为他们的产品非常棒。在我看来，他们的 Linux 应用程序为现代跨平台应用程序树立了标杆。我最近开始在 CLI 中使用他们的秘密功能——能够存储一个包含一堆无害秘密引用的 `.env` 文件，并将实际的 [秘密注入到环境][68] 或 [配置文件][69] 中，这非常方便。

我还拥有一小批不同接口的 [Yubikeys][70]。一个放在我的办公桌上，连接着我的台式机，另一个放在我的口袋里或随身携带，还有一个放在保险箱里。它们都支持 NFC，因此可以很好地与我的移动设备配合使用。我将 Yubikeys 配置为使用 ed25519 [常驻密钥][71] 进行 SSH，同时还存储了我的 GPG 密钥（虽然现在很少使用……）。

我最喜欢 Yubikey 的一点是它们能够存储 TOTP 代码。虽然每次添加新账户时都需要将新密钥添加到每个 Yubikey 上有点麻烦，但好处是我不用每次换手机时都去更新/转移它们！在桌面上运行 `ykman oath accounts code <name>` 也很方便。

## Productivity Apps 

我的大部分工作都是在浏览器中完成的。Canonical 使用 [Google Workspace][73] 来处理邮件、文档、幻灯片等，因此自从加入公司以来，我的默认模式是使用 Google Chrome 处理工作事务，而 Firefox 则用于个人事务。我知道 Firefox 有 [Account Containers][74] 和其他功能可以帮助区分两者，但我发现将工作和个人事务完全分开的浏览器使用方式非常有用。

在 [Droplet][75] 上运行了几年 [Nextcloud][76] 服务器后，我最终意识到我**只**使用了文件同步功能，而且即使如此，也没有移动太多数据。我转而使用 [Syncthing][77] 来避免运行服务器实例的开销，特别是与 Tailscale 结合使用时效果非常好。

我所有的笔记，无论是工作还是个人，都存放在 [Obsidian][78] 中。当我第一次发现 Obsidian 时，我陷入了安装**所有扩展**的经典陷阱，后来逐渐减少了使用。我曾经深入使用 [Dataview][79]，用它来整理我的笔记库中的各种任务，分为不同的类别（会议议程、个人、回顾等），但随着笔记库的增长，性能受到了很大影响。几个月前，我移除了 Dataview，对我的笔记进行了一些痛苦的整理（大量使用 `sed` 和 `grep`！），并重新使用 Obsidian 的 [查询][80] 功能。

我尝试过 [Zettelkasten][81] 方法，但发现维护起来有点……无聊？我最终采用了一个简单的结构，这在我的日常工作中确实帮助很大。每天都会有一个"每日笔记"，其中包括我的议程，链接到与相关人员或定期会议的笔记。每日笔记也是我整理零散笔记的地方，这些笔记可能会在以后被搜索到：


![obsidian.md 截图显示我的每日笔记模板][136]

议程和链接是通过我编写的一个小型 Go 应用程序自动生成的——这个应用程序会抓取我的 Google 日历，并根据一些规则和它对笔记库的了解，生成 Markdown 格式的议程并复制到剪贴板。每天，我坐下来在命令行中输入 `agenda`，然后将其粘贴到 Obsidian 中。每个人的笔记都包含我与该人或小组的按日期记录的笔记。

我使用了一些 Obsidian 插件来帮助完成这些工作——包括 [Templater][84]、[QuickAdd][85]、[Omnisearch][86] 和 [Linter][87]。前两个插件特别方便，可以快速插入常见的会议议程、面试问题集、操作手册等。

我不再在 Obsidian 中跟踪任务，而是从去年年底开始使用 [Todoist][88]。Todoist 非常棒——我喜欢为每个人保留一个任务列表，这样当我下次与他们进行一对一会议或其他会议时，我可以快速参考所有需要与他们讨论的事情——而且我可以很容易地通过 Todoist 的标签功能实现这一点。Obsidian 的集成意味着我可以将议程与特定人员的会议笔记整合在一起：



![obsidian and todoist side-by-side showing the integration][137]

## 开发环境 

我长期使用 [Alacritty][92] 作为终端模拟器。在桌面上，我主要使用 [Visual Studio Code][93]——我喜欢它的插件和主题等社区支持。我也非常擅长使用 vim——我仍然保留着一个相当炫酷的 [Neovim][94] 配置，每当我在终端时都会使用它。你可以在 Github 上看到我的 [neovim 配置][95]——我没有安装太多插件，但我逐渐喜欢上了 [lightline][96]、[telescope][97] 和 [nvim-tree-lua][98]。



![Alacritty 终端模拟器显示加载了 Neovim 的 tmux 会话][138]

我主要通过命令行使用 `git`，但最近我开始使用 [Sublime Merge][101] 来处理复杂的 rebase 操作，或者当我想暂存文件中的多个小块时。我曾经是 [Sublime Text][102] 的忠实用户，但后来觉得它在功能上逐渐落后于 Visual Studio Code——尽管我仍然对 Sublime Text 的极速体验有些上瘾。



![Visual Studio Code 和 Sublime Merge 并排显示][139]

## OS / Desktop 

如果你之前读过我的博客，那么我对 NixOS 的全情投入应该不会让你感到意外。我大约两年前开始了这段旅程，并且再也没有回头。多年来，我在 Linux 桌面上的经历相当多样化：我最早的 Linux 桌面体验是在 2003 年使用 [Knoppix][106]。然后我花了几年时间尝试各种 Ubuntu 版本，直到 2014 年左右开始全职在桌面上使用 Linux。从那以后，我在 Arch Linux 上度过了几年，大约每 12 个月就会在 Plasma 和 GNOME 之间切换。

我已经成为一个相当专注的平铺窗口管理器用户，尽管我承认在它真正"粘住"我之前，我几次放弃了它。当我在 2021 年切换到 [Sway][107] 时，某些东西突然"点击"了，从那以后我再也没有离开过平铺窗口管理器。我在各种配置下坚持使用 Sway 相当长一段时间，直到大约 15 个月前，我切换到了一个看起来几乎相同的基于 [Hyprland][108] 的设置。Hyprland 看起来不错——它大部分时间都很稳定，而且我喜欢它的视觉效果。

所有东西都使用了 [Catppuccin Macchiato][109] 主题。我不仅喜欢这个主题，还喜欢它在我使用的所有应用程序/工具中的普遍性——我对一致性非常着迷！

你可以在 Github 上看到我的 Hyprland、waybar、rofi、mako 等的所有细节 [on Github][110]。



[Hyprland 桌面的截图，显示了编辑器、浏览器等][141]

## Server / Homelab [#][113]

我的服务器也运行着 NixOS，上面部署了一系列媒体服务和实用工具。尽可能的情况下，所有服务都以"原生"NixOS 模块的形式运行，有些则使用内置的 [NixOS 容器][115] 语言运行在 [systemd-nspawn][114] 容器中。如果我在实验新服务，有时会先用 Docker 运行，特别是当还没有现成的 NixOS 模块时，我需要决定是否值得花时间编写一个！

我使用 [Caddy][116] 作为反向代理（最近从 Traefik 切换过来）。它可以 [直接与 Tailscale 守护进程通信][118]，为你的 tailnet 上的设备颁发 LetsEncrypt 证书。这个 Caddy 实例充当所有运行在服务器上的服务的反向代理，同时还包括我家庭局域网中的一些其他服务，所有通信都通过 TLS 加密。我通常通过 [Homepage][119] 访问这些服务（我之前 [写过相关博客][120]）：



![使用 gethomepage.dev 的个人仪表盘][140]

每天晚上，我的 iCloud 照片库的内容会通过 [icloud-photos-downloader][123] 进行备份，这样如果我的 iCloud 账户发生任何意外，我仍然可以在本地保留一份照片的备份。

它还运行着一个 [Home Assistant][124] 实例，用于管理我房子里的地暖和太阳能逆变器的自定义集成（仍在进行中！）。我还没有花足够的时间在 Home Assistant 上，但我计划在未来几个月内更好地设置它。我最近搬到了一栋有很多"智能"设备的房子，我希望将所有设备的控制集中到一个应用程序中。一旦我的实验完成，我可能会将 Home Assistant 部署迁移到一个专用的低功耗设备上。

这台机器的数据每晚都会备份到 [Borgbase][125]。如上所述，我使用 [Syncthing][126] 来同步文件，并将这台服务器配置为所有同步目录的"仅接收"目标。这意味着我的数据始终至少存在于三个地方：我的台式机或笔记本电脑、我的服务器，以及备份到 Borgbase。有时我会使用 [Files][127] 临时访问未同步到某台机器的文件，这是一个外观漂亮、单 PHP 文件的文件画廊。我一直想用**非 PHP** 的东西替换它，但至今还没有找到一个在简洁性和用户体验上更吸引人的替代品。

## **总结**

我不知道有多少人对别人如何使用他们的计算机感兴趣——但我希望你喜欢这篇文章。如果你觉得我可以做得更好，或者你有一个杀手级应用推荐给我，请随时联系我！


作者
Jon Seager
丈夫、父亲、领导者、软件工程师、极客。

---

[←→ 使用 LXD 和 Multipass 的工作站虚拟机 2024 年 6 月 25 日][133]

[3]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/
[22]: https://www.amd.com/en/products/processors/desktops/ryzen/7000-series/amd-ryzen-9-7950x.html
[23]: https://www.xfxforce.com/shop/xfx-speedster-merc310-7900xt
[24]: https://www.gskill.com/product/165/390/1665020865/F5-6000J3040G32GX2-TZ5NR
[25]: https://www.corsair.com/uk/en/p/psu/cp-9020259-uk/hx1000i-fully-modular-ultra-low-noise-platinum-atx-1000-watt-pc-power-supply-cp-9020259-uk
[26]: https://www.westerndigital.com/products/internal-drives/wd-black-sn850x-nvme-ssd?sku=WDS100T2X0E
[27]: https://www.westerndigital.com/products/internal-drives/wd-black-sn850x-nvme-ssd?sku=WDS200T2X0E
[28]: https://www.msi.com/Motherboard/MPG-X670E-CARBON-WIFI
[29]: https://www.bequiet.com/en/cpucooler/4466
[30]: https://www.bequiet.com/en/case/1501
[31]: https://www.bequiet.com/en/casefans/3703
[32]: https://www.durgod.com/product/k320-space-gray/
[33]: https://www.razer.com/ap-en/gaming-mice/razer-deathadder-v2-pro
[40]: https://www.samsung.com/uk/monitors/gaming/odyssey-neo-g9-g95nc-57-inch-240hz-curved-dual-uhd-ls57cg952nuxxu/
[41]: https://www.amazon.co.uk/gp/product/B0B73XXDP5/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1
[42]: https://www.lg.com/us/monitors/lg-27un850-w-4k-uhd-led-monitor
[44]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/01.png
[45]: https://canonical.com
[46]: https://www.sony.co.uk/interchangeable-lens-cameras/products/ilme-fx3-body---kit
[47]: https://www.sony.co.uk/electronics/camera-lenses/sel2870
[48]: https://www.elgato.com/uk/en/p/cam-link-4k
[49]: https://rode.com/en/microphones/on-camera/videomic-go-ii
[50]: https://audioengineeu.com/products/audioengine-a2-wireless-bluetooth-computer-speakers-60w-bluetooth-speaker-system-for-home-studio-gaming
[52]: https://ark.intel.com/content/www/us/en/ark/products/89187/intel-nuc-kit-nuc6i7kyk.html
[54]: https://www.lenovo.com/gb/en/p/laptops/thinkpad/thinkpadz/thinkpad-z13-%2813-inch-amd%29/len101t0036?srsltid=AfmBOor-8ic5yZrW3rlDXTTRwK8r05y-gjCpJK04fA4qtote0u2HZ7I6
[56]: https://www.apple.com/uk/iphone-15-pro/
[57]: https://uk.mous.co/products/limitless-5-0-magsafe-compatible-phone-case-aramid_fibre
[58]: #connectivity--security
[59]: https://tailscale.com/
[60]: https://tailscale.com/kb/1258/mullvad-exit-nodes
[61]: https://mullvad.net/en
[62]: https://nextdns.io/
[63]: https://pi-hole.net/
[64]: https://tailscale.com/kb/1218/nextdns
[65]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[66]: https://tailscale.com/kb/1018/acls
[67]: https://1password.com/
[68]: https://developer.1password.com/docs/cli/secrets-scripts
[69]: https://developer.1password.com/docs/cli/secrets-config-files
[70]: https://www.yubico.com/products/yubikey-5-overview/
[71]: https://developers.yubico.com/SSH/Securing_git_with_SSH_and_FIDO2.html
[72]: #productivity-apps
[73]: https://workspace.google.com/intl/en_uk/
[74]: https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/
[75]: https://nextcloud.com/
[76]: https://github.com/jnsgruk/nextcloud-docker-compose
[78]: https://obsidian.md/
[79]: https://blacksmithgu.github.io/obsidian-dataview/
[80]: https://publish.obsidian.md/tasks/Queries/About+Queries
[81]: https://zettelkasten.de/introduction/
[83]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/02.png
[84]: https://github.com/SilentVoid13/Templater
[85]: https://github.com/chhoumann/quickadd
[86]: https://github.com/scambier/obsidian-omnisearch
[87]: https://github.com/platers/obsidian-linter
[88]: https://todoist.com/
[90]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/03.png
[92]: https://alacritty.org/
[93]: https://code.visualstudio.com/
[94]: https://neovim.io/
[95]: https://github.com/jnsgruk/nixos-config/blob/main/home/common/shell/vim.nix
[96]: https://github.com/itchyny/lightline.vim
[97]: https://github.com/nvim-telescope/telescope.nvim
[98]: https://github.com/nvim-tree/nvim-tree.lua
[100]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/04.png
[101]: https://www.sublimemerge.com/
[102]: https://www.sublimetext.com/
[104]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/05.png
[106]: https://www.knopper.net/knoppix/index-en.html
[107]: https://swaywm.org/
[108]: https://hyprland.org/
[109]: https://github.com/catppuccin/catppuccin
[110]: https://github.com/jnsgruk/nixos-config
[112]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/07.png
[114]: https://www.freedesktop.org/software/systemd/man/latest/systemd-nspawn.html
[115]: https://nixos.wiki/wiki/NixOS_Containers
[116]: https://caddyserver.com/
[117]: https://github.com/jnsgruk/nixos-config/commit/dffac1dc0635f377865bcdfc2349387d41fc965d
[118]: https://tailscale.com/kb/1190/caddy-certificates
[119]: https://gethomepage.dev/latest/
[120]: https://jnsgr.uk/2024/03/a-homelab-dashboard-for-nixos/
[122]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/08.png
[123]: https://github.com/icloud-photos-downloader/icloud_photos_downloader
[124]: https://www.home-assistant.io/
[125]: https://www.borgbase.com/
[126]: https://syncthing.net/
[127]: https://www.files.gallery/
[129]: https://github.com/jnsgruk
[130]: https://hachyderm.io/@jnsgruk
[131]: https://linkedin.com/in/jnsgruk
[132]: https://t.me/jnsgruk
[133]: https://jnsgr.uk/2024/06/desktop-vms-lxd-multipass/
[134]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/06_hu19d68ad61b5dede6fd1f1d19283c6ff2_18899216_660x0_resize_box_3.png
[135]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/01_hu930cbab5a5dfad6c7189f290c5bed7dd_1798110_660x0_resize_box_3.png
[136]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/02_huf93b0c707cf46312a8fd9228e1b4cdb5_92067_660x0_resize_box_3.png
[137]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/03_hu961bfa5d6384c4efa7dfa90c87eca21a_361619_660x0_resize_box_3.png
[138]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/04_hu05bf6628177fb41deef80d4a55f42632_404501_660x0_resize_box_3.png
[139]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/05_hu2d6d25863bc408587fe8518d526b1cf7_752958_660x0_resize_box_3.png
[140]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/08_hu7f790721995065d374827ee567548c83_94865_660x0_resize_box_3.png
[141]: https://jnsgr.uk/2024/07/how-i-computer-in-2024/07_hu19d68ad61b5dede6fd1f1d19283c6ff2_2075377_660x0_resize_box_3.png
