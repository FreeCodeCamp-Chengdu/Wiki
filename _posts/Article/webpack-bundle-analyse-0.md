---
title: webpack 打包文件分析（上）
date: 2020-5-17
updated: 2020-05-18
authors:
  - zhangyanling77
categories:
  - Article
tags:
  - webpack

original: https://github.com/earlyBirdCamp/articles/issues/42
toc: true
---
# webpack 打包文件分析（上）

## 前言
![webpack](https://github.com/zhangyanling77/learn-webpack/blob/master/webpack.png)

`Webpack` 是一个用于静态资源打包的工具。它分析你的项目结构，会递归的构建依赖关系，找到其中脚本、图片、样式等将其转换和打包输出为浏览器能识别的资源。<br>
本篇文章仅对webpack打包输出的文件进行简要的分析。

## 项目准备
[项目地址](https://github.com/zhangyanling77/learn-webpack)

看一下几个关键文件：
- 依赖文件：src/foo.js
```javascript
module.exports = 'foo';
```
- 入口文件：src/index.js
```javascript
const foo = require('./foo.js');
console.log(foo)
```
- webpack配置文件：webpack.config.js
```javascript
const path = require('path');

module.exports = {
  mode: 'development', // 标识不同的环境，development 开发 | production 生产
  devtool: 'none', // 不生成 sourcemap 文件
  entry: './src/index.js', // 文件入口
  output: {
    path: path.resolve(__dirname, 'dist'), // 输出目录
    filename: 'bundle.js', // 输出文件名称
  }
}
```
## bundle分析
首先放上打包输出文件：dist/bundle.js

```javascript
 (function(modules) {
  // 模块缓存对象
  var installedModules = {};
  function __webpack_require__(moduleId) {
    if(installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // 创建一个新的模块对象
    var module = installedModules[moduleId] = {
      i: moduleId, // 模块id，即模块所在的路径
      l: false, // 该模块是否已经加载过了
      exports: {} // 导出对象
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    // 标识模块已经加载过了
    module.l = true;
    return module.exports;
  }
  // 该属性用于公开modules对象 (__webpack_modules__)
  __webpack_require__.m = modules;
  // 该属性用于公开模块缓存对象
  __webpack_require__.c = installedModules;
  // 该属性用于定义兼容各种模块规范输出的getter函数，d即Object.defineProperty
  __webpack_require__.d = function(exports, name, getter) {
    if(!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };
  // 该属性用于在导出对象exports上定义 __esModule = true，表示该模块是一个es6模块
  __webpack_require__.r = function(exports) {
    // 定义这种模块的Symbol.toStringTag为 [object Module]
    if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };
  // 创建一个命名空间对象
  // mode & 1: 传入的value为模块id，使用__webpack_require__加载该模块
  // mode & 2: 将传入的value的所有的属性都合并到ns对象上
  // mode & 4: 当ns对象已经存在时，直接返回value。表示该模块已经被包装过了
  // mode & 8|1: 行为类似于require
  __webpack_require__.t = function(value, mode) {
    if(mode & 1) value = __webpack_require__(value);
    if(mode & 8) return value;
    if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
    // 创建一个命名空间对象
    var ns = Object.create(null);
    // 将ns对象标识为es模块
    __webpack_require__.r(ns);
    // 给ns对象定义default属性，值为传入的value
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
    return ns;
  };
  // 获取模块的默认导出对象，这里区分 commonjs 和 es modlue两种方式
  __webpack_require__.n = function(module) {
    var getter = module && module.__esModule ?
      function getDefault() { return module['default']; } :
      function getModuleExports() { return module; };
    __webpack_require__.d(getter, 'a', getter);
    return getter;
  };
  // 该属性用于判断对象自身属性中是否具有指定的属性，o即Object.prototype.hasOwnProperty
  __webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
  // 该属性用于存放公共访问路径，默认为'' (__webpack_public_path__)
  __webpack_require__.p = "";
  // 加载入口模块并返回模块的导出对象
  return __webpack_require__(__webpack_require__.s = "./src/index.js");
})
({
  "./src/foo.js":
  (function(module, exports) {
    module.exports = 'foo';
  }),
  "./src/index.js":
  (function(module, exports, __webpack_require__) {
    const foo = __webpack_require__("./src/foo.js");
    console.log(foo)
  })
});
```

根据上面的源码可以看出，最终打包出的是一个自执行函数。<br>
首先，这个自执行函数它接收一个参数`modules`，`modules`为一个对象，其中`key`为打包的模块文件的路径，对应的`value`为一个函数，其内部为模块文件定义的内容。<br>
然后，我们再来看一看自执行函数的函数体部分。函数体返回 `__webpack_require__(__webpack_require__.s = "./src/index.js")` 这段代码，此处为加载入口模块并返回模块的导出对象。<br>
可以发现，webpack自己实现了一套加载机制，即`__webpack_require__`，可以在浏览器中使用。该方法接收一个moduleId，返回当前模块的导出对象。
### webpack文件加载（ __webpack_require__  ）
```javascript
  var installedModules = {};
  function __webpack_require__(moduleId) {
    if(installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    module.l = true;
    return module.exports;
  }
  // ...
```
首先，当前作用域顶端声明了`installedModules`这个对象，它用于缓存加载过的模块。在`__webpack_require__`方法内部，会对于传入的moduleId在缓存对象中查找对应的模块是否存在，如果已经存在，返回该模块对象的导出对象；否则，创建一个新的模块对象，记录当前模块id、标识模块是否加载过、以及定义导出对象，同时将它放到缓存对象中。<br>
接下来就是重要的一步，执行模块的函数内容，传入module、module.exports及__webpack_require__作为参数。
```javascript
 modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
```
也就是去执行自执行函数传入的modules对象中当前moduleId对应的函数。接着将该模块标识为已经加载的状态，最后返回当前模块的导出对象。此时便完成了模块的加载任务。<br>
接着，再来看看传入的modules对象部分。
```javascript
 ({
  "./src/foo.js":
  (function(module, exports) {
    module.exports = 'foo';
  }),
  "./src/index.js":
  (function(module, exports, __webpack_require__) {
    const foo = __webpack_require__("./src/foo.js");
    console.log(foo)
  })
})
```
观察函数体内容，可以看到对于依赖模块`foo.js`而言，函数体内即为`foo.js`文件中的定义内容。而对于入口模块`index.js`，则需要执行`__webpack_require__`方法将依赖的文件加载进来使用。

那么，到此为止，我们已经明白了webpack加载模块的基本原理。但细心的你一定发现了，我们的文件导入导出遵循的是commonjs规范，而webpack是基于nodejs实现的，所以在文件加载部分并没有特别的处理。因此，这里我们来看看不同模块规范相互加载时，webpack是如何处理的。

**harmony（和谐，即对于不同模块规范加载的一个兼容处理）**

- commonjs 加载 commonjs

这种方式即我们上面示例的加载方式，就不做赘述了。

- commonjs 加载 es module

src/foo.js
```javascript
export default 'foo';
```
src/index.js
```javascript
const foo = require('./foo.js');
console.log(foo)
```
dist/bundle.js
```javascript
({
  "./src/foo.js":
  (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);
    __webpack_exports__["default"] = ('foo');
  }),
  "./src/index.js":
  (function(module, exports, __webpack_require__) {
    const foo = __webpack_require__("./src/foo.js");
    console.log(foo)
  })
})
```

由打包后的源码可以发现，当`foo.js`使用es module方式导出，与之前的相比，多了`__webpack_require__.r(__webpack_exports__)`这段代码，`__webpack_exports__`很好理解，即模块的导出对象。那么，`__webpack_require__.r`方法是干嘛的呢？
```javascript
// ...
__webpack_require__.r = function(exports) {
  if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
    Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
  }
  Object.defineProperty(exports, '__esModule', { value: true });
};
// ...
```
根据其实现可知，该方法将传入的对象标识上`__esModule=true`，即表明该模块为es6模块。同时定义该对象的`Symbol.toStringTag`为`Module`，即当使用`Object.prototype.toString.call`时将返回`[object Module]`。<br>
最后，将模块的内容挂在`__webpack_exports__`的`default`属性上。

- es module 加载 es module

src/foo.js
```javascript
export default 'foo';
```
src/index.js
```javascript
import foo from './foo.js';
console.log(foo)
```
dist/bundle.js
```javascript
({
  "./src/foo.js":
  (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);
    __webpack_exports__["default"] = ('foo');         
  }),
  "./src/index.js":
  (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);
    var _foo_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/foo.js");
    console.log(_foo_js__WEBPACK_IMPORTED_MODULE_0__["default"])
  })
})
```

当入口文件`index.js`和依赖文件`foo.js`都遵循es module的方式时，可以发现在`index.js`中，对于获取导出对象的方式也有所不同。`_foo_js__WEBPACK_IMPORTED_MODULE_0__`用来接收导入的文件，并通过`default`属性获取到文件的默认导出内容。<br>
那么，是如何实现这种方式的呢？

```javascript
// ...
__webpack_require__.d = function(exports, name, getter) {
  if(!__webpack_require__.o(exports, name)) {
    Object.defineProperty(exports, name, { enumerable: true, get: getter });
  }
};
// ...
__webpack_require__.n = function(module) {
  var getter = module && module.__esModule ?
    function getDefault() { return module['default']; } :
    function getModuleExports() { return module; };
  __webpack_require__.d(getter, 'a', getter);
  return getter;
};

__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
// ...
```

分析这几个方法可以发现，`__webpack_require__.o`其实就是`Object.prototype.hasOwnProperty`的一个重写，用于判断对象自身属性中是否具有指定的属性。而`__webpack_require__.d`即`Object.defineProperty`，这里用于定义兼容各种模块规范输出的getter函数。`__webpack_require__.n`则是用于获取模块的默认导出对象，兼容commonjs 和 es modlue两种方式。

- es module 加载 commonjs

src/foo.js
```javascript
module.exports = 'foo';
```
src/index.js
```javascript
import foo from './foo.js';
console.log(foo)
```
dist/bundle.js
```javascript
({
  "./src/foo.js":
  (function(module, exports) {
    module.exports = 'foo';
  }),
  "./src/index.js":
  (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);
    var _foo_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/foo.js");
    var _foo_js__WEBPACK_IMPORTED_MODULE_0___default = __webpack_require__.n(_foo_js__WEBPACK_IMPORTED_MODULE_0__);
    console.log(_foo_js__WEBPACK_IMPORTED_MODULE_0___default.a)
  })
})
```
当入口文件`index.js`以es module的方式加载遵循commonjs规范的`foo.js`时，通过`__webpack_require__`加载传入的模块，将得到的模块`_foo_js__WEBPACK_IMPORTED_MODULE_0__`再传入`__webpack_require__.n`方法获取到该模块的默认导出对象。因为`foo.js`中的内容是通过**export**导出，而非**export default**导出。因此`foo`被挂在了`default`的一个`a`属性上。

webpack对于不同模块规范的相互加载的处理，我们已经有了基本的了解。但此时我们的文件加载都是同步的，那么文件的异步加载又是怎么样的呢？

请听下回分解。
