---
title: 如何用开源软件办一场技术大会？
date: 2019-11-28 01:30:03
authors:
  - TechQuery
categories:
  - Article
tags:
  - Web
  - conference
  - Open-Source
toc: true
thumbnail: WebConf-2019-banner.png
---

> [2019 成都 Web 全栈大会][1]技术工作总结

其实今年大会水歌也有不少要分享的心得，可惜忙于大会筹备，没精力准备一个讲题。不过本届大会倒是全面应用了水歌新钻研的技术，会后就和大家简单分享一下。

<!-- more -->

## 官网 Web 前端

<figure>
    <img src="/article/web-conf-2019-guide/WebConf-2019-PWA-0.jpg">
    <figcaption>大会官网 PWA</figcaption>
</figure>

### 组件引擎 —— WebCell

<figure>
    <img src="https://web-cell.dev/WebCell-1.fb612fdb.png">
    <figcaption>WebCell</figcaption>
</figure>

[2018 成都 Web 前端大会][2]筹办时，大会官网就用大会[开源市集][3]参展者 [WebCell][4] 的第一版开发，WebCell 也成为大会当天最火的展位之一。

一年来，其作者[水歌][5]一直保持对**让 Web 开发更简单**的不懈追求，在[贺老][6]对 WebCell “装饰器退回 TypeScript 版本”的建议下，花两天时间用 TypeScript 重写出了 [WebCell v2][7] ——

- [Web Components 标准][8]提供的**轻量级运行时隔离环境**让组件引擎可以用更少的奇技淫巧，降低复杂度（本届大会上[慕阳老师的《DevCloud Web Components 实践》][9]也有深入讲解）

- TypeScript 给 ECMAScript 带来的类型系统不但能充分激发**程序员的肌肉记忆**，还让本与 HTML 5 `<template />` 相差不大的 JSX 通过**类型推导**如虎添翼

### 状态管理 —— MobX

<figure>
    <img src="https://cn.mobx.js.org/mobx.png">
    <figcaption>MobX</figcaption>
</figure>

WebCell 自 v1 以来的**装饰器**写法与 MobX 十分搭调，v2 用了 TypeScript 就更是无缝兼容，而且还带来了更清爽的开发思路：

- [前端路由][10]可以基于状态管理与普通组件来实现，而不再是“特殊的库” —— **路径即状态，容器即组件**

- **登录框**也无需做成单独的页面，而变成一个容器组件，避免*复杂需求下页面跳转时状态的混乱* —— 有登录状态就渲染 `children`，否则渲染登录框（像**后端中间件**一样，让具体业务页面不用管登录状态）

### 组件库 —— BootCell

<figure>
{% asset_img BootStrap.png %}
    <figcaption>BootStrap</figcaption>
</figure>

说到 UI 组件库，很多人可能觉得 [BootStrap][11] “不好看”，但水歌却对它情有独钟 ——

- 官方文档示例代码中的 HTML 结构**语义化**、[**无障碍** (Accessability)][12] 堪称“教科书”，对 SEO（搜索引擎爬虫）、屏幕阅读器（视障人士）极为友好

- 它首先是个 CSS 库，其 [CSS 工具类][13]很丰富，几乎覆盖所有常见定制需求，日常开发基本不需要写自己的 CSS 文件，且能保持一致的**设计规范**，对小项目团队非常友好

因此，在如此规范的设计之上实现一个特定框架的组件库，边写业务边封装都是很快的。

### 工具链 —— Parcel

<figure>
    <img src="https://user-images.githubusercontent.com/19409/31321658-f6aed0f2-ac3d-11e7-8100-1587e676e0ec.png">
    <figcaption>Parcel</figcaption>
</figure>

现在 **Web 前端开发的第一道门槛**恐怕不是 `this`、闭包、异步了，当属 webpack ——

> 我司业务代码都写好了，就差一个 *webpack 配置工程师*了……

所以，我们转而用**面向资源的打包器** [Parcel][14]，它能在保持 **Web 开发原生资源引入方式**的同时，自动处理资源的路径转换、依赖安装、编译流程，**没有特殊需求则无需配置文件**，三条命令搞定 ——

```shell
# 安装
npm install parcel-bundler -D
# 开发
parcel source/index.html
# 构建
parcel build source/index.html
```

### PWA 框架 —— Workbox

