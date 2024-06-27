---
title: Recent Docker BuildKit Features You're Missing Out On
date: 2024-05-13T00:00:00.000Z
authorURL: ""
originalURL: https://martinheinz.dev/blog/111
translator: ""
reviewer: ""
---

随着 BuildKit 的引入，Docker 的构建后端得到了显著改进，并增添了许多强大的新功能。然而，很多用户并不了解这些新功能。因此，本文将向你介绍那些你绝对应该了解并开始使用的 BuildKit 功能，助你更好地利用 Docker。

## Debugging（调试）

让我们从最常见的任务 - **调试** 开始。一直以来，调试 `docker build` 都是一件痛苦的事情。当 `RUN` 或 `COPY` 命令失败时，你很难查看上下文并调试问题所在，通常只能求助于添加 `RUN ls -la` 等命令来获取更多信息。

然而，随着 **`docker buildx debug`** 的引入，这一切都将成为过去！

```shell
export BUILDX_EXPERIMENTAL=1
docker buildx debug --invoke /bin/sh --on=error build .
```

```shell
[+] Building 1.2s (14/18)
docker:default
...
------
 > [builder 5/6] RUN exit 1:
------
Dockerfile:10
--------------------
   8 |     RUN pip3 install -r requirements.txt
   9 |
  10 | >>> RUN exit 1
  11 |
  12 |     COPY . /app
--------------------
ERROR: process "/bin/sh -c exit 1" did not complete successfully: exit code: 1
[+] Building 0.0s (0/0)                  docker:default
Launching interactive container. Press Ctrl-a-c to switch to monitor console
Interactive container was restarted with process "u6agxp1ywqapemxrt8iexfv4h". Press Ctrl-a-c to switch to the new container
/ # ls -la
total 72
drwxr-xr-x    1 root     root          4096 May  5 12:59 .
drwxr-xr-x    1 root     root          4096 May  5 12:59 ..
drwxr-xr-x    1 root     root          4096 May  4 10:11 app
...
```

如上述代码片段所示，我们首先通过设置 `BUILDX_EXPERIMENTAL` 环境变量来启用实验性的 BuildKit 功能。然后，我们使用 `docker buildx debug` 命令启动构建过程。如果构建过程中的任何步骤发生错误，我们将自动进入容器内部，并可以自由探索上下文和进行调试。

需要注意的是，我们使用了 `--on=error` 选项，这表示只有在构建失败时才会启动调试会话。

[调试文档，获取更多细节][12]

## 环境变量

如果你之前使用 BuildKit 运行过构建，你一定注意到了它那花哨的新日志输出格式。虽然看起来很漂亮，但在调试时却不太实用。

好消息是，我们可以通过设置一个环境变量来切换回简洁的日志输出格式：

```shell
export BUILDKIT_PROGRESS=plain
```

你也可以将其设置为 `rawjson` ，虽然这种格式对人类来说绝对不可读，但如果你想以某种方式处理日志，它可能会很有用。

或者，如果你喜欢基于 TTY 的动态输出，但不喜欢默认的颜色，那么你可以通过以下方式更改它们：

```shell
BUILDKIT_COLORS="run=green:warning=yellow:error=red:cancel=cyan"
docker buildx debug --invoke /bin/sh --on=error build .
```

会有如下输出:

![Buildx Colors][1]

查看 [环境变量文档][2]

## 导出容器

BuildKit 还引入了 _导出器(exporters)_ 的概念，它定义了如何保存构建的输出。其中两个最有用的选项是 `image` 和 `registry`。`image` 正如你所期望的那样，将输出保存为容器镜像，而 `registry` 导出器会自动将镜像推送到指定的镜像仓库：

```shell
docker buildx build --output type=registry,name=martinheinz/testimage:latest .
```

我们只需要使用 `--output` 选项并指定类型为 `registry` 以及目标地址即可。此选项还支持一次指定多个镜像仓库：

```shell
docker buildx build --output type=registry,\"name=docker.io/martinheinz/testimage,docker.io/martinheinz/testimage2\" .
```

最后，我们还可以提供 `--cache-to` 和 `--cache-from` 选项，例如使用镜像仓库中的现有镜像作为缓存源：

```shell
docker buildx build --output type=registry,name=martinheinz/testimage:latest \
 --cache-to type=inline \
 --cache-from type=registry,ref=docker.io/martinheinz/testimage .
```

```shell
...
 => CACHED docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a8...1e24ef1a0dbc
...
 => CACHED [builder 2/5] WORKDIR /app                                                                            0.0s
 => CACHED [builder 3/5] COPY requirements.txt /app                                                              0.0s
 => CACHED [builder 4/5] RUN --mount=type=cache,target=/root/.cache/pip     pip3 install -r requirements.txt     0.0s
 => CACHED [builder 5/5] COPY . /app                                                                             0.0s
 => CACHED [dev-envs 1/3] RUN <<EOF (apk update...)                                                              0.0s
 => CACHED [dev-envs 2/3] RUN <<EOF (addgroup -S docker...)                                                      0.0s
 => CACHED [dev-envs 3/3] COPY --from=gloursdocker/docker / /                                                    0.0s
 => preparing layers for inline cache                                                                            0.0s
...
```

## 镜像工具

`docker buildx` 有一个简单但方便的子命令叫做 `imagetools`，它允许我们在不拉取镜像的情况下检查镜像仓库中的镜像。[文档][3] 中包含许多示例，但对我来说最有用的是获取远程镜像的哈希值：

```shell
docker buildx imagetools inspect alpine --format "{{json .Manifest}}" | jq .digest
"sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b"
```

## 最新的 Dockerfile 语法

