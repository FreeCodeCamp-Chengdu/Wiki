---
title: ECMAScript + DOM 骚操作
date: 2020-03-26 16:52:58
authors:
  - TechQuery
categories:
  - Article
tags:
  - ECMAScript
  - DOM
  - API
toc: true
thumbnail: https://perfectacle.github.io/images/js-001-es/thumb.png
---

前一阵有篇传播较广的 Web 技术博文，我看后觉得，作者题为“ES 6”，但有些示例代码却不够 ES 6，且看我一一优化~

## 对标文章

- 中文译文：[《记好这 24 个 ES6 方法，用来解决实际开发的 JS 问题》][1]
- 英文原文：["24 modern ES6 code snippets to solve practical JS problems"][2]

<!-- more -->

## 我来优化

### 1. 如何隐藏所有指定的元素？

```JavaScript
for (const element of document.querySelectorAll('.your-selector'))
    element.hidden = true;
```

【知识点】

- [`for ... of`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of)
- [`HTMLElement.prototype.hidden`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/hidden)

### 4. 如何获取当前页面的滚动位置？

```JavaScript
const getScrollPosition = (
    { pageXOffset, pageYOffset, scrollLeft, scrollTop } = window
) => ({
    x: pageXOffset ?? scrollLeft,
    y: pageYOffset ?? scrollTop
});

getScrollPosition();  // {x: 0, y: 200}
```

【知识点】（ES 11/2020 特性，略微超纲）

- [`??` 空值合并运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Nullish_Coalescing_Operator)

### 5. 如何平滑滚动到页面顶部？

```JavaScript
window.scrollTo({
    top: 0,
    behavior: 'smooth'
});
```

【知识点】

