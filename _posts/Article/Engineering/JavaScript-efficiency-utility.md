---
title: JavaScript 效率工具
date: 2020-06-08 10:15:48
authors:
  - TechQuery
categories:
  - Article
  - Engineering
tags:
  - ECMAScript
  - DOM
  - BOM
  - API
thumbnail: https://perfectacle.github.io/images/js-001-es/thumb.png
toc: true
---

每当看到发在 [FCC 成都社区][1]群里的技术文章，[水歌][2]都忍不住去指出它的不足。

今天评注的文章题为[《一批提升你工作效率的 JS 工具方法》][3]，文中的 60 个方法与[上次评注的“24 个 ES 方法”][4]类似，不够**简洁**、**优雅**，与**最新 ECMAScript、DOM 标准**有些差距，有些“复制粘贴老文章片段”的感觉。

接下来，我就按功能类别来对一些有必要优化的工具方法一一重构。

<!-- more -->

## 数据校验

完全基于[正则表达式][5]的检验规则其实可以不用封装成函数，全放在独立的模块中，导入后直接 `/regexp/.test(data)` 即可。

### 电邮地址

```javascript
export const Email = /^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$/;
// 原文：/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+((.[a-zA-Z0-9_-]{2,3}){1,2})$/
```

「注解」

- `\w` 即为 `[a-zA-Z0-9_]`
- `[]` 表示一个**字符范围**，就是一个整体，无需 `()` 包围
- Gmail 等服务商还支持形如 `name.filter@gmail.com` 这样的用户别名邮箱
- `(.[a-zA-Z0-9_-]{2,3}){1,2}` 只适用于前些年常见的 `.cn`、`.com.cn` 一类根域名，近几年新增的 `.name`、`.info`、`.club`、`.camp` 等域名就失效了，形如 `vip.xxmail.com` 的多级域名也不适用

### 手机号码

其实以下只适用于中国大陆手机号，其它国家手机号似乎与固定电话号之间没有明显的区分。

```javascript
export const Mobile = /^1[3-9]\d{9}$/;
// 原文：/^1[0-9]{10}$/
```

「注解」

- `\d` 即为 `[0-9]`
- 中国大陆手机号第二位目前没有 1、2

### 固话号码

中国大陆固定电话号码“区号 + 机号”始终为 11 位。

```javascript
export const Phone = /^((0\d{2}-)?\d{8}|(0\d{3}-)?\d{7})$/;
// 原文：/^([0-9]{3,4}-)?[0-9]{7,8}$/
```

### 网址

```javascript
export const URL = /^\w+:\/\/\S+$/;
// 原文：/^http[s]?:\/\/.*/
```

「注解」

- URL 协议不仅包括 `http`、`https`，还有 `ftp`（文件传输）、`file`（本机文件系统）、`ed2k`（电驴 2000）等各种各样的**网络协议**
- URL 主机名、路径可以是 [Unicode][6] 中各种**可见字符**，但遇到空白符就结束

### 日期格式

判断是否为合法的日期格式除了用正则之外，还可利用 [`Date` 构造函数][7]内部的算法：

```javascript
export const isDate = raw => !isNaN(+new Date(raw));
```

对于无法解析为日期的数据，`date.toString()` 会返回“Invalid Date”，`date.getTime()` 对应的返回值则是 `NaN`。而**算数运算符**会调用对象的 `valueOf()` 方法，`date.valueOf()` 的返回值又与 `date.getTime()` 相同。

### 汉字

“汉字”在计算机领域的学名叫**中日韩统一表意文字**（俗称 CJK），在 2017 年 6 月发布的 Unicode 10 标准中，它有了代码级明确的指代：

```javascript
export const HanZi = /\p{Unified_Ideograph}/u;
```

[【详情参考】][8]

### 最佳实践

- 学习：正则分析器 [RegExr][9]、[Regex101][10]
- 前端：[HTML 5 表单校验 API][11]
- 后端：[基于装饰器的数据校验][12]

## 数据转换

### 阿拉伯数字转中文

ECMA-402 标准（[ECMAScript 国际化 API][13]）把各语言之间的**数据格式转换算法**都封装好了，我们引入 polyfill 就可以直接用：

```javascript
export const toChineseNumber = raw =>
  new Intl.NumberFormat("zh-Hans-u-nu-hanidec").format(raw);
```

## 数据类型

判断一个值的类型，用比较**构造函数名**或**类名**的方式兼容性比较差，因为线上环境通常是压缩后的代码，自定义的函数名、类名不再是原名，应用开发者一般也不会实现 [`Symbol.toStringTag` getter 类成员][14]，导致 `Object.prototype.toString.call()` 只会返回默认值 `[object Object]`。

### JavaScript

综上，我们应该利用 [JavaScript 原型继承][15]，来统一判断“值的类型归属”：

```javascript
export const isType = (value, constructor) =>
  Object(value) instanceof constructor;
```

「注解」

- `Object` 构造函数会返回所有基本值的包装对象

### TypeScript

