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

![](https://www.docker.com/wp-content/uploads/2024/04/jay-schmidt.jpeg)

[Jay Schmidt][1]

Docker 作为一款容器化工具，其灵活性和健壮性毋庸置疑，但同时也带来了一定的复杂性，这可能会让用户望而却步。完成类似的任务，Docker 提供了多种方法，用户必须了解每种方法的优缺点，才能为自己的项目选择最佳方案。

Dockerfile 指令中，[RUN][2], [CMD][3], 和 [ENTRYPOINT][4] 是容易混淆的领域之一。本文将讨论这些指令之间的区别，并描述每种指令的用例。

![2400x1260 choosing between run cmd and entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_choosing-between-run-cmd-and-entrypoint-1110x583.png "- 2400X1260 Choosing Between Run Cmd And Entrypoint")

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

### Signal handling and flexibility

Using `ENTRYPOINT` in exec form and `CMD` to specify parameters ensures that Docker containers can handle operating system signals gracefully, respond to user inputs dynamically, and maintain secure and predictable operations.

This setup is particularly beneficial for containers that run critical applications needing reliable shutdown and configuration behaviors. The following table shows key differences between the forms.

#### **Key differences between shell and exec**

<table><tbody><tr><td></td><td><strong>Shell Form</strong></td><td><strong>Exec Form</strong></td></tr><tr><td><strong>Form</strong></td><td>Commands without <code>[]</code> brackets. Run by the container’s shell, e.g., <code>/bin/sh -c</code>.</td><td>Commands with <code>[]</code> brackets. Run directly, not through a shell.</td></tr><tr><td><strong>Variable Substitution</strong></td><td>Inherits environment variables from the shell, such as <code>$HOME</code> and <code>$PATH</code>.</td><td>Does not inherit shell environment variables but behaves the same for <code>ENV</code> instruction variables.</td></tr><tr><td><strong>Shell Features</strong></td><td>Supports sub-commands, piping output, chaining commands, I/O redirection, etc.</td><td>Does not support shell features.</td></tr><tr><td><strong>Signal Trapping &amp; Forwarding</strong></td><td>Most shells do not forward process signals to child processes.</td><td>Directly traps and forwards signals like <code>SIGINT</code>.</td></tr><tr><td><strong>Usage with ENTRYPOINT</strong></td><td>Can cause issues with signal forwarding.</td><td>Recommended due to better signal handling.</td></tr><tr><td><strong>CMD as ENTRYPOINT Parameters</strong></td><td>Not possible with the shell form.</td><td>If the first item in the array is not a command, all items are used as parameters for the <code>ENTRYPOINT</code>.</td></tr></tbody></table>

Figure 1 provides a decision tree for using `RUN`, `CMD`, and `ENTRYPOINT` in building a Dockerfile.

![2400x1260 run cmd entrypoint](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20583'%3E%3C/svg%3E "- 2400X1260 Run Cmd Entrypoint")

![2400x1260 run cmd entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_run-cmd-entrypoint-1110x583.png "- 2400X1260 Run Cmd Entrypoint")

Figure 1: Decision tree — RUN, CMD, ENTRYPOINT.

Figure 2 shows a decision tree to help determine when to use exec form or shell form.

![2400x1260 decision tree exec vs shell](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20583'%3E%3C/svg%3E "- 2400X1260 Decision Tree Exec Vs Shell")

![2400x1260 decision tree exec vs shell](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_decision-tree-exec-vs-shell-1110x583.png "- 2400X1260 Decision Tree Exec Vs Shell")

Figure 2: Decision tree — exec vs. shell form.

## Examples

The following section will walk through the high-level differences between `CMD` and `ENTRYPOINT`. In these examples, the `RUN` command is not included, given that the only decision to make there is easily handled by reviewing the two different formats.

### Test Dockerfile

\# Use syntax version 1.3-labs for Dockerfile

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

### First build

We will build this image and tag it as `ab`.

$ docker build -t ab .

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

### **Run with `CMD ab`**

Without any arguments, we get a usage block as expected.

$ docker run ab
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

However, if I run `ab` and include a URL to test, I initially get an error:

$ docker run --rm ab https://jayschmidt.us
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "https://jayschmidt.us": stat https://jayschmidt.us: no such file or directory: unknown.

The issue here is that the string supplied on the command line — `https://jayschmidt.us` — is overriding the `CMD` instruction, and that is not a valid command, resulting in an error being thrown. So, we need to specify the command to run:

$ docker run --rm ab ab https://jayschmidt.us/
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

### Run with ENTRYPOINT

In this run, we remove the `CMD ab` instruction from the Dockerfile, replace it with `ENTRYPOINT ["ab"]`, and then rebuild the image.

This is similar to but different from the `CMD` command — when you use `ENTRYPOINT`, you cannot override the command unless you use the `–entrypoint` flag on the `docker run` command. Instead, any arguments passed to `docker run` are treated as arguments to the `ENTRYPOINT`.

$ docker run --rm ab "https://jayschmidt.us/"
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

### What about syntax?

In the example above, we use `ENTRYPOINT ["ab"]` syntax to wrap the command we want to run in square brackets and quotes. However, it is possible to specify `ENTRYPOINT ab` (without quotes or brackets).

Let’s see what happens when we try that.

$ docker run --rm ab "https://jayschmidt.us/"
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

Your first thought will likely be to re-run the `docker run` command as we did for `CMD ab` above, which is giving both the executable and the argument:

$ docker run --rm ab ab "https://jayschmidt.us/"
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

This is because `ENTRYPOINT` can only be overridden if you explicitly add the `–entrypoint` argument to the `docker run` command. The takeaway is to always use `ENTRYPOINT` when you want to force the use of a given executable in the container when it is run.

## Wrapping up: Key takeaways and best practices

The decision-making process involving the use of `RUN`, `CMD`, and `ENTRYPOINT`, along with the choice between shell and exec forms, showcases Docker’s intricate nature. Each command serves a distinct purpose in the Docker ecosystem, impacting how containers are built, operate, and interact with their environments.

By selecting the right command and form for each specific scenario, developers can construct Docker images that are more reliable, secure, and optimized for efficiency. This level of understanding and application of Docker’s commands and their formats is crucial for fully harnessing Docker’s capabilities. Implementing these best practices ensures that applications deployed in Docker containers achieve maximum performance across various settings, enhancing development workflows and production deployments.

## Learn more

-   [CMD][10]
-   [ENTRYPOINT][11]
-   [RUN][12]
-   Stay updated on the latest Docker news! Subscribe to the [Docker Newsletter][13].
-   Get the latest release of [Docker Desktop][14].
-   New to Docker? [Get started][15].
-   Have questions? The [Docker community is here to help][16].

[developers][17], [Docker][18], [Docker Desktop][19], [dockerfile][20]

[

][21]

#### [Docker Desktop 4.32: Beta Releases of Compose File Viewer, Terminal Shell Integration, and Volume Backups to Cloud Providers][22]

By [Deanna Sparks][23]

[

][24]

#### [How an AI Assistant Can Help Configure Your Project’s Git Hooks][25]

By [Docker Labs][26] July 15, 2024

[

][27]

#### [How to Run Hugging Face Models Programmatically Using Ollama and Testcontainers][28]

By [Ignasi Lopez Luna][29] July 11, 2024

#### Posted

Jul 15, 2024

-   [][30]
-   [][31]
-   [][32]

#### Post Tags

[developers][33][Docker][34][Docker Desktop][35][dockerfile][36]

#### Categories

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
