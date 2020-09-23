---
title: 【直播】志愿者贡献平台 PWA 开发实战
date: 2020-09-21 17:48:00
categories:
  - Activity
  - Salon
tags:
  - Web
  - RESTful
  - WebCell
  - Strapi
  - online
thumbnail: /activity/workshop/web-components-development/Web-Component-Cell.png
original: https://mp.weixin.qq.com/s/-IxDmlEDcoKQxnZRL58nPw

start: 2020-09-27 19:00:00
end: 2020-09-27 21:00:00
links:
  报名: https://bbs.huaweicloud.com/signup/0117d00ad31a4558905c1db48c2f54ca
mentors:
  - TechQuery
partners:
  - HDZ

toc: true
slidehtml: true
---

> 【时间】2020 年 9 月 27 日（周日）19:00 ~ 21:00

本次沙龙将为大家直播基于 [Headless CMS][1]、[Web 标准组件][2]、[PWA][3] 和 [TypeScript][4] 开发**志愿者贡献平台**。

[1]: https://github.com/search?q=Headless+CMS&ref=opensearch
[2]: https://www.webcomponents.org/
[3]: https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps
[4]: https://www.typescriptlang.org/zh/

<!-- more -->

---

<img
    style="width: 100%; max-height: none"
    src="https://developer-res-cbc-cn.obs.myhwclouds.com/devcenter/activitysign/img/20200916/1600242478177823.jpg"
/>

---

## 入门概念

---

### Headless CMS

[![](https://strapi.io/assets/strapi-logo-light.svg)](https://strapi.io/)

---

### Web Components

[![](https://web-cell.dev/WebCell-0.f1ffd28b.png)](https://web-cell.dev/)

---

## 生成项目

---

### Strapi

```shell
yarn create strapi-app my-project --quickstart
```

---

### WebCell

<iframe
    style="width: 100%; max-width: 100%; height: 100vh"
    src="https://web-cell.dev/"
></iframe>

---

## 开发项目

---

### 数据结构

- 任务 Task
- 贡献 Contribution
- 评价 Evaluation

---

### OAuth 登录

https://strapi.io/documentation/v3.x/plugins/users-permissions.html#providers
