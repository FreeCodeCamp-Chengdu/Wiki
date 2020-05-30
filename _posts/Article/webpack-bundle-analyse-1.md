---
title: webpack 打包文件分析（下）
date: 2020-05-28
authors:
  - zhangyanling77
categories:
  - Article
tags:
  - webpack
  - bundle

original: https://github.com/earlyBirdCamp/articles/issues/103
thumbnail: https://www.ma-no.org/cache/galleries/contents-1806/1200-700/webpack-how-it-works.jpeg
toc: true
---

## 回顾

上一篇[webpack 打包文件分析（上）][1]我们讲到 `webpack` 打包源码中文件加载的部分，通过分析了解了在 `webpack` 中不同模块规范相互加载的处理。而至此，只包括了文件的**同步加载**分析，对于文件的异步加载又是如何处理的呢？

我们使用 `webpack` 将项目打包为一个 `bundle.js` 文件，通过 `script` 标签插入到页面中引用。但如果这个 `bundle.js` 体积特别大，就会导致我们加载时间过长，阻塞页面的渲染。

其次，这个打包出来的 `bundle.js` 中其实部分的代码资源是当前加载页面用不到的，这样也导致了浪费。于是，资源加载的优化就成了必须要考虑的问题，而异步加载（或者说动态加载）就是解决这个问题的方案之一。

<!-- more -->

## 异步加载

在 `webpack` 中提供了符合 [ECMAScript 的 `import()` 语法][2]，允许我们动态的加载模块。（在 `webpack` 版本较低时，我们使用的代码动态加载方案是 `require.ensure` 方法，后面已经被 `import()` 取代）。

那么接下来，就继续探究一下**异步加载**的实现。

### 关键文件

- `src/foo.js`

```javascript
export default "foo";
```

- `src/index.js`

```javascript
// /* webpackChunkName: "foo"*/: 魔法字符串，设置打包后的chunk名
import(/* webpackChunkName: "foo" */ "./foo").then(foo => {
  console.log(foo);
});
```

- `webpack.config.js`

```javascript
// ...
  output:  {
    path: path.resolve(__dirname, 'dist'), // 输出目录
    filename: '[name].bundle.js', // 输出文件名称
  },
// ...
```

### bundle 分析

打包后输出两个文件：

> `foo.bundle.js` 因为是异步加载的方式，单独打包为一个文件。由于打包后的源码内容过长，这里省略部分已经分析过的代码块。

- `index.bundle.js`

```javascript
(function(modules) {
  function webpackJsonpCallback(data) {
    // ...
  }
  /**
   * 该对象用于存储已经加载和正在加载中的chunks
   * undefined：表示chunk未加载
   * null：表示chunk预加载 / 预获取
   * Promise：表示chunk正在加载中
   * 0: 表示chunk已经加载了
   */
  var installedChunks = {
    index: 0 // 默认入口模块已经加载完毕
  };

  function __webpack_require__(moduleId) {
    // ...
  }

  // 设置加载chunk的脚本路径 此处的 __webpack_require__.p 为 publicPath，默认为""
  function jsonpScriptSrc(chunkId) {
    return (
      __webpack_require__.p +
      "" +
      ({ foo: "foo" }[chunkId] || chunkId) +
      ".bundle.js"
    );
  }
  // ...

  // 作用：懒加载代码块，原理使用 JSONP
  __webpack_require__.e = function requireEnsure(chunkId) {
    var promises = [];
    // ...
    return Promise.all(promises);
  };
  // ...

  // 异步加载时触发的错误函数
  __webpack_require__.oe = function(err) {
    console.error(err);
    throw err;
  };
  // 存储的是传入的chunk
  var jsonpArray = (window["webpackJsonp"] = window["webpackJsonp"] || []);
  // 存储旧的 jsonpArray.push 方法
  var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
  // 用 webpackJsonpCallback 覆盖 jsonpArray.push 方法
  jsonpArray.push = webpackJsonpCallback;
  jsonpArray = jsonpArray.slice();
  for (var i = 0; i < jsonpArray.length; i++)
    webpackJsonpCallback(jsonpArray[i]);
  var parentJsonpFunction = oldJsonpFunction;
  // __webpack_require__.s 用于缓存入口模块id
  return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
  "./src/index.js": function(module, exports, __webpack_require__) {
    // 异步加载 foo
    __webpack_require__
      .e("foo")
      .then(__webpack_require__.bind(null, "./src/foo.js"))
      .then(foo => {
        console.log(foo);
      });
  }
});
```

- `foo.bundle.js`

```javascript
// [[这里存chunk的名称], {这个对象是存放chunk路径及chunk内容定义的键值对}]
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
  ["foo"],
  {
    "./src/foo.js": function(module, __webpack_exports__, __webpack_require__) {
      "use strict";
      // 将模块标识为 ES Module
      __webpack_require__.r(__webpack_exports__);
      // 将函数内容定义挂在 default 上
      __webpack_exports__["default"] = "foo";
    }
  }
]);
```

