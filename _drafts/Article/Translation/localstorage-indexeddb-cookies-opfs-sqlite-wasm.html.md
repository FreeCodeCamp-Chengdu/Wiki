---
title: LocalStorage vs. IndexedDB vs. Cookies vs. OPFS vs. WASM-SQLite
date: 2024-11-29T02:51:26.286Z
authorURL: ""
originalURL: https://rxdb.info/articles/localstorage-indexeddb-cookies-opfs-sqlite-wasm.html
translator: ""
reviewer: ""
---

当你正在构建一个Web应用程序时，你可能希望**将数据存储在用户的浏览器中**。也许你只需要存储一些小标志，或者你甚至需要一个完整的数据库。

我们构建的Web应用程序类型已经发生了显著变化。在Web的早期，我们提供静态的HTML文件。然后我们提供动态渲染的HTML，后来我们构建了**单页应用程序**，这些应用程序在客户端运行大部分逻辑。而在未来几年，你可能希望构建所谓的[本地优先应用程序][4]，这些应用程序在客户端处理大量复杂的数据操作，甚至在离线时也能工作，这为你提供了构建**零延迟**用户交互的机会。

在Web的早期，**Cookies**是存储小型键值对的唯一选择。但JavaScript和浏览器已经显著发展，并添加了更好的存储API，为更大、更复杂的数据操作铺平了道路。

在本文中，我们将深入探讨在浏览器中存储和查询数据的各种技术。我们将探索传统方法，如**Cookies**、**localStorage**、**WebSQL**、**IndexedDB**，以及较新的解决方案，如**OPFS**和**通过WebAssembly的SQLite**。我们将比较这些技术的功能和限制，并通过性能测试揭示在Web应用程序中使用各种方法读写数据的速度。

注意

你正在[RxDB][5]文档中阅读本文。RxDB是一个JavaScript数据库，具有不同的存储适配器，可以利用不同的存储API。**自2017年以来**，我大部分时间都在与这些API打交道，进行性能测试，并构建[技巧][6]和插件，以达到浏览器数据库操作速度的极限。

[![Image 1: JavaScript Database][1]][5]

现代浏览器中可用的存储API
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

首先，让我们简要概述不同的API、它们的预期用例和历史：

### 什么是Cookies

Cookies最早由[网景公司于1994年][7]引入。Cookies存储小型的键值数据，主要用于会话管理、个性化和跟踪。Cookies可以具有多个安全设置，如生存时间或`domain`属性，以在多个子域之间共享Cookies。

Cookies的值不仅存储在客户端，还会随**每个HTTP请求**发送到服务器。这意味着我们不能在Cookie中存储太多数据，但Cookie的访问性能与其他方法相比仍然值得关注。特别是因为Cookies是Web的重要基础功能，许多性能优化已经完成，甚至在这些日子里，仍然有进展，如Chromium的[共享内存版本控制][8]或异步的[CookieStore API][9]。

### 什么是LocalStorage

[localStorage API][10]最早作为[WebStorage规范的一部分于2009年][11]提出。LocalStorage提供了一个简单的API，用于在Web浏览器中存储键值对。它具有`setItem`、`getItem`、`removeItem`和`clear`方法，这些方法是你从键值存储中所需的一切。LocalStorage仅适用于存储少量需要在会话之间持久化的数据，并且它[受到5MB存储上限的限制][12]。存储复杂数据只能通过将其转换为字符串来实现，例如使用`JSON.stringify()`。该API不是异步的，这意味着它在执行操作时会完全阻塞你的JavaScript进程。因此，在其上运行繁重的操作可能会阻止你的UI渲染。

> 还有**SessionStorage** API。关键区别在于localStorage数据会无限期持久化，直到显式清除，而sessionStorage数据在浏览器标签或窗口关闭时会被清除。

### 什么是IndexedDB

IndexedDB最早作为"Indexed Database API"[于2015年][13]引入。

