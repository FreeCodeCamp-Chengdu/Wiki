---
title: 我不再使用 React.setState 的 3 个理由
date: 2024-02-09 22:05
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
original: https://blog.cloudboost.io/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e
authors:
  - TechQuery
---

[![Michel Weststrate](https://miro.medium.com/v2/resize:fill:88:88/1*XWCjUzWvB5KUrmXT1kxOOA.jpeg)][2]

[![CloudBoost](https://miro.medium.com/v2/resize:fill:48:48/1*a8_IkAXKt7ff5oUv_QmQSw.png)][3]

[Michel Weststrate][4] · [Follow][5]

Published in [CloudBoost][6] · Jun 15, 2016

自几个月前，我已在所有我新写的 React 组件弃用 React 的 _setState_ 。别误会我，我没有弃用本地组件状态，我只是不再用 React 去管理它，并且令人愉快！

使用 _setState_ 对初学者来说很棘手。即使经验丰富的 React 程序员在使用 React 自有状态机制时，也很容易引入微妙的 bug，例如：

{% asset_img React-state.gif %}

忘记 React 状态是异步的而引入了 bug，日志输出总是在后面一项。

这篇优秀的 React [文档][9]总结了错误使用 _setState_ 的各种情况：

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
      /* up */
      this.setState({
        selection: values[idx - 1]
      });
    } else if (e.keyCode === 40 && idx < values.length - 1) {
      /* down */
      this.setState({
        selection: values[idx + 1]
      });
    }
    this.fireOnSelect();
  };

  fireOnSelect() {
    if (typeof this.props.onSelect === "function")
      this.props.onSelect(this.state.selection); /* not what you expected..*/
  }
}

ReactDOM.render(
  <Select
    values={["State.", "Should.", "Be.", "Synchronous."]}
    onSelect={value => console.log(value)}
  />,
  document.getElementById("app")
);
```

猛地一看，这可能看起来还不错。两个事件处理器和一个工具函数在 _onSelect_ 存在时触发事件。然而，这个 _Select_ 组件有个 bug，在上面的 GIF 图中被很好地演示。 _onSelect_ 总是带着 _state.selection_ 旧值被触发，因为 _fireOnSelect_ 工具函数执行时，_setState_ _还_ 没做完他的工作。我认为此处 React 至少可以把这个方法改名为 _scheduleState_，或让回调参数变成必填。

> 这个 bug 很容易修复，最棘手的部分是意识到它在这儿。

**_编辑于 2018 年 1 月 25 日：_** [**_Dan Abramov_**][10] **_已经热心地写下一个广为人知而有力的_** [**_解释_**][11] **_，从设计的角度讲为什么 setState 是异步的。_**

## 2. setState 引起没必要的渲染

使用 _setState_ 的第二个问题是，它总会触发重渲染。通常这些重渲染是不必要的。你可以使用 React 性能工具中的 [_printWasted_][12] 方法来找出它什么时间发生。但粗略地讲，有几种原因是一次重渲染为什么可能是不必要的：

- 新状态实际上和之前的一模一样。这通常可以通过实现 _shouldComponentUpdate_ 来解决。你可能已经使用了一个（纯渲染）库来解决。
- 有时改变的状态与渲染有关，但并不是所有情况都是。比如当有些数据只条件性可见。
- 第三，正如 Aria Buckles [在 React Europe 2015 上的演讲][13]所指出的那样，有时候实例状态根本与渲染无关！这通常是与管理事件监听器、计时器 ID 等相关的家务状态。

## 3. setState 无法捕获所有组件状态

接上面最后一点，不是所有组件状态都应该用 _setState_ 存储和更新。更多复杂组件通常有基于生命周期钩子的管理需求，去管理计时器、网络请求和事件等。用 _setState_ 管理这些不仅引起不必要的渲染，还会引起相关生命周期钩子被重新触发，引起奇怪的问题。

# 用 MobX 管理本地组件状态

（惊喜，惊喜）在 [Mendix][14]，我们已经依赖 MobX 管理我们所有的状态存储。然而，我们曾使用 React 自有状态机制来处理本地组件状态。最近，我们已经切换到用 MobX 管理本地组件状态。它看起来就像这样：

```jsx
import { observable } from "mobx";
import { observer } from "mobx-react";

@observer
class Select extends React.Component {
  @observable selection = null; /* MobX managed instance state */

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
      /* up */
      this.selection = values[idx - 1];
    } else if (e.keyCode === 40 && idx < values.length - 1) {
      /* down */
      this.selection = values[idx + 1];
    }
    this.fireOnSelect();
  };

  fireOnSelect() {
    if (typeof this.props.onSelect === "function")
      this.props.onSelect(this.selection); /* solved! */
  }
}