可以看出，对于同步加载的部分（`index.js`），依然是使用 `__webpack_require__` 来进行加载的，这里不做赘述。

观察 `index.js` 中对于 `foo.js` 的加载可以发现，使用到了 `__webpack_require__.e` 方法，该方法接收 chunkName，返回一个 `promise`，再传入 chunk 的路径，通过 `__webpack_require__` 加载 chunk 的内容，最后输出。

那么关键点就是 `__webpack_require__.e` 这个方法了。

```javascript
__webpack_require__.e = function requireEnsure(chunkId) {
  var promises = [];
  // 获取加载的chunk内容
  var installedChunkData = installedChunks[chunkId];
  if (installedChunkData !== 0) {
    // 0 表示已经加载过了
    // Promise 意味着 chunk 正在加载
    if (installedChunkData) {
      promises.push(installedChunkData[2]);
    } else {
      // 在chunk缓存中设置 Promise
      var promise = new Promise(function(resolve, reject) {
        installedChunkData = installedChunks[chunkId] = [resolve, reject];
      });
      // 此时 installedChunkData = [resolve, reject, promise]
      promises.push((installedChunkData[2] = promise));

      // 开始加载chunk，jsonp方式
      var script = document.createElement("script");
      var onScriptComplete;

      script.charset = "utf-8"; // 设置字符集
      script.timeout = 120;
      // 和CSP相关
      if (__webpack_require__.nc) {
        script.setAttribute("nonce", __webpack_require__.nc);
      }
      // 设置脚本的加载路径
      script.src = jsonpScriptSrc(chunkId);
      // 脚本加载完成、超时、出错的事件处理函数
      var error = new Error();
      onScriptComplete = function(event) {
        // 避免IE内存泄漏
        script.onerror = script.onload = null;
        clearTimeout(timeout);
        var chunk = installedChunks[chunkId];
        if (chunk !== 0) {
          if (chunk) {
            var errorType =
              event && (event.type === "load" ? "missing" : event.type);
            var realSrc = event && event.target && event.target.src;
            error.message =
              "Loading chunk " +
              chunkId +
              " failed.\n(" +
              errorType +
              ": " +
              realSrc +
              ")";
            error.name = "ChunkLoadError";
            error.type = errorType;
            error.request = realSrc;
            chunk[1](error);
          }
          installedChunks[chunkId] = undefined;
        }
      };
      var timeout = setTimeout(function() {
        onScriptComplete({ type: "timeout", target: script });
      }, 120000);
      script.onerror = script.onload = onScriptComplete;
      document.head.appendChild(script);
    }
  }
  return Promise.all(promises);
};
```

分析这个方法，它的核心作用就是异步加载的实现。

- 获取传入的 chunkName 在 `installedChunks` 对象中对应的加载状态，如果状态为非加载完成，则构造一个 `promise`，将它的 `resolve`、`reject` 作为该 chunk 的正在加载状态，并存入到 `promises` 中。

- 创建 `script` 标签，将 chunk 的路径作为脚本的加载路径，然后插入到页面的 `<head>` 中，让浏览器去下载这个 chunk。

- 最后返回 `promises` 的执行结果，让所有的 `promise` 都变为完成态，即完成所有 chunk 的加载。

接着再来看 `foo.bundle.js`，整个代码体的目的是在向 `window.webpackJsonp` 中 `push` 一个数组，这个数组的结构为 `[["chunk的名字"], { "chunk的路径": function(){ chunk的内容定义 }}]` 。

回到 `index.bundle.js` 中我们可以找到关于 `window.webpackJsonp` 的相关定义及使用。

```javascript
var jsonpArray = (window["webpackJsonp"] = window["webpackJsonp"] || []);
// 存储旧的 jsonpArray.push 方法
var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
// 用 webpackJsonpCallback 覆盖 jsonpArray.push 方法
jsonpArray.push = webpackJsonpCallback;
jsonpArray = jsonpArray.slice();
// 依次调用 webpackJsonpCallback
for (var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
// 缓存上一次的jsonpArray.push方法，形成链条，将模块共享挂载，起到缓存的作用。
var parentJsonpFunction = oldJsonpFunction;
```

这里用 `webpackJsonpCallback` 覆盖了 `window.webpackJson` 的 `push` 方法，也就是说，在 `foo.bundle.js` 中其实是调用了 `webpackJsonpCallback` 方法。

那么，这个 `webpackJsonpCallback` 方法究竟又做了什么呢？

