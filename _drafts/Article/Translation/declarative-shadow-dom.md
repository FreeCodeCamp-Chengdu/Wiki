---
title: 宣言型の Shadow DOM
date: 2024-07-27T07:40:40.781Z
author: X
authorURL: https://twitter.com/_developit
originalURL: https://developer.chrome.com/docs/css-ui/declarative-shadow-dom
translator: ""
reviewer: ""
---

![](https://developer.chrome.com/_static/images/translated.svg?hl=ja) このページは [Cloud Translation API][1] によって翻訳されました。

<!-- more -->

-   [ホーム][2]
-   [Docs][3]
-   [CSS and UI][4]

# 宣言型の Shadow DOM

コレクションでコンテンツを整理 必要に応じて、コンテンツの保存と分類を行います。

HTML で直接 Shadow DOM を実装して使用する新しい方法です。

![Jason Miller](https://web.dev/images/authors/developit.jpg?hl=ja)

Jason Miller

[X][5] [GitHub][6] [ホームページ][7]

![Mason Freed](https://web.dev/images/authors/masonfreed.jpg?hl=ja)

Mason Freed

[X][8] [GitHub][9]

宣言型 Shadow DOM は[標準のウェブ プラットフォーム機能][10]であり、Chrome バージョン 90 以降でサポートされています。なお、この機能は 2023 年に仕様が変更され（`shadowroot` の `shadowrootmode` への名前変更を含む）、この機能のすべての部分の最新の標準化されたバージョンが Chrome バージョン 124 で利用可能になりました。

[Shadow DOM][11] は 3 つの Web Components 標準の一つであり、[HTML テンプレート][12]と[カスタム要素][13]によって丸められています。Shadow DOM を使用すると、CSS スタイルのスコープを特定の DOM サブツリーに制限し、そのサブツリーをドキュメントの他の部分から分離できます。`<slot>` 要素を使用すると、カスタム要素の子をシャドウツリー内のどこに挿入するかを制御できます。これらの機能を組み合わせることで、組み込みの HTML 要素と同様に、既存のアプリケーションにシームレスに統合される自己完結型の再利用可能なコンポーネントを構築できます。

これまでは、Shadow DOM を使用するには、JavaScript を使用して Shadow ルートを構築する必要がありました。

```
const host = document.getElementById('host');
const shadowRoot = host.attachShadow({mode: 'open'});
shadowRoot.innerHTML = '<h1>Hello Shadow DOM</h1>';
```

このような命令型 API は、クライアント側のレンダリングで問題なく機能します。カスタム要素を定義するのと同じ JavaScript モジュールが、シャドウルートの作成とコンテンツの設定も行います。ただし、多くのウェブ アプリケーションでは、コンテンツをサーバーサイドでレンダリングするか、ビルド時に静的 HTML にレンダリングする必要があります。これは、JavaScript を実行できない可能性のあるユーザーに合理的なエクスペリエンスを提供するうえで重要な役割を果たします。

[サーバーサイド レンダリング][14]（SSR）を行う理由は、プロジェクトによって異なります。ウェブサイトによっては、ユーザー補助ガイドラインを満たすために、完全に機能するサーバー レンダリング HTML を提供する必要があります。また、低速な接続やデバイスで優れたパフォーマンスを保証する方法として、JavaScript を使用しないベースライン エクスペリエンスを提供することを選択するウェブサイトもあります。

サーバーで生成された HTML に Shadow Root を表現する方法が組み込まれていなかったため、従来は Shadow DOM をサーバーサイド レンダリングと組み合わせて使用することは困難でした。Shadow Root なしですでにレンダリングされている DOM 要素に Shadow Root をアタッチする場合も、パフォーマンスに影響します。これにより、ページの読み込み後にレイアウトがずれたり、Shadow Root のスタイルシートの読み込み中に、スタイルが設定されていないコンテンツ（「FOUC」）が一時的に表示される原因となったりすることがあります。

[宣言型 Shadow DOM][15]（DSD）はこの制限を取り除き、サーバーに Shadow DOM を導入します。

## 宣言型シャドウルートのビルド

宣言型シャドウルートは、`shadowrootmode` 属性を持つ `<template>` 要素です。

```
<host-element>
  <template shadowrootmode="open">
    <slot></slot>
  </template>
  <h2>Light content</h2>
</host-element>
```

`shadowrootmode` 属性を持つテンプレート要素は HTML パーサーによって検出され、親要素のシャドウ ルートとしてすぐに適用されます。上記のサンプルの純粋な HTML マークアップを読み込むと、次のような DOM ツリーが生成されます。

```
<host-element>
  #shadow-root (open)
  <slot>
    ↳
    <h2>Light content</h2>
  </slot>
</host-element>
```

このコードサンプルは、Chrome DevTools の \[Elements\] パネルの規則に従って、Shadow DOM コンテンツを表示します。たとえば、🚫? 文字はスロット化された Light DOM コンテンツを表します。

これにより、静的 HTML での Shadow DOM のカプセル化とスロット投影のメリットが活かされます。Shadow Root を含むツリー全体を生成するために JavaScript は必要ありません。

## 成分水分補給

宣言型 Shadow DOM は、スタイルをカプセル化したり、子の配置をカスタマイズしたりするための手段として単独で使用できますが、カスタム要素と組み合わせて使用すると最も効果的です。カスタム要素を使用して作成されたコンポーネントは、静的 HTML から自動的にアップグレードされます。宣言型 Shadow DOM の導入により、アップグレード前にカスタム要素に Shadow ルートを設定できるようになりました。

宣言型シャドウルートを含む HTML からアップグレードされるカスタム要素には、すでにそのシャドウルートがアタッチされています。つまり、コードが明示的に作成しなくても、要素のインスタンス化時には、すでに利用可能な shadowRoot プロパティが要素に含まれます。要素のコンストラクタに既存のシャドウルートがあるかどうかを `this.shadowRoot` でチェックすることをおすすめします。値がすでに存在する場合、このコンポーネントの HTML には宣言型 Shadow ルートが含まれます。値が null の場合は、HTML に宣言型 Shadow Root が存在しないか、ブラウザで宣言型 Shadow DOM がサポートされていません。

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

カスタム要素は以前から存在していますが、これまでは `attachShadow()` を使用して作成する前に、既存のシャドウルートを確認する理由はありませんでした。宣言型 Shadow DOM には小さな変更が加えられており、既存のコンポーネントを動作させることができます。既存の宣言型 Shadow ルートがある要素で `attachShadow()` メソッドを呼び出しても、エラーはスローされません。代わりに、宣言型シャドウルートが空になり、返されます。これにより、宣言型 Shadow DOM 用にビルドされていない古いコンポーネントが動作を継続できるようになります。これは、命令型置換が作成されるまで宣言型ルートが保持されるためです。

新たに作成されたカスタム要素では、新しい [`ElementInternals.shadowRoot`][16] プロパティを使用して、要素の既存の宣言型シャドウルート（開いた状態と閉じた状態の両方）への参照を明示的に取得できます。これを使用すると、宣言型シャドウルートを確認して使用できますが、宣言型ルートが指定されていない場合は `attachShadow()` にフォールバックできます。

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

## ルートごとに 1 つのシャドウ

宣言型シャドウルートは、その親要素にのみ関連付けられます。つまり、シャドウルートは常に関連する要素と同じ場所に配置されます。この設計上の決定により、シャドウルートも HTML ドキュメントの他の部分と同じようにストリーミング可能になります。また、要素にシャドウルートを追加しても、既存のシャドウルートのレジストリを維持する必要がないため、作成や生成にも役立ちます。

シャドウルートを親要素に関連付けるトレードオフは、複数の要素を同じ宣言型シャドウルート `<template>` から初期化できないことです。ただし、宣言型 Shadow DOM が使用されているほとんどのケースでは、各 Shadow ルートの内容がほとんど同一ではないため、この点で問題が生じる可能性はほとんどありません。サーバーでレンダリングされる HTML には多くの場合、繰り返しの要素構造が含まれていますが、テキストや属性がわずかに異なるなど、通常はコンテンツが異なります。シリアル化された宣言型シャドウルートの内容は完全に静的であるため、1 つの宣言型シャドウルートから複数の要素をアップグレードしても、要素が同じであった場合にのみ機能します。最後に、圧縮の影響により、類似したシャドウルートが繰り返された場合のネットワーク転送サイズへの影響は比較的小さくなります。

将来的には、共有シャドウルートを再検討できるようになる可能性があります。DOM で[組み込みテンプレート][17]がサポートされている場合、宣言型シャドウルートは、特定の要素のシャドウルートを作成するためにインスタンス化されるテンプレートとして扱うことができます。現在の宣言型 Shadow DOM の設計では、Shadow ルートの関連付けを 1 つの要素に制限することで、このような可能性に対応できます。

## ストリーミングは優れている

宣言型シャドウルートを親要素に直接関連付けると、アップグレードしてその要素にアタッチするプロセスが簡単になります。宣言型シャドウルートは HTML 解析中に検出され、**開始** `<template>` タグが検出されるとすぐに添付されます。`<template>` 内の解析された HTML は直接 Shadow ルートに解析されるため、これを「ストリーミング」して、受信時にレンダリングできます。

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

## パーサーのみ

宣言型 Shadow DOM は HTML パーサーの機能です。つまり、宣言型シャドウルートは、HTML 解析中に存在する `shadowrootmode` 属性を持つ `<template>` タグに対してのみ解析され、添付されます。つまり、宣言型シャドウルートは最初の HTML 解析時に構築できます。

```
<some-element>
  <template shadowrootmode="open">
    shadow root content for some-element
  </template>
</some-element>
```

`<template>` 要素の `shadowrootmode` 属性を設定しても何も行われず、テンプレートは通常のテンプレート要素のままです。

```
const div = document.createElement('div');
const template = document.createElement('template');
template.setAttribute('shadowrootmode', 'open'); // this does nothing
div.appendChild(template);
div.shadowRoot; // null
```

セキュリティ上の重要な考慮事項を回避するため、宣言型シャドールートは、`innerHTML` や `insertAdjacentHTML()` などのフラグメント解析 API を使用して作成することもできません。宣言型シャドウルートが適用された HTML を解析する唯一の方法は、`setHTMLUnsafe()` または `parseHTMLUnsafe()` を使用することです。

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

## スタイルを使用したサーバー レンダリング

インライン スタイルシートと外部スタイルシートは、標準の `<style>` タグと `<link>` タグを使用する宣言型シャドウルート内で完全にサポートされています。

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

この方法で指定するスタイルは高度に最適化されています。同じスタイルシートが複数の宣言型シャドウルートに存在する場合でも、読み込みと解析は 1 回だけ行われます。ブラウザは、すべてのシャドウルートで共有される単一のバッキング `CSSStyleSheet` を使用するため、重複するメモリのオーバーヘッドがなくなります。

[構成可能なスタイルシート][18]は宣言型 Shadow DOM ではサポートされていません。これは、現在 HTML で構成可能なスタイルシートをシリアル化する方法がなく、`adoptedStyleSheets` を入力する際にそれらを参照する方法もないためです。

## スタイルが設定されていないコンテンツが大量に発生しないようにする

宣言型 Shadow DOM にまだ対応していないブラウザでは、「スタイル化されていないコンテンツのフラッシュ」（FOUC）が回避される可能性があります。これは、まだアップグレードされていないカスタム要素に未加工のコンテンツが表示されることです。宣言型 Shadow DOM が導入される前、FOUC を回避する一般的な手法の 1 つは、まだ読み込まれていないカスタム要素に `display:none` スタイルルールを適用することでした。カスタム要素には Shadow ルートがアタッチされず、データが入力されていないためです。このようにして、コンテンツは「準備完了」になるまで表示されません。

```
<style>
  x-foo:not(:defined) > * {
    display: none;
  }
</style>
```

宣言型 Shadow DOM の導入により、HTML でカスタム要素をレンダリングまたは作成できるため、クライアント側のコンポーネントの実装が読み込まれる前に、シャドウ コンテンツが適切な場所に配置され、準備が整います。

```
<x-foo>
  <template shadowrootmode="open">
    <style>h2 { color: blue; }</style>
    <h2>shadow content</h2>
  </template>
</x-foo>
```

この場合、`display:none` の「FOUC」ルールにより、宣言型シャドウルートのコンテンツが表示されなくなります。ただし、このルールを削除すると、宣言型 Shadow DOM のサポートがないブラウザでは、宣言型 Shadow DOM の[polyfill][19]が読み込まれて実際の Shadow ルートに変換されるまで、誤ったコンテンツやスタイルが設定されていないコンテンツが表示されます。

幸いなことに、これは CSS で FOUC スタイルルールを変更することで解決できます。宣言型 Shadow DOM をサポートするブラウザでは、`<template shadowrootmode>` 要素は直ちに Shadow ルートに変換され、DOM ツリーに `<template>` 要素は残りません。宣言型 Shadow DOM をサポートしていないブラウザは、`<template>` 要素を保持します。これにより、FOUC を防ぐことができます。

```
<style>
  x-foo:not(:defined) > template[shadowrootmode] ~ *  {
    display: none;
  }
</style>
```

修正後の「FOUC」ルールは、まだ定義されていないカスタム要素を非表示にするのではなく、`<template shadowrootmode>` 要素の後にある子要素を非表示にします。カスタム要素が定義されると、そのルールは一致しなくなります。HTML 解析中に `<template shadowrootmode>` の子が削除されるため、宣言型 Shadow DOM をサポートしているブラウザでは、このルールは無視されます。

## 機能検出とブラウザ サポート

宣言型 Shadow DOM は Chrome 90 と Edge 91 以降で利用できますが、標準化された `shadowrootmode` 属性ではなく、`shadowroot` という標準以外の古い属性が使用されていました。新しい `shadowrootmode` 属性とストリーミング動作は、Chrome 111 と Edge 111 で使用できます。

宣言型 Shadow DOM は新しいウェブ プラットフォーム API であるため、まだすべてのブラウザで広くサポートされているわけではありません。ブラウザのサポートは、`HTMLTemplateElement` のプロトタイプで `shadowRootMode` プロパティの存在を確認することで検出できます。

```
function supportsDeclarativeShadowDOM() {
  return HTMLTemplateElement.prototype.hasOwnProperty('shadowRootMode');
}
```

## ポリフィル

宣言型 Shadow DOM の簡素化されたポリフィルの作成は比較的簡単です。ポリフィルは、ブラウザ実装に関連するタイミング セマンティクスやパーサーのみの特性を完全に複製する必要がないためです。宣言型 Shadow DOM をポリフィルするには、DOM をスキャンしてすべての `<template shadowrootmode>` 要素を検出し、それらを親要素に適用された Shadow ルートに変換します。このプロセスは、ドキュメントの準備ができたら実行することも、カスタム要素のライフサイクルなど、より具体的なイベントによってトリガーすることもできます。

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

## 参考資料

-   [ストリーミング宣言型 Shadow DOM の Chromestatus][20]
-   [HTML 仕様の pull リクエスト][21]

[1]: //cloud.google.com/translate/?hl=ja
[2]: https://developer.chrome.com/?hl=ja
[3]: https://developer.chrome.com/docs?hl=ja
[4]: https://developer.chrome.com/docs/css-ui?hl=ja
[5]: https://twitter.com/_developit
[6]: https://github.com/developit
[7]: https://jasonformat.com
[8]: https://twitter.com/Mfreed777
[9]: https://github.com/mfreed7
[10]: https://html.spec.whatwg.org/#parsing-main-inhead:attr-template-shadowrootmode:%7E:text=Let%20declarative%20shadow%20host%20element%20be%20adjusted%20current%20node
[11]: https://web.dev/articles/shadowdom-v1?hl=ja
[12]: https://developer.mozilla.org/docs/Web/Web_Components/Using_templates_and_slots
[13]: https://developer.mozilla.org/docs/Web/Web_Components/Using_custom_elements
[14]: https://web.dev/articles/rendering-on-the-web?hl=ja
[15]: https://github.com/mfreed7/declarative-shadow-dom/blob/master/README.md
[16]: https://github.com/WICG/webcomponents/issues/871
[17]: https://github.com/WICG/webcomponents/blob/gh-pages/proposals/Template-Instantiation.md
[18]: https://web.dev/articles/constructable-stylesheets?hl=ja
[19]: https://web.dev/declarative-shadow-dom?hl=ja#polyfill
[20]: https://chromestatus.com/feature/5161240576393216
[21]: https://github.com/whatwg/html/pull/5465