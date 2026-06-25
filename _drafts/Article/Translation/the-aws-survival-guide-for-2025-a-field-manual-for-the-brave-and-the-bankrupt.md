---
title: "2025年 AWS 生存指南：勇敢者和破产者的实战手册"
date: 2025-07-09T00:00:00.000Z
author: Corey Quinn
authorURL: https://www.lastweekinaws.com/blog/author/cquinn/
originalURL: https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/
translator: ""
reviewer: ""
---

欢迎，勇敢的云探索者！你决定在 2025 年踏入 AWS 这片丛林，这里的服务增长速度比你的月账单还要快。忘掉那些老掉牙的服务，比如 S3、EC2 和 RDS……

## 第一章：服务名称生成器

首先，你需要理解现代 AWS 服务的命名方式。他们显然请了一支拼字游戏冠军团队，这些人喝了太多咖啡，在早期就把所有好名字都用完了。需要无服务器 AI 驱动的量子计算吗？那就是 AWS QuantumLambdaForgeMaxProUltra™（但他们简称`Amazon Q`）。想部署一个简单的 API？你需要 AWS HyperGatewayMeshFabricOrchestrator360™。

专业提示：如果服务名称听起来不像一个被淘汰的变形金刚，那它可能已经被弃用了，我当然是指`谷歌产品`。

## 第二章：文档迷宫

2025 年的 AWS 文档是一种"沉浸式体验"，就像被折磨一样。把它想象成一个密室逃脱游戏，奖品就是理解这个服务到底是做什么的。每个文档页面包含：

-   47 个架构图，看起来像有人把一堆廉价剪贴画塞进鼻子里，然后在电路板上打了个喷嚏
-   3 个相互矛盾的`入门指南`，它们之间差异巨大，假设你要么已经读过其他 2 个，要么至少在 AWS 服务团队工作了 5 年
-   1 个在 2023 年有效的例子，为一个人准备的，为了赶 re:Invent 截止日期而匆忙完成
-   0 个价格解释。（如果你觉得我在胡说八道，告诉我 Aurora DSQL Compute DPU 在现实世界中代表什么。我等着。）

记住：如果你第一次阅读就能理解文档，那你一定是在看错误服务的文档。如果这能给你任何安慰，你现在看的这个服务几乎肯定运行容器。

## 第三章：IAM 策略——黑暗艺术

编写 IAM 策略就像蒙着眼睛骑独轮车下四维象棋。在 2025 年，你需要权限来请求查看你所需权限的权限。最小权限原则已经演变成了"祝你好运，搞清楚为什么这不起作用，撞墙撞到放弃并允许*"的原则。

现代 IAM 策略示例：
```json
{
  "Effect": "Deny",
  "Action": "Everything:YouActuallyNeed",
  "Resource": "\*",
  "Condition": {
    "StringEquals": {
      "aws:PleaseWork": "false"
    }
  }
}
```

## 第四章：计费仪表板——恐怖故事

打开你的 AWS 账单就像打开了潘多拉魔盒的现代版本。你会发现以下费用：

-   你从未听说过的服务
-   你念不出来的地区——也被称为 _贾西先生的地理课_
-   `AWS ThoughtAboutProcessingYourRequest`费用
-   来自你噩梦的数据出口

成本浏览器现在需要自己的成本浏览器来理解为什么成本浏览器本身成本这么高。令人讨厌的是，尽管资源浏览器做了平庸的努力来填补这个空白，但仍然没有对你的账户中所有内容的统一概览。

## 第五章：re:Invent公告

每次 re:Invent（_re:Invent：感恩节后的一周，因为我们讨厌我们的家人，觉得你也是_™），AWS 都会宣布 73 个新服务，它们都做同一件事的略微不同版本。到 2025 年，有[51 种在 AWS 上运行容器的方法][27]，每种都有自己的定价模型，足以让衍生品交易员哭泣。你需要在以下选项中选择：

-   AWS ContainerThing
-   AWS KubernetesButSlightlyDifferent
-   AWS DockerWithExtraSteps
-   Amazon JustUseOurManagedEverything
-   AWS ContainerContainerContainer
-   Amazon ScrewItJustUseLambda

## 第六章：支持——神话生物

