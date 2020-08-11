---
title: 【公益】高考志愿填报助手
date: 2020-07-27 14:31:38
authors:
  - TechQuery
categories:
  - Article
  - Propaganda
tags:
  - NGO
  - college
  - PWA
toc: true
original: https://github.com/freeCodeCamp-China/activity/issues/11
thumbnail: https://img14.360buyimg.com/n1/jfs/t1/119147/20/5048/404108/5eb260c6E8e88fd49/94ddfbc9426f974e.jpg
---

2020 年高考是一场非常不容易的高考，毕业生和社会各界都历经艰辛，希望我们的拼搏都能有更加美好的明天。

说起**填志愿**，回想起我高考时正值汶川大地震。那时我们获取**可靠、有效、全面的志愿参考信息**，基本只能通过学校发的一本大部头参考书，里面汇集了**全国招生的院校、专业和往年录取数据**。但这样一本像过去每个城市的电话黄页一样的书，**反复查阅它的“低效”痛苦**就不用多说了。

<figure>
    <img src="https://img14.360buyimg.com/n1/jfs/t1/119147/20/5048/404108/5eb260c6E8e88fd49/94ddfbc9426f974e.jpg">
</figure>

进入[大学 IT 技术社团][1]之后，就一直想用自己学到的编程技术，来为学弟学妹解决我当年那般的痛苦。但彼时，我国**政府部门信息公开**政策的保守和技术的落后，让我们很难获得好用的数据，商业平台又有巨大的封闭壁垒，遂作罢……

直到前一阵子，FCC 一位城市社区组织者在群里分享了[黄希彤][2]老师的[填教授][3]公益项目，看到他**把多年收集的公开数据用统计学方法进一步处理**，免费为广大考生做参考，内心十分激动！于是想尽自己现在的专长所能，帮黄老师优化一下前端界面，让广大的考生和家长用起来更方便。

<!-- more -->

同时黄老师也欣然接受了我一直以来的**开源直播**方式，于是便有了 [FCC 中文社区 B 站直播间][4]长期栏目[《水歌酱的开源日常》][5]的首期特别节目[《编程帮我填志愿》][6]，也欢迎大家关注 [FCC B 站账号][7]以接收日常节目的开播提醒~

## 使用入门

### 基本操作

<figure>
{% asset_img PWA-3.jpg %}
</figure>

1. 选择考生所在**省份**、**分科**后，再填入**高考成绩**，点击查询按钮即可查询到**根据往年录取情况有可能考上的学校和专业**
2. 可以下拉选择不同的**上线概率**后重新查询，来筛选相对保险（高概率、保底）的专业和相对有挑战（低概率）的专业
3. 对于公布**位次**的省份，可以选择按照位次来查询筛选专业
4. 筛选后页面出现过滤按钮，可以输入感兴趣的学校和专业**关键字**在结果中进行进一步的筛选
5. 每年招生情况都会发生变化，因此考生在查询后可点击**学校、专业名称的链接**，进一步了解相关专业当年的招生信息

### 安装为 App

<figure>
{% asset_img PWA-0.jpg %}
<figcaption>现代浏览器首次访问自动提示安装</figcatpion>
</figure>

<figure>
{% asset_img PWA-1.jpg %}
<figcaption>确认安装到桌面</figcatpion>
</figure>

<figure>
{% asset_img PWA-2.jpg %}
<figcaption>已安装到桌面</figcatpion>
</figure>

<figure>
{% asset_img PWA-4.jpg %}
<figcaption>App 在系统中独立运行</figcatpion>
</figure>

## 技术知识

为了满足黄老师提出的“除 CSS 样式外，简单业务不依赖任何库”要求，[水歌][8]本次对[填教授 Web 前端代码][9]的重构采用了最新版 **BootStrap**、**DOM API**、**JavaScript 标准**，刚学完 **freeCodeCamp HTML、CSS、JS 基础课程**的菜鸟也能快速上手~

以下是这些**易学、易用的标准、通用技术**的入门文档：

### HTML 标签、属性

1. [`tabindex` 全局属性](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/tabindex)
2. [`hidden` 全局属性](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/hidden)
3. [`iframe` 标签的 `name` 属性](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe#%E5%B1%9E%E6%80%A7)
4. [`template` 标签](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/template)

### CSS

1.  [`:empty` 伪类选择符](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:empty)

### DOM API

1. [CSS 选择器 API](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Locating_DOM_elements_using_selectors)
2. [`document.forms`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/forms)
3. [`ParentNode` 接口](https://developer.mozilla.org/zh-CN/docs/Web/API/ParentNode)
4. [`ChildNode` 接口](https://developer.mozilla.org/zh-CN/docs/Web/API/ChildNode)
5. [`Element.prototype.classList`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList)
6. [`HTMLFormElement.prototype.elements`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLFormElement/elements)

### BOM API

1. [`fetch()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)
2. [`URL()`](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/URL)

### ECMAScript API、语法

1. [`Array.prototype.map()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
2. [`Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
3. [ECMAScript 6 模板字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/template_strings)
4. [`with` 语句的利弊](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)
5. [`async` 函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

### 第三方开源库、云服务

1. [BootStrap 4.5](https://getbootstrap.com/)
2. [API Polyfill 自动补丁服务](https://polyfill.io/)
3. [CodeSandbox 在线 Web 前端项目沙盒](https://codesandbox.io/)
4. [Parcel 零配置打包器](https://parceljs.org/)

### 参考文档

以上知识点在 [FCC 成都社区](https://fcc-cd.dev/)之前的技术博文有介绍：

1. [《ECMAScript + DOM 骚操作》](https://mp.weixin.qq.com/s/Yyl27IShg6jhiuW3yvjycg)
2. [《JavaScript 效率工具》](https://mp.weixin.qq.com/s/i7JJtDg6zUvwoGi05FrEGw)
3. [《如何用开源软件办一场技术大会？》](https://mp.weixin.qq.com/s/hxCwiokl4uPXJscTQi42-A)

[1]: https://www.fyscu.com/
[2]: https://cloud.tencent.com/tvp/member/157
[3]: https://professortian.net/
[4]: https://live.bilibili.com/22218677
[5]: https://github.com/freeCodeCamp-China/activity/issues/5
[6]: https://mp.weixin.qq.com/s/JX3aHGXIWCTrZho-O2XSaA
[7]: https://space.bilibili.com/335505768/
[8]: https://github.com/TechQuery
[9]: https://github.com/stonelf/China-college-application