[IndexedDB][14]是一个用于存储大量结构化JSON数据的低级API。虽然该API有点难以使用，但IndexedDB可以利用索引和异步操作。它缺乏对复杂查询的支持，只允许在索引上进行迭代，这使得它更像是其他库的基础层，而不是一个完全成熟的数据库。

2018年，IndexedDB 2.0版本[被引入][15]。这增加了一些重大改进。最显著的是`getAll()`方法，它在获取大量JSON文档时显著提高了性能。

IndexedDB [3.0版本][16]正在开发中，包含了许多改进。最重要的是添加了基于`Promise`的调用，这使得现代JS功能如`async/await`更加有用。

### 什么是OPFS

[Origin Private File System][17]（OPFS）是一个[相对较新][18]的API，允许Web应用程序直接在浏览器中存储大文件。它专为数据密集型应用程序设计，这些应用程序希望在模拟的文件系统中写入和读取**二进制数据**。

OPFS可以在两种模式下使用：

*   在[主线程][19]上异步使用
*   或者在WebWorker中使用更快的异步访问，通过`createSyncAccessHandle()`方法。

由于只能处理二进制数据，OPFS被设计为库开发者的基础文件系统。当你构建一个"普通"应用程序时，你不太可能直接使用OPFS，因为它太复杂了。这仅适用于存储纯文件（如图像），而不是高效地存储和查询[JSON数据][20]。我为RxDB构建了一个[基于OPFS的存储][17]，具有适当的索引和查询功能，这花了我几个月的时间。

### 什么是WASM SQLite

![Image 2: WASM SQLite][2]

[WebAssembly][21]（Wasm）是一种二进制格式，允许在Web上执行高性能代码。Wasm在2017年被添加到主要浏览器中，这为在浏览器中运行的内容开辟了广泛的机会。你可以将本地库编译为WebAssembly，只需稍作调整即可在客户端运行。WASM代码可以发送到浏览器应用程序，并且通常比JavaScript运行得更快，但仍然比本地代码[慢约10%][22]。

许多人开始在浏览器中使用编译后的SQLite作为数据库，这就是为什么将这种设置与本地API进行比较也是有意义的。

SQLite的编译字节码大小约为[938.9 kB][23]，必须在首次页面加载时由用户下载和解析。WASM不能直接访问浏览器中的任何持久存储API。相反，它需要数据从WASM流向主线程，然后可以放入浏览器API之一。这是通过所谓的[VFS（虚拟文件系统）适配器][24]完成的，这些适配器处理从SQLite到其他任何东西的数据访问。

### 什么是WebSQL

WebSQL**曾经**是一个Web API，[于2009年][25]引入，允许浏览器使用SQL数据库进行客户端存储，基于SQLite。其想法是让开发者能够使用SQL在客户端存储和查询数据，类似于服务器端数据库。WebSQL在近年已**从浏览器中移除**，原因如下：

*   WebSQL没有标准化，并且基于SQLite源代码的单一特定实现的API很难成为标准。
*   WebSQL要求浏览器使用[特定版本][26]的SQLite（版本3.6.19），这意味着每当SQLite有任何更新或错误修复时，如果不破坏Web，就不可能将其添加到WebSQL中。
*   像Firefox这样的主要浏览器从未支持WebSQL。

因此，在接下来的内容中，我们将**忽略WebSQL**，即使通过设置特定的浏览器标志或使用旧版本的Chromium，仍然可以对其进行测试。

* * *

功能比较
-------------------------------------------------------------------------------------------------------------------------------------------------------------

现在你已经了解了这些API的基本概念，让我们比较一些特定的功能，这些功能对于使用RxDB和基于浏览器的存储的人来说非常重要。

### 存储复杂的JSON文档

当你在Web应用程序中存储数据时，大多数情况下你希望存储复杂的JSON文档，而不仅仅是存储在服务器端数据库中的`整数`和`字符串`等"普通"值。