2025 年的 AWS 支持就像通过谷歌翻译玩的传话游戏。你关于网络连接的简单问题会导致为期 3 周的邮件往来，讨论秘鲁的香蕉进口法规。AWS 支持团队的人员都很棒和珍贵（说真的，他们令人难以置信），这就是为什么 AWS 采取了重要措施来阻止对他们的任何访问，除非首先通过无用的 GenAI 的考验。这是为了测试你的毅力，确保你决心解决问题，而不是只是空希望服务能做文档说它应该做的事情。

### 支持等级指南

-   基础版：想法和祈祷
-   开发者版：自动回复，指责你的代码
-   商业版：从未使用过 AWS 的真实人类
-   企业入门版：AWS 学会了每次只摘取你一个肾脏
-   企业版：Andy Jassy 亲自忽略你的工单
-   胡说八道危机：你现在正在读这个

## 第七章：认证跑步机

AWS 现在发布新认证的速度比你获得它们的速度还要快。当你通过 `AWS 认证量子区块链解决方案架构师——助理级 3.5 测试版` 时，它已经过时了，因为他们的测试合作伙伴 Pearson Vue 纯粹出于恶意，从折磨考生中获得企业满足感。你的 LinkedIn 个人资料将需要自己的 CDN 来托管你所有的认证徽章，但他们也忙着用乏味的 GenAI 填充_那个_产品。

## 生存技巧

1.  **像准备世界末日一样做预算**——因为你的 AWS 账单可能会引发一个
2.  **学会爱缩写词**——你现在的生活是 EKS、ECS、ECR、EMR、EBS、EFS 和哭泣
3.  **拥抱混乱**——如果某件事第一次就成功，你肯定做错了
4.  **把心理治疗师设为快速拨号**——最好是接受 AWS 积分付款的；你可以在 AWS 市场中找到他们
5.  **记住黄金法则**——永远是 DNS。即使不是 DNS，也是 DNS。这是一个数据库。

## 后记：前进的道路

恭喜！你现在准备好开始你的 AWS 之旅了。记住，每个专家都曾经是初学者，他们想知道为什么他们的[`简单`WordPress 网站][28]每月花费 3000 美元运行。

愿你的 lambda 函数保持温暖，你的区域保持接近，你的账单……嗯，让我们只关注前两个。

_免责声明：本指南对使用 AWS 导致的任何情感伤害、财务破产或存在危机不承担责任。副作用可能包括：强迫性仪表板刷新、关于级联故障的噩梦，以及对 `数据传输成本`这个词的非理性恐惧。_