<figure>
    <img src="https://developers.google.cn/web/tools/workbox/images/Workbox-Logo-Grey.svg">
    <figcaption>Workbox</figcaption>
</figure>

虽然天朝因为墙的原因而难以使用 [Web Push][15]，但 PWA **图标添加到桌面**、[Service Worker][16]（后台网络缓存）这些特性，国内用的主流操作系统、浏览器基本都支持了，该标准的亲爹 Google 还推出了强大而方便的工具包，让 PWA 真正 Quick start：

```shell
# 安装
npm install workbox-cli -g
# 配置
workbox wizard
# 构建
workbox generateSW
```

【参考】https://tech-query.me/development/pwa-quick-start/

## 电子邀请函

<figure>
    <img src="/article/web-conf-2019-guide/WebConf-2019-Invitation-2.png">
    <figcaption>前端生成截图</figcaption>
</figure>

### 前端截图 —— SVG foreignObject

在 Web 前端页面中截图，最负盛名的库莫过于 [html2canvas][17]，但它需要把当前网页的 DOM 树克隆进一个 `<iframe />`，会意外地触发 Web Components 的重绘，遂放弃。

除此之外，在浏览器更新较快的今天，我们终于可以放心地使用 SVG 带来的福利 —— [`<foreignObject />`][18]。它能将 HTML 包在 SVG DOM 中渲染，而 SVG 又是可以被 `<img />` 引用的图片，截图只需用 `<canvas />` 画一下再输出，就变成 PNG、JPG 这些常用格式了~

```html
<body>
  <svg>
    <foreignObject>
      <h1>Hello, SVG!</h1>
    </foreignObject>
  </svg>
</body>
```

【封装】https://github.com/bubkoo/html-to-image

### 前端请求库 —— KoAJAX

国内前端同学最常用的 HTTP 请求库应该是 axios 了吧？虽然它的 Interceptor（拦截器）API 是 `.use()`，但和 Node.js 的 Express、[Koa][19] 等框架的**中间件模式**完全不同，相比 jQuery `.ajaxPrefilter()`、`dataFilter()` 并没什么实质改进；上传、下载进度比 `jQuery.Deferred()` 还简陋，只是两个专门的回调选项。所以，它还是要对特定的需求记忆特定的 API，不够简洁。

幸运的是，水歌在研究如何[用 ES 2018 异步迭代器实现一个类 Koa 中间件引擎][20]的过程中，做出了一个更有实际价值的上层应用 —— [KoAJAX][21]。它的整个执行过程基于 **Koa 式的中间件**，而且它自己就是一个**中间件调用栈**。除了 RESTful API 常用的 `.get()`、`.post()`、`.put()`、`.delete()` 等快捷方法外，开发者就只需记住 `.use()` 和 `next()`，其它都是 **ES 标准语法**和 **TS 类型推导**：

```javascript
import { HTTPClient } from "koajax";

var token = "";

export const client = new HTTPClient().use(
  async ({ request: { method, path, headers }, response }, next) => {
    if (token) headers.Authorization = "token " + token;

    await next();

    if (method === "POST" && path.startsWith("/session"))
      token = response.headers.Token;
  }
);
```

不仅如此，其**上传下载进度**还是个 Observable 对象（RxJS 粉丝们喜闻乐见），而且是个下文会详述的升级版：

```javascript
import { request } from "koajax";

const { upload, download, response } = request({
  method: "POST",
  path: "/files",
  body: new Blob(["Hello, Observable!"]),
  responseType: "json"
});

for await (const { loaded } of upload)
  console.log(`Upload ${file.name} : ${(loaded / file.size) * 100}%`);

const { body } = await response;

console.log(`Upload ${file.name} : ${body.url}`);
```

### BaaS 后端云服务 —— NodeTS-LeanCloud

【声明】笔者是自来水，绝无商业广告之意。

在云服务大行其道的今日，开发应用貌似必谈**云原生**，本届大会也有阿里云的[《从 Infrastructure as Code 到 Open Application Model —— 填补开发到运维的鸿沟》][22]和 AWS 的[《无服务器计算架构》][23]两个主题来着力科普**云计算架构**。但无论是前者的 IaaS（基础设施即服务）还是后者的 FaaS（函数即服务），对于初创全栈应用而言，其粒度不是太大就是太小，水歌还是青睐**粒度适中的 BaaS**（后端即服务）。

