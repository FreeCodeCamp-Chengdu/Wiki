---
title: Understanding Container Image Layers
date: 2024-06-07T03:12:52.216Z
author: Ken Muse
authorURL: https://github.com/kenmuse
originalURL: https://www.kenmuse.com/blog/understanding-container-image-layers/
translator: ""
reviewer: ""
---


**发布时间:** [6 月 27][5], [2024][6] **阅读时间:** 9 分钟

---

容器非常了不起。它们允许简单的进程像虚拟机一样运行。这种优雅的背后是一套模式和实践，最终使一切都能正常工作。设计的根源在于*层*。层是存储和分发容器化文件系统内容的基本方式。这种设计既非常简单，同时又非常强大。在今天的文章中，我将解释什么是层，以及它们在概念上是如何工作的。

## 构建分层镜像

当你创建一个镜像时，你通常会使用一个 `Dockerfile` 来定义容器的内容。它包含一系列命令，例如：

```dockerfile
FROM scratch
RUN echo "hello" > /work/message.txt
COPY content.txt /work/content.txt
RUN rm -rf /work/message.txt
```

在底层，容器引擎会按顺序执行这些命令，为每个命令创建一个 `层`。 但这究竟是如何实现的呢？ 最简单的理解方式是将每一层都视为一个目录，其中包含所有已修改的文件。

让我们通过一个可能的实现方法的例子来逐步解释。

1.  `FROM scratch` 表示此容器从零内容开始。 这是第一层，它可以用一个空目录 `/img/layer1` 来表示。
2.  创建一个第二个目录 `/img/layer2` 并将 `/img/layer1` 中的所有内容复制到其中。 然后，执行 Dockerfile 中的下一条命令（将一个文件写入 `/work/message.txt`）。 这些内容被写入 `/img/layer2/work/message.txt`。 这是第二层。
3.  创建一个第三个目录 `/img/layer3`，将 `img/layer2` 中的所有内容复制到其中。 下一个 Dockerfile 命令需要将主机的 `content.txt` 复制到该目录。 该文件被写入 `/img/layer3/work/content.txt`。 这是第三层。
4.  最后，创建一个第四个目录 `/img/layer4`，将 `img/layer3` 中的所有内容复制到其中。 下一条命令删除消息文件 `img/layer4/work/message.txt`。 这是第四层。

为了共享这些层，最简单的方法是为每个目录创建一个压缩的 `.tar.gz` 文件。为了减小总文件大小，任何未经修改、只是从上一层复制的文件都会被删除。为了明确何时删除了文件，可以使用 `whiteout 文件` 作为占位符。该文件只需在原始文件名前加上前缀 `.wh.`。例如，第四层会将删除的文件替换为名为 `.wh.message.txt` 的占位符。当一个层被解包时，任何以 `.wh.` 开头的文件都可以被删除。

继续我们的例子，压缩文件将包含：

| File                                                | Contents                                                            |
| --------------------------------------------------- | ------------------------------------------------------------------- |
| `layer1.tar.gz`                                     | 空文件                                                          |
| `layer2.tar.gz`                                     | 包含 `/work/message.txt`                                        |
| `layer3.tar.gz`                                     | 包含 `/work/content.txt` （因为 `message.txt` 文件没有修改） |
| `layer4.tar.gz`                                     | 包含 `/work/.wh.message.txt` （因为 `message.txt` 删除了） |
| 文件 `content.txt` 没有被修改，所以没有被包含在内。 |

以这种方式构建大量镜像会导致大量的 `layer1` 目录。为了确保名称的唯一性，压缩文件的命名基于内容的摘要。这类似于 Git 的工作方式。它的好处是在识别文件下载过程中任何损坏的同时，还能识别相同的内容。如果内容的摘要（哈希值）与文件名不匹配，则文件已损坏。

为了使结果可重复，还需要一个文件来解释如何对层进行排序（清单）。清单会标识要下载哪些文件以及解包它们的顺序。这使得能够重新创建目录结构。它还提供了一个重要的好处：层可以在镜像之间重复使用和共享。这最大限度地减少了本地存储需求。

