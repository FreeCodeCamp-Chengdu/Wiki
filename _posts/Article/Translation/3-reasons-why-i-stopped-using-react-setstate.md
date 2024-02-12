---
title: 我不再使用 React.setState 的 3 个理由
date: 2024-02-09 22:05
updated: 2024-02-12 16:47
categories:
  - Article
  - Translation
tags:
  - React
  - state
  - MobX
  - Web
  - front-end
toc: true
thumbnail: React-MobX-TS.png
original: https://blog.cloudboost.io/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e
authors:
  - TechQuery
---

[![Michel Weststrate](https://miro.medium.com/v2/resize:fill:88:88/1*XWCjUzWvB5KUrmXT1kxOOA.jpeg)][1]

[![CloudBoost](https://miro.medium.com/v2/resize:fill:48:48/1*a8_IkAXKt7ff5oUv_QmQSw.png)][2]

[Michel Weststrate][3] published in [CloudBoost][4] · Jun 15, 2016

自几个月前，我已在所有我新写的 React 组件弃用 React 的 _setState_ 。别误会我，我没有弃用本地组件状态，我只是不再用 React 去管理它，并且令人愉快！

使用 _setState_ 对初学者来说很棘手。即使经验丰富的 React 程序员在使用 React 自有状态机制时，也很容易引入微妙的 bug，例如：

{% asset_img React-state.gif %}

忘记 React 状态是异步的而引入了 bug，日志输出总是在后面一项。

这篇优秀的 React [文档][5]总结了错误使用 _setState_ 的各种情况：

{% asset_img React-docs.webp %}

<!-- more -->

总的来说，使用 _setState_ 有三种问题：

## 1. setState 是异步的

很多开发者一开始没有意识到这一点，但 setState 是 _异步的_ 。如果你设置一些状态后再查看一下，你将仍看到旧状态。这是 setState 最棘手的部分。setState 调用看起来不像异步的，天真地调用 setState 会引入非常微妙的 bug。接下来的 gist 非常好地演示了这一点：

```jsx
class Select extends React.Component {
  constructor(props, context) {
    super(props, context);
    this.state = {
      selection: props.values[0]
    };
  }

  render() {
    return (
      <ul onKeyDown={this.onKeyDown} tabIndex={0}>
        {this.props.values.map(value => (
          <li
            className={value === this.state.selection ? "selected" : ""}
            key={value}
            onClick={() => this.onSelect(value)}
          >
            {value}
          </li>
        ))}
      </ul>
    );
  }

  onSelect(value) {
    this.setState({
      selection: value
    });
    this.fireOnSelect();
  }

  onKeyDown = e => {
    const { values } = this.props;
    const idx = values.indexOf(this.state.selection);
    if (e.keyCode === 38 && idx > 0) {
      /* 上 */
      this.setState({
        selection: values[idx - 1]
      });
    } else if (e.keyCode === 40 && idx < values.length - 1) {
      /* 下 */
      this.setState({
        selection: values[idx + 1]
      });
    }
    this.fireOnSelect();
  };

  fireOnSelect() {
    if (typeof this.props.onSelect === "function")
      this.props.onSelect(this.state.selection); /* 不是你预期的…… */
  }
}

ReactDOM.render(
  <Select
    values={["状态", "应该", "是", "同步的"]}
    onSelect={value => console.log(value)}
  />,
  document.getElementById("app")
);
```

猛地一看，这可能看起来还不错。两个事件处理器和一个工具函数在 _onSelect_ 存在时触发事件。然而，这个 _Select_ 组件有个 bug，在上面的 GIF 图中被很好地演示。 _onSelect_ 总是带着 _state.selection_ 旧值被触发，因为 _fireOnSelect_ 工具函数执行时，_setState_ _还_ 没做完他的工作。我认为此处 React 至少可以把这个方法改名为 _scheduleState_，或让回调参数变成必填。

> 这个 bug 很容易修复，最棘手的部分是意识到它在这儿。

**_编辑于 2018 年 1 月 25 日：_** [**_Dan Abramov_**][6] **_已经热心地写下一个广为人知而有力的_** [**_解释_**][7] **_，从设计的角度讲为什么 setState 是异步的。_**

## 2. setState 引起没必要的渲染

使用 _setState_ 的第二个问题是，它总会触发重渲染。通常这些重渲染是不必要的。你可以使用 React 性能工具中的 [_printWasted_][8] 方法来找出它什么时间发生。但粗略地讲，有几种原因是一次重渲染为什么可能是不必要的：

- 新状态实际上和之前的一模一样。这通常可以通过实现 _shouldComponentUpdate_ 来解决。你可能已经使用了一个（纯渲染）库来解决。
- 有时改变的状态与渲染有关，但并不是所有情况都是。比如当有些数据只条件性可见。
- 第三，正如 Aria Buckles [在 React Europe 2015 上的演讲][9]所指出的那样，有时候实例状态根本与渲染无关！这通常是与管理事件监听器、计时器 ID 等相关的家务状态。

## 3. setState 无法捕获所有组件状态

接上面最后一点，不是所有组件状态都应该用 _setState_ 存储和更新。更多复杂组件通常有基于生命周期钩子的管理需求，去管理计时器、网络请求和事件等。用 _setState_ 管理这些不仅引起不必要的渲染，还会引起相关生命周期钩子被重新触发，引起奇怪的问题。

# 用 MobX 管理本地组件状态

（惊喜，惊喜）在 [Mendix][10]，我们已经依赖 MobX 管理我们所有的状态存储。然而，我们曾使用 React 自有状态机制来处理本地组件状态。最近，我们已经切换到用 MobX 管理本地组件状态。它看起来就像这样：

```jsx
import { observable } from "mobx";
import { observer } from "mobx-react";

@observer
class Select extends React.Component {
  @observable selection = null; /* MobX 管理的实例 state */

  constructor(props, context) {
    super(props, context);
    this.selection = props.values[0];
  }

  render() {
    return (
      <ul onKeyDown={this.onKeyDown} tabIndex={0}>
        {this.props.values.map(value => (
          <li
            className={value === this.selection ? "selected" : ""}
            key={value}
            onClick={() => this.onSelect(value)}
          >
            {value}
          </li>
        ))}
      </ul>
    );
  }

  onSelect(value) {
    this.selection = value;
    this.fireOnSelect();
  }

  onKeyDown = e => {
    const { values } = this.props;
    const idx = values.indexOf(this.selection);
    if (e.keyCode === 38 && idx > 0) {
      /* 上 */
      this.selection = values[idx - 1];
    } else if (e.keyCode === 40 && idx < values.length - 1) {
      /* 下 */
      this.selection = values[idx + 1];
    }
    this.fireOnSelect();
  };

  fireOnSelect() {
    if (typeof this.props.onSelect === "function")
      this.props.onSelect(this.selection); /* 解决啦！ */
  }
}

ReactDOM.render(
  <Select
    values={["状态", "应该", "是", "同步的"]}
    onSelect={value => console.log(value)}
  />,
  document.getElementById("app")
);
```

为了完整性的缘故：

{% asset_img MobX-state.gif %}

当使用一个同步状态机制时，没有未预期的 bug。

上面的代码片段不但更简洁，MobX 也解决了所有 setState 相关问题：

对状态的改变被立即反应到本地组件状态，让我们的逻辑更简单、代码复用更容易。你不必找补“状态可能还没更新”的事实。

MobX 在运行时确定哪些可观察量与渲染相关。所以，暂时与渲染无关的可观察量将不会触发重渲染，直到它们重新相关为止。因此，当把渲染无关的类属性变为 *@observable*时，也完全不存在渲染惩罚（或生命周期问题）。

所以，可渲染和不可渲染的状态都能被统一处理。同时，现在我们组件存储的状态和存在其它存储的状态工作方式一模一样。这让重构组件有些琐碎，并移动本地组件状态进一个独立存储，反之亦然。详见这个 [egghead][11] 教程的演示。

> MobX 高效地把你的组件转化为小型 store

此外，当为 state 应用 observable 时，不会再犯直接向 _state_ 对象赋值的菜鸟错误了。哦，不再操心实现 _shouldComponentUpdate_ 或 _PureRenderMixin_，MobX 已处理好这些。最后，你可能好奇，如果我想等到 _setState_ 完成呢？嗯，你仍可用 _compentDidUpdate_ 生命周期钩子来实现。

## 听起来好酷！我如何开始使用 MobX？

非常简单，照着 [10 分钟交互介绍][12] 或观看前述视频。你可以简单地从你的代码库挑一个组件，把 _@observer_ 拍在上面，并引入一些 _@observable_ 属性。你甚至都不需要替换你现有的 _setState_ 调用，当使用 MobX 时它们依然可用。尽管不出几分钟你可能就会发现它们是如此复杂，无论如何你将会把它们换掉 🙂。（哦，如果你不喜欢装饰器，不用担心，它也[适用于 ES5][13]）。

## 长话短说：

我已不再用 React 管理本地组件状态，我用 MobX 代替，现在 React 就真的成了“仅为视图”🙂。MobX 现在同时管理本地组件状态和 store 状态。它是简洁的、同步的、高效的和统一的。从经验来看，我发现 MobX 比 React 自有 _setState_ 更容易向 React 初学者解释，它让我们的组件干净而简单。

- 把 _setState_ 用于状态管理的 [JSBin][14]
- 把 _MobX observables_ 用于状态管理的 [JSBin][15]

---

## 译者后记

MobX 之父这篇文章给我的启发不限于废弃 React 类组件的 `this.setState()`，更提醒我 ——

> **MobX 装饰器 API** 的设计是为了增强 ECMAScript class，**为面向对象前端代码提供响应式状态管理**，而非发明一套全新的写法。

因此，一切基于 `class` 的组件引擎均可用 MobX 管理组件内外部状态。而**基于类继承的 Web components** 不但完全兼容 MobX 装饰器 API，其 DOM props（元素对象属性）的**可变数据**特性则更是与 MobX 的响应式状态完美契合！

于是，在通读全文、醍醐灌顶之后，我遵循上述思路，一气呵成地在 2022 年正月十五重写出了 [WebCell v3 原型版][16]。接下来的两年虽因忙于创业公司而搁置开发，但在 React 生态中[对原文 MobX 思想的深入实践][17]，为 2024 新年一个月的闭关重写提供了[丰富的设计经验][18]。

最后，原文中的 React + MobX + JS 代码若改写为 [WebCell][19] + MobX + TS，同样简洁清晰：

[![Edit WebCell + MobX state](https://codesandbox.io/static/img/play-codesandbox.svg)][20]

```ts
import { DOMRenderer } from "dom-renderer";
import { observable } from "mobx";
import { WebCell, attribute, component, observer } from "web-cell";

interface SelectProps {
  values: string[];
}

interface Select extends WebCell<SelectProps> {}

@component({ tagName: "wc-select" })
@observer
class Select extends HTMLElement implements WebCell<SelectProps> {
  @attribute
  @observable
  accessor values: string[] = []; /* MobX 管理的实例 props */

  @observable
  accessor selection = this.values[0]; /* MobX 管理的实例 state */

  render() {
    const { values, selection } = this;

    return (
      <ul tabIndex={0} onKeydown={this.onKeyDown}>
        {values.map(value => (
          <li
            key={value}
            className={value === selection ? "selected" : ""}
            onClick={() => this.onSelect(value)}
          >
            {value}
          </li>
        ))}
      </ul>
    );
  }

  onSelect(value: string) {
    this.selection = value;
    this.fireOnSelect();
  }

  onKeyDown = ({ key }: KeyboardEvent) => {
    const { values } = this;
    const index = values.indexOf(this.selection);

    if (key === "ArrowUp" && index > 0) {
      this.selection = values[index - 1];
    } else if (key === "ArrowDown" && index < values.length - 1) {
      this.selection = values[index + 1];
    }
    this.fireOnSelect();
  };

  fireOnSelect() {
    this.emit("select", this.selection);
  }
}

new DOMRenderer().render(
  <Select
    values={["状态", "应该", "是", "同步的"]}
    onSelect={({ detail }: CustomEvent) => console.log(detail)}
  />,
  document.querySelector("#app")
);
```

[1]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[2]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[3]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[4]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[5]: https://facebook.github.io/react/docs/component-api.html
[6]: https://medium.com/u/a3a8af6addc1?source=post_page-----ab73fc67a42e--------------------------------
[7]: https://github.com/facebook/react/issues/11527#issuecomment-360199710
[8]: https://facebook.github.io/react/docs/perf.html#perf.printwastedmeasurements
[9]: https://youtu.be/2Qu-Ulrsfl8?t=12m09s
[10]: http://www.mendix.com/
[11]: https://egghead.io/lessons/javascript-mobx-and-react-intro-syncing-the-ui-with-the-app-state-using-observable-and-observer
[12]: https://mobxjs.github.io/mobx/getting-started.html
[13]: https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/syntax.md#react-components
[14]: http://jsbin.com/yelazuvamo/edit?js%2Cconsole%2Coutput=
[15]: http://jsbin.com/sofezamavi/1/edit?js%2Cconsole%2Coutput=
[16]: https://github.com/EasyWebApp/WebCell/commit/91ee45e7f951c28819bbe93c52139b34dd45f053
[17]: https://github.com/orgs/idea2app/repositories?q=mobx&type=all
[18]: https://www.zhihu.com/question/456685038/answer/3142780397
[19]: https://web-cell.dev/
[20]: https://codesandbox.io/p/devbox/webcell-mobx-state-pjk262?file=%2Fsrc%2Fselect.tsx&embed=1&showConsole=true