- [`window.scrollTo()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/scrollTo)

### 8. 如何获取元素中的所有图像？

```JavaScript
const getImages = (element = document) => [
    ...new Set(Array.from(
        element.querySelectorAll('img, input[type="image"]'),
        ({ src }) => src
    ))
];

getImages();  // ['http://fcc-cd.dev/xxx.jpg', 'http://fcc-cd.dev/yyy.png']
```

【知识点】

- [`Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- [`input[type="image"]`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input/image)

### 9. 如何确定设备是移动设备还是台式机、笔记本电脑？

```JavaScript
const isMobile = 'ontouchend' in document;

console.log(isMobile);
```

【知识点】

- [`touchend` 事件](https://developer.mozilla.org/en-US/docs/Web/API/Document/touchend_event)

### 11. 如何创建一个包含当前 URL 参数的对象？

```JavaScript
const parseURLData = (raw = location.search) =>
    Object.fromEntries([...new URLSearchParams(raw)]);

parseURLData('http://url.com/page?n=Adam&s=Smith');  // {n: 'Adam', s: 'Smith'}
```

【知识点】（ES 10/2019 特性，略微超纲）

- [`Object.fromEntries()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)
- [`URLSearchParams()`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams)

若想支持**重名参数**、**参数值转基本类型**，请参考 [Web Utility 的实现][3]。

### 12. 如何将一组表单元素转化为对象？

```JavaScript
const formToJSON = form => Object.fromEntries([...new FormData(form)]);

formToJSON(document.forms[0]);  // {email: 'test@email.com', name: 'Test Name'}
```

【知识点】

- [`FormData()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)

若想支持**同名多值**、 `fieldset` 转**嵌套对象**、**参数值转基本类型**，请参考 [Web Utility 的实现][4]。

### 14. 如何在等待指定时间后调用提供的函数？

```JavaScript
const delay = seconds =>
    new Promise(resolve => setTimeout(resolve, seconds * 1000));

await delay(1);

console.log('1 second later');
```

【知识点】

- [Promise()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [顶层 `await` 提案](https://wenjun.me/2019/11/top-level-await.html)

### 17. 如何获得给定毫秒数的可读格式？

```JavaScript
const unitISO = ['Y', 'M', 'D', 'H', 'm', 's', 'ms'],
    patternISO = /[YMDHms]+/g;

function formatDate(
    time = new Date(), template = 'YYYY-MM-DD HH:mm:ss'
) {
    time = time instanceof Date ? time : new Date(time);

    const temp = new Date(+time - time.getTimezoneOffset() * 60 * 1000)
        .toJSON()
        .split(/\D/)
        .reduce((temp, section, index) => {
            temp[unitISO[index]] = section;

            return temp;
        }, {});

    return template.replace(patternISO, section =>
        temp[section[0]].padStart(section.length, '0')
    );
}

formatDate(new Date(1989, 05, 04), 'YYYY年MM月DD日');  // '1989年06月04日'
```

【知识点】

- [`Date.prototype.toJSON()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/toJSON)
- [`Array.prototype.reduce()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
- [`String.prototype.padStart()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/padStart)

上述函数的核心是 [ISO 时间格式][5]，它也是 **Date 对象序列化**到 JSON 中的标准格式。又因为 ISO 时间总是 0 时区的，所以要事先做好**时区偏移**，使转出的时间正确。将 ISO 时间字符串中的数值与**时间单位占位符**一一对应后，就可用[正则表达式][6]把时间数据替换进模板里了~

### 20. 如何对传递的 URL 发出 POST 请求？

```JavaScript
const request = (
    path,
    method = 'GET',
    body = null,
    header = { 'Content-Type': 'application/json' },
    option = { responseType: 'json' }
) => new Promise((resolve, reject) => {
    const client = new XMLHttpRequest()

    client.onload = () => resolve(client.response),
    client.onerror = reject;

    client.open(method, path);

    for (const name in header)
        client.setRequestHeader(name, header[name]);

    Object.assign(client, option);

    client.send(
        (body && typeof body === 'object') ?
            JSON.stringify(body) : body
    );
});

console.log(await request(
    'https://jsonplaceholder.typicode.com/posts', 'POST', {
        userId: 1,
        id: 1337,
        title: 'Foo',
        body: 'bar bar bar'
    }
));
```

【知识点】

- [`XMLHttpRequest()`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
- [`Object.assign()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

有些小伙伴看了上面的代码可能要说：

> 要用基于 Promise 的 AJAX，干嘛不用 `fetch()`？XHR 还要自己封装……

其实标准化后的 XHR 功能很强大，比 `fetch()` 还灵活，水歌开发的[网络库 KoAJAX][7] 就基于它实现~

## 总结

自 HTML 5、DOM 4、ECMAScript 6 以来，各种**新 API**、**语法糖**层出不穷，再加上 **API polyfill 补丁**、Babel **语法转译器**，Web 前端工程师早已不需担心**浏览器兼容性**，大胆使用原生 API、语法写出**简洁的代码**，专注于业务和上层架构。

以上经验来自[水歌][8]开源的 [WebCell 组件引擎][9]升级最新 API 和语法之路的一些心得。同时，WebCell 也全面拥抱了 TypeScript，并已形成[官方生态库矩阵][10]，支撑了多个生产项目，欢迎大家一起研讨、开发！

## 相关推荐

[《如何用开源软件办一场技术大会？》](https://mp.weixin.qq.com/s/hxCwiokl4uPXJscTQi42-A)

[1]: https://mp.weixin.qq.com/s/kwl1_KhM9d2viJixh5e-qw
[2]: https://dev.to/madarsbiss/20-modern-es6-snippets-to-solve-practical-js-problems-3n83
[3]: https://github.com/EasyWebApp/web-utility/blob/b238a0a/source/URL.ts#L13-L23
[4]: https://github.com/EasyWebApp/web-utility/blob/b238a0a/source/DOM.ts#L71-L116
[5]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString
[6]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions
[7]: https://web-cell.dev/KoAJAX/
[8]: https://github.com/TechQuery
[9]: https://web-cell.dev/
[10]: https://web-cell.dev/WebCell/#ecosystem
