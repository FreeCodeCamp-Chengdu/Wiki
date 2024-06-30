---
title: "Building containers from scratch: Layers"
date: 2024-06-07T03:14:58.423Z
authorURL: ""
originalURL: https://depot.dev/blog/building-container-layers-from-scratch
translator: ""
reviewer: ""
---

在 Depot，我们专注于为容器镜像提供最快的构建服务。我们主要通过以下方式实现这一目标：

<!-- more -->

1.  提供对强大计算和存储的即时访问。
2.  优化构建过程本身以使其尽可能快。

我们将 Depot 运行在 AWS 之上，为每个 Depot 项目使用大型 16 核机器。这些机器使用原生 Intel 和 Arm CPU，避免了多平台镜像的仿真。并且我们使用带有 NVMe SSD 的 Ceph 集群为它们提供分布式缓存存储。这一切都使得执行 `RUN` 语句变得快速，并使缓存查找和写入变得快速。

对于构建过程本身，除了对构建过程进行许多[高级优化][1]之外，我们目前正在对构建过程本身进行许多低级优化。

为了更好地理解其中一些优化，了解 OCI 容器镜像层格式本身很有帮助。

## [][2]层格式

它们只是 tar 包！

[OCI 镜像规范][3] 为容器镜像（“Docker 镜像”）定义了一个容器镜像，它是由“层”和元数据组成的集合。每一层都是一个以 [tar 归档][4] 形式存储的文件集合。

当镜像被解包时，各层相互堆叠形成容器的文件系统。从概念上讲，这可以看作是在彼此之上提取每个 tar 包，直到拥有整个文件系统，或者将所有层“联合”在一起。

以这个 Dockerfile 为例：

```yaml
FROM ubuntu:22.04
COPY hello.txt .
COPY world.txt .
```

构建时，这将生成一个包含三层的容器镜像：

1. 一个包含 Ubuntu 22.04 基础镜像文件的“层”（tar 包）
2. 一个包含 `hello.txt` 文件的“层”（tar 包）
3. 一个包含 `world.txt` 文件的“层”（tar 包）

解压生成的镜像后，容器文件系统将如下所示：

```bash
/
├── bin/
├── boot/
├── dev/
├── etc/
├── home/
├── lib/
├── media/
├── mnt/
├── opt/
├── proc/
├── root/
├── run/
├── sbin/
├── srv/
├── sys/
├── tmp/
├── usr/
├── var/
├── hello.txt
└── world.txt
```

这将包含 Ubuntu 22.04 基础层中的所有文件，以及第二层中的 `hello.txt` 文件，以及第三层中的 `world.txt` 文件。

如今，像 Docker 和 BuildKit 这样的工具会为 Dockerfile（或多阶段 Dockerfile 的目标阶段）中的每个 `RUN`、`COPY`、`ADD` 等语句生成一个 tar 层。请注意，这不是 OCI 镜像规范的要求，该规范没有指定如何创建层。

OCI 容器镜像只是一组 tar 包。

## [][5]组装 OCI 镜像

如果容器镜像层只是 tar 包，并且它们被联合在一起形成容器文件系统，那么如何才能在后面的层中删除或修改文件呢？

### [][6]处理已修改的文件

修改文件很简单：如果一个文件包含在多个层中，则最后包含该文件的层 `wins`。例如

```yaml
FROM scratch
RUN echo "hello" > example.txt
RUN echo "world" > example.txt
```

这将生成一个包含两层的镜像：

1. 一个包含 `example.txt` 文件的 tar 包，其中包含文本 `hello`
2. 一个包含 `example.txt` 文件的 tar 包，其中包含文本 `world`

解压后，第一层的 `example.txt` 文件将被第二层的 `example.txt` 文件覆盖，从而生成一个容器文件系统，其中包含一个 `example.txt` 文件，其中包含文本 `world`。

### [][7]处理已删除的文件

但是，已删除的文件该如何处理？ 例如：

```yaml
FROM scratch
RUN echo "hello" > example.txt
RUN rm example.txt
```

为此，OCI 镜像规范定义了一种特殊的文件，称为 `whiteout` 文件。

Whiteout 文件是带有特殊名称的空文件，它告诉容器运行时应从容器文件系统中删除某个路径。 Whiteout 文件有一个特殊的`.wh.`前缀，后跟要删除的文件的名称。

例如，上面示例生成的第二层将包含一个名为 `.wh.example.txt` 的零字节文件。 这指示容器运行时在解压该层时从容器文件系统中删除 `example.txt` 文件。

**注意：** 这是容器构建中安全漏洞的常见原因。如果一个文件包含在较早的层中，然后在较晚的层中被删除，则该文件的内容仍然存在于容器镜像内容中。这可能导致泄露敏感信息，例如凭据或私钥。

还有一种特殊的 whiteout 文件，称为 `opaque whiteout`，名为 `.wh..wh..opq`。该文件指示容器运行时删除与 opaque whiteout 文件位于同一目录中的所有文件和目录。例如，如果一个层包含一个名为`/example/.wh..wh..opq`的文件，则指示容器运行时删除 `/example` 目录下的所有文件和目录。

