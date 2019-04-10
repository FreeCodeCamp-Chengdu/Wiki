---
title: NodeJS 网页爬虫一小时实战
date: 2019-04-10 18:31:21
categories:
  - Activity
  - Workshop
tags:
  - Node.JS
  - Web
  - crawler
  - Puppeteer
toc: true

description: "freeCodeCamp 成都社区 在线工作坊 #2"
start: 2019-04-14 20:00:00
end: 2019-04-14 22:00:00
mentors:
  - TechQuery
---

> freeCodeCamp 成都社区 在线工作坊 #2
>
> 2019 年 4 月 14 日（周日）晚 8 ~ 10 点

## 学习收获

一小时内学会用 Node.JS 从多个网站**汇总最新本地 IT 活动**列表

## 内容大纲

- [ ] **JavaScript 标准项目**生成
- [ ] **静态网页**抓取
- [ ] **动态网页**抓取
- [ ] **数据接口**分析（选修）

<!-- more -->

## 课前准备

请学员务必提前执行以下命令，安装好**开发环境**！（[操作图解][1]）

### Windows

```shell
choco install -y git tortoisegit nodejs-lts vscode googlechrome zoom
```

### Mac OS X

```shell
brew install node cask
brew cask install sourcetree visual-studio-code google-chrome zoomus
```

## 操作要点

### JavaScript 标准项目生成

```shell
npm init es-pack ~/Desktop/web-crawler

code ~/Desktop/web-crawler
```

### 静态网页抓取

#### 安装依赖包

```shell
npm install web-cell jsdom
```

#### 核心代码

`static.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import "web-cell/dist/polyfill";

import { request, $ } from "web-cell";

(async () => {
  const { body } = await request("https://segmentfault.com/events?city=510100");

  const list = $(".all-event-list .widget-event", body).map(item => ({
    title: $(".title", item)[0].textContent.trim(),
    date: $(".widget-event__meta :first-child", item)[0]
      .textContent.trim()
      .slice(3),
    address: $(".widget-event__meta :last-child", item)[0]
      .textContent.trim()
      .slice(3),
    banner: $(".widget-event__banner", item)[0].src
  }));

  console.info(list);
})();
```

### 动态网页抓取

#### 安装依赖包

```shell
npm install puppeteer-core

npm install fs-match -D
```

#### 增加项目配置

`package.json`

```json
{
  "scripts": {
    "install": "app-find chrome -c"
  }
}
```

首次安装需手动应用配置：

```shell
npm run install
```

#### 核心代码

`dynamic.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import Puppeteer from "puppeteer-core";

(async () => {
  const browser = await Puppeteer.launch({
    executablePath: process.env.npm_config_chrome
  });

  const page = await browser.newPage();

  await page.goto("https://juejin.im/events/chengdu");

  const list = await page.$$eval(".events-list .events-inner", list =>
    list.map(item => ({
      title: item.querySelector(".title").textContent.trim(),
      date: item.querySelector(".date").textContent.trim(),
      address: item.querySelector(".address").textContent.trim(),
      banner: (item
        .querySelector(".banner")
        .style.backgroundImage.match(/url\((?:'|")?(.+)(?:'|")?\)/) || "")[1]
    }))
  );

  console.info(list);
})();
```

## 样本数据

1. https://www.huodongxing.com/events?orderby=n&tag=IT%E4%BA%92%E8%81%94%E7%BD%91&city=%E6%88%90%E9%83%BD

2. https://www.bagevent.com/eventlist.html?f=1&tag=17&city=%E6%88%90%E9%83%BD

3. https://www.oschina.net/event?tab=latest&city=%E6%88%90%E9%83%BD&time=all

4. https://juejin.im/events/chengdu

5. https://segmentfault.com/events?city=510100

[1]: ../hexo-web-app/#%E3%80%90%E9%99%84-0%E3%80%91Windows-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%9B%BE%E8%A7%A3

## 参考文档

- [项目创意](https://github.com/FreeCodeCamp-Chengdu/cd-events)

- [WebCell API 文档](https://web-cell.tk/WebCell/)

- [Puppeteer 中文文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/)
