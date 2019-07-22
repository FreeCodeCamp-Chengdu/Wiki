---
title: Deno 入门与 Chrome 性能调试
date: 2019-06-27 19:12:15
categories:
  - Activity
  - Salon
tags:
  - offline
  - Deno
  - Chrome
  - JavaScript
  - performance
toc: true
thumbnail: End-all.jpeg

# Activity meta
description: JavaScript 进阶技术分享会
start: 2019-06-30 13:30:00
end: 2019-06-30 17:30:00
address: 成都市高新区天益南巷18号创客大使馆
links:
  报名: https://jinshuju.net/f/Pgvh6G
mentors:
  - manyuanrong
  - TingYinHelen
workers:
  - TechQuery
  - TingYinHelen
partners:
  - CD-CKplus
---

> 2019 年 6 月 30 日 13:30 ~ 17:30
>
> 成都市高新区天益南巷 18 号[创客大使馆](/partner/cd-ckplus/)

**JavaScript 前后端**的小伙伴有没有被一天到晚填坑而累得没脾气呢？

想不想学点**高端武艺**来对 bug **降维打击**呢？

**FCC 成都社区**半月一次的小活动又来了！！！

## 主题简介

### 《Deno 初体验》

![](https://avatars1.githubusercontent.com/u/42048915?s=200&v=4)

让我们来了解 Node.js 之父 Ryan Dahl 的新“造物”Deno 背后的动机，来一起学一下别人“学不动的” Deno，从背景及语法再到简单 HTTP 服务入手，畅谈 Deno 的现在和未来。

#### 讲师简介

**满远荣**，前重庆优启科技架构师、前重庆奇燎科技 CTO，现 ThoughtWorks 切图仔。

Deno contributor；Deno 中国发起人、[Deno 中文社区][1]站长及发起人；[denolib 组织][2] Member；Deno 生态核心基础库作者和贡献者。

### 《Chrome DevTools 之 performance》

![](https://avatars3.githubusercontent.com/u/1778935?s=200&v=4)

从**浏览器渲染**到**动画性能调优**，让自己写的前端代码渲染性能可控。

#### 讲师简介

**Helen**，爱玩的程序媛，喜欢写代码、玩游戏、架子鼓、马拉松。

她也是 **FCC 成都社区**核心成员、[Vue Beauty 组件库][3]活跃开发者，并在 [Google Women TechMaker 2018（成都站）][4]、[第 0 届学生开源年会][5]等会议上发表演讲。

<!-- more -->

## 课前准备

### 安装 Deno

#### Linux、Mac OS X

在**命令行终端**执行以下命令：

```shell
curl -fsSL https://deno.land/x/install/install.sh | sh
```

#### Windows

在 PowerShell 中执行以下命令：

```powershell
iwr https://deno.land/x/install/install.ps1 | iex
```

#### Mac OS X

在**命令行终端**执行以下命令：

```shell
brew install deno
```

### 安装 Google Chrome

https://google.cn/chrome

## 参考资料

- [Deno 中文手册](https://nugine.github.io/deno-manual-cn/)
- [Deno 核心指南](https://github.com/denolib/guide/tree/master/chinese)
- [Deno 生态集锦](https://github.com/denolib/awesome-deno)

---

## 活动总结

由于本次活动的出品人水歌上周经历了重感冒、在公司项目写微信小程序的双重折磨，没能尽早**确定活动日期**并**联系场地方**，导致本次场地没有可用的 **HDMI 投影仪**、**智能电视**，只能临时用 AK 同学的私人 **Zoom 会议室**共享讲师屏幕……

但幸好大家一起把三张大桌子连起来，人手一台电脑地坐一起，反而拉近了**人与人的距离**，又找回 2016 ~ 17 年在 [@Too][6] 的【拾级咖啡】办活动时的那种感觉！

{% asset_img Share-all.jpeg %}

第一个主题 Deno 乍看 PPT 很简单，但随着满远荣老师的逐步发散，让在座很多 Web 工程师发现 Deno 独特设计的奥妙 ——

1. 小巧的**单可执行文件** —— 安装、部署非常简单
2. 开箱即用的 **TypeScript** 支持 —— 保持 JS 灵活的同时又强健
3. 内置最新 **Web API** —— 前端同构代码更好移植
4. **依赖包**一条 URL 搞定 —— 基于 ECMAScript 模块标准

虽然现在 **Deno 标准库**尚不完善，但正因为基于 [ES module][7]，一个常用功能官方是否提供，甚至有没有[官方软件源][8]，都已不再重要，Deno 完全变成了**分布式系统**。

尽管如此，官方还是希望维护一批由核心开发者维护的**高质量常用库**，满老师自己开发的 [SMTP 工具库][9]就是 Deno 创始人中意的之一，我们也有幸现场跟着老师从零写一遍这个库的核心代码，通过实践来体会 **SMTP 协议**和 Deno 的简洁。

{% asset_img Start-all.jpeg %}

紧接着的第二个主题由我们 **FCC 成都社区颜值担当**之一的 Helen 小姐姐讲解，几张言简意赅的图文 PPT 之后，便直接开始用 demo 程序演示如何用 **Chrome 调试器**来分析、优化**动画渲染性能**。

勤于思考的小伙伴们在讲完后提出了多个值得思考的问题，Helen 在会后的活动总结中做了[进一步解答][10]。

活动结束时，小伙伴们纷纷表示**收获颇丰**，但希望**动手实践环节**能再更易上手些，并通过一些**思考题**来现场编码，提升动手的参与度。这些建议我们会在和满老师一起准备下期 Deno 活动时充分采纳，敬请期待！

[1]: https://denocn.org/
[2]: https://github.com/denolib
[3]: https://fe-driver.github.io/vue-beauty/
[4]: https://www.meetup.com/Chengdu-GDG/events/249594885/
[5]: https://openingsource.org/3447/
[6]: https://too.github.io/
[7]: http://es6.ruanyifeng.com/#docs/module-loader
[8]: https://deno.land/x/
[9]: https://github.com/manyuanrong/deno-smtp
[10]: https://fcc-cd.tk/article/chrome-performance-summary/