*   只有IndexedDB原生支持JSON对象。
*   使用SQLite WASM，你可以[存储JSON][27]在`text`列中，自版本3.38.0（2022-02-22）起，甚至可以在其上运行深度查询，并使用单个属性作为索引。

其他API只能存储字符串或二进制数据。当然，你可以使用`JSON.stringify()`将任何JSON对象转换为字符串，但在API中没有JSON支持会使查询变得复杂，并且多次运行`JSON.stringify()`可能会导致性能问题。

### 多标签支持

构建Web应用程序与[Electron][28]或[React-Native][29]相比，一个很大的区别是用户会**同时打开和关闭多个浏览器标签**。因此，你不仅有一个JavaScript进程在运行，而且可能存在多个进程，并且它们可能需要在彼此之间共享状态更改，以避免向用户显示**过时的数据**。

> 如果你的用户的肌肉记忆让他们在使用你的网站时将左手放在**F5**键上，那么你做错了什么！

并非所有存储API都支持在标签之间自动共享写入事件。

只有localStorage有一种通过API本身在标签之间自动共享写入事件的方式，即[storage-event][30]，它可以用于观察更改。

```
// localStorage可以通过storage事件观察更改。// 这个功能在IndexedDB和其他API中缺失addEventListener("storage", (event) => {});
```

曾经有[实验性的IndexedDB观察者API][31]用于Chrome，但提案仓库已被归档。

为了解决这个问题，有两种解决方案：

*   第一种选择是使用[BroadcastChannel API][32]，它可以在浏览器标签之间发送消息。因此，每当你向存储写入数据时，你还可以向其他标签发送通知，告知这些更改。这是最常见的解决方法，RxDB也使用了这种方法。注意，还有[WebLocks API][33]，它可以用于在浏览器标签之间实现互斥锁。
*   另一种解决方案是使用[SharedWorker][34]，并在该worker中执行所有写入操作。所有浏览器标签都可以订阅来自该**单一**SharedWorker的消息，并了解更改。

### 索引支持

数据库与将数据存储在普通文件中的最大区别在于，数据库以允许通过索引运行操作的格式写入数据，从而支持快速查询。在我们的技术列表中，只有**IndexedDB**和**WASM SQLite**原生支持索引。理论上，你可以在任何存储（如localStorage或OPFS）上构建索引，但你可能不想自己动手。

例如，在IndexedDB中，我们可以通过给定的索引范围获取一批文档：

```
// 查找所有价格在10到50之间的产品const keyRange = IDBKeyRange.bound(10, 50);const transaction = db.transaction('products', 'readonly');const objectStore = transaction.objectStore('products');const index = objectStore.index('priceIndex');const request = index.getAll(keyRange);const result = await new Promise((res, rej) => {  request.onsuccess = (event) => res(event.target.result);  request.onerror = (event) => rej(event);});
```

注意，IndexedDB有一个限制，即[不支持布尔值的索引][35]。你只能索引字符串和数字。为了解决这个问题，你必须在存储数据时将布尔值转换为数字，并在读取时转换回来。

### WebWorker支持

当运行繁重的数据操作时，你可能希望将处理从JavaScript主线程中移出。这确保了我们的应用程序保持响应和快速，而处理可以在后台并行运行。在浏览器中，你可以使用[WebWorker][36]、[SharedWorker][34]或[ServiceWorker][37] API来实现这一点。在RxDB中，你可以使用[WebWorker][38]或[SharedWorker][39]插件将存储移动到worker中。

最常见的用例是生成一个**WebWorker**，并在第二个JavaScript进程中执行大部分工作。worker是从一个单独的JavaScript文件（或base64字符串）生成的，并通过`postMessage()`与主线程通信。

不幸的是，**LocalStorage**和**Cookies**[不能在WebWorker或SharedWorker中使用][40]，这是由于设计和安全限制。WebWorkers在与主浏览器线程分离的全局上下文中运行，因此不能执行可能影响主线程的操作。它们无法直接访问某些Web API，如DOM、localStorage或cookies。