在实践中，还有更多可用的优化。例如，`FROM scratch` 实际上意味着没有父层，所以我们的示例实际上是从 `layer2` 的内容开始的。引擎还可以查看构建中使用的文件，以确定是否需要重新创建层。这是层缓存的基础，它最大限度地减少了构建或重新创建层的需要。作为额外的优化，不依赖于前一层的层可以使用 `COPY --link` 来指示该层不需要删除或修改前一层中的任何文件。这允许压缩层文件与其他步骤并行创建。

## 快照

在容器可以运行之前，它需要一个文件系统来挂载。本质上，它需要一个包含所有需要可用的文件的目录。压缩的层文件包含文件系统的组件，但它们不能直接挂载和使用。相反，它们需要被解包并组织成一个文件系统。这个解包后的目录被称为*快照*（好吧，它是具有该名称的几样东西之一 😄）。

创建快照的过程与镜像构建相反。它首先下载清单并构建要下载的层列表。对于每一层，都会创建一个包含该层父层内容的目录。此目录称为*活动快照*。接下来，*差异应用器*负责解压缩压缩的层文件并将更改应用于活动快照。然后，生成的目录称为*已提交快照*。最终提交的快照是作为容器文件系统挂载的快照。

使用我们之前的例子：

1.  初始层 `FROM scratch` 意味着我们可以从下一层和一个空目录开始。没有父层。
2.  创建一个 `layer2` 的目录。这个空目录现在是一个活动快照。下载文件 `layer2.tar.gz`，验证（通过将摘要与文件名进行比较），并解压缩到目录中。结果是一个包含 `/work/message.txt` 的目录。这是第一个提交的快照。
3.  创建一个 `layer3` 的目录，并将 `layer2` 的内容复制到其中。这是一个新的活动快照。下载文件 `layer3.tar.gz`，验证并解压缩。结果是一个包含 `/work/message.txt` 和 `/work/content.txt` 的目录。这是第二个提交的快照。
4.  创建一个 `layer4` 的目录，并将 `layer3` 的内容复制到其中。下载文件 `layer4.tar.gz`，验证并解压缩。差异应用器识别出删除文件 `/work/.wh.message.txt`，并删除 `/work/message.txt`。只剩下 `/work/content.txt`。这是第三个提交的快照。
5.  由于 `layer4` 是最后一层，因此它是容器的基础。为了使其支持读写操作，会创建一个新的快照目录，并将 `layer4` 的内容复制到其中。此目录作为容器的文件系统挂载。运行中的容器所做的任何更改都将发生在此目录中。

如果这些目录中的任何一个已经存在，则表明另一个镜像具有相同的依赖项。因此，引擎可以跳过下载和差异应用器。它可以按原样使用该层。在实践中，这些目录和文件中的每一个都根据内容的摘要进行命名，以便更容易识别。例如，一组快照可能如下所示：

```text
1/var/path/to/snapshots/blobs
2└─ sha256
3   ├─ 635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c
4   ├─ 9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1
5   ├─ fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f
6   └─ fb124ec4f943662ecf7aac45a43b096d316f1a6833548ec802226c7b406154e9
```

或者，换句话说:

| 镜像                                                                   | 父镜像                                                                  |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| sha256:635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c |                                                                         |
| sha256:9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1 | sha256:635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c |
| sha256:fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f | sha256:9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1 |
| sha256:fb124ec4f943662ecf7aac45a43b096d316f1a6833548ec802226c7b406154e9 | sha256:fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f |

实际的快照系统支持插件，可以改善其中一些行为。例如，它可以允许预先组合和解压缩快照，从而加快进程。这允许将快照存储在远程。它还允许进行特殊优化，例如按需下载所需的文件和层。

## 覆盖层