![Corey Quinn Headshot](https://www.lastweekinaws.com/wp-content/uploads/2019/09/Corey_Quinn_2019_02-150x150.jpg)

作者 Corey Quinn

Corey 是 The Duckbill Group 的首席云经济学家，他专门帮助公司通过减少和降低 AWS 账单的可怕程度来改善账单。他还主持 `Screaming in the Cloud` 和 `AWS Morning Brief` 播客；并策划 `Last Week in AWS`，这是一个每周通讯，总结 AWS 新闻、博客和工具的最新动态，大致同等程度地融入了尖刻评论和深思熟虑的分析。

## Corey 的更多文章

![](https://www.lastweekinaws.com/wp-content/uploads/2025/06/pexels-olly-3760809-350x200.jpg)

### [AWS 证书管理器宣布可导出 TLS 证书，我基本可以接受][31]

[作者 Corey Quinn][32]

我认为说免费的 TLS 证书服务如 Let's Encrypt 和 AWS 证书管理器已经将加密连接推向主流并不为过。

[阅读更多关于 AWS 证书管理器宣布可导出 TLS 证书，我基本可以接受][33]

![](https://www.lastweekinaws.com/wp-content/uploads/2025/06/pexels-lilartsy-1925536-350x200.jpg)

### [服务器#47B-2 的一天：AWS 数据中心回忆录][35]

[作者 Corey Quinn][36]

或者：惊喜！我包含多重性，还有你的 `无服务器` 函数

[阅读更多关于服务器#47B-2 的一天：AWS 数据中心回忆录][37]

![Photo by Pei Peng on Unsplash](https://www.lastweekinaws.com/wp-content/uploads/2025/05/pei-peng-3DJfbRW__Fo-unsplash-350x200.png)


### [云服务回迁变得复杂][39]

[作者 Corey Quinn][40]

五年前，我相当有信心地表示云服务回迁不是一件事，我大体上坚持我写的观点。也就是说，现在是 2025 年，情况有所变化。

[阅读更多关于云服务回迁正变得复杂][41]

![Billie Holding Mail Email Subscribe Icon](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/blocks/billie-newsletter.svg)

## 获取通讯！

了解最新的 AWS 新闻、观点和工具，都带着一点尖刻评论的亲切点缀。

云计算行业太过严肃了。我们要让它变得有趣起来。

[请降低我的 AWS 账单！][42]

-   [通讯][47]
-   [播客][48]
-   [博客][49]
-   [商品][50]
-   [贡献][51]

-   [关于][52]
-   [联系][53]
-   [赞助][54]
-   [披露][55]


© 2025 The Duckbill Group. All Rights Reserved.


[3]: https://www.lastweekinaws.com
[4]: https://duckbillgroup.com
[5]: https://www.lastweekinaws.com/about/
[6]: https://www.lastweekinaws.com/community/
[7]: https://www.lastweekinaws.com/contact/
[8]: https://www.lastweekinaws.com/contribute/
[9]: https://www.lastweekinaws.com/blog/
[10]: https://www.lastweekinaws.com/newsletter/
[11]: https://www.lastweekinaws.com/podcast/
[12]: https://www.lastweekinaws.com/podcast/aws-morning-brief/
[13]: https://www.lastweekinaws.com/podcast/screaming-in-the-cloud/
[14]: https://www.lastweekinaws.com/nominate-a-guest/
[15]: https://store.lastweekinaws.com/collections/all
[16]: https://www.lastweekinaws.com/resources/
[17]: https://awsnetwork.lastweekinaws.com/
[18]: https://www.lastweekinaws.com/sponsorship/
[19]: https://www.lastweekinaws.com/blog/author/cquinn/
[20]: https://www.facebook.com/sharer/sharer.php?u=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/&display=popup&ref=plugin&src=share_button "Share on Facebook"
[21]: https://twitter.com/share?url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/&text=The%20AWS%20Survival%20Guide%20for%202025%3A%20A%20Field%20Manual%20for%20the%20Brave%20and%20the%C2%A0Bankrupt "Share on Twitter"
[22]: https://www.linkedin.com/shareArticle?mini=true&url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/ "Share on LinkedIn"
[23]: https://www.reddit.com/submit?url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/ "Share on Reddit"
[24]: https://www.lastweekinaws.com/
[25]: https://www.lastweekinaws.com/blog/
[26]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[27]: https://www.lastweekinaws.com/blog/17-final-ways-to-run-containers/
[28]: https://bsky.app/profile/quinnypig.com/post/3lqfctagyue2y
[29]: https://www.lastweekinaws.com/blog/
[30]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[31]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[32]: https://www.lastweekinaws.com/blog/author/cquinn/
[33]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[34]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[35]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[36]: https://www.lastweekinaws.com/blog/author/cquinn/
[37]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[38]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[39]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[40]: https://www.lastweekinaws.com/blog/author/cquinn/
[41]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[42]: https://www.duckbillgroup.com/
[43]: https://twitter.com/LastWeekinAWS
[44]: https://www.linkedin.com/company/last-week-in-aws/
[45]: https://www.lastweekinaws.com/feed/
[46]: https://www.lastweekinaws.com/community
[47]: https://www.lastweekinaws.com/newsletter/
[48]: https://www.lastweekinaws.com/podcast/
[49]: https://www.lastweekinaws.com/blog/
[50]: https://www.lastweekinaws.com/products/
[51]: https://www.lastweekinaws.com/contribute/
[52]: https://www.lastweekinaws.com/about/
[53]: https://www.lastweekinaws.com/contact/
[54]: https://www.lastweekinaws.com/sponsorship/
[55]: https://www.lastweekinaws.com/disclosures/
[56]: https://www.lastweekinaws.com/privacy-policy/
[57]: https://www.lastweekinaws.com/cookie-policy/