而 BaaS 中的翘楚，国外莫过于 Google Cloud 收购的 Firebase，国内则是 [LeanCloud][24] 深耕数年（它的创始人还是 YouTube 的创始人陈士骏，说起来都是谷歌家的，哈哈）。

BaaS 不但能像 IaaS、PaaS 一样基于 Docker 运行完整的各种语言写的后端项目，而且还封装了**数据库**、**对象存储**、**消息推送**、**实时通讯**等各种常用基础设施，数据库也默认实现了**基于角色权限的用户系统**和**应用内搜索**，这一切只需调用 SDK 的 API，而无需关心安装与运维。其数据库的 API 和后端常用的 **ORM**（对象关系管理）库差不多，SQL、分库分表、负载均衡这些都不用操心了，**抄起键盘就撸业务代码**才是王道！

同时，借助其 SDK 的云函数 API，也能在同一个项目里实现 FaaS 的能力，对热衷代码本身的人来说更加灵活。

【模板】https://github.com/TechQuery/NodeTS-LeanCloud

### Email as a Hook —— IterableObserver

经常制作表单的天朝程序员想必知道，国内各大**表单服务商**无论界面是否花哨、功能是否强大，都没有方便的**后端 API**，与 Google Forms 等国外平台相去甚远。数据的后续处理只能手工导出 Excel 再转 CSV 导入其它数据库，无论实时性、便捷性，简直不可同日而语。

如何在参会者购票成功后，大会官网可查到其报名信息？这让我们头疼了好久……

幸好我们的合作方 MikeCRM 有个**邮件通知**功能，可以用它模拟一个 **Web Hook 接口**。于是，便找了个基于 IMAP 协议的邮件通知监听库 [mail-notifier][25]。

虽然它能实现功能，但随之而来的问题是，邮件客户端重启时集中发来的新邮件事件会并发大量 API 请求，BaaS 服务商会直接断掉后续请求……

如何将**事件流串行**起来？大家自然会想到 RxJS，但它有点重。再仔细想一下，事件通常是异步的，异步的串行不就是**异步迭代**吗？那我们能否将 Observable 封装成 ES 2018 的 Async Iteration 呢？

其实最初版的 Async Iteration 提案中**异步生成器**的返回值就是一个 Observable，后来才拆成单独的提案孵化，可以推测二者是能相互转化的。

于是，水歌尝试用已成为标准的 Async Generator 来实现一个 Observable，便有了 [Iterable Observer][26]，新邮件的串行处理也可以**从离散的 Callback 改写成线性的 Iteration**：

```javascript
const newMail = Observable.fromEvent(client, "mail");

(async () => {
  for await (const { html } of newMail) {
    await new MikeCRM(html).saveUser();
  }
})();
```

[1]: https://web-conf.dev/#2019/
[2]: https://web-conf.dev/#2018/
[3]: https://mp.weixin.qq.com/s/ROvIe7ggETVyTZAscgd7xw
[4]: https://web-cell.dev/
[5]: https://github.com/TechQuery
[6]: https://johnhax.net/
[7]: https://github.com/EasyWebApp/WebCell/tree/v2
[8]: https://www.webcomponents.org/
[9]: https://docs.qq.com/slide/DTnlwYmFham56YkFI
[10]: https://github.com/EasyWebApp/cell-router/tree/v2
[11]: https://getbootstrap.com/
[12]: https://developer.mozilla.org/zh-CN/docs/Web/Accessibility
[13]: https://getbootstrap.com/docs/4.4/utilities/borders/
[14]: https://parceljs.org/
[15]: https://developers.google.cn/web/updates/2015/03/push-notifications-on-the-open-web
[16]: https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers
[17]: https://html2canvas.hertzen.com/
[18]: https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/foreignObject
[19]: https://koajs.com/
[20]: https://tech-query.me/onion-stack/
[21]: https://web-cell.dev/KoAJAX/
[22]: https://docs.qq.com/slide/DTnVmRlRkcHhKQXhs
[23]: https://docs.qq.com/pdf/DTlJFSE5pVk16WHJV
[24]: https://www.leancloud.cn/
[25]: https://github.com/jcreigno/nodejs-mail-notifier
[26]: https://web-cell.dev/iterable-observer/