下面，我再给出一个 TypeScript 的实现，让**类型推断**更加准确：

```typescript
export function isType<T>(
  value: T,
  constructor: { new (...data: any[]): T }
): value is T {
  return Object(value) instanceof constructor;
}
```

```typescript
import { isType } from "./utility";

let test;

if (isType(test, Number)) console.log(test!.toFixed(2));
```

[【在编辑器中体验 TS 类型提示】][16]

## 浏览器检测

以下使用 `globalThis` 是为了兼容浏览器主线程、[Web Worker][17]、[Node.js][18]、[Deno][19] 等不同 JavaScript 运行时环境。

### 品牌

```javascript
export const isBrowserVendor = (
  name,
  UA = globalThis.navigator?.userAgent || ""
) => UA.toLowerCase().includes(name);
```

### 爬虫

```javascript
export const isRobot = (UA = globalThis.navigator?.userAgent || "") =>
  /bot|spider|crawler/i.test(UA);
```

## 去除 HTML 标签

### 正则表达式

以下使用了 [non-greedy（非贪婪模式）][20]来提升性能，并规避正文中可能出现的示例代码没完全转译尖括号，导致删除错误。

```javascript
export const removeHtmlTag = raw => raw.replace(/<[\s\S]+?>/g, "");
```

### DOM API

下面再提供一种借助 DOM 引擎的实现：

```javascript
const box = document.createElement("template");

export function removeHtmlTag(raw) {
  box.innerHTML = raw;

  return box.content.textContent;
}
```

## URL 参数追加

[`URL()`][21]、[`URLSearchParams()`][22] 在浏览器主线程、Web Worker、Node.js 10+、Deno 均全局可用。

```javascript
export function appendQuery(path, data, base = globalThis.location.href) {
  const URI = new URL(path, base);
  const { searchParams } = URI;

  for (const key in data) searchParams.append(key, data[key]);

  return URI + "";
}
```

## W3C、ECMA 标准

还有一些可以用新标准（部分为提案）直接实现的特性，集中罗列如下：

- [`.trim()`][23]、[`.trimStart()`][24]、[`.trimEnd()`][25]（原文第 53 条）
- [`.includes()`][26]（原文第 42 条）
- [`Array.from()`][27]（原文第 48 条）
- [数组去重][28]（原文第 44 条）
- [动态 `import`][29]（原文第 27 条）
- [`element.classList`][30]（原文第 29 ~ 31 条）
- [`saveAs()`][31]（原文第 28 条）
- [`text-transform`][32]（原文第 54 条）

## 开源库

![WebCell](https://web-cell.dev/WebCell-1.fb612fdb.png)

水歌把日常开发中积累的各种工具方法，用 TypeScript 写成一个 **Web 开源工具库** —— https://web-cell.dev/web-utility/ ，欢迎大家使用、改进！~

[1]: https://fcc-cd.dev/
[2]: https://github.com/TechQuery
[3]: https://mp.weixin.qq.com/s/4oQc_SYxK4vIKCWWOKwoCw
[4]: https://git-pager.leanapp.cn/article/engineering/ecmascript-dom-skills/
[5]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions
[6]: https://home.unicode.org/
[7]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/Date
[8]: https://zhuanlan.zhihu.com/p/33335629
[9]: https://regexr.com/
[10]: https://regex101.com/
[11]: https://developer.mozilla.org/zh-CN/docs/Learn/HTML/Forms/Data_form_validation
[12]: https://github.com/typestack/class-validator
[13]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Intl
[14]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag
[15]: https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Inheritance
[16]: https://www.typescriptlang.org/play/index.html#code/GYVwdgxgLglg9mABDAzgFQJ4AcCmAeNAPgAoA3AQwBsQcAuRNAGkQgRSgCcRo4P6BvRGBwB3YgDpJAE3JRy9cmAwBtALoBKemkQBfTYgrUcyFA0T8AUImuIOOKCA5IA8gCMAVjmhkqNdcjB2RQgcOGAWNk5uKF4AbgsdCwtKe0QoHHZ4ixhw4lRMXGJ09mYAORAAW1ccDnV-VkC4FPFKOABzIoyoAEJxGIAxGAAPHCliACY62KA
[17]: https://developer.mozilla.org/zh-CN/docs/Web/API/WorkerGlobalScope
[18]: https://nodejs.org/dist/latest-v12.x/docs/api/globals.html#globals_global
[19]: https://doc.deno.land/https/github.com/denoland/deno/releases/latest/download/lib.deno.d.ts
[20]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Cheatsheet
[21]: https://developer.mozilla.org/zh-CN/docs/Web/API/URL
[22]: https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams
[23]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/Trim
[24]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/TrimLeft
[25]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/TrimRight
[26]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes
[27]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from
[28]: https://github.com/TechQuery/array-unique-proposal
[29]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import#%E5%8A%A8%E6%80%81import
[30]: https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList
[31]: https://github.com/eligrey/FileSaver.js
[32]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-transform