虽然挂载很容易，但我们刚才描述的快照方法会创建大量文件变更和重复文件。这会减慢第一次启动容器的速度并浪费空间。值得庆幸的是，这是容器化过程中可以通过文件系统处理的众多方面之一。Linux 本身支持将目录挂载为覆盖层，为我们实现了大部分过程。

在 Linux 中（或以 `--privileged` 或 `--cap-add=SYS_ADMIN` 运行的 Linux 容器中）：

1.  创建一个 `tmpfs` 挂载点（基于内存的文件系统，将用于探索覆盖过程）

    ```bash
    mkdir /tmp/overlay
    mount -t tmpfs tmpfs /tmp/overlay
    ```

2.  创建我们流程所需的目录。我们将使用 `lower` 作为下层（父层），`upper` 作为上层（子层），`work` 作为文件系统的工作目录，`merged` 包含合并后的文件系统。

    ```bash
    mkdir /tmp/overlay/{lower,upper,work,merged}
    ```

3.  为实验创建一些文件。你也可以选择在 `upper` 中添加文件。

    ```bash
    cd /tmp/overlay
    echo hello > lower/hello.txt
    echo "I'm only here for a moment" > lower/delete-me.txt
    echo message > upper/upper-message.txt
    ```

4.  将这些目录挂载为 `overlay` 类型的文件系统。这将在 `merged` 目录中创建一个新的文件系统，其中包含 `lower` 和 `upper` 目录的组合内容。 `work` 目录将用于跟踪文件系统的更改。

    ```bash
    mount -t overlay overlay -o lowerdir=lower,upperdir=upper,workdir=work merged
    ```

5.  探索文件系统。你会注意到 `merged` 包含 `upper` 和 `lower` 的组合内容。然后，进行一些更改：

    ```bash
    rm -rf merged/delete-me.txt
    echo "I'm new" > merged/new.txt
    echo world >> merged/hello.txt
    ```

6.  正如预期的那样，`delete-me.txt` 已从 `merged` 中删除，并在同一目录中创建了一个新文件 `new.txt`。如果你对这些目录执行 `tree` 命令，你会发现一些有趣的事情：

    ```txt
       |-- lower
       |   |-- delete-me.txt
       |   `-- hello.txt
       |-- merged
       |   |-- hello.txt
       |   |-- new.txt
       |   `-- upper-message.txt
       |-- upper
       |   |-- delete-me.txt
       |   |-- hello.txt
       |   |-- new.txt
       |   `-- upper-message.txt
    ```

    运行 `ls -l upper` 显示：

    ```text
    total 12
    c--------- 2 root root 0, 0 Jan 20 00:17 delete-me.txt
    -rw-r--r-- 1 root root   12 Jan 20 00:20 hello.txt
    -rw-r--r-- 1 root root    8 Jan 20 00:17 new.txt
    -rw-r--r-- 1 root root    8 Jan 20 00:17 upper-message.txt
    ```

虽然 `merged` 显示了我们更改的效果，但 `upper`（作为父层）存储的更改类似于我们手动过程中的示例。它包含新文件 `new.txt` 和修改后的 `hello.txt`。它还创建了一个 whiteout 文件。对于 overlay 文件系统，这涉及将文件替换为字符设备（以及设备号 0，0）。简而言之，它拥有打包目录所需的一切！

你可以看到这种方法如何也可以用于实现快照系统。 `mount` 命令本身可以接受一个以冒号 (`:`) 分隔的 `lowerdir` 路径列表，所有这些路径都合并到一个文件系统中。这是现代容器本质的一部分——容器是使用原生操作系统特性组成的。

这就是创建一个基本系统的全部内容。事实上，Kubernetes（以及最近发布的 Docker Desktop 4.27.0）使用的 `containerd` 运行时使用类似的方法来构建和管理其镜像（更详细的内容在 [Content Flow][1] 中介绍）。希望这有助于揭开容器镜像工作方式的神秘面纱！

[1]: https://github.com/containerd/containerd/blob/main/docs/content-flow.md
