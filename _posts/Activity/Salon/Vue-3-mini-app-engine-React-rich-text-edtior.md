---
title: 【开源】Vue 3 小程序引擎 & React 富文本编辑器
date: 2021-04-12 01:38:18
categories:
  - Activity
  - Salon
tags:
  - Vue
  - React
  - HTML
  - editor
toc: true
thumbnail: https://miro.medium.com/max/1050/1*DQjqzOr8dgD0Av1252yUww.png

description: 「开源市集」日常活动
start: 2021-04-18 13:30:00
end: 2021-04-18 17:30:00
address: 成都市高新区天府五街 200 号箐蓉汇 2 栋 5 楼少灏厅
mentors:
  - yangmingshan
  - TechQuery
partners:
  - SKFI
---

**开源项目**都很高大上吗？

不，**开源作者**就在你身边！

些许的**实践**与**反思**，就能催生一个小小的开源项目~

阳春三月，一起和[去年 1024「开源市集」][1]中的两位开源人感受春暖花开吧~

> **时间**：2021 年 4 月 18 日 13:30 ~ 17:30
> **地点**：天府五街 200 号箐蓉汇 2 栋 5 楼少灏厅

<!-- more -->

## Vue Mini：Vue 3 小程序引擎

<figure>
  <img src="https://vue-mini.github.io/logo.png">
</figure>

### 简介

[Vue Mini][2] 是一个基于 [Vue 3][3] 的小程序开发库，它能让你用 Composition API 写小程序。与某些小程序开发方案不同的是 Vue Mini 仅仅是一个轻量的运行时库，它既不依赖任何编译步骤，也不涉及任何 Virtual DOM。并且 Vue Mini 从一开始就被设计为能跟小程序原生语法协同工作，你甚至能在同一个页面或组件内混用原生语法与 Vue Mini，这能让你很轻松的将其整合进既有项目中。当然，你也能完全使用 Vue Mini 开发一个小程序。

Vue Mini 仅聚焦于小程序逻辑部分，也就是 JS 部分，它并不影响小程序的模版、样式及配置。

### 作者

<figure>
  <img src="https://github.com/yangmingshan.png">
  <figcaption>杨明山</figcaption>
  <figcaption>
    <a target="_blank" href="https://www.thoughtworks.com/zh-cn">ThoughtWorks</a>
    高级 Web 工程师
  </figcaption>
</figure>

## React Bootstrap editor：轻量级富文本编辑器

<figure>
  <img src="https://github.com/react-bootstrap.png">
</figure>

### 简介

[React Bootstrap editor][4] 是一个基于 [TypeScript][5]、[React][6] 和 [Bootstrap][7] 的轻量级**富文本编辑器**，没有繁杂的 API，只有基于**抽象类**的平台封装：

1. [`document.execCommand()`][8]
2. [`document.queryCommandState()`][9]
3. [`document.queryCommandSupported()`][10]
4. [`document.queryCommandEnabled()`][11]
5. [Selection API][12]

而其自身代码不过 400 行，原型实现也只花了外包项目恼火一天后的 4 小时熬夜。

### 作者

<figure>
  <img src="/activity/salon/web-and-javascript-new-age/ShuiGe.jpg">
  <figcaption>水歌</figcaption>
  <figcaption>
    <a target="_blank" href="https://ideapp.dev/">idea2app</a>
    团队创始人
  </figcaption>
</figure>

[1]: https://fcc-cd.dev/activity/conference/coscon-2020-chengdu/
[2]: https://vue-mini.github.io/
[3]: https://v3.vuejs.org/guide/introduction.html
[4]: https://github.com/idea2app/React-Bootstrap-editor
[5]: https://www.typescriptlang.org/zh/
[6]: https://reactjs.org/
[7]: https://getbootstrap.com/
[8]: https://developer.mozilla.org/zh-CN/docs/Web/API/document/execCommand
[9]: https://developer.mozilla.org/zh-CN/docs/Web/API/document/queryCommandState
[10]: https://developer.mozilla.org/zh-CN/docs/Web/API/document/queryCommandSupported
[11]: https://developer.mozilla.org/zh-CN/docs/Web/API/Document/queryCommandEnabled
[12]: https://developer.mozilla.org/zh-CN/docs/Web/API/Selection