其他所有内容都可以在WebWorker中使用。OPFS的快速版本`createSyncAccessHandle`方法**只能**[在WebWorker中使用][41]，而**不能在主线程中使用**。这是因为返回的`AccessHandle`的所有操作都是**非异步的**，因此会阻塞JavaScript进程，所以你肯定不希望在主线程中执行这些操作并阻塞一切。

* * *

存储大小限制
----------------------------------------------------------------------------------------------------------------------------------------------------------------

*   **Cookies** 在 [RFC-6265][42] 中被限制为大约 `4 KB` 的数据。由于存储的 Cookies 会随每个 HTTP 请求发送到服务器，这个限制是合理的。你可以在这里测试浏览器的 Cookies 限制[43]。需要注意的是，你永远不应该填满 `4 KB` 的 Cookies，因为你的 Web 服务器不会接受过长的请求头，并会以 `HTTP ERROR 431 - 请求头字段过大` 拒绝请求。一旦达到这个点，你甚至无法向用户提供更新的 JavaScript 来清理 Cookies，用户将被锁定，直到手动清理 Cookies。
    
*   **LocalStorage** 的存储大小限制因浏览器而异，但通常在 4 MB 到 10 MB 之间。你可以在这里测试你的 localStorage 大小限制[44]。
    
    *   Chrome/Chromium/Edge: 每个域名 5 MB
    *   Firefox: 每个域名 10 MB
    *   Safari: 每个域名 4-5 MB（不同版本略有差异）
*   **IndexedDB** 没有像 localStorage 那样的固定大小限制。IndexedDB 的最大存储大小取决于浏览器的实现。上限通常基于用户设备上的可用磁盘空间。在 Chromium 浏览器中，它可以使用高达 80% 的总磁盘空间。你可以通过调用 `await navigator.storage.estimate()` 来估算存储大小限制。通常你可以存储千兆字节的数据，可以在这里尝试[45]。请注意，我们有一篇关于 [IndexedDB 存储最大限制][46] 的完整文章，涵盖了该主题。
    
*   **OPFS** 的存储大小限制与 IndexedDB 相同。它的限制取决于可用磁盘空间。也可以在这里测试[45]。
    

* * *

性能比较
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

现在我们已经回顾了每种存储方法的功能，让我们深入探讨性能比较，重点关注初始化时间、读写延迟和批量操作。

请注意，我们只运行了简单的测试，对于你的应用程序中的特定用例，结果可能会有所不同。此外，我们只在 Google Chrome（版本 128.0.6613.137）中比较了性能。Firefox 和 Safari 有类似但**不完全相同**的性能模式。你可以在这个 [GitHub 仓库][47] 中在自己的机器上运行测试。对于所有测试，我们将网络限制为德国平均互联网速度。（下载：135,900 kbit/s，上传：28,400 kbit/s，延迟：125ms）。此外，所有测试都存储一个"平均"的 JSON 对象，可能需要根据存储方式将其字符串化。我们还只测试了按 ID 存储文档的性能，因为某些技术（Cookies、OPFS 和 localStorage）不支持索引范围操作，因此比较这些操作的性能没有意义。

### 初始化时间

在存储任何数据之前，许多 API 需要一个设置过程，比如创建数据库、启动 WebAssembly 进程或下载额外的内容。为了确保你的应用程序快速启动，初始化时间非常重要。

localStorage 和 Cookies 的 API 没有任何设置过程，可以直接使用。IndexedDB 需要打开一个数据库并在其中创建一个存储。WASM SQLite 需要下载一个 WASM 文件并处理它。OPFS 需要下载并启动一个 worker 文件，并初始化虚拟文件系统目录。

以下是存储第一比特数据所需的时间测量结果：