```javascript
function webpackJsonpCallback(data) {
  var chunkIds = data[0]; // 对应加载的chunk的名称的数组
  var moreModules = data[1]; // 对应加载的chunk的路径和chunk定义组成的对象
  var moduleId,
    chunkId,
    i = 0,
    resolves = [];
  for (; i < chunkIds.length; i++) {
    chunkId = chunkIds[i];
    if (
      Object.prototype.hasOwnProperty.call(installedChunks, chunkId) &&
      installedChunks[chunkId]
    ) {
      resolves.push(installedChunks[chunkId][0]); // 收集所有的resolve
    }
    installedChunks[chunkId] = 0; // 标识chunk加载完毕
  }
  // 让modules中包含同步和异步加载的所有模块
  for (moduleId in moreModules) {
    if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
      modules[moduleId] = moreModules[moduleId]; // 将异步加载的chunk添加到 modules 对象中
    }
  }
  if (parentJsonpFunction) parentJsonpFunction(data);

  while (resolves.length) {
    resolves.shift()(); // 依次执行resolve，将所有的promise变为完成态
  }
}
```

根据代码内容分析，该方法

- 首先，判断异步加载的 chunk 是否已经完成加载，如果还在加载中就收集所有 `promise` 的 `resolve`方法，接着在 `installedChunks` 对象中标记 chunk 为加载完成状态
- 然后，再把这些 chunk 都添加到 `modules` 对象中，这样就可通过 `modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)` 来同步加载 chunk，也就是 `foo.bundle.js` 中第一个 `then` 执行的内容，传入模块的路径，使用 `__webpack_require__` 进行同步加载。
- 最后，依次执行收集的 `promise` 的 `resolve` 回调，将所有的 `promise` 变为完成态。

到此，异步加载原理我们就有了一个基本的了解了。

### 补充

源码中还有部分的方法因为没有用到，所以没有做具体的分析。其中 `__webpack_require__.t` 这个方法很有必要提一下。

这个方法会在异步加载中用到，比如，`foo.js` 是 CommonJS 规范的内容。

```javascript
module.exports = "foo";
```

这个时候打包出来的入口文件中就可以看到 `__webpack_require__.t` 的身影。

```javascript
({
  "./src/index.js": function(module, exports, __webpack_require__) {
    __webpack_require__
      .e("foo")
      .then(__webpack_require__.t.bind(null, "./src/foo.js", 7))
      .then(foo => {
        console.log(foo);
      });
  }
});
```

该方法传入模块的路径，以及一个数字 `7`，作用当然也是为了加载模块内容。但它和 `__webpack_require__` 相比究竟有什么区别呢？

```javascript
__webpack_require__.t = function(value, mode) {
  if (mode & 1) value = __webpack_require__(value);
  if (mode & 8) return value;
  if (mode & 4 && typeof value === "object" && value && value.__esModule)
    return value;
  // 创建一个命名空间对象
  var ns = Object.create(null);
  // 将ns对象标识为 ES 模块
  __webpack_require__.r(ns);
  // 给ns对象定义default属性，值为传入的value
  Object.defineProperty(ns, "default", { enumerable: true, value: value });
  if (mode & 2 && typeof value != "string")
    for (var key in value)
      __webpack_require__.d(
        ns,
        key,
        function(key) {
          return value[key];
        }.bind(null, key)
      );
  return ns;
};
```

分析源码可以发现，该方法最终返回一个命名空间对象，接收的第二个参数是个数字，它接下来与 1，2，4，8 进行了按位与操作。想必你已经很快联想到了二进制吧，没错，这几个数字正是对应 0b0001、0b0010、0b0100、0b1000 这几个二进制数。为什么要用数字呢？当然是为了提高运算比较的效率。

回到正题，该方法通过传入的第二个参数进行了以下处理。

- 当 `mode & 1` 为`true`，表示传入的`value`是一个模块 id，需要使用 `__webpack_require__`来加载模块内容

- 当 `mode & 2` 为`true`，首先构造了一个 `ns` 的命名空间对象，将该对象传入 `__webpack_require__.r` 方法中，被标识为一个 ES Module （即拥有`__esModule`属性）。接着定义 `ns` 对象的 `default` 属性，并将传入的 `value` 挂上去作为该对象的值。然后遍历传入的 `value`，将它的属性和值都拷贝定义到 `ns` 上

- 当 `mode & 4` 为`true`，并且传入的 `value` 是个对象且拥有`__esModule`属性（表示已经是或者已经被包装为 ES Module 了），则直接返回这个 `value` 对象

- 当 `mode & 8` 为`true`，其行为等同于 `require`，直接返回 `value` 即可

## 结语

简单总结一下，为了减少打包的体积，去掉非必要资源加载的浪费，我们需要异步加载方案来优化资源的加载。简单说，就是在需要用到某个文件的时候，通过 `import()` 引入这个文件，在返回的 `promise` 的 `then` 中去获取文件内容，以达到动态加载的目的。当然，这并不是唯一的方法，`webpack` 还提供了代码分割方案，也可以达到加载优化的效果。

[1]: /article/webpack-bundle-analyse-0/
[2]: https://tc39.es/proposal-dynamic-import/
