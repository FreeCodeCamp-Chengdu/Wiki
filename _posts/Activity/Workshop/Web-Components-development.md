---
title: 【工作坊】Web 标准组件开发
date: 2019-12-16 18:08:27
categories:
  - Activity
  - Workshop
tags:
  - Web
  - component
  - WebCell
  - BootStrap
toc: true
thumbnail: Web-Component-Cell.png

start: 2019-12-21 13:30:00
end: 2019-12-21 17:30:00
address: 成都市高新区天益南巷18号创客大使馆
description:
links:
  报名: http://fcc-chengdu.mikecrm.com/35zDvFg
mentors:
  - TechQuery
workers:
  - demongodYY
  - zhangyanling77
  - Akagilnc
partners:
  - CD-CKplus
---

正在筹划本期工作坊时，就看到 [FCC 成都社区][1]的微信群里有小伙伴在苦恼**前端技能提升**的问题，也有小伙伴想一起开发一个组件库，那你们这次可来对了~

本期工作坊是[水歌][2]在 2019 年 [Google DevFest][3] 成都站[《Web 组件标准实践》][4]演讲后的首次配套工作坊，可以错过上次，但一定不要错过这次！

> 【时间】2019 年 12 月 21 日（周六）13:30 ~ 17:30
>
> 【地点】成都市高新区天益南巷 18 号创客大使馆

<!-- more -->

## 实践收获

1. 学习 **Web Components 标准原生 API** 的基本用法

2. 了解 **WebCell** 如何基于 TypeScript、JSX、Parcel 简化 **Web 组件**开发

3. 亲手封装一个基于 BootStrap 的**通用 Web 组件**，现场**为开源项目做贡献**

## 实践路线图

### Step 0 | Web Components 标准简介

<figure>
{% asset_img Web-Component-evolution.png %}
</figure>

[Web Components][5] 是一套浏览器提供的新标准 API，用于实现**框架无关的 Web 组件**。其官方 polyfill 补丁支持 IE 11+，[Polymer][6]、Angular、[Ionic Stencil][7] 等国际 Web 前端框架已全面应用。

### Step 1 | 手写一个 Web 组件

只用 [Web Components API][8] 来实现一个简单的组件。

<iframe
  style="width: 100%" height="600" frameborder="no"
  scrolling="no" allowtransparency="true" allowfullscreen="true"
  src="https://codepen.io/tech_query/embed/jONqOzj/?height=600&amp;theme-id=31315&amp;default-tab=html,result">
</iframe>

### Step 2 | 用 WebCell 简化组件

<figure>
  <img src="https://web-cell.dev/WebCell-1.fb612fdb.png">
</figure>

[WebCell][9] 是一个基于 Web Components API 的**轻量级组件引擎**，在保留 Web Components 核心写法的同时，基于 TypeScript、JSX、[Parcel][10] 等成熟技术，进一步简化 Web 组件的开发。

<iframe
  title="WebCell scaffold"
  style="width: 100%; height: 90vh; border: 0; border-radius: 5px"
  src="https://codesandbox.io/embed/github/EasyWebApp/scaffold/tree/master/?autoresize=1&amp;fontsize=14&amp;hidenavigation=1&amp;module=%2Fsrc%2FClock.tsx&amp;theme=dark"
  allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
  sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
></iframe>

### Step 3 | 封装一个 BootStrap 组件

<figure>
    <img src="/article/web-conf-2019-open-source/BootStrap.png">
    <figcaption>BootStrap</figcaption>
</figure>

[BootStrap][11] 可谓 **CSS 框架**时代的开创者，一直被模仿、从未被超越。在同时代的大量同类框架中，现在基本只有它还在持续演进。

在当今基于 Angular、React、Vue 等的 **JS 组件库**大行其道之时，BootStrap 为何仍有很大的优势？来基于它亲自封装一个**通用 Web 组件**、开发一个 Web 应用，你就能体会到其设计的独到之处~

[1]: https://fcc-cd.tk/
[2]: https://github.com/TechQuery
[3]: https://devfest.withgoogle.com/
[4]: https://tech-query.me/programming/web-components-practise/slide.html
[5]: https://www.webcomponents.org/
[6]: https://www.polymer-project.org/
[7]: https://stenciljs.com/
[8]: https://developer.mozilla.org/zh-CN/docs/Web/Web_Components
[9]: https://web-cell.dev/
[10]: https://parceljs.org/
[11]: https://getbootstrap.com/