ReactDOM.render(
  <Select
    values={["State.", "Should.", "Be.", "Synchronous."]}
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

所以，可渲染和不可渲染的状态都能被统一处理。同时，现在我们组件存储的状态和存在其它存储的状态工作方式一模一样。这让重构组件有些琐碎，并移动本地组件状态进一个独立存储，反之亦然。详见这个 [egghead][15] 教程的演示。

> MobX 高效地把你的组件转化为小型 store

此外，当为 state 应用 observable 时，不会再犯直接向 _state_ 对象赋值的菜鸟错误了。哦，不再操心实现 _shouldComponentUpdate_ 或 _PureRenderMixin_，MobX 已处理好这些。最后，你可能好奇，如果我想等到 _setState_ 完成呢？嗯，你仍可用 _compentDidUpdate_ 生命周期钩子来实现。

## 听起来好酷！我如何开始使用 MobX？

非常简单，照着 [10 分钟交互介绍][16] 或观看前述视频。你可以简单地从你的代码库挑一个组件，把 _@observer_ 拍在上面，并引入一些 _@observable_ 属性。你甚至都不需要替换你现有的 _setState_ 调用，当使用 MobX 时它们依然可用。尽管不出几分钟你可能就会发现它们是如此复杂，无论如何你将会把它们换掉 🙂。（哦，如果你不喜欢装饰器，不用担心，它也[适用于 ES5][17]）。

## 长话短说：

我已不再用 React 管理本地组件状态，我用 MobX 代替，现在 React 就真的成了“仅为视图”🙂。MobX 现在同时管理本地组件状态和 store 状态。它是简洁的、同步的、高效的和统一的。从经验来看，我发现 MobX 比 React 自有 _setState_ 更容易向 React 初学者解释，它让我们的组件干净而简单。

- 把 _setState_ 用于状态管理的 [JSBin][18]
- 把 _MobX observables_ 用于状态管理的 [JSBin][19]

[2]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[3]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[4]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[5]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Fde4496bfa1e2&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&user=Michel+Weststrate&userId=de4496bfa1e2&source=post_page-de4496bfa1e2----ab73fc67a42e---------------------post_header-----------
[6]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[9]: https://facebook.github.io/react/docs/component-api.html
[10]: https://medium.com/u/a3a8af6addc1?source=post_page-----ab73fc67a42e--------------------------------
[11]: https://github.com/facebook/react/issues/11527#issuecomment-360199710
[12]: https://facebook.github.io/react/docs/perf.html#perf.printwastedmeasurements
[13]: https://youtu.be/2Qu-Ulrsfl8?t=12m09s
[14]: http://www.mendix.com/
[15]: https://egghead.io/lessons/javascript-mobx-and-react-intro-syncing-the-ui-with-the-app-state-using-observable-and-observer
[16]: https://mobxjs.github.io/mobx/getting-started.html
[17]: https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/syntax.md#react-components
[18]: http://jsbin.com/yelazuvamo/edit?js%2Cconsole%2Coutput=
[19]: http://jsbin.com/sofezamavi/1/edit?js%2Cconsole%2Coutput=
