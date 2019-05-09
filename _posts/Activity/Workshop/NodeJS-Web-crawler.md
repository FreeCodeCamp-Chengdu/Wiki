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
  - Chrome
toc: true
authors:
  - TechQuery

description: "freeCodeCamp 成都社区 在线工作坊 #2"
start: 2019-04-14 20:00:00
end: 2019-04-14 22:00:00
mentors:
  - TechQuery
  - too
  - demongodYY
  - Akagilnc
---

> freeCodeCamp 成都社区 在线工作坊 #2
>
> 2019 年 4 月 14 日（周日）晚 8 ~ 10 点
>
> 加 [QQ 群 (466129470)](/contact/#QQ-群) 上课（入群注明课程名）

## 学习收获

一小时内学会用 Node.JS 从多个网站**汇总最新本地 IT 活动**列表，并了解一些知识点 ——

1. 用 **Chrome 调试器**分析 **HTML 结构**
2. 用 Chrome 调试器分析 **HTTP 接口**
3. Puppeteer **无界面浏览器**操作
4. JavaScript 最新**标准语法**

## 内容大纲

- [ ] **JavaScript 标准项目**生成
- [ ] **静态网页**抓取
- [ ] **动态网页**抓取
- [ ] **数据接口**分析
- [ ] **数据分页**处理

<!-- more -->

## 课前准备

请学员务必提前执行以下命令，安装好**开发环境**！（[操作图解][1]）

### Windows

```powershell
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

#### 为 Node.JS 定制配置

```shell
npm uninstall amd-bundle

npm install -D \
    @babel/cli \
    @babel/core \
    @babel/plugin-transform-runtime

npm install @babel/runtime
```

`package.json`

```json
{
  "engines": {
    "node": "^6.13.0"
  },
  "script": {
    "lint": "eslint source/ --fix",
    "pack": "babel source/ -d dist/ -s"
  },
  "babel": {
    "presets": [
      [
        "@babel/preset-env",
        {
          "targets": {
            "node": "6.13.0"
          }
        }
      ]
    ],
    "plugins": ["@babel/plugin-transform-runtime"]
  }
}
```

### 静态网页抓取

#### 安装依赖包

```shell
npm install jsdom
```

#### 核心代码

`source/static.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import { JSDOM } from "jsdom";

(async () => {
  const {
    window: { document }
  } = await JSDOM.fromURL("https://segmentfault.com/events?city=510100");

  const list = [
    ...document.querySelectorAll(".all-event-list .widget-event")
  ].map(item => ({
    title: item.querySelector(".title").textContent.trim(),
    date: item
      .querySelector(".widget-event__meta :first-child")
      .textContent.trim()
      .slice(3),
    address: item
      .querySelector(".widget-event__meta :last-child")
      .textContent.trim()
      .slice(3),
    banner: item.querySelector(".widget-event__banner").dataset.original
  }));

  console.info(list);
})();
```

#### 编译并运行

```shell
npm run build

node dist/static
```

### 动态网页抓取

#### 安装依赖包

```shell
npm install puppeteer-core @tech_query/node-toolkit

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

`source/dynamic.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import Puppeteer from "puppeteer-core";

import { getNPMConfig } from "@tech_query/node-toolkit";

(async () => {
  const browser = await Puppeteer.launch({
    executablePath: getNPMConfig("chrome")
  });

  const [page] = await browser.pages();

  await page.goto("https://juejin.im/events/chengdu");

  await page.waitFor(".events-list .events-inner");

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

  process.exit();
})();
```

#### 编译并运行

```shell
npm run build

node dist/dynamic
```

### 数据接口分析

![](HTTP-API.png)

#### 安装依赖包

```shell
npm install node-fetch
```

#### 核心代码

`source/data.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import { URLSearchParams } from "url";

import fetch from "node-fetch";

(async () => {
  const response = await fetch(
    `https://event-storage-api-ms.juejin.im/v2/getEventList?${new URLSearchParams(
      {
        src: "web",
        orderType: "startTime",
        cityAlias: "chengdu"
      }
    )}`
  );

  const data = await response.json();

  console.info(data.d);
})();
```

#### 编译并运行

```shell
npm run build

node dist/data
```

### 数据分页处理

#### 传统思路

`source/data.js`

```javascript
import { URLSearchParams } from "url";

import fetch from "node-fetch";

export default async function(index = 1) {
  const URL = `https://event-storage-api-ms.juejin.im/v2/getEventList?${new URLSearchParams(
    {
      src: "web",
      orderType: "startTime",
      cityAlias: "chengdu",
      pageNum: index
    }
  )}`;

  console.warn(URL);

  return (await (await fetch(URL)).json()).d;
}
```

`source/index.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import crawler from "./data";

(async () => {
  const list = [];

  for (let i = 0; ; )
    try {
      const page = await crawler(++i);

      if ((page || "")[0]) list.push(...page);
      else break;
    } catch (error) {
      console.warn(error);
      break;
    }

  console.info(JSON.stringify(list, null, 4));
})();
```

通过命令输出数据到文件 ——

```shell
npm run build

node dist/ 1> index.json
```

#### 现代思路

`source/data.js`

```javascript
export default async function*(start = 1) {
  while (true) {
    const URL = `https://event-storage-api-ms.juejin.im/v2/getEventList?${new URLSearchParams(
      {
        src: "web",
        orderType: "startTime",
        cityAlias: "chengdu",
        pageNum: start++
      }
    )}`;

    const data = (await (await fetch(URL)).json()).d;

    if ((data || "")[0]) yield { URL, data };
    else break;
  }
}
```

`source/index.js`

```javascript
#! /usr/bin/env node

import "@babel/polyfill";

import crawler from "./data";

(async () => {
  const list = [];

  for await (let { URL, data } of crawler()) {
    console.warn(URL);

    list.push(...data);
  }

  console.info(JSON.stringify(list, null, 4));
})();
```

## Think more...

1. 多源数据去重
2. 定期抓取
3. 展示界面
4. 本地应用打包
5. 服务器部署

## 【附】样本数据

1. https://www.huodongxing.com/events?orderby=n&tag=IT%E4%BA%92%E8%81%94%E7%BD%91&city=%E6%88%90%E9%83%BD

2. https://www.bagevent.com/eventlist.html?f=1&tag=17&city=%E6%88%90%E9%83%BD

3. https://www.oschina.net/event?tab=latest&city=%E6%88%90%E9%83%BD&time=all

4. https://juejin.im/events/chengdu

5. https://segmentfault.com/events?city=510100

## 【附】参考文档

- [项目创意](https://github.com/FreeCodeCamp-Chengdu/IT-events/blob/master/Contributing.md)

- [DOM API 文档](https://developer.mozilla.org/zh-CN/docs/Web/API)

- [Node.JS 中文文档](http://nodejs.cn/api/)

- [Puppeteer 中文文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/)

[1]: ../hexo-web-app/#%E3%80%90%E9%99%84-0%E3%80%91Windows-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%9B%BE%E8%A7%A3
