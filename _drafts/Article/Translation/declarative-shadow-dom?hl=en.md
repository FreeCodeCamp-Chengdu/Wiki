---
title: 声明式 Shadow DOM
date: 2024-07-27T07:45:09.743Z
authors:
  - TechQuery
authorURL: https://twitter.com/_developit
original: https://developer.chrome.com/docs/css-ui/declarative-shadow-dom?hl=en
---

Stay organized with collections Save and categorize content based on your preferences.

一种直接在 HTML 中实现和使用 Shadow DOM 的新方法。

![Jason Miller](https://web.dev/images/authors/developit.jpg)

Jason Miller

[X][4] [GitHub][5] [Homepage][6]

![Mason Freed](https://web.dev/images/authors/masonfreed.jpg)

Mason Freed

[X][7] [GitHub][8]

<!-- more -->

声明式 Shadow DOM 是一种[标准 Web 平台特性][9]，已在 Chrome 90 中获得支持。请注意，这个特性的规范在 2023 年有所变更（包括 `shadowroot` 到 `shadowrootmode` 的重命名），还有这个特性所有部分最近标准化的版本已在 Chrome 124 实现。

[Shadow DOM][10] 是包括 [HTML templates][11] 和 [Custom Elements][12] 在内的三个 Web Components 标准之一。Shadow DOM 提供一种方式，可将 CSS 样式的作用域限定于一棵特定 DOM 子树，并把该子树与文档的其余部分隔离开来。`<slot>` 元素给我们一种方式去控制一个自定义元素的子节点应该插入在它的影子树中的什么位置。这些特性联合成一个系统来构建自包含、可复用的组件，以便像一个内置 HTML 元素一样无缝集成进现有应用。

在此之前，使用 Shadow DOM 的唯一方法是用 JavaScript 构造一个影子根节点：

```js
const host = document.getElementById('host');
const shadowRoot = host.attachShadow({mode: 'open'});
shadowRoot.innerHTML = '<h1>Hello Shadow DOM</h1>';
```

这样的命令式 API 适用于客户端渲染：定义我们自定义元素的同一个 JavaScript 也用于创建它们的影子根节点并设置它们的内容。然而，很多 Web 需要在服务端渲染内容，或在构建时生成静态 HTML。想为或许无法运行 JavaScript 的访问者提供一个合理的体验，这可能是一个重要的举措。

使用[服务端渲染][13] (SSR) 的原因因项目而异。有些网站必须提供全功能的服务端渲染 HTML，以便符合无障碍准则；另一些则选择提供一个基准无 JavaScript 体验，以确保在慢速连接和设备上拥有良好性能。

历史上，将 Shadow DOM 与服务端渲染结合使用一直很困难，因为没有内建的方式在服务端生成的 HTML 中表示影子根。当把影子根附加到已单独渲染的 DOM 元素上也会有性能影响。这可能导致在页面已经加载完成时导致布局偏移，或当加载影子根的样式表时暂时闪烁一下未样式化的内容 ("FOUC")。

[声明式 Shadow DOM][14] (DSD) 解除了这些限制，把 Shadow DOM 带到了服务端。

## 构建一个声明式影子根

一个声明式影子根是一个带有 `shadowrootmode` 属性的 `<template>` 元素：

```html
<host-element>
  <template shadowrootmode="open">
    <slot></slot>
  </template>
  <h2>Light content</h2>
</host-element>
```

A template element with the `shadowrootmode` attribute is detected by the HTML parser and immediately applied as the shadow root of its parent element. Loading the pure HTML markup from the above sample results in the following DOM tree:

```
<host-element>
  #shadow-root (open)
  <slot>
    ↳
    <h2>Light content</h2>
  </slot>
</host-element>
```

This code sample is following the Chrome DevTools Elements panel's conventions for displaying Shadow DOM content. For example, the ↳ character represents slotted Light DOM content.

This gives us the benefits of Shadow DOM's encapsulation and slot projection in static HTML. No JavaScript is needed to produce the entire tree, including the Shadow Root.

## Component hydration

Declarative Shadow DOM can be used on its own as a way to encapsulate styles or customize child placement, but it's most powerful when used with Custom Elements. Components built using Custom Elements get automatically upgraded from static HTML. With the introduction of Declarative Shadow DOM, it's now possible for a Custom Element to have a shadow root before it gets upgraded.

A Custom Element being upgraded from HTML that includes a Declarative Shadow Root will already have that shadow root attached. This means the element will have a shadowRoot property already available when it is instantiated, without your code explicitly creating one. It's best to check `this.shadowRoot` for any existing shadow root in your element's constructor. If there is already a value, the HTML for this component includes a Declarative Shadow Root. If the value is null, there was no Declarative Shadow Root present in the HTML or the browser doesn't support Declarative Shadow DOM.

```
<menu-toggle>
  <template shadowrootmode="open">
    <button>
      <slot></slot>
    </button>
  </template>
  Open Menu
</menu-toggle>
<script>
  class MenuToggle extends HTMLElement {
    constructor() {
      super();

      // Detect whether we have SSR content already:
      if (this.shadowRoot) {
        // A Declarative Shadow Root exists!
        // wire up event listeners, references, etc.:
        const button = this.shadowRoot.firstElementChild;
        button.addEventListener('click', toggle);
      } else {
        // A Declarative Shadow Root doesn't exist.
        // Create a new shadow root and populate it:
        const shadow = this.attachShadow({mode: 'open'});
        shadow.innerHTML = `<button><slot></slot></button>`;
        shadow.firstChild.addEventListener('click', toggle);
      }
    }
  }

  customElements.define('menu-toggle', MenuToggle);
</script>
```

Custom Elements have been around for a while, and until now there was no reason to check for an existing shadow root before creating one using `attachShadow()`. Declarative Shadow DOM includes a small change that allows existing components to work despite this: calling the `attachShadow()` method on an element with an existing **Declarative** Shadow Root will **not** throw an error. Instead, the Declarative Shadow Root is emptied and returned. This allows older components not built for Declarative Shadow DOM to continue working, since declarative roots are preserved until an imperative replacement is created.

For newly-created Custom Elements, a new [`ElementInternals.shadowRoot`][15] property provides an explicit way to get a reference to an element's existing Declarative Shadow Root, both open and closed. This can be used to check for and use any Declarative Shadow Root, while still falling back to `attachShadow()` in cases where one was not provided.

```
class MenuToggle extends HTMLElement {
  constructor() {
    super();

    const internals = this.attachInternals();

    // check for a Declarative Shadow Root:
    let shadow = internals.shadowRoot;
    if (!shadow) {
      // there wasn't one. create a new Shadow Root:
      shadow = this.attachShadow({
        mode: 'open'
      });
      shadow.innerHTML = `<button><slot></slot></button>`;
    }

    // in either case, wire up our event listener:
    shadow.firstChild.addEventListener('click', toggle);
  }
}
customElements.define('menu-toggle', MenuToggle);
```

## One shadow per root

A Declarative Shadow Root is only associated with its parent element. This means shadow roots are always colocated with their associated element. This design decision ensures shadow roots are streamable like the rest of an HTML document. It's also convenient for authoring and generation, since adding a shadow root to an element does not require maintaining a registry of existing shadow roots.

The tradeoff of associating shadow roots with their parent element is that it is not possible for multiple elements to be initialized from the same Declarative Shadow Root `<template>`. However, this is unlikely to matter in most cases where Declarative Shadow DOM is used, since the contents of each shadow root are seldom identical. While server-rendered HTML often contains repeated element structures, their content generally differs–for example, slight variations in text, or attributes. Because the contents of a serialized Declarative Shadow Root are entirely static, upgrading multiple elements from a single Declarative Shadow Root would only work if the elements happened to be identical. Finally, the impact of repeated similar shadow roots on network transfer size is relatively small due to the effects of compression.

In the future, it might be possible to revisit shared shadow roots. If the DOM gains support for [built-in templating][16], Declarative Shadow Roots could be treated as templates that are instantiated in order to construct the shadow root for a given element. The current Declarative Shadow DOM design allows for this possibility to exist in the future by limiting shadow root association to a single element.

## Streaming is cool

Associating Declarative Shadow Roots directly with their parent element simplifies the process of upgrading and attaching them to that element. Declarative Shadow Roots are detected during HTML parsing, and attached immediately when their **opening** `<template>` tag is encountered. Parsed HTML within the `<template>` is parsed directly into the shadow root, so it can be "streamed": rendered as it is received.

```
<div id="el">
  <script>
    el.shadowRoot; // null
  </script>

  <template shadowrootmode="open">
    <!-- shadow realm -->
  </template>

  <script>
    el.shadowRoot; // ShadowRoot
  </script>
</div>
```

## Parser-only

Declarative Shadow DOM is a feature of the HTML parser. This means that a Declarative Shadow Root will only be parsed and attached for `<template>` tags with a `shadowrootmode` attribute that are present during HTML parsing. In other words, Declarative Shadow Roots can be constructed during initial HTML parsing:

```
<some-element>
  <template shadowrootmode="open">
    shadow root content for some-element
  </template>
</some-element>
```

Setting the `shadowrootmode` attribute of a `<template>` element does nothing, and the template remains an ordinary template element:

```
const div = document.createElement('div');
const template = document.createElement('template');
template.setAttribute('shadowrootmode', 'open'); // this does nothing
div.appendChild(template);
div.shadowRoot; // null
```

To avoid some important security considerations, Declarative Shadow Roots also can't be created using fragment parsing APIs like `innerHTML` or `insertAdjacentHTML()`. The only way to parse HTML with Declarative Shadow Roots applied is to use `setHTMLUnsafe()` or `parseHTMLUnsafe()`:

```
<script>
  const html = `
    <div>
      <template shadowrootmode="open"></template>
    </div>
  `;
  const div = document.createElement('div');
  div.innerHTML = html; // No shadow root here
  div.setHTMLUnsafe(html); // Shadow roots included
  const newDocument = Document.parseHTMLUnsafe(html); // Also here
</script>
```

## Server-rendering with style

Inline and external stylesheets are fully supported inside Declarative Shadow Roots using the standard `<style>` and `<link>` tags:

```
<nineties-button>
  <template shadowrootmode="open">
    <style>
      button {
        color: seagreen;
      }
    </style>
    <link rel="stylesheet" href="/comicsans.css" />
    <button>
      <slot></slot>
    </button>
  </template>
  I'm Blue
</nineties-button>
```

Styles specified this way are also highly optimized: if the same stylesheet is present in multiple Declarative Shadow Roots, it is only loaded and parsed once. The browser uses a single backing `CSSStyleSheet` that is shared by all of the shadow roots, eliminating duplicate memory overhead.

[Constructable Stylesheets][17] are not supported in Declarative Shadow DOM. This is because there is currently no way to serialize constructable stylesheets in HTML, and no way to refer to them when populating `adoptedStyleSheets`.

## Avoiding the flash of unstyled content

One potential issue in browsers that do not yet support Declarative Shadow DOM is avoiding "flash of unstyled content" (FOUC), where the raw contents are shown for Custom Elements that have not yet been upgraded. Prior to Declarative Shadow DOM, one common technique for avoiding FOUC was to apply a `display:none` style rule to Custom Elements that haven't been loaded yet, since these have not had their shadow root attached and populated. In this way, content is not displayed until it is "ready":

```
<style>
  x-foo:not(:defined) > * {
    display: none;
  }
</style>
```

With the introduction of Declarative Shadow DOM, Custom Elements can be rendered or authored in HTML such that their shadow content is in-place and ready before the client-side component implementation is loaded:

```
<x-foo>
  <template shadowrootmode="open">
    <style>h2 { color: blue; }</style>
    <h2>shadow content</h2>
  </template>
</x-foo>
```

In this case, the `display:none` "FOUC" rule would prevent the declarative shadow root's content from showing. However, removing that rule would cause browsers without Declarative Shadow DOM support to show incorrect or unstyled content until the Declarative Shadow DOM [polyfill][18] loads and converts the shadow root template into a real shadow root.

Fortunately, this can be solved in CSS by modifying the FOUC style rule. In browsers that support Declarative Shadow DOM, the `<template shadowrootmode>` element is immediately converted into a shadow root, leaving no `<template>` element in the DOM tree. Browsers that don't support Declarative Shadow DOM preserve the `<template>` element, which we can use to prevent FOUC:

```
<style>
  x-foo:not(:defined) > template[shadowrootmode] ~ *  {
    display: none;
  }
</style>
```

Instead of hiding the not-yet-defined Custom Element, the revised "FOUC" rule hides its _children_ when they follow a `<template shadowrootmode>` element. Once the Custom Element is defined, the rule no longer matches. The rule is ignored in browsers that support Declarative Shadow DOM because the `<template shadowrootmode>` child is removed during HTML parsing.

## Feature detection and browser support

Declarative Shadow DOM has been available since Chrome 90 and Edge 91, but it used an older non-standard attribute called `shadowroot` instead of the standardized `shadowrootmode` attribute. The newer `shadowrootmode` attribute and streaming behavior is available in Chrome 111 and Edge 111.

As a new web platform API, Declarative Shadow DOM does not yet have widespread support across all browsers. Browser support can be detected by checking for the existence of a `shadowRootMode` property on the prototype of `HTMLTemplateElement`:

```
function supportsDeclarativeShadowDOM() {
  return HTMLTemplateElement.prototype.hasOwnProperty('shadowRootMode');
}
```

## Polyfill

Building a simplified polyfill for Declarative Shadow DOM is relatively straightforward, since a polyfill doesn't need to perfectly replicate the timing semantics or parser-only characteristics that a browser implementation concerns itself with. To polyfill Declarative Shadow DOM, we can scan the DOM to find all `<template shadowrootmode>` elements, then convert them to attached Shadow Roots on their parent element. This process can be done once the document is ready, or triggered by more specific events like Custom Element lifecycles.

```
(function attachShadowRoots(root) {
  root.querySelectorAll("template[shadowrootmode]").forEach(template => {
    const mode = template.getAttribute("shadowrootmode");
    const shadowRoot = template.parentNode.attachShadow({ mode });
    shadowRoot.appendChild(template.content);
    template.remove();
    attachShadowRoots(shadowRoot);
  });
})(document);
```

## Further reading

-   [Chromestatus for Streaming Declarative Shadow DOM][19]
-   [HTML Spec Pull Request][20]

[1]: https://developer.chrome.com/
[2]: https://developer.chrome.com/docs
[3]: https://developer.chrome.com/docs/css-ui
[4]: https://twitter.com/_developit
[5]: https://github.com/developit
[6]: https://jasonformat.com
[7]: https://twitter.com/Mfreed777
[8]: https://github.com/mfreed7
[9]: https://html.spec.whatwg.org/#parsing-main-inhead:attr-template-shadowrootmode:%7E:text=Let%20declarative%20shadow%20host%20element%20be%20adjusted%20current%20node
[10]: https://web.dev/articles/shadowdom-v1
[11]: https://developer.mozilla.org/docs/Web/Web_Components/Using_templates_and_slots
[12]: https://developer.mozilla.org/docs/Web/Web_Components/Using_custom_elements
[13]: https://web.dev/articles/rendering-on-the-web
[14]: https://github.com/mfreed7/declarative-shadow-dom/blob/master/README.md
[15]: https://github.com/WICG/webcomponents/issues/871
[16]: https://github.com/WICG/webcomponents/blob/gh-pages/proposals/Template-Instantiation.md
[17]: https://web.dev/articles/constructable-stylesheets
[18]: https://web.dev/declarative-shadow-dom#polyfill
[19]: https://chromestatus.com/feature/5161240576393216
[20]: https://github.com/whatwg/html/pull/5465