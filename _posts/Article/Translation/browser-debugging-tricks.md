---
title: 67 Weird Debugging Tricks Your Browser Doesn't Want You to Know
authorURL: ""
originalURL: https://alan.norbauer.com/articles/browser-debugging-tricks
translator: ""
reviewer: ""
---

一份实用而不显眼的黑客高手列表，让您充分利用浏览器的调试器。假设您对开发工具有中级或更高程度的了解。

<!-- more -->

## [高级条件断点](#advanced-conditional-breakpoints)

通过在您意想不到的地方使用会产生副作用的表达式 (有预期之外的结果)，我们可以从诸如条件断点等基本功能中榨取出更多的功能。

### [Logpoints (日志断点) / Tracepoints (跟踪断点)](#logpoints--tracepoints)

例如，我们可以在断点中使用 `console.log`。日志断点是指不会暂停执行并将日志记录到控制台的断点。尽管 Microsoft Edge 已经内置了日志断点有一段时间，Chrome 则是在 v73 版本中才添加了这个功能，但 Firefox 却没有。不过，我们可以使用条件断点 (conditional breakpoints) 在任何浏览器中模拟这一功能。

![Conditional Breakpoint - console.log](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-console-log.2d18d3e4.gif&w=828&q=75)

如果您还想记录该行代码的执行次数，请使用 `console.count` 代替 `console.log`。