最后，层通常以 gzip 压缩的 tar 包（扩展名为`.tar.gz`）的形式分发，以节省存储空间并减少网络数据传输。请注意，该规范还支持未压缩的 tar 包（`.tar`）和 zstd 压缩的 tar 包（`.tar.zstd`）。

## [][8]Overlay filesystems（叠加文件系统）

像 containerd 或 podman 这样的容器运行时负责在运行容器之前将镜像的层（tar 包）解压到一个目录中。这被称为容器的`rootfs`或根文件系统。

实际上，对于每个启动的容器，依次将每一层解压到 rootfs 目录中，并注意应用文件修改或删除 whiteout 文件，这将非常缓慢。相反，容器运行时通常使用特殊的文件系统来有效地将各层组合成一个单一的文件系统。

其中一种文件系统是 [overlayfs][9]，它是 Linux 内核的一项功能，允许多个目录组合成一个单一目录。这是 Docker 和 Podman 使用的默认文件系统。

使用 overlayfs 时，每一层都会被解压到一个单独的目录中，然后容器运行时会告诉 Linux 内核将这些目录相互叠加挂载。然后，内核将组合后的目录作为单个目录呈现给容器运行时。

Overlayfs 本身支持 whiteout 文件的概念，因此容器运行时在挂载层时会将 OCI 镜像 whiteout 文件转换为 overlayfs whiteout 文件。

这使得每个镜像层只需解压一次，因此从同一个镜像运行多个容器非常快，并且在镜像之间共享公共基础层非常高效。Linux 内核处理将各层组合成单个文件系统的所有复杂性。

## [][10]使用 eStargz 的惰性镜像层

虽然镜像层“仅仅”是包含文件的 tar 归档文件，但可以通过扩展格式来提高存储和传输效率。[eStargz][11] 镜像格式就是一个例子。

eStargz 镜像仍然是有效的 tar 归档文件，但它们的构建方式很特殊，允许元数据文件描述压缩 tar 包中每个文件的具体位置。这允许在无需从注册表下载整个层并解压缩的情况下访问这些文件。

例如，要在 eStargz 层中找到特定文件，容器运行时将执行以下操作：

1. 获取压缩 tar 包的末尾，其中包含层中所有文件的索引（`TOC`）
2. 从 TOC 中读取文件的字节偏移量和长度
3. 从注册表中获取指定的字节范围

对于大型层来说，这可以显著节省下载大小和时间。

这种优化在启动容器时非常有用，因为容器可以在整个镜像下载完成之前就开始运行。然后，随着容器的运行和文件的访问，这些文件会从注册表中延迟加载。

这对构建镜像也很有帮助。我们构建的 [depot.ai][12] 就是一个例子。借助 depot.ai 镜像，Hugging Face 中流行的机器学习模型被打包成与 eStargz 兼容的镜像。然后，当将这些机器学习模型复制到新的容器镜像中时，Depot 只需要下载 `COPY` 请求的文件。这比下载整个模型仓库要快得多，尤其是在模型仓库包含多种格式的模型而只需要一种格式的情况下。

这类优化的其他例子还包括 AWS 的 [SOCI 快照程序][13]、[Nydus][14] 镜像格式和 Red Hat 的 [`zstd:chunked`][15]。

## [][16]优化层构建

考虑到容器镜像层本质上是 tar 包，我们正在探索各种层构建过程的优化方法，使其更快、更高效：

1. Depot 目前支持创建和使用 eStargz 镜像。
2. Depot 使用多个 CPU 核心并行压缩 tar 包。目前，我们会将层 tar 包分成多个块，并使用 gzip 并行压缩每个块，然后将这些块连接在一起。这会导致压缩后的 tar 包略大一些，但压缩速度会大大提高。
3. 我们正在研究并行构建单层的方法，构建一层意味着从文件目录创建 tar 包。目前，这是按顺序完成的，但 tar 存档头的构建可以并行化。
4. 我们正在探索构建层不需要 `Dockerfile` 的替代方法。目前，Dockerfile 是描述容器构建的最常见方式，但考虑到层是 tar 包，因此可以直接制作 tar 包。

---

优化容器构建过程还有很多方面，我们希望将来能分享更多我们的工作。如果您对这些内容感兴趣，请随时通过 [Twitter][17] 或 [Discord][18] 与我们联系。

[1]: https://twitter.com/kylegalbraith/status/1746161367290167705
[2]: #layer-format
[3]: https://github.com/opencontainers/image-spec/blob/main/spec.md
[4]: https://en.wikipedia.org/wiki/Tar_(computing)
[5]: #assembling-an-oci-image
[6]: #handling-modified-files
[7]: #handling-removed-files
[8]: #overlay-filesystems
[9]: https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html
[10]: #lazy-image-layers-with-estargz
[11]: https://github.com/containerd/stargz-snapshotter/blob/main/docs/estargz.md
[12]: https://depot.ai/
[13]: https://github.com/awslabs/soci-snapshotter
[14]: https://nydus.dev/
[15]: https://www.redhat.com/sysadmin/faster-container-image-pulls
[16]: #optimizing-layer-construction
[17]: https://twitter.com/jacobwgillespie
[18]: https://discord.gg/MMPqYSgDCg
