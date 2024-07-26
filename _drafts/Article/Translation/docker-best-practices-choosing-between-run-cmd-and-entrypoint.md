---
title: "Docker Best Practices: Choosing Between RUN, CMD, and ENTRYPOINT"
date: 2024-07-17T00:00:00.000Z
author: Jay Schmidt
authorURL: https://www.docker.com/author/jay-schmidt/
originalURL: https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/
translator: ""
reviewer: ""
---

# Docker Best Practices: Choosing Between RUN, CMD, and ENTRYPOINT

<!-- more -->

<img src="https://www.docker.com/wp-content/uploads/2024/04/jay-schmidt.jpeg" width="50">

[Jay Schmidt][1]

Docker 作为一款容器化工具，其灵活性和健壮性毋庸置疑，但同时也带来了一定的复杂性，这可能会让用户望而却步。完成类似的任务，Docker 提供了多种方法，用户必须了解每种方法的优缺点，才能为自己的项目选择最佳方案。

Dockerfile 指令中，[RUN][2], [CMD][3], 和 [ENTRYPOINT][4] 是容易混淆的领域之一。本文将讨论这些指令之间的区别，并描述每种指令的用例。

![2400x1260 choosing between run cmd and entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_choosing-between-run-cmd-and-entrypoint.png)

### RUN

`RUN` 指令在 [Dockerfiles][5] 中用于执行构建和配置 Docker 镜像的命令。这些命令在镜像构建过程中执行，并且每个 `RUN` 指令都会在 Docker 镜像中创建一个新层。例如，如果您要创建一个需要安装特定软件或库的镜像，则可以使用 `RUN` 来执行必要的安装命令。

以下示例展示了如何在镜像构建期间指示 Docker 构建过程更新 `apt 缓存` 并安装 Apache：

```shell
RUN apt update && apt -y install apache2
```

应谨慎使用 `RUN` 指令，尽可能将相关命令合并到单个 `RUN` 指令中，以最大程度地减少镜像层数，从而减小镜像大小。

### CMD

`CMD` 指令指定了从 Docker 镜像启动容器时默认运行的命令。如果在容器启动期间（即在 `docker run` 命令中）未指定命令，则使用此默认命令。可以通过向 `docker run` 提供命令行参数来覆盖 `CMD`。

`CMD` 对于设置默认命令和易于覆盖的参数非常有用。它通常在镜像中用于定义默认运行参数，并且可以在运行容器时从命令行覆盖。

例如，默认情况下，您可能希望启动 Web 服务器，但用户可以将其覆盖为运行 shell：

```shell
CMD ["apache2ctl", "-DFOREGROUND"]
```

用户可以使用 `docker run -it <image> /bin/bash` 启动容器以获取 Bash shell，而不是启动 Apache。

### ENTRYPOINT

`ENTRYPOINT` 指令设置容器的默认可执行文件。它类似于 `CMD`，但会被传递给 `docker run` 的命令行参数覆盖。不同的是，任何命令行参数都会被附加到 `ENTRYPOINT` 命令之后。

**注意：** 当您需要容器始终运行相同的基础命令，并且希望允许用户在末尾追加其他命令时，请使用 `ENTRYPOINT`。

`ENTRYPOINT` 对于将容器转换为独立的可执行文件特别有用。例如，假设您正在打包一个需要参数的自定义脚本（例如，`my_script extra_args`）。在这种情况下，您可以使用 `ENTRYPOINT` 始终运行脚本进程 (`my_script`)，然后允许镜像用户在 `docker run` 命令行上指定 `extra_args`。您可以执行以下操作：

```shell
ENTRYPOINT ["my_script"]
```

### 组合使用 CMD 和 ENTRYPOINT

如果 `CMD` 指令以 exec 形式指定，则它可以用于为 `ENTRYPOINT` 提供默认参数。这种设置允许入口点作为主可执行文件，而 `CMD` 指定可以被用户覆盖的附加参数。

例如，您可能有一个运行 Python 应用程序的容器，您希望始终使用相同的应用程序文件，但允许用户指定不同的命令行参数：