| 技术 | 时间（毫秒） |
| --- | --- |
| IndexedDB | 46 |
| OPFS 主线程 | 23 |
| OPFS WebWorker | 26.8 |
| WASM SQLite（内存） | 504 |
| WASM SQLite（IndexedDB） | 535 |

这里我们可以注意到几点：

*   打开一个新的 IndexedDB 数据库并创建一个存储需要的时间出人意料地长
*   将数据从主线程发送到 WebWorker OPFS 的延迟开销约为 4 毫秒。这里我们只发送了最少的数据来初始化 OPFS 文件句柄。如果处理更多数据，这个延迟是否会增加将是一个有趣的问题。
*   下载并解析 WASM SQLite 并创建一个表大约需要半秒钟。使用 IndexedDB VFS 持久化存储数据还会额外增加 31 毫秒。在启用缓存并已准备好表的情况下重新加载页面会稍微快一些，为 420 毫秒（内存）。

### 少量写操作的延迟

接下来，我们测试少量写操作的延迟。这在你有许多独立发生的小数据更改时非常重要，比如从 WebSocket 流式传输数据或持久化伪随机发生的事件（如鼠标移动）。

| 技术 | 时间（毫秒） |
| --- | --- |
| Cookies | 0.058 |
| LocalStorage | 0.017 |
| IndexedDB | 0.17 |
| OPFS 主线程 | 1.46 |
| OPFS WebWorker | 1.54 |
| WASM SQLite（内存） | 0.17 |
| WASM SQLite（IndexedDB） | 3.17 |

这里我们可以注意到几点：

*   LocalStorage 的写延迟最低，仅为 0.017 毫秒每次写操作。
*   IndexedDB 的写操作比 localStorage 慢约 10 倍。
*   将数据发送到 WASM SQLite 进程并通过 IndexedDB 持久化存储的速度较慢，每次写操作超过 3 毫秒。

OPFS 操作大约需要 1.5 毫秒将 JSON 数据写入每个文档的一个文件。我们可以看到，先将数据发送到 WebWorker 会稍微慢一些，这是由于在两端序列化和反序列化数据的开销。如果我们不为每个文档创建一个 OPFS 文件，而是将所有内容追加到单个文件中，性能模式会发生显著变化。然后，来自 `createSyncAccessHandle()` 的更快文件句柄每次写操作仅需约 1 毫秒。但这需要以某种方式记住每个文档的存储位置。因此，在我们的测试中，我们将继续使用每个文档一个文件的方式。

### 少量读操作的延迟

现在我们已经存储了一些文档，让我们测量按 `id` 读取单个文档所需的时间。

| 技术 | 时间（毫秒） |
| --- | --- |
| Cookies | 0.132 |
| LocalStorage | 0.0052 |
| IndexedDB | 0.1 |
| OPFS 主线程 | 1.28 |
| OPFS WebWorker | 1.41 |
| WASM SQLite（内存） | 0.45 |
| WASM SQLite（IndexedDB） | 2.93 |

这里我们可以注意到几点：

*   LocalStorage 的读取速度**非常非常快**，仅为 0.0052 毫秒每次读操作。
*   其他技术的读取速度与其写延迟相似。

### 大批量写操作

接下来，我们进行一些大批量操作，一次性写入 200 个文档。

| 技术 | 时间（毫秒） |
| --- | --- |
| Cookies | 20.6 |
| LocalStorage | 5.79 |
| IndexedDB | 13.41 |
| OPFS 主线程 | 280 |
| OPFS WebWorker | 104 |
| WASM SQLite（内存） | 19.1 |
| WASM SQLite（IndexedDB） | 37.12 |

这里我们可以注意到几点：

*   将数据发送到 WebWorker 并通过更快的 OPFS API 运行的速度大约快两倍。
*   WASM SQLite 在批量操作中的表现优于其单次写延迟。这是因为如果一次性发送数据到 WASM 并返回，而不是每次文档发送一次，速度会更快。

### 大批量读操作

现在让我们一次性读取 100 个文档。