BuildKit 还通过所谓的 _[Dockerfile 前端][4]_ 带来了新的 Dockerfile 语法。要启用当前最新的语法，我们需要在 Dockerfile 的顶部添加一个指令，例如：

```shell
# syntax=docker/dockerfile:1.3
FROM ...
```

要查找版本，你可以查看 [`dockerfile-upstream`][5]。

## Here-docs

我想提到的第一个 Dockerfile 语法改进是 _here-docs_，它允许我们将多行脚本传递给 `RUN` 和 `COPY` 命令：

```Dockerfile
# syntax = docker/dockerfile:1.3-labs
FROM debian
RUN <<eot bash
  apt-get update
  apt-get install -y vim
eot
```

```Dockerfile
# 与下面的相同:
RUN apt-get update && apt-get install -y vim
```

过去，如果我们想将多个命令放入单个 `RUN` 中，我们必须使用 `&&`，现在使用 here 文档，我们可以编写一个正常的脚本。

此外，第一行可以指定解释器，因此我们也可以，编写 Python 脚本：

```shell
# syntax = docker/dockerfile:1.3-labs
FROM python:3.6
RUN <<eot
#!/usr/bin/env python
print("hello world")
eot
```

## `COPY` 和 `ADD` 功能

在这种新的 Dockerfile 语法中，`COPY` 和 `ADD` 也有更细微的变化和改进，以新选项的形式出现。

`COPY` 现在支持 `--parents` 选项：

```shell
# syntax=docker/dockerfile:1.7.0-labs
FROM ubuntu

COPY ./one/two/some.txt /normal/

RUN find /normal
#10 [3/5] RUN find /normal
#10 0.223 /normal
#10 0.223 /normal/some.txt

COPY --parents ./one/two/some.txt /parents/

RUN find /parents
#12 [5/5] RUN find /parents
#12 0.509 /parents
#12 0.509 /parents/one
#12 0.509 /parents/one/two
#12 0.509 /parents/one/two/some.txt
```

如果你使用普通的 `COPY` 复制一个嵌套文件，镜像将只包含文件本身，而不包含父目录，使用 `--parents` 将复制整个文件树，类似于 `cp --parents` 的工作方式。

与 `--parents` 选项类似，你也可以使用 `--exclude`：

```shell
COPY --exclude=*.txt ./some-dir/* ./some-dest
```

这将在复制时省略排除的文件（和模式）。

最后，`ADD` 命令也得到了改进，现在可以直接添加 Git 仓库：

```shell
# syntax=docker/dockerfile:1.7.0-labs
FROM ubuntu

ADD git@github.com:kelseyhightower/helloworld.git /repo
RUN ls -la /repo
```

并且在为此 Dockerfile 运行构建时，我们将获得：

```shell
docker buildx build --ssh default --progress=plain .
```

```text
#8 [2/3] ADD git@github.com:kelseyhightower/helloworld.git /repo
#8 0.478 Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
#8 1.738 ref: refs/heads/master HEAD
#8 1.738 96a652519d1aaca11085ca3a7806bead4d2c273f HEAD
#8 3.478 96a652519d1aaca11085ca3a7806bead4d2c273f refs/heads/master
#8 1.829 ref: refs/heads/master HEAD
#8 1.829 96a652519d1aaca11085ca3a7806bead4d2c273f HEAD
#8 3.838 From github.com:kelseyhightower/helloworld
#8 3.838  * [new branch]      master     -> master
#8 3.838  * [new branch]      master     -> origin/master
#8 DONE 7.4s

#9 [2/3] ADD git@github.com:kelseyhightower/helloworld.git /repo
#9 DONE 0.0s
```

这同样适用于[私有仓库][6]。

在[文档][7]中查看更多有趣的选项，例如用于验证工件校验和的 `ADD --keep-git-dir` 或 `ADD --checksum`。

## 额外奖励：缩进

虽然不是 BuildKit 功能，但我最近发现的一件事是，你可以在 Dockerfile 中缩进行，并且它可以正常工作，这允许更好的可读性，尤其是在多阶段构建中：

```shell
# syntax=docker/dockerfile:1
FROM golang:1.21
  WORKDIR /src

  COPY main.go .
  RUN go build -o /bin/hello ./main.go

FROM scratch
  COPY --from=0 /bin/hello /bin/hello
  CMD ["/bin/hello"]
```

乍一看可能很奇怪，但我认为这样可读性更高，可以更清楚地看出每个阶段从哪里开始以及哪些命令属于它。

## 结语

本文中的示例仅展示了我认为最有用的功能，但还有更多功能，因此请务必查看官方 [Docker 文档][8]，以及包含最新更改的 [BuildKit 文档][9]。Docker 博客也是一个很好的资源，特别是打了 _[buildkit][10]_ 或 _[buildx][11]_ 标签的帖子。

[1]: https://i.imgur.com/gTasZHC.png
[2]: https://docs.docker.com/build/building/variables/#build-tool-configuration-variables
[3]: https://docs.docker.com/reference/cli/docker/buildx/imagetools/inspect/
[4]: https://docs.docker.com/build/dockerfile/frontend/
[5]: https://hub.docker.com/r/docker/dockerfile-upstream
[6]: https://docs.docker.com/reference/dockerfile/#adding-private-git-repositories
[7]: https://docs.docker.com/reference/dockerfile/#add
[8]: https://docs.docker.com/reference/cli/docker/buildx/
[9]: https://github.com/moby/buildkit/tree/master/docs
[10]: https://www.docker.com/blog/tag/buildkit/
[11]: https://www.docker.com/blog/tag/buildx/
[12]: https://github.com/docker/buildx/blob/master/docs/debugging.md