```shell
ENTRYPOINT ["python", "/app/my_script.py"]
CMD ["--default-arg"]
```

运行 `docker run myimage --user-arg` 将执行 `python /app/my_script.py --user-arg`。

下表概述了这些命令和用例。

#### **命令描述和用例**

| **命令**                                                                     | **描述**                                                            | **用例**                                                   |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------- |
| [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd)               | 定义 Docker 镜像的默认可执行文件。它可以被 `docker run` 参数覆盖。  | 实用程序镜像允许用户在命令行上传递不同的可执行文件和参数。 |
| [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint) | 定义默认可执行文件。它可以被 `"--entrypoint" docker run` 参数覆盖。 | 为特定目的构建的镜像，其中不希望覆盖默认可执行文件。       |
| [`RUN`](https://docs.docker.com/engine/reference/builder/#run)               | 执行命令以构建层。                                                  | 构建镜像                                                   |

### 什么是 PID 1，及其重要性

在 Unix 和类 Unix 系统中，包括 Docker 容器，PID 1 指的是系统启动期间启动的第一个进程。所有其他进程都是由 PID 1 启动的，在进程树模型中，它是系统中每个进程的父进程。

在 Docker 容器中，作为 PID 1 运行的进程至关重要，因为它负责管理容器内的所有其他进程。此外，PID 1 是负责接收和处理来自 Docker 主机的信号的进程。例如，进入容器的 `SIGTERM` 信号将由 PID 1 捕获和处理，容器应该正常关闭。

当使用 shell 形式在 Docker 中执行命令时，shell 进程 (`/bin/sh -c`) 通常会成为 PID 1。然而，它不能正确处理这些信号，这可能会导致容器无法正常关闭。相反，当使用 exec 形式时，命令会直接作为 PID 1 运行，而不涉及 shell，这使得它能够直接接收和处理信号。

这种行为确保了容器可以正常停止、重启或处理中断，使得 exec 形式更适用于需要强大且响应迅速的信号处理的应用程序。

### Shell 和 exec

在之前的例子中，我们使用了两种方式将参数传递给 `RUN`、`CMD` 和 `ENTRYPOINT` 指令。它们被称为 shell 形式和 exec 形式。

**注意：** 两种形式的主要视觉区别是，exec 形式以逗号分隔的命令和参数数组的形式传递，每个元素一个参数/命令。相反，shell 形式则表示为一个组合了命令和参数的字符串。

每种形式都会对容器内的命令执行产生影响，从信号处理到环境变量扩展都会有所不同。下表提供了一个关于不同形式的快速参考指南。

#### **Shell 形式和 Exec 形式参考指南**

| 表格           | 描述                                          | 示例                                                    |
| -------------- | --------------------------------------------- | ------------------------------------------------------- |
| **Shell 形式** | 采用 `<指令> <命令>` 的形式。                 | `CMD echo TEST` 或 `ENTRYPOINT echo TEST`               |
| **Exec 形式**  | 采用 `<指令> ["可执行文件", "参数"]` 的形式。 | `CMD ["echo", "TEST"]` 或 `ENTRYPOINT ["echo", "TEST"]` |

在 shell 形式下，命令会在一个子 shell 中运行，在 Linux 系统上通常是 `/bin/sh -c`。这种形式很有用，因为它允许进行 shell 处理（如变量扩展、通配符等），使其对于某些类型的命令更加灵活（有关 shell 处理的示例，请参阅这篇[shell 脚本文章][9]）。然而，这也意味着运行命令的进程不是容器的 PID 1，这可能导致信号处理问题，因为 Docker 发送的信号（如用于正常关闭的 `SIGTERM`）会被 shell 接收，而不是预期的进程。

exec 形式不调用命令 shell。这意味着您指定的命令将作为容器的 PID 1 直接执行，这对于正确处理发送到容器的信号非常重要。此外，这种形式不执行 shell 扩展，因此它更安全、更可预测，尤其是在指定来自外部源的参数或命令时。

### 一起使用

为了说明 Docker 的 `RUN`、`CMD` 和 `ENTRYPOINT` 指令以及 shell 和 exec 形式的选择的实际应用和细微差别，让我们回顾一些示例。这些示例演示了如何在现实世界的 Dockerfile 场景中有效地利用每条指令，并重点介绍了 shell 和 exec 形式之间的区别。

通过这些示例，您将更好地理解何时以及如何使用每个指令来精确地根据您的需要定制容器行为，从而确保 Docker 容器的正确配置、安全性和性能。这种实践方法将帮助您将我们讨论过的理论知识整合到可操作的见解中，这些见解可以直接应用于您的 Docker 项目。

### RUN 指令

对于在 Docker 构建过程中用于安装软件包或修改文件的 `RUN` 指令，在 shell 和 exec 形式之间进行选择取决于是否需要 shell 处理。对于需要 shell 功能的命令（例如管道或文件通配符），shell 形式是必要的。但是，对于没有 shell 功能的简单命令，exec 形式更可取，因为它降低了复杂性和潜在错误。

Shell 形式，适用于复杂的脚本编写。

```shell
RUN apt-get update && apt-get install -y nginx
```

Exec 形式，用于直接执行命令

```shell
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "nginx"]
```

### CMD 和 ENTRYPOINT

这些指令控制容器的运行时行为。将 exec 形式与 `ENTRYPOINT` 一起使用可确保容器的主应用程序直接处理信号，这对于正确的启动和关闭行为至关重要。 `CMD` 可以为以 exec 形式定义的 `ENTRYPOINT` 提供默认参数，从而提供灵活性和强大的信号处理能力。

ENTRYPOINT 使用 exec 形式进行直接进程控制

```shell
ENTRYPOINT ["httpd"]
```

CMD 提供默认参数，可以在运行时被覆盖

```shell
CMD ["-D", "FOREGROUND"]
```

### 信号处理与灵活性

使用 `ENTRYPOINT` (exec 格式) 和 `CMD` 来指定参数，可以确保 Docker 容器能够优雅地处理操作系统信号，动态响应用户输入，并维护安全可预测的操作。

这种设置对于运行需要可靠关闭和配置行为的关键应用程序的容器特别有用。下表显示了两种形式之间的主要区别：

#### **shell 和 exec 形式的主要区别**

| 特性                    | **Shell 形式**                                                                           | **Exec 形式**                                                                                               |
|-------------------------|-------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **形式**                | 没有 `[]` 括号的命令。由容器的 shell 运行，例如 `/bin/sh -c`。                                | 带有 `[]` 括号的命令。直接运行，不通过 shell。                                                                      |
| **变量替换**             | 继承 shell 的环境变量，例如 `$HOME` 和 `$PATH`。                                          | 不继承 shell 环境变量，但对 `ENV` 指令变量的行为相同。                                                               |
| **Shell 功能**          | 支持子命令、管道输出、命令链、I/O 重定向等。                                                | 不支持 shell 功能。                                                                                                |
| **信号捕获和转发**       | 大多数 shell 不会将进程信号转发给子进程。                                                    | 直接捕获并转发 `SIGINT` 等信号。                                                                                 |
| **与 ENTRYPOINT 一起使用** | 可能导致信号转发问题。                                                                    | 建议使用，因为信号处理更好。                                                                                      |
| **CMD 作为 ENTRYPOINT 参数** | shell 形式不支持。                                                                        | 如果数组中的第一项不是命令，则所有项都用作 `ENTRYPOINT` 的参数。                                                          | 

图 1 提供了一个决策树，用于在构建 Dockerfile 时使用 `RUN`、`CMD` 和 `ENTRYPOINT` 指令。 

![2400x1260 run cmd entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_run-cmd-entrypoint-1110x583.png "- 2400X1260 Run Cmd Entrypoint")

图 1：决策树 —— RUN, CMD, ENTRYPOINT。


图 2 显示了一个决策树，用以帮助确定何时使用 exec 格式或 shell 格式。

![2400x1260 decision tree exec vs shell](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_decision-tree-exec-vs-shell-1110x583.png "- 2400X1260 Decision Tree Exec Vs Shell")

图 2：决策树 —— exec 格式对比 shell 格式。

## 例子

下一节将概述 `CMD` 和 `ENTRYPOINT` 之间的高层次差异。在这些示例中，没有包括 `RUN` 命令，因为只需查看两种不同的格式即可轻松做出决定。

### Test Dockerfile
```yaml
# Use syntax version 1.3-labs for Dockerfile

# syntax=docker/dockerfile:1.3-labs

# Use the Ubuntu 20.04 image as the base image

FROM ubuntu:20.04

# Run the following commands inside the container:

# 1. Update the package lists for upgrades and new package installations

# 2. Install the apache2-utils package (which includes the 'ab' tool)

# 3. Remove the package lists to reduce the image size

#

# This is all run in a HEREDOC; see

# https://www.docker.com/blog/introduction-to-heredocs-in-dockerfiles/

# for more details.

#

RUN <<EOF
apt-get update;
apt-get install -y apache2-utils;
rm -rf /var/lib/apt/lists/\*;
EOF

# Set the default command

CMD ab
```
### 第一次构建

我们将构建这个镜像，并将其 tag 记为 `ab`。

```shell
$ docker build -t ab .
```

```text
\[+\] Building 7.0s (6/6) FINISHED docker:desktop-linux
=> \[internal\] load .dockerignore 0.0s
=> => transferring context: 2B 0.0s
=> \[internal\] load build definition from Dockerfile 0.0s
=> => transferring dockerfile: 730B 0.0s
=> \[internal\] load metadata for docker.io/library/ubuntu:20.04 0.4s
=> CACHED \[1/2\] FROM docker.io/library/ubuntu:20.04@sha256:33a5cc25d22c45900796a1aca487ad7a7cb09f09ea00b779e 0.0s
=> \[2/2\] RUN <<EOF (apt-get update;...) 6.5s
=> exporting to image 0.0s
=> => exporting layers 0.0s
=> => writing image sha256:99ca34fac6a38b79aefd859540f88e309ca759aad0d7ad066c4931356881e518 0.0s
=> => naming to docker.io/library/ab
```

### **运行 `CMD ab`**

在没有任何参数的情况下，我们会按预期得到一个使用说明块。 

```shell
$ docker run ab
```

```text
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
-n requests Number of requests to perform
-c concurrency Number of multiple requests to make at a time
-t timelimit Seconds to max. to spend on benchmarking
This implies -n 50000
-s timeout Seconds to max. wait for each response
Default is 30 seconds
<-- SNIP -->
```

然而，如果我运行 `ab` 并且包含了一个要测试的 URL，我最初会得到一个错误：

```shell
$ docker run --rm ab https://jayschmidt.us
```

```text
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "https://jayschmidt.us": stat https://jayschmidt.us: no such file or directory: unknown.
```

这里的问题是，命令行上提供的字符串 — `https://jayschmidt.us` — 覆盖了 `CMD` 指令，而这不是一个有效的命令，导致抛出了一个错误。所以，我们需要指定要运行的命令：

```shell
$ docker run --rm ab ab https://jayschmidt.us/
```
```text
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking jayschmidt.us (be patient).....done

Server Software: nginx
Server Hostname: jayschmidt.us
Server Port: 443
SSL/TLS Protocol: TLSv1.2,ECDHE-ECDSA-AES256-GCM-SHA384,256,256
Server Temp Key: X25519 253 bits
TLS Server Name: jayschmidt.us

Document Path: /
Document Length: 12992 bytes

Concurrency Level: 1
Time taken for tests: 0.132 seconds
Complete requests: 1
Failed requests: 0
Total transferred: 13236 bytes
HTML transferred: 12992 bytes
Requests per second: 7.56 \[#/sec\] (mean)
Time per request: 132.270 \[ms\] (mean)
Time per request: 132.270 \[ms\] (mean, across all concurrent requests)
Transfer rate: 97.72 \[Kbytes/sec\] received

Connection Times (ms)
min mean\[+/-sd\] median max
Connect: 90 90 0.0 90 90
Processing: 43 43 0.0 43 43
Waiting: 43 43 0.0 43 43
Total: 132 132 0.0 132 132
```

### Run 使用 ENTRYPOINT

在这次运行中，我们从 Dockerfile 中移除了 `CMD ab` 指令，用 `ENTRYPOINT ["ab"]` 替换它，然后重新构建镜像。

这与 `CMD` 命令相似但又有所不同 — 当你使用 `ENTRYPOINT` 时，除非你在 `docker run` 命令上使用 `--entrypoint` 标志，否则你不能覆盖命令。相反，传递给 `docker run` 的任何参数都会被当作是 `ENTRYPOINT` 的参数。

```shell
$ docker run --rm ab "https://jayschmidt.us/"
```

```text
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking jayschmidt.us (be patient).....done

Server Software: nginx
Server Hostname: jayschmidt.us
Server Port: 443
SSL/TLS Protocol: TLSv1.2,ECDHE-ECDSA-AES256-GCM-SHA384,256,256
Server Temp Key: X25519 253 bits
TLS Server Name: jayschmidt.us

Document Path: /
Document Length: 12992 bytes

Concurrency Level: 1
Time taken for tests: 0.122 seconds
Complete requests: 1
Failed requests: 0
Total transferred: 13236 bytes
HTML transferred: 12992 bytes
Requests per second: 8.22 \[#/sec\] (mean)
Time per request: 121.709 \[ms\] (mean)
Time per request: 121.709 \[ms\] (mean, across all concurrent requests)
Transfer rate: 106.20 \[Kbytes/sec\] received

Connection Times (ms)
min mean\[+/-sd\] median max
Connect: 91 91 0.0 91 91
Processing: 31 31 0.0 31 31
Waiting: 31 31 0.0 31 31
Total: 122 122 0.0 122 122
```

### What about syntax?

在上面的示例中，我们使用 `ENTRYPOINT ["ab"]` 语法将要运行的命令用方括号和引号括起来。但是，也可以指定 `ENTRYPOINT ab`（不带引号或括号）。 

让我们看看尝试这样做会发生什么。 

```shell
$ docker run --rm ab "https://jayschmidt.us/"
```

```text
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
-n requests Number of requests to perform
-c concurrency Number of multiple requests to make at a time
-t timelimit Seconds to max. to spend on benchmarking
This implies -n 50000
-s timeout Seconds to max. wait for each response
Default is 30 seconds
<-- SNIP -->
```

您的第一个想法可能是像上面对 `CMD ab` 所做的那样重新运行 `docker run` 命令，该命令同时提供了可执行文件和参数： 

```shell
$ docker run --rm ab ab "https://jayschmidt.us/"
```

```text
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
-n requests Number of requests to perform
-c concurrency Number of multiple requests to make at a time
-t timelimit Seconds to max. to spend on benchmarking
This implies -n 50000
-s timeout Seconds to max. wait for each response
Default is 30 seconds
<-- SNIP -->
```

这是因为只有在 `docker run` 命令中显式添加 `--entrypoint` 参数时，才能覆盖 `ENTRYPOINT`。  要点是，当您希望在运行容器时强制使用给定的可执行文件时，请始终使用 `ENTRYPOINT`。 

## 总结：关键要点和最佳实践

涉及使用 `RUN`、`CMD` 和 `ENTRYPOINT` 以及 shell 和 exec 形式的选择的决策过程，展现了 Docker 的复杂性。每个命令在 Docker 生态系统中都有其独特的用途，影响着容器的构建、运行以及与其环境的交互方式。

通过为每个特定场景选择正确的命令和形式，开发人员可以构建更可靠、更安全且针对效率进行优化的 Docker 镜像。这种对 Docker 命令及其格式的理解和应用水平，对于充分利用 Docker 的功能至关重要。实施这些最佳实践可确保在 Docker 容器中部署的应用程序在各种设置下都能实现最佳性能，从而增强开发工作流程和生产部署。 

## 学习更多

-   [CMD][10]
-   [ENTRYPOINT][11]
-   [RUN][12]
-   及时了解 Docker 最新资讯！订阅 [Docker 简报][13]。
-   获取最新版的 [Docker Desktop][14]。
-   Docker 新手？[开始使用][15]。
-   有问题？[Docker 社区随时为您提供帮助][16]。

[developers][17], [Docker][18], [Docker Desktop][19], [dockerfile][20]



#### [Docker Desktop 4.32：Compose 文件查看器、终端 Shell 集成和卷备份到云提供商的测试版本][22]

By [Deanna Sparks][23]


#### [人工智能助手如何帮助配置项目的 Git hook][25]

By [Docker Labs][26] July 15, 2024


#### [如何使用 Ollama 和 Testcontainers 以编程方式运行 Hugging Face 模型][28]

By [Ignasi Lopez Luna][29] July 11, 2024

#### Posted

#### Post Tags

[developers][33][Docker][34][Docker Desktop][35][dockerfile][36]

#### 分类

-   [Community][37]
-   [Company][38]
-   [Engineering][39]
-   [Products][40]

[1]: https://www.docker.com/author/jay-schmidt/ "Posts by Jay Schmidt"
[2]: https://docs.docker.com/engine/reference/builder/#run
[3]: https://docs.docker.com/engine/reference/builder/#cmd
[4]: https://docs.docker.com/engine/reference/builder/#entrypoint
[5]: https://docs.docker.com/reference/dockerfile/
[6]: https://docs.docker.com/engine/reference/builder/#cmd
[7]: https://docs.docker.com/engine/reference/builder/#entrypoint
[8]: https://docs.docker.com/engine/reference/builder/#run
[9]: https://en.wikibooks.org/wiki/Bourne_Shell_Scripting/Variable_Expansion
[10]: https://docs.docker.com/engine/reference/builder/#cmd
[11]: https://docs.docker.com/engine/reference/builder/#entrypoint
[12]: https://docs.docker.com/engine/reference/builder/#run
[13]: https://www.docker.com/newsletter-subscription/
[14]: https://www.docker.com/products/docker-desktop/
[15]: https://docs.docker.com/desktop/
[16]: https://www.docker.com/community/
[17]: https://www.docker.com/blog/tag/developers/
[18]: https://www.docker.com/blog/tag/docker/
[19]: https://www.docker.com/blog/tag/docker-desktop/
[20]: https://www.docker.com/blog/tag/dockerfile/
[21]: https://www.docker.com/blog/docker-desktop-4-32/
[22]: https://www.docker.com/blog/docker-desktop-4-32/
[23]: https://www.docker.com/author/deanna-sparks/ "Posts by Deanna Sparks"
[24]: https://www.docker.com/blog/how-an-ai-assistant-can-help-configure-your-projects-git-hooks/
[25]: https://www.docker.com/blog/how-an-ai-assistant-can-help-configure-your-projects-git-hooks/
[26]: https://www.docker.com/author/docker-labs/ "Posts by Docker Labs"
[27]: https://www.docker.com/blog/how-to-run-hugging-face-models-programmatically-using-ollama-and-testcontainers/
[28]: https://www.docker.com/blog/how-to-run-hugging-face-models-programmatically-using-ollama-and-testcontainers/
[29]: https://www.docker.com/author/ignasi-lopez-luna/ "Posts by Ignasi Lopez Luna"
[30]: https://twitter.com/intent/tweet?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[31]: https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[32]: https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[33]: https://www.docker.com/blog/tag/developers/
[34]: https://www.docker.com/blog/tag/docker/
[35]: https://www.docker.com/blog/tag/docker-desktop/
[36]: https://www.docker.com/blog/tag/dockerfile/
[37]: https://www.docker.com/blog/category/community-content/
[38]: https://www.docker.com/blog/category/company/
[39]: https://www.docker.com/blog/category/engineering/
[40]: https://www.docker.com/blog/category/products/