| 技术 | 时间（毫秒） |
| --- | --- |
| Cookies | 6.34 |
| LocalStorage | 0.39 |
| IndexedDB | 4.99 |
| OPFS 主线程 | 54.79 |
| OPFS WebWorker | 25.61 |
| WASM SQLite（内存） | 3.59 |
| WASM SQLite（IndexedDB） | 5.84（无缓存时为35毫秒） |

这里我们可以注意到几点：

*   在 OPFS WebWorker 中读取多个文件的速度比主线程模式**快大约两倍**。
*   WASM SQLite 的速度出人意料地快。进一步检查发现，WASM SQLite 进程将文档缓存在内存中，这提高了在写入后直接读取相同数据时的延迟。如果在写入和读取之间重新加载浏览器标签，查找 100 个文档大约需要 **35 毫秒**。

性能结论
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

*   LocalStorage 确实非常快，但请记住它有一些缺点：
    *   它会阻塞主 JavaScript 进程，因此不应用于大型批量操作。
    *   只能进行键值对操作，当你需要对数据进行基于索引的范围查询时，无法高效使用它。
*   OPFS 在 WebWorker 中使用 `createSyncAccessHandle()` 方法时，比直接在主线程中使用要快得多。
*   SQLite WASM 可以很快，但你必须先下载完整的二进制文件并启动它，这大约需要半秒钟。如果你的应用程序启动一次并长时间使用，这可能无关紧要。但对于在多个浏览器标签中多次打开和关闭的 Web 应用程序来说，这可能是一个问题。

* * *

可能的改进
----------------------------------------------------------------------------------------------------------------------------------------------------------------------

有许多可能的改进和性能优化技巧可以加速操作：

*   对于 IndexedDB，我在这里整理了一份[性能优化技巧列表][6]。例如，你可以在多个数据库和 WebWorker 之间进行分片，或者使用自定义的索引策略。
*   OPFS 在每份文档写入一个文件时速度较慢。但你不需要这样做，而是可以像普通数据库那样将所有内容存储在单个文件中。这可以显著提高性能，就像 RxDB 的 [OPFS RxStorage][17] 所做的那样。
*   你可以混合使用多种技术来优化不同的场景。例如，在 RxDB 中有一个 [localstorage 元数据优化器][48]，它将初始元数据存储在 localStorage 中，而将"普通"文档存储在 IndexedDB 中。这提高了初始启动时间，同时仍然能够高效地查询文档。
*   RxDB 中有一个 [内存映射][49] 存储插件，它直接将数据映射到内存中。与 SharedWorker 结合使用可以显著提高页面加载和查询速度。
*   在存储数据之前[压缩][50]数据可能会提高某些存储的性能。
*   通过[分片][51]在[多个 WebWorker][38] 之间分配工作可以利用用户设备的全部性能，从而提高性能。

在这里你可以看到各种 RxDB 存储实现的[性能比较][52]，这为实际性能提供了更好的视角：

![图 3: RxStorage 性能 - 浏览器][3]

未来的改进
----------------------------------------------------------------------------------------------------------------------------------------------------------------

你正在 2024 年阅读这篇文章，但网络并没有停滞不前。浏览器很可能会增强以支持更快、更好的数据操作。

*   目前还没有办法直接从 WebAssembly 进程中访问持久存储。如果未来这一点发生变化，在浏览器中运行 SQLite（或类似的数据库）可能是最佳选择。
*   在主线程和 WebWorker 之间发送数据速度较慢，但未来可能会有所改进。这里有一篇[关于为什么 `postMessage()` 很慢的好文章][53]。
*   IndexedDB 最近[支持][54]了存储桶（仅限 Chrome），这可能会提高性能。

后续
----------------------------------------------------------------------------------------------------------------------------------

*   分享我的[公告推文][55] --\>
*   在 [GitHub 仓库][47] 中复现基准测试
*   学习如何使用 RxDB 的 [RxDB 快速入门][56]
*   查看 [RxDB GitHub 仓库][57] 并留下星星 ⭐