更新 (2020 年 5 月)：所有主流浏览器现在都直接支持日志断点/跟踪断点 ([Chrome Logpoints](https://developers.google.com/web/updates/2019/01/devtools#logpoints)，[Edge Tracepoints](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger#breakpoints)，[Firefox Logpoints](https://developer.mozilla.org/en-US/docs/Tools/Debugger/Set_a_logpoint))

#### [Watch Pane](#watch-pane)

您还可以在监视窗格中使用 `console.log`。例如，要在调试器中每次暂停应用程序时转储 `localStorage` 的快照，可以创建一个 `console.table(localStorage)` watch Pane：

![console.table in watch pane](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-table-in-watch.03919d55.png&w=1080&q=75)

或者，要在 DOM 突变后执行表达式，请设置 DOM 变化断点 (在元素检查器 Element Inspector 中)：
 ![DOM Mutation Breakpoint](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-DOM-mutation-chrome.27f07619.png&w=1920&q=75)

然后添加观察表达式 (watch expression)，例如记录 DOM 的快照：`(window.doms = window.doms || []).push(document.documentElement.outerHTML)`。现在，在修改任何 DOM 子树之后，调试器将暂停执行，新的 DOM 快照将位于 `window.doms` 数组的最后一个。(没有办法创建不暂停执行的 DOM 突变断点)。

#### [跟踪调用堆栈 (Tracing Callstacks)](#tracing-callstacks)

举个例子，你有一个 showSpinner 的函数和一个 hideSpinner 的函数，但在代码的某个地方，你调用 show 方法时没有调用匹配的 hide 方法。如何找到未配对显示调用的来源？在 show 方法的条件断点中使用 `console.trace`，运行代码，找到 show 方法的最后一次堆栈跟踪，然后点击调用者进入代码：

![console.trace in conditional breakpoint](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-trace-find-stack.d107e89c.gif&w=1920&q=75)

### [改变程序行为 (Changing Program Behavior)](#changing-program-behavior)

通过使用对程序行为有副作用的表达式，我们可以在浏览器中即时更改程序行为。

例如，你可以覆盖 `getPerson` 函数的参数 `id`。由于 `id=1` 的值为 true，这个条件断点会暂停调试器。为避免这种情况，可在表达式中添加 `, false`。

![Conditional Breakpoint - parameter override](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-parameter-override.375af5d5.gif&w=1920&q=75)

### [快速、简单的性能分析 (Quick and Dirty Performance Profiling)](#quick-and-dirty-performance-profiling)

您不应该用条件断点 (conditional breakpoint) 评估时间之类的东西来性能分析，但如果您想快速而又肮脏地测量某个程序运行所需的时间，可以在条件断点中使用控制台计时 API。在起点设置一个条件为 `console.time('label')` 的断点，在终点设置一个条件为 `console.timeEnd('label')` 的断点。每次运行要测量的内容时，浏览器都会在控制台中记录运行时间。

![Conditional Breakpoint - performance profile](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-time-performance-profile.9b494665.gif&w=1920&q=75)

### [利用函数的参数个数](#using-function-arity)

#### [按参数个数中断](#break-on-number-of-arguments)

只有在当前函数被调用并包含 3 个参数时才暂停：`arguments.callee.length === 3`

在重载函数 (overloaded function) 有可选参数时非常有用。

![Conditional Breakpoint - argument length](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-argument-length.eedb2e1c.gif&w=1920&q=75)

#### [在函数参数个数不匹配时设置断点](#break-on-function-arity-mismatch)

仅在调用当前函数时使用了错误的参数数时暂停：`(arguments.callee.length) != arguments.length`

![Conditional Breakpoint - arity check](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-arity-check.70c0a60c.gif&w=1920&q=75)

在查找函数调用站点中的错误时非常有用。

### [运行时间](#using-time)

#### [跳过页面加载](#skip-page-load)

页面加载 5 秒后才暂停：performance.now() > 5000

当您想设置断点，但只想在初始页面加载后暂停执行时很有用。

#### [跳过 N 秒](#skip-n-seconds)

如果在接下来的 5 秒钟内遇到断点，则不暂停执行，但随时可以在之后暂停：`window.baseline = window.baseline || Date.now(), (Date.now() - window.baseline) > 5000`。

可以随时从控制台 (console) 重置计数器：`window.baseline = Date.now()`

### [Using CSS](#using-css)

根据计算的 CSS 值暂停执行，例如，仅在文档正文背景颜色为红色时暂停执行：`window.getComputedStyle(document.body).backgroundColor === "rgb(255,0,0)"`

### [仅限偶数次调用](#even-calls-only)

每执行一次后暂停一次：`window.counter = window.counter || 0, window.counter % 2 === 0`

### [根据采样暂停](#break-on-sample)

仅在随机抽样执行该行时暂停，例如，仅在每执行 10 次该行时暂停 1 次：`Math.random() < 0.1`

### [此处永不暂停](#never-pause-here)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">

右键单击边栏位置并选择 `Never Pause Here（此处永不暂停）`，Chrome 浏览器会创建一个条件断点，`false`，永远不会触发。这样调试器就不会在这一行暂停。

![Never Pause Here](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fnever-pause-here.a4010ee4.png&w=640&q=75) 
![Never Pause Here Result](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fnever-pause-here-result.32dc71c3.png&w=750&q=75)

当你想将某一行从 XHR 断点中排除、忽略正在抛出的异常等时，它就会派上用场。

### [自动分配实例 ID](#automatic-instance-ids)

通过在构造函数中设置以下条件断点，自动为类的每个实例分配唯一 ID：`(window.instances = window.instances || []).push(this)`

然后检索唯一 ID：`window.instances.indexOf(instance)` (例如，在类方法中使用 `window.instances.indexOf(this)`)

### [编程触发](#programmatically-toggle)

使用全局布尔值选取一个或多个条件断点：

![Boolean gate](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-gated.d32764ce.png&w=1920&q=75)

然后通过编程切换布尔值，例如

-   从控制台手动输入

    ```javascript
    window.enableBreakpoints = true;
    ```
-   从其他断点 ![Boolean gate - enable from other breakpoint](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-gated-enable-from-breakpoint.1c568b6e.png&w=1920&q=75)
-   从控制台上的计时器

    ```javascript
    setTimeout(() => (window.enableBreakpoints = true), 5000);
    ```

-   等等

## [监视类 (class) 的调用](#monitor-class-calls)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">

您可以使用 Chrome 浏览器的 `monitor` 命令行方法轻松跟踪对类方法的所有调用。例如，给定一个类 `Dog`

```javascript
class Dog {
  bark(count)  {   //计数
    /* ... */   
  }
}
```

如果我们想知道对 “Dog” 的所有实例的所有调用，请将此内容粘贴到命令行中：

```javascript
var p = Dog.prototype;
Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

就会在控制台中得到输出结果：

```javascript
> function bark called with arguments: 2
```

如果想暂停任何方法调用的执行 (而不只是记录到控制台)，可以使用 `debug` 代替 `monitor`。

### [从特定实例](#from-a-specific-instance)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">

如果您不知道该类，但有一个实例：

```javascript
var p = instance.constructor.prototype;
Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

当您想为任何类的任何实例 (而不仅仅是 `Dog` 类) 编写一个执行此操作的函数时，该函数非常有用。

## [调用并调试一个函数](#call-and-debug-a-function)

在调用要在控制台中调试的函数之前，调用 `debugger`。例如

```javascript
function fn() {
    /* ... */
}
```

从您的控制台：
```js
debugger; fn(1);
```

然后 `进入下一个函数调用(Step into next function call)`，以调试 `fn` 的实现。

当你不想查找 `fn` 的定义并手动添加断点时，或者当 `fn` 与函数动态绑定，而你不知道源代码在哪里时，这种调试器就很有用。

在 Chrome 浏览器中，您还可以选择在命令行中调用 `debug(fn)`，调试器会在每次调用 `fn` 时暂停执行。

## [在 URL 改变时暂停执行](#pause-execution-on-url-change)

在单页面应用程序修改 URL (即发生路由事件) 之前暂停执行：

```javascript
const dbg = () => {
    debugger;
};
history.pushState = dbg;
history.replaceState = dbg;
window.onhashchange = dbg;
window.onpopstate = dbg;
```


至于如何创建一个能在不中断导航的情况下暂停执行的 `dbg` 版本，读者可自行决定。

另外，请注意，这种方法无法处理直接调用 `window.location.replace/assign` 的情况，因为页面在赋值后会立即销毁，所以没有任何调试内容。如果你仍然想要查看这些重定向的原网址 (并在重定向时调试你的状态)，在 Chrome 中你可以使用 `debug` 来调试相关的方法：

```javascript
debug(window.location.replace);
debug(window.location.assign);
```

## [调试属性读取操作](#debugging-property-reads)

如果你有一个对象，并想知道它的某个属性何时被读取，可以使用一个带有 `debugger` 调用的对象获取器。例如，将 `{configOption: true}` 转换为 `{get configOption() { debugger; return true; }}` (在原始源代码中或使用条件断点)。

当你向某个程序传递一些配置选项，并希望查看这些选项的使用情况时，它就会派上用场。

## [Use copy()](#use-copy)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">
<img src="https://alan.norbauer.com/_next/static/media/firefox.583d9a58.svg" width="40" height="40">

您可以使用 `copy()` 控制台 API 将浏览器中有趣的信息直接复制到剪贴板，而无需截断任何字符串。您可能想复制一些有趣的内容：

-   当前 DOM 的快照：`copy(document.documentElement.outerHTML)`
-   资源的元数据 (如图像)：`copy(performance.getEntriesByType("resource"))`
-   格式化后的大型 JSON blob：`copy(JSON.parse(blob))`
-   本地存储的转储：`copy(localStorage)`
-   等等。

## [Debugging HTML/CSS](#debugging-htmlcss)

JS 控制台有助于诊断 HTML/CSS 的问题。

### [在禁用 JavaScript 的情况下检查 DOM](#inspect-the-dom-with-js-disabled)

在 DOM 检查器中按下 `ctrl+\` (Chrome/Windows) 可以随时暂停 JS 的执行。这样您就可以检查 DOM 的快照，而不必担心 JS 会改变 DOM 或事件 (如鼠标悬停) 会导致 DOM 从您脚下发生变化。

### [检查一个难以捉摸的元素](#inspect-an-elusive-element)

假设您要检查一个 DOM 元素，而该元素只有在有条件的情况下才会出现。检查该元素需要将鼠标移动到该元素上，但当您尝试移动时，该元素却消失了：

![Elusive element](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Felusive-element.495b0945.gif&w=1920&q=75)

为了检查元素，您可以将以下内容粘贴到控制台中：`setTimeout(function(){debugger; }, 5000);`。这将为您提供 5 秒钟的时间来触发用户界面，一旦 5 秒计时器计时结束，JS 的执行就会暂停，元素也不会消失。您可以自由地将鼠标移到开发工具上，而不会丢失元素：

![Elusive element - inspected](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Felusive-element-inspected.f5f036b4.gif&w=1920&q=75)

在暂停执行 JS 时，您可以检查元素、编辑 CSS、在 JS 控制台中执行命令等。

在检查依赖于特定光标位置、焦点等的 DOM 时非常有用。

### [记录 DOM 的快照](#record-snapshots-of-the-dom)

复制当前状态下的 DOM：

```javascript
copy(document.documentElement.outerHTML);
```

每秒记录一次 DOM 的快照：

```javascript
doms = [];
setInterval(() => {
    const domStr = document.documentElement.outerHTML;
    doms.push(domStr);
}, 1000);
```

或者直接转存 (dump) 到控制台：

```javascript
setInterval(() => {
    const domStr = document.documentElement.outerHTML;
    console.log("snapshotting DOM: ", domStr);
}, 1000);
```

### [监视焦点元素](#monitor-focused-element)

```javascript
(function () {
    let last = document.activeElement;
    setInterval(() => {
        if (document.activeElement !== last) {
            last = document.activeElement;
            console.log("Focus changed to: ", last);
        }
    }, 100);
})();
```

![Monitor focused element](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fmonitor-focus.b9692b99.gif&w=1920&q=75)

### [查找加粗元素](#find-bold-elements)

```javascript
const isBold = (e) => {
    let w = window.getComputedStyle(e).fontWeight;
    return w === "bold" || w === "700";
};
Array.from(document.querySelectorAll("*")).filter(isBold);
```

#### [仅限后代元素](#just-descendants)

或者只是检查器中当前所选元素的后代：

```javascript
Array.from($0.querySelectorAll("*")).filter(isBold);
```

### [引用当前选定的元素](#reference-currently-selected-element)

控制台中的 `$0` 是对元素检查器中当前选定元素的自动引用 (automatic reference)。

#### [前面的元素](#previous-elements)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">
<img src="https://alan.norbauer.com/_next/static/media/edge.c22c90ce.svg" width="40" height="40">

在 Chrome 和 Edge 中，你可以使用 `$1` 访问你上次检查的元素，使用 `$2` 访问上上次检查的元素，以此类推。

#### [查看事件监听器](#get-event-listeners)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">

在 Chrome 浏览器中，您可以使用 `getEventListeners($0)` 检查当前选定元素的事件监听器，例如

![getEventListeners](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2FgetEventListeners.4ae6f43e.png&w=1920&q=75)

### [监视元素的事件](#monitor-events-for-element)

<img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40">

调试选定元素的所有事件：`monitorEvents($0)`

调试选定元素的特定事件：`monitorEvents($0, ["control", "key"])`

![monitorEvents](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2FmonitorEvents.a03f9e53.gif&w=1920&q=75)

## [脚注](#footnote-label)

1.  除非浏览器标识另有说明，否则 Chrome、Firefox 和 Edge 浏览器均支持提示功能： <img src="https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg" width="40" height="40"><img src="https://alan.norbauer.com/_next/static/media/firefox.583d9a58.svg" width="40" height="40"><img src="https://alan.norbauer.com/_next/static/media/edge.c22c90ce.svg" width="40" height="40">