[1]: https://rxdb.info/files/logo/rxdb_javascript_database.svg
[2]: https://rxdb.info/files/icons/sqlite.svg
[3]: https://rxdb.info/files/rx-storage-performance-browser.png
[4]: https://rxdb.info/offline-first.html
[5]: https://rxdb.info/
[6]: https://rxdb.info/slow-indexeddb.html
[7]: https://www.baekdal.com/thoughts/the-original-cookie-specification-from-1997-was-gdpr-compliant/
[8]: https://blog.chromium.org/2024/06/introducing-shared-memory-versioning-to.html
[9]: https://developer.mozilla.org/en-US/docs/Web/API/Cookie_Store_API
[10]: https://rxdb.info/articles/localstorage.html
[11]: https://www.w3.org/TR/2009/WD-webstorage-20090423/#the-localstorage-attribute
[12]: https://rxdb.info/articles/localstorage.html#understanding-the-limitations-of-local-storage
[13]: https://www.w3.org/TR/IndexedDB/#sotd
[14]: https://rxdb.info/rx-storage-indexeddb.html
[15]: https://hacks.mozilla.org/2016/10/whats-new-in-indexeddb-2-0/
[16]: https://w3c.github.io/IndexedDB/
[17]: https://rxdb.info/rx-storage-opfs.html
[18]: https://caniuse.com/mdn-api_filesystemfilehandle_createsyncaccesshandle
[19]: https://rxdb.info/rx-storage-opfs.html#using-opfs-in-the-main-thread-instead-of-a-worker
[20]: https://rxdb.info/articles/json-based-database.html
[21]: https://webassembly.org/
[22]: https://www.usenix.org/conference/atc19/presentation/jangda
[23]: https://sqlite.org/download.html
[24]: https://www.sqlite.org/vfs.html
[25]: https://www.w3.org/TR/webdatabase/
[26]: https://developer.chrome.com/blog/deprecating-web-sql#reasons_for_deprecating_web_sql
[27]: https://www.sqlite.org/json1.html
[28]: https://rxdb.info/electron-database.html
[29]: https://rxdb.info/react-native-database.html
[30]: https://rxdb.info/articles/localstorage.html#localstorage-vs-indexeddb
[31]: https://stackoverflow.com/a/33270440
[32]: https://github.com/pubkey/broadcast-channel
[33]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API
[34]: https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker
[35]: https://github.com/w3c/IndexedDB/issues/76
[36]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
[37]: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
[38]: https://rxdb.info/rx-storage-worker.html
[39]: https://rxdb.info/rx-storage-shared-worker.html
[40]: https://stackoverflow.com/questions/6179159/accessing-localstorage-from-a-webworker
[41]: https://rxdb.info/rx-storage-opfs.html#opfs-limitations
[42]: https://datatracker.ietf.org/doc/html/rfc6265#section-6.1
[43]: http://www.ruslog.com/tools/cookies.html
[44]: https://arty.name/localstorage.html
[45]: https://demo.agektmr.com/storage/
[46]: https://rxdb.info/articles/indexeddb-max-storage-limit.html
[47]: https://github.com/pubkey/localstorage-indexeddb-cookies-opfs-sqlite-wasm
[48]: https://rxdb.info/rx-storage-localstorage-meta-optimizer.html
[49]: https://rxdb.info/rx-storage-memory-mapped.html
[50]: https://rxdb.info/key-compression.html
[51]: https://rxdb.info/rx-storage-sharding.html
[52]: https://rxdb.info/rx-storage-performance.html
[53]: https://surma.dev/things/is-postmessage-slow/
[54]: https://developer.chrome.com/blog/maximum-idb-performance-with-storage-buckets
[55]: https://x.com/rxdbjs/status/1846145062847062391
[56]: https://rxdb.info/quickstart.html
[57]: https://github.com/pubkey/rxdb
