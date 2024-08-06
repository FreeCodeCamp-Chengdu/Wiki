---
title: 高级 Dockerfile 指令
date: 2024-07-08T06:25:58.000Z
authorURL: ""
originalURL: https://dev.to/kalkwst/advanced-dockerfile-directives-193f
translator: ""
reviewer: ""
---

[![Cover image for Advanced Dockerfile Directives](https://media.dev.to/cdn-cgi/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fud6oyinws6xha4i4f179.png)][14]

[![Kostas Kalafatis](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][15]

![](https://dev.to/assets/sparkle-heart-5f9bee3767e18deb1bb725290cb151c25234768a0e9a2bd39370c382d02920cf.svg)  ![](https://dev.to/assets/multi-unicorn-b44d6f8c23cdd00964192bedc38af3e82463978aa611b4365bd33a0f1f4f3e97.svg)  ![](https://dev.to/assets/exploding-head-daceb38d627e6ae9b730f36a1e390fca556a4289d5a41abb2c35068ad3e2c4b5.svg)  ![](https://dev.to/assets/raised-hands-74b2099fd66a39f2d7eed9305ee0f4553df0eb7b4f11b01b6b1b499973048fe5.svg) ![](https://dev.to/assets/fire-f60e7a582391810302117f987b22a8ef04a2fe0df7e3258a5f49332df1cec71e.svg)

## [The Docker Workshop (6 Part Series)][21]

在本篇文章中，我们将讨论更高级的 **Dockerfile** 指令。这些指令可用于创建更高级的 Docker 镜像。

例如，我们可以使用**VOLUME**指令将主机的文件系统绑定到 Docker 容器上。这样，我们就能将 Docker 容器生成和使用的数据保存到本地机器上。

我们将在本篇文章中介绍以下指令：

1. **ENV** 指令
2. **ARG** 指令
3. **WORKDIR** 指令
4. **COPY** 指令
5. **ADD** 指令
6. **USER** 指令
7. **VOLUME** 指令
8. **EXPOSE** 指令
9. **HEALTHCHECK** 指令
10. **ONBUILD** 指令

## ENV 指令

Dockerfile 中的 ENV 指令可用于设置环境变量。环境变量用于设置环境变量。环境变量是键值对，用于向容器内运行的应用程序和进程提供信息。它们可以通过在运行时提供动态值来影响程序和脚本的行为。

环境变量的定义方式如下所示的键值对：

```dockerfile
ENV <key>=<value>
```

例如，我们可以使用 **ENV** 指令设置路径，如下所示： 
```dockerfile
ENV PATH $PATH:/usr/local/app/bin/
```

我们可以在同一行中设置多个环境变量，并用空格分隔。但是，在这种形式中，**键**和**值**应该用等号（`=`）符号分隔：

```dockerfile
ENV <key>=<value> <key=value> ...
```

下面，我们设置了两个环境变量。**PATH** 环境变量配置为 `$PATH:/usr/local/app/bin`，**VERSION** 环境变量配置为 `1.0.0`：

```dockerfile
ENV PATH=$PATH:/usr/local/app/bin/ VERSION=1.0.0
```

一旦使用 **ENV** 指令在 **Dockerfile** 中设置了环境变量，该变量在所有后续的 Docker 镜像层中都可用。此变量甚至在从此 Docker 镜像启动的 Docker 容器中也可用。

## ARG 指令

Dockerfile 中的 **ARG** 指令用于定义用户可以在构建时使用 `docker build` 命令传递给构建器的变量。这些变量的行为类似于环境变量，可以在整个 Dockerfile 中使用，但不会持久化到最终镜像中，除非使用 **ENV** 指令显式声明。

**ARG** 指令的格式如下： 
```dockerfile
ARG <varname>
```

我们还可以添加多个 **ARG** 指令，如下所示：
```dockerfile
ARG USER
ARG VERSION
```

这些参数还可以在 Dockerfile 本身中指定可选的默认值。如果用户在构建过程中没有提供值，Docker 会使用 **ARG** 指令中定义的默认值：
```dockerfile
ARG USER=TestUser
ARG VERSION=1.0.0
```

与 **ENV** 变量不同，**ARG** 变量在运行的容器中不可访问。它们仅在构建过程中可用。 

### 在 Dockerfile 中使用 ENV 和 ARG 指令 
我们将创建一个使用 ubuntu 作为父镜像的 **Dockerfile**，但我们能够在构建时更改 ubuntu 版本。我们还将指定环境名称和应用程序目录作为 Docker 镜像的环境变量。

使用 `mkdir` 命令创建一个名为 `env-arg-example` 的新目录： 


```dockerfile
mkdir env-arg-example
```

使用 cd 命令导航到新创建的 env-arg-example 目录：
```shell
cd env-arg-example
```

现在，我们来创建一个新的 Dockerfile。我将使用 VS Code，但你可以随意使用任何你喜欢的编辑器：

```shell
code Dockerfile
```

将以下内容添加到 Dockerfile 中。然后保存并退出：
```dockerfile
ARG TAG=latest
FROM ubuntu:$TAG
LABEL maintainer=ananalogguyinadigitalworld@example.com
ENV ENVIRONMENT=dev APP_DIR=/usr/local/app/bin
CMD ["env"]
```

这段文字详细解释了一个 Dockerfile 的内容和作用。

**Dockerfile 解释：**

* **`ARG TAG=latest`**:  定义了一个名为 `TAG` 的构建参数，默认值为 `latest`。这个参数可以在构建镜像时通过 `docker build` 命令的 `--build-arg`  选项进行修改。
* **`FROM ubuntu:$TAG`**:  使用 `ubuntu:$TAG` 作为基础镜像。这意味着 Docker 将会拉取标签为 `TAG` 值 (默认为 `latest`) 的 Ubuntu 镜像作为构建的基础。
* **`LABEL maintainer="user@example.com"`**: 为镜像添加了维护者的信息，方便其他人了解镜像的来源。
* **`ENV ENVIRONMENT=dev`**: 设置了一个名为 `ENVIRONMENT` 的环境变量，其值为 `dev`。环境变量可以在容器内部被应用程序访问，用于配置应用程序的行为。
* **`ENV APP_DIR=/usr/local/app/bin`**: 设置了另一个名为 `APP_DIR` 的环境变量，其值为 `/usr/local/app/bin`，用于指定应用程序的目录。
* **`CMD ["env"]`**: 定义了容器启动时执行的命令。在这里，容器启动后会执行 `env` 命令，用于显示容器内部的所有环境变量。

**构建 Docker 镜像：**

接下来就可以使用 `docker build` 命令构建 Docker 镜像了。 


```shell
docker image build -t env-arg --build-arg TAG=23.10 .
```

输出结果应该类似于以下内容：

```text
[+] Building 34.9s (6/6) FINISHED                                                                                                            docker:default
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [internal] load build definition from Dockerfile                                                                                                   0.0s
 => => transferring dockerfile: 189B                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:23.10                                                                                        3.3s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                          0.0s
 => [1/1] FROM docker.io/library/ubuntu:23.10@sha256:fd7fe639db24c4e005643921beea92bc449aac4f4d40d60cd9ad9ab6456aec01                                 31.6s
 => => resolve docker.io/library/ubuntu:23.10@sha256:fd7fe639db24c4e005643921beea92bc449aac4f4d40d60cd9ad9ab6456aec01                                  0.0s
 => => sha256:fd7fe639db24c4e005643921beea92bc449aac4f4d40d60cd9ad9ab6456aec01 1.13kB / 1.13kB                                                         0.0s
 => => sha256:c57e8a329cd805f341ed7ee7fcc010761b29b9b8771b02a4f74fc794f1d7eac5 424B / 424B                                                             0.0s
 => => sha256:77081d4f1e7217ffd2b55df73979d33fd493ad941b3c1f67f1e2364b9ee7672f 2.30kB / 2.30kB                                                         0.0s
 => => sha256:cd0bff360addc3363f9442a3e0b72ff44a74ccc0120d0fc49dfe793035242642 27.23MB / 27.23MB                                                      30.3s
 => => extracting sha256:cd0bff360addc3363f9442a3e0b72ff44a74ccc0120d0fc49dfe793035242642                                                              1.1s
 => exporting to image                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                0.0s
 => => writing image sha256:86b2f4c440c71f37c3f29f5dd5fe79beac30f5a6ce878bd14dc17f439bd2377d                                                           0.0s
 => => naming to docker.io/library/env-arg                                                                                                             0.0s
```

现在，执行 `docker container run` 命令，从上一步构建的 Docker 镜像启动一个新的容器： 

```shell
docker container run env-arg
```

并且输出结果应该类似于以下内容：

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=d6020a144f39
ENVIRONMENT=dev
APP_DIR=/usr/local/app/bin
HOME=/root
```

## WORKDIR 指令

这段文字解释了 Dockerfile 中 `WORKDIR` 指令的作用和用法。

**WORKDIR 指令：**

Dockerfile 中的 `WORKDIR` 指令用于设置后续指令在容器内部执行时的**当前工作目录**。 这意味着，在 `WORKDIR` 指令之后出现的 `ADD`、`CMD`、`COPY`、`ENTRYPOINT` 和 `RUN` 等指令，都会在 `WORKDIR` 所指定的目录下执行。

**WORKDIR 指令格式：**

`WORKDIR` 指令的格式如下： 

```dockerfile
WORKDIR /path/to/workdir
```
如果指定的目录在镜像中不存在，Docker 会在构建过程中创建它。 此外，`WORKDIR` 指令有效地结合了类 Unix 系统中 `mkdir` 和 `cd` 命令的功能。 它会在目录不存在时创建目录，并将当前目录更改为指定的路径。

我们可以在一个 Dockerfile 中使用多个 `WORKDIR` 指令。 如果后续的 `WORKDIR` 指令使用相对路径，则它们将相对于最后设置的 `WORKDIR`。

例如： 
```dockerfile
WORKDIR /one
WORKDIR two
WORKDIR three
WORKDIR drink
```

`WORKDIR /one` 会将 `/one` 设置为初始工作目录。 然后，`WORKDIR two` 会将目录更改为 `/one/two`。 `WORKDIR three` 会进一步将其更改为 `one/two/three`。 最后，`WORKDIR drink` 会将其更改为最终形式 `one/two/three/drink`。 

## COPY 指令

在构建 Docker 镜像时，通常会将本地开发环境中的文件包含到镜像本身中。 这些文件可以是从应用程序源代码到配置文件以及应用程序在容器内正常运行所需的其他资源。 Dockerfile 中的 **COPY** 指令通过允许我们指定应将本地文件系统中的哪些文件或目录复制到正在构建的镜像中来实现此目的。

**COPY** 命令的语法如下所示： 

```dockerfile
COPY <source> <destination>
```
`<source>` 指定了本地文件系统上相对于构建上下文的文件或目录的路径。 `<destination>` 指定了文件或目录在 Docker 镜像文件系统中应复制到的路径。

在以下示例中，我们使用 **COPY** 指令将本地文件系统中的 `index.html` 文件复制到 Docker 镜像的 `/var/www/html/` 目录： 
```dockerfile
COPY index.html /var/www/html/index.html
```

我们还可以使用通配符复制与给定模式匹配的所有文件。 在下文中，我们会将当前目录中所有扩展名为 `.html` 的文件复制到 Docker 镜像的 `/var/www/html/` 目录中： 
```dockerfile
COPY *.html /var/www/html/
```

在 Dockerfile 中使用 **COPY** 指令将文件从本地文件系统传输到 Docker 镜像时，我们还可以指定 `--chown` 标志。 此标志允许我们设置 Docker 镜像中复制文件的用户和组所有权。 
```dockerfile
COPY --chown=myuser:mygroup *.html /var/www/html/
```

在这个例子中，`--chown=myuser:mygroup` 指定了所有从本地目录复制到 Docker 镜像中 `/var/www/html/` 的 `.html` 文件都应该归 `myuser`（用户）和 `mygroup`（组）所有。 

## ADD 指令

Dockerfile 中的 **ADD** 指令与 **COPY** 指令功能类似，但具有一些额外的功能。 

```dockerfile
ADD <source> <destination>
```

`<source>` 指定本地文件系统或远程 URL 上的文件或目录的路径或 **URL**。 `<destination>` 再次指定文件或目录在 Docker 镜像文件系统中应复制到的路径。

在下面的示例中，我们将使用 **ADD** 从本地文件系统复制文件： 
```dockerfile
ADD index.html /var/www/html/index.html
```

在这个例子中，Docker 将把 `index.html` 文件从本地文件系统（相对于 Docker 构建上下文）复制到 Docker 镜像内的 `/var/www/html/index.html`。

在下面的例子中，我们将使用 **ADD** 从远程 URL 复制文件： 

```dockerfile
ADD http://example.com/test-data.csv /tmp/test-data.csv
```

与 **COPY** 不同，**ADD** 指令允许指定一个 URL（在本例中是 `http://example.com/test-data.csv`）作为 `<source>` 参数。Docker 将从 URL 下载文件并将其复制到 Docker 镜像内的 `/tmp/test-data.csv`。

**ADD** 指令不仅可以从本地文件系统复制文件或从 URL 下载文件，还可以从某些类型的压缩存档中自动解压缩。当 `<source>` 是压缩存档文件（例如，`.tar`、`.tar.gz`、`.tgz`、`.bz2`、`.tbz2`、`.txz`、`.zip`）时，Docker 会自动将其内容解压缩到 Docker 镜像文件系统内的 `<destination>` 中。

例如： 
```dockerfile
ADD myapp.tar.gz /opt/myapp/
```

在上面的例子中，`myapp.tar.gz` 是一个压缩的存档文件，Docker 会自动将 `myapp.tar.gz` 的内容解压缩到 Docker 镜像内的 `/opt/myapp/` 中。 

### 最佳实践: Dockerfile 中 COPY 与 ADD 的比较 
在编写 Dockerfile 时，在 **COPY** 和 **ADD** 指令之间进行选择对于维护镜像构建过程的清晰度、安全性和可靠性至关重要。 

##### 清晰度和意图（Clarity and Intent）

**COPY** 的使用方式直观明了，它明确指出将本地文件系统中的文件或目录复制到 Docker 镜像中。这种清晰性有助于理解 Dockerfile 的目的，并使其更易于维护。

另一方面，**ADD** 引入了额外的功能，例如从 URL 下载文件和自动解压缩压缩包。虽然这些功能在某些情况下很方便，但它们也可能掩盖简单复制文件的原始意图。如果管理不当，这种缺乏透明度可能会导致意外行为或安全风险。

##### 安全性和可预测性

使用 **COPY** 可以增强安全性，因为它可以避免从任意 URL 下载文件所带来的潜在风险。Docker 镜像的构建应该使用受控的、经过验证的源，以防止包含意外或恶意内容。将文件下载与构建过程分离并使用 `COPY` 可以确保 Docker 构建环境保持安全和可预测。 

##### 与 Docker 理念一致

Docker 鼓励构建轻量级、高效且可预测的容器化应用程序。 **COPY** 通过提升简洁性和降低镜像构建过程中出现意外副作用的风险，与这一理念完美契合。 

### 在 Dockerfile 中使用 WORKDIR、COPY 和 ADD 指令 
在这个例子中，我们要将一个自定义 HTML 文件部署到 Apache Web 服务器上。我们将使用 Ubuntu 作为基础镜像，并在其上安装 Apache。然后，我们将自定义的 `index.html` 文件复制到 Docker 镜像中，并下载一个 Docker logo。

使用 `mkdir` 命令创建一个名为 `workdir-copy-add-example` 的新目录：

```shell
mkdir workdir-copy-add-example
```

进入到新建的 `workdir-copy-add-example` 目录：

```shell
cd .\workdir-copy-add-example\
```

在 `workdir-copy-add-example` 目录中，创建一个名为 `index.html` 的文件。这个文件将在构建镜像时被复制到 Docker 镜像中。我将使用 VS Code，但你可以随意使用你更习惯的任何编辑器： 

```shell
code index.html
```

将以下内容添加到 index.html 文件中，保存并关闭编辑器： 

```html
<html>
    <body>
        <h1>
            Welcome to Docker!
        </h1>
        <img src="logo.png" height="350" width="500"/>
    </body>
</html>
```

这段 HTML 代码创建了一个简单的网页，用一个写着“Welcome to Docker!”的大标题来迎接访问者。在标题下方，它包含了一个使用 `<img>` 标签显示的图片，其源属性 (`src="logo.png"`) 指示它应该获取并显示一个名为 `logo.png` 的图片文件。图片的大小被设置为高 350 像素，宽 500 像素 (`height="350"` 和 `width="500"`)。 

现在，在这个目录下创建一个 **Dockerfile**：

```shell
code Dockerfile
```

将以下内容添加到 `Dockerfile` 文件中，保存并退出：

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
WORKDIR /var/www/html/
COPY index.html .
ADD https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg ./logo.png
CMD ["ls"]
```

这个 Dockerfile 首先指定了 `FROM ubuntu:latest`，表示它将基于可用的最新 Ubuntu 基础镜像构建。接下来的 `RUN apt-get update && apt-get upgrade` 命令更新并升级容器内的软件包列表。然后，`apt-get install apache2 -y` 使用包管理器安装 Apache Web 服务器。`WORKDIR /var/www/html/` 指令将工作目录设置为 `/var/www/html/`，这是 Apache 中用于提供 Web 内容的常见位置。

在这个目录中，`COPY index.html .` 将主机上的本地 `index.html` 文件复制到容器中。此外，`ADD https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg ./logo.png` 从 URL 检索一个 SVG 图像文件，并将其作为 `logo.png` 保存在同一目录的本地。

最后，`CMD ["ls"]` 指定在容器启动时，将执行 `ls` 命令，显示 `/var/www/html/` 中的文件和目录列表。 

现在，使用 `workdir-copy-add` 标签构建 Docker 镜像：
```shell
docker build -t workdir-copy-add .
```

你应该看到以下输出：

```text
[+] Building 4.0s (13/13) FINISHED                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 290B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                   3.6s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                      0.0s
 => [1/6] FROM docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc4362  0.0s
 => => resolve docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc4362  0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 32B                                                                                   0.0s
 => https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg                     0.3s
 => CACHED [2/6] RUN apt-get update && apt-get upgrade                                                             0.0s
 => CACHED [3/6] RUN apt-get install apache2 -y                                                                    0.0s
 => CACHED [4/6] WORKDIR /var/www/html/                                                                            0.0s
 => CACHED [5/6] COPY index.html .                                                                                 0.0s
 => CACHED [6/6] ADD https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg .  0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:646864d79dc576f862a980ef6ddab550ef9801790d2c91967c3c9596cf85b81a                       0.0s
 => => naming to docker.io/library/workdir-copy-add                                                                0.0s
```

执行 `docker container run` 命令，从我们之前构建的 Docker 镜像启动一个新容器：

```shell
docker run workdir-copy-add
```

正如我们所见，`index.html` 和 `logo.png` 都在 `/var/www/html/` 目录中：

```text
index.html
logo.png
```

## USER 指令
在 Docker 中，默认情况下，容器以 root 用户身份运行，该用户在容器环境中拥有广泛的权限。为了降低安全风险，Docker 允许我们使用 **Dockerfile** 中的 **USER** 指令指定一个非 root 用户。此指令设置容器的默认用户，并且 Dockerfile 中指定的所有后续命令（例如 **RUN**、**CMD** 和 **ENTRYPOINT**）都将在该用户的上下文中执行。

在 Docker 安全性中，实现 **USER** 指令被认为是最佳实践，它符合最小权限原则。它确保容器以其功能所需的最小权限运行，从而增强整体系统安全性并减少攻击面。

**USER** 指令采用以下格式：

```
USER <user>
```
除了用户名之外，我们还可以指定可选的组名来运行 Docker 容器：

```
USER <user>:<group>
```
您需要确保 `<user>` 和 `<group>` 值是有效的用户名和组名。否则，Docker 守护进程会在尝试运行容器时抛出错误。

### 在 Dockerfile 中使用 USER 指令  
在这个例子中，我们将使用 **Dockerfile** 中的 **USER** 指令来设置默认用户。我们将安装 Apache Web 服务器并将用户更改为 **www-data**。最后，我们将执行 `whoami` 命令以通过打印用户名来验证当前用户。

创建一个名为 `user-example` 的新目录。


```shell
mkdir user-example
```
进入新创建的目录 `user-example`

```shell
cd .\user-example\
```
在 `user-example` 目录中，创建一个新的 **Dockerfile** 文件。 

```shell
code Dockerfile
```

将以下内容添加到您的 **Dockerfile** 中，保存并关闭编辑器： 

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
USER www-data
CMD ["whoami"]
```

这个 Dockerfile 首先使用最新的 Ubuntu 基础镜像，并在安装 Apache Web 服务器 (`apache2`) 之前更新系统软件包。它通过切换到 `www-data` 用户（通常用于 Web 服务器）来增强安全性，以最大程度地减少潜在漏洞。 `CMD ["whoami"]` 指令确保容器启动时显示当前用户 (`www-data`)，展示了适合在 Docker 环境中托管 Web 应用程序的安全设置。 

构建 Docker 镜像： 

```shell
docker build -t user .
```

And you should see the following output:

```text
[+] Building 5.0s (8/8) FINISHED                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                   0.0s
 => => transferring dockerfile: 157B                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                      0.1s
 => => transferring context: 2B                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                       4.8s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                          0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => => resolve docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => CACHED [2/3] RUN apt-get update && apt-get upgrade                                                                                                 0.0s
 => CACHED [3/3] RUN apt-get install apache2 -y                                                                                                        0.0s
 => exporting to image                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                0.0s
 => => writing image sha256:ce1de597471f741f4fcae898215cfcb0d847aacf7c201690c5d4e95289476768                                                           0.0s
 => => naming to docker.io/library/user                                                                                                                0.0s
```

现在，执行 `docker container run` 命令，从上一步构建的 Docker 镜像启动一个新容器： 

```shell
docker container run user
```
输出应该显示 `www-data` 作为与 Docker 容器关联的当前用户： 

```text
www-data
```

## VOLUME 指令
在 Docker 中，容器旨在以可移植和轻量级的方式封装应用程序及其依赖项。但是，默认情况下，在 Docker 容器文件系统中生成或修改的任何数据都是短暂的，这意味着它仅在容器运行时存在。当容器被删除或替换时，这些数据就会丢失，这对需要持久存储的应用程序（例如数据库或文件存储系统）提出了挑战。

为了应对这一挑战，Docker 引入了卷的概念。卷提供了一种独立于容器生命周期来持久化数据的方法。它们充当 Docker 容器和主机之间的桥梁，确保即使在容器停止、移除或替换时，存储在卷中的数据也能保留下来。这使得卷对于需要跨容器实例维护状态信息的应用程序（例如存储数据库、配置文件或应用程序日志）至关重要。

当您使用 `VOLUME` 指令在 Dockerfile 中定义卷时，Docker 会在容器的文件系统中创建一个托管目录。此目录用作卷的挂载点。至关重要的是，Docker 还会在主机上建立一个相应的目录，用于存储卷的实际数据。这种映射确保了从容器内部对卷中的文件所做的任何更改都会立即与主机上的映射目录同步，反之亦然。

Docker 中的卷支持多种类型，包括命名卷和主机挂载卷。命名卷由 Docker 创建和管理，对卷生命周期和存储管理提供了更多控制和灵活性。另一方面，主机挂载卷允许您将主机文件系统中的目录直接挂载到容器中，从而提供对主机资源的直接访问。

**VOLUME** 指令通常将 JSON 数组作为参数： 

```dockerfile
VOLUME ["path/to/volume"]
```

或者，我们可以指定一个包含多个路径的普通字符串： 

```dockerfile
VOLUME /path/to/volume1 /path/to/volume2
```

我们可以使用 `docker container inspect <容器>` 命令来查看容器中可用的卷。 docker container inspect 命令的输出 JSON 将打印类似于以下内容的卷信息： 

```text
[
   {
      "CreatedAt":"2024-06-21T22:52:52+03:00",
      "Driver":"local",
      "Labels":null,
      "Mountpoint":"/var/lib/docker/volumes/f46f82ea6310d0db3a13897a0c3ab45e659ff3255eaeead680b48bca37cc0166/_data",
      "Name":"f46f82ea6310d0db3a13897a0c3ab45e659ff3255eaeead680b48bca37cc0166",
      "Options":null,
      "Scope":"local"
   }
]
```

### 在 Dockerfile 中使用 VOLUME 指令
在这个例子中，我们将设置一个 Docker 容器来运行 Apache Web 服务器。但是，我们不希望在 Docker 容器出现故障时丢失 Apache 日志文件。作为解决方案，我们将通过将 Apache 日志路径挂载到基础 Docker 主机来持久化日志文件。

创建一个名为 `volume-example` 的新目录 

```shell
mkdir volume-example
```

进入新创建的 `volume-example` 目录

```shell
cd volume-example
```
在 `volume-example`目录里创建新的 **Dockerfile** 文件

```shell
code Dockerfile
```
将以下内容添加到 **Dockerfile**，保存并退出。

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
VOLUME ["/var/log/apache2"]
```

此 Dockerfile 首先使用最新版本的 Ubuntu 作为基础镜像，并通过运行 `apt-get update` 和 `apt-get upgrade` 来更新所有已安装的软件包，以确保其是最新的。然后，它使用 `apt-get install apache2 -y` 安装 Apache HTTP 服务器 (`apache2`)。 `VOLUME ["/var/log/apache2"]` 指令在 `/var/log/apache2` 处定义了一个 Docker 卷，这是 Apache 通常存储其日志文件的位置。

现在，让我们构建 Docker 镜像： 

```shell
docker build -t volume .
```
如下输出:
```text
[+] Building 3.6s (8/8) FINISHED                                                                                                             docker:default
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [internal] load build definition from Dockerfile                                                                                                   0.0s
 => => transferring dockerfile: 155B                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                       3.5s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                          0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => => resolve docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => CACHED [2/3] RUN apt-get update && apt-get upgrade                                                                                                 0.0s
 => CACHED [3/3] RUN apt-get install apache2 -y                                                                                                        0.0s
 => exporting to image                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                0.0s
 => => writing image sha256:9c7a81e379553444e0b4f3bbf45bdd17880aea251db8f8b75669e13964b9c30f                                                           0.0s
 => => naming to docker.io/library/volume
```

执行 `docker container run` 命令从先前构建的镜像启动一个新容器。注意，你还需要使用 `--interactive` 和 `--tty` 标志来打开一个交互式的 bash 会话，这样你就可以从容器的 bash shell 执行命令。同时，你需要使用 `--name` 标志来定义容器的名称为 `volume-container`。


```shell
docker container run --interactive --tty --name volume-container volume /bin/bash
```

你的 bash shell 将如下打开：
```shell
root@8aa0f5fb8a6d:/#
```

进入 `/var/log/apache2` 目录

```shell
root@8aa0f5fb8a6d:/# cd /var/log/apache2/
```

这将产生以下输出：

```shell
root@8aa0f5fb8a6d:/var/log/apache2#
```

现在，列出目录中的可用文件。

```shell
root@8aa0f5fb8a6d:/var/log/apache2# ls -l
```

如下输出
```text
total 0
-rw-r----- 1 root adm 0 Jun 20 13:42 access.log
-rw-r----- 1 root adm 0 Jun 20 13:42 error.log
-rw-r----- 1 root adm 0 Jun 20 13:42 other_vhosts_access.log
```

这些是由 Apache 在运行过程中创建的日志文件。 当您检查此卷的宿主机的挂载点时，应该能够看到相同的文件。 

退出容器，回到宿主机。

```shell
root@8aa0f5fb8a6d:/var/log/apache2# exit
```

检查 `volume-container` 以查看挂载信息。 

```shell
docker container inspect volume-container
```

在 `Mounts` 键下，你将能够看到与挂载相关的信息。 

```text
"Mounts":[
    {
         "Type":"volume",
         "Name":"50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8",
         "Source":"/var/lib/docker/volumes/50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8/_data",
         "Destination":"/var/log/apache2",
         "Driver":"local",
         "Mode":"",
         "RW":true,
         "Propagation":""
    }
]
```

使用 `docker volume inspect <volume_name>` 命令检查卷。 您可以在先前输出的 `Name` 字段中找到 `<volume_name>` 。 

```shell
docker volume inspect 50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8
```

你应该会得到类似于以下内容的输出： 

```text
[{
   "CreatedAt":"2024-06-21T11:02:32Z",
   "Driver":"local",
   "Labels":{
      "com.docker.volume.anonymous":""
   },
   "Mountpoint":"/var/lib/docker/volumes/50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8/_data",
   "Name":"50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8",
   "Options":null,
   "Scope":"local"
}]
```


列出主机文件路径中可用的文件。 主机文件路径可以通过先前输出的 `Mountpoint` 字段识别。 


```shell
ls -l /var/lib/docker/volumes/50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8/_data
```

## EXPOSE 指令

Docker 中的 `EXPOSE` 指令用于指示容器在运行时将在特定端口上进行监听。 此声明主要用于提供信息，默认情况下不会将端口发布到主机系统或使其可从容器外部访问。 相反，它记录了哪些端口打算用于 Docker 环境中的容器间通信或网络服务。

`EXPOSE` 指令支持 TCP 和 UDP 协议，允许灵活地公开端口以满足各种网络需求。 此指令是容器运行时使用的 `-p` 或 `-P` 选项的前身，用于将这些公开的端口实际映射到主机上的端口，以便在需要时启用外部访问。

EXPOSE 指令的格式如下：

```dockerfile
EXPOSE <port>
```

## HEALTHCHECK 指令

健康检查是一个关键机制，旨在评估容器的运行健康状况。它提供了一种方法来验证在 Docker 容器内运行的应用程序是否正常工作。如果没有指定健康检查，Docker 就缺乏自主确定容器健康状态的能力。这在生产环境中尤为关键，因为可靠性和系统正常运行时间至关重要。

Docker 中的 **HEALTHCHECK** 指令允许开发人员定义自定义健康检查，通常以命令或脚本的形式，定期检查容器的状态并报告其健康状况。这个指令确保了主动监控，并帮助 Docker 编排工具根据健康状态做出有关容器生命周期管理的明智决策。

在一个 **Dockerfile** 中只能有一个 **HEALTHCHECK** 指令。如果有多个 **HEALTHCHECK** 指令，只有最后一个会生效。

例如，我们可以使用以下指令来确保容器能够在 `http://localhost/` 上接收流量：

```dockerfile
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

在前面命令的最后，退出码被用来指定容器的健康状态，`0` 和 `1` 是这个字段的有效值。`0` 表示容器健康，`1` 表示容器不健康。

在 Docker 中使用 **HEALTHCHECK** 指令时，可以配置额外的参数来定制健康检查的执行方式：

-   **`--interval`**：指定执行健康检查的频率，默认间隔为30秒。
-   **`--timeout`**：定义健康检查命令成功完成所允许的最大时间。如果在这个时间段内没有收到成功响应，则健康检查被标记为失败。默认超时时间也设置为30秒。
-   **`--start-period`**：指定在 Docker 开始执行第一次健康检查之前的初始延迟。这个参数允许容器在开始健康检查之前有时间进行初始化，默认开始期为0秒。
-   **`--retries`**：定义在 Docker 认为容器不健康之前允许的连续失败的健康检查次数。默认情况下，Docker 允许最多3次重试。

在以下示例中，通过提供自定义值，覆盖了 **HEALTHCHECK** 的默认值：

```dockerfile
HEALTHCHECK \
    --interval=1m \
    --timeout=2s \
    --start-period=2m \
    --retries=3 \
    CMD curl -f http://localhost/ || exit 1

```

### 在 Dockerfile 中使用 EXPOSE 和 HEALTHCHECK 

我们将要将 Apache Web 服务器容器化，以便从 Web 浏览器访问 Apache 主页。此外，我们将配置健康检查来确定 Apache Web 服务器的健康状态。

创建一个名为 `expose-heathcheck-example` 的新目录。

```shell
mkdir expose-healthcheck-example
```

进入新创建的 `expose-healthcheck-example` 目录。

```shell
cd .\expose-healthcheck-example\
```

创建 **Dockerfile** 文件，并加入如下内容。

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade

RUN apt-get install apache2 curl -y

HEALTHCHECK CMD curl -f http://localhost/ || exit 1

EXPOSE 80

ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]
```
这个 Dockerfile 从拉取最新的 Ubuntu 基础镜像并更新它开始。然后它使用 `apt-get` 安装 Apache Web 服务器和 curl。`HEALTHCHECK` 指令设置为运行健康检查命令（`curl -f http://localhost/ || exit 1`），确保基于 localhost 连接的容器健康。暴露 80 端口以允许外部访问 Apache。最后，容器被配置为使用 `ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]` 以前台模式运行 Apache，确保它作为主进程保持活动和响应。这个设置使得在 Docker 环境内通过端口 80 访问托管的 Web 服务器成为可能。


构建镜像

```shell
docker image build -t expose-healthcheck-example .
```

你会获得如下输出:

```text
[+] Building 29.0s (8/8) FINISHED                                                                                                            docker:default
 => [internal] load build definition from Dockerfile                                                                                                   0.0s
 => => transferring dockerfile: 244B                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                       3.4s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                          0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => => resolve docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30                                 0.0s
 => CACHED [2/3] RUN apt-get update && apt-get upgrade                                                                                                 0.0s
 => [3/3] RUN apt-get install apache2 curl -y                                                                                                         24.8s
 => exporting to image                                                                                                                                 0.6s
 => => exporting layers                                                                                                                                0.6s
 => => writing image sha256:3323e865b3888a4e45852c6a8c163cb820739735716f8783a0d126b43d810f1e                                                           0.0s
 => => naming to docker.io/library/expose-healthcheck-example                                                                                          0.0s
```
执行 `docker container run` 命令来启动一个新容器。您将使用 `-p` 标志将主机的端口 `80` 重定向到容器的端口 `8080`。此外，您将使用 `--name` 标志指定容器名称为 `expose-healthcheck-container`，并使用 `-d` 标志以分离模式运行容器。

```shell
docker container run -p 8080:80 --name expose-healthcheck-container -d expose-healthcheck-example
```

使用 `docker container list` 命令列出正在运行的容器。

```shell
docker container list
```

在输出中，你将看到 `expose-healthcheck-container` 的 `STATUS` 显示为健康（healthy）。

```text
CONTAINER ID   IMAGE                        COMMAND                  CREATED              STATUS                        PORTS                            NAMES
3ff16b11275c   expose-healthcheck-example   "apache2ctl -D FOREG…"   About a minute ago   Up About a minute (healthy)   80/tcp, 0.0.0.0:8080->8080/tcp   expose-healthcheck-container
```

现在，您应该能够查看 Apache 主页。请从您的浏览器访问 `http://127.0.0.1:8080`。

[![Image description](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fynq7tnklqzp44zhosob2.png)][47]

## ONBUILD 指令

Dockerfile 中的 ONBUILD 指令有助于创建可重用的基础镜像，这些基础镜像用于后续的镜像构建。它允许开发人员定义只有当另一个 Docker 镜像使用当前镜像作为其基础时才会触发的指令。例如，你可以构建一个 Docker 镜像，包含运行应用程序所需的所有必备前提条件和配置。

通过在这个“前提条件”镜像中应用 ONBUILD 指令，可以将特定指令推迟到该镜像作为父镜像在另一个 Dockerfile 中使用时再执行。这些推迟的指令在当前 Dockerfile 的构建过程中不会被执行，而是在构建子镜像时被继承和执行。这种方法简化了设置环境的过程，并确保基础镜像衍生的多个项目或应用程序中一致地应用了常见的依赖项和配置。

**ONBUILD** 指令的格式如下：

```dockerfile
ONBUILD <instruction>
```

例如，设想我们在自定义基础镜像的 **Dockerfile** 中有以下的 **ONBUILD** 指令。
```dockerfile
ONBUILD ENTRYPOINT ["echo", "Running an ONBUILD Directive"]
```

如果我们从自定义基础镜像创建一个 Docker 容器，`Running an ONBUILD Directive` 的值不会被打印出来，但如果我们使用它作为另一个 Docker 镜像的基础，则会被打印出来。

### 在 Dockerfile 中使用 ONBUILD 指令

在这个例子中，我们将构建一个带有 Apache 网络服务器的父镜像，并使用 **ONBUILD** 指令来复制 HTML 文件。

创建文件夹 `onbuild-parent-example`

```shell
mkdir onbuild-parent-example
```

进入刚创建 `onbuild-parent-example` 目录:

```shell
cd .\onbuild-parent-example\
```

创建 **Dockerfile** 文件，加入如下内容：

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade

RUN apt-get install apache2 -y

ONBUILD COPY *.html /var/www/html

EXPOSE 80

ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]
```

这个 Dockerfile 从使用最新的 Ubuntu 基础镜像开始。它更新并升级了系统包，然后安装了 Apache 网络服务器。ONBUILD 指令指定任何从这个 Dockerfile 构建的子镜像都将自动将所有 HTML 文件从构建上下文复制到容器内的 `/var/www/html` 目录。暴露 80 端口以允许流入的流量访问 Apache 服务器。最后，`ENTRYPOINT` 命令配置容器以前台模式运行 Apache，确保它作为主进程保持活跃和响应。这个设置使得容器能够通过 Apache 在 80 端口提供网页内容。

现在，构建 Docker 镜像:

```shell
docker image build -t onbuild-parent-example .
```

输出如下内容:

```text
[+] Building 3.5s (8/8) FINISHED                                                                         docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 221B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.1s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                   3.3s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                      0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc4362  0.0s
 => => resolve docker.io/library/ubuntu:latest@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc4362  0.0s
 => CACHED [2/3] RUN apt-get update && apt-get upgrade                                                             0.0s
 => CACHED [3/3] RUN apt-get install apache2 -y                                                                    0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:4a6360882fb65415cdd7326392de35c2336a8599c2c4b8b7a1e4d962d81df7e4                       0.0s
 => => naming to docker.io/library/onbuild-parent-example                                                          0.0s
```

执行 `docker container run` 命令从上一步构建的 Docker 镜像启动一个新容器：
```shell
docker container run -p 8080:80 --name onbuild-parent-container -d onbuild-parent-example
```


If you navigate to the `http://127.0.0.1:8080/` endpoint you should see the default Apache home page

[![Image description](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fynq7tnklqzp44zhosob2.png)][50]

---

Remove the container so it won't interfere with the ports

```shell
docker container stop onbuild-parent-container
docker container rm onbuild-parent-container
```

---

Now, lets create another Docker image using `onbuild-parent-container` as the base image, to deploy a custom HTML home page. To do that let's create a new directory named `onbuild-child-example`

```shell
cd ..
mkdir onbuild-child-example
```

---

Create a new html page with the following content

```html
<html>

    <body>

        <h1>Demonstrating Docker ONBUILD Directive</h1>

    </body>

</html>
```

---

In the same directory create a **Dockerfile**

```
FROM onbuild-parent-example
```

This **Dockerfile** has a single directive. This will use the **FROM** directive to utilize the `onbuild-parent-example` Docker image that we created previously as the base image.

---

Now, build the docker image

```shell
docker image build -t onbuild-child-example .
```

The output should be something like the following

```text
[+] Building 0.3s (7/7) FINISHED                                                                         docker:default
 => [internal] load .dockerignore                                                                                  0.1s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                               0.1s
 => => transferring dockerfile: 64B                                                                                0.0s
 => [internal] load metadata for docker.io/library/onbuild-parent-example:latest                                   0.0s
 => [internal] load build context                                                                                  0.1s
 => => transferring context: 134B                                                                                  0.0s
 => [1/1] FROM docker.io/library/onbuild-parent-example                                                            0.1s
 => [2/1] COPY *.html /var/www/html                                                                                0.0s
 => exporting to image                                                                                             0.1s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:9fb3629a292e2536300724db933eb59a6fb918f9d01e46a01aff18fe1ad6fe69                       0.0s
 => => naming to docker.io/library/onbuild-child-example                                                           0.0s
```

---

Execute the `docker container run` command to start a new container with the image we just built

```shell
docker container run -p 8080:80 --name onbuild-child-container -d onbuild-child-example
```

---

You should be able now to view our custom `index.html` page if you navigate to the `http://127.0.0.1:8080/` endpoint.

[![Image description](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdecdrnoany4idkkrrllx.png)][51]

## [][52]Summary

In this post we focused on building Docker images. We we discussed more advanced Dockerfile directives, including the **ENV**, **ARG**, **WORKDIR**, **COPY**, **ADD**, **USER**, **VOLUME**, **EXPOSE**, **HEALTHCHECK**, and **ONBUILD** directives.

In the next few posts we are going to discuss what a Docker registry is, look at private and public Docker registries and how we can publish images to Docker registries.

---

Buckle up, buttercup! This docker journey is about to get even wilder. For references, just check out the first post in this series. It's your one-stop shop for all the nitty-gritty details.

## [The Docker Workshop (6 Part Series)][53]

[1 Running our First Docker Image][54] [2 Basic Container Lifecycle Management][55] [... 2 more parts...][56] [3 Getting Started with Dockerfiles][57] [4 Attaching To Containers Using the Attach Command][58] [5 Building Docker Images][59] [6 Advanced Dockerfile Directives][60]

## Top comments (12)

Subscribe

Collapse Expand



[![rouqe profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1293582%2F1fb5203a-fb01-4377-94c6-fb6c77699e59.jpg)][63]

[Jimben Honeyfield][64]

Jimben Honeyfield

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1293582%2F1fb5203a-fb01-4377-94c6-fb6c77699e59.jpg)Jimben Honeyfield][65]

Follow

Developer, and DevOps Practitioner.

-   Joined

    Feb 22, 2024

• [Jul 10][66]

Dropdown menu

-   [Copy link][67]

-   Hide

Great article writeup! I especially liked the **ONBUILD COPY** part!

I always felt having a single container image that copies the contents of a file didn't lend itself very well to an iterative approach for larger container sizes. For instance, write some code, test, build container over, and over. By splitting out the requirements into a separate container image, and using the **ONBUILD COPY** command I see how this can work effectively.

Thanks for the great article writeup! Looking forward to the next one!!

Like comment: Like comment: 2 likes Like [Comment button Reply][68]

Collapse Expand



[![kalkwst profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][69]

[Kostas Kalafatis][70]

Kostas Kalafatis

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)Kostas Kalafatis][71]

Follow

Sitecore employee by day, gamer by night. Battling my impostor syndrome and coding useless stuff since 2008.

-   Email

    [kalafatiskwstas@gmail.com][72]

-   Location

    Athens, Greece

-   Education

    MSc in Applied Informatics

-   Work

    Senior .NET Developer at Sitecore

-   Joined

    Jan 25, 2018

• [Jul 10][73]

Dropdown menu

-   [Copy link][74]

-   Hide

Thank you for your feedback! I'm glad you found the article helpful, especially the discussion on `ONBUILD COPY`.

You're absolutely right about the challenges of maintaining larger container images in iterative development cycles. `ONBUILD COPY` offers a great solution by allowing us to separate the dependencies and build artifacts into a builder image. This approach not only streamlines the Docker build process but also enhances flexibility and repeatability in deployments.

By decoupling requirements handling from the main application image, we can manage changes and updates more efficiently. It also ensures that each build incorporates the latest dependencies without bloating the final image size unnecessarily.

Keep in mind that the same can be achieved with multi-stage Docker builds though.

In multistage builds, you define multiple `FROM` instructions in your Dockerfile, each representing a different stage or phase of the build process. Typically, you start with a builder stage where you compile or build your application along with its dependencies. Once the build artifacts are ready, you can then copy only the necessary files from the builder stage into a smaller, final stage image. This final image is what you ultimately use for deployment.

For example, you first create a _builder stage_ image. This stage is where you install dependencies, compile code, and generate any necessary artifacts. You can use a larger image with build tools and dependencies.

```
FROM base-image as builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
```

And then you can use a final stage image. In this stage, you start with a minimal base image (like `alpine` or `scratch`) and copy only the built artifacts from the builder stage. This ensures that the final image is as small and efficient as possible.

```
FROM base-image
COPY --from=builder /app/dist /app
CMD ["npm", "start"]
```

This approach not only reduces the size of your Docker image by excluding unnecessary build dependencies but also enhances security and build efficiency. Each stage runs in isolation, allowing Docker to cache intermediate images and speed up subsequent builds.

When choosing between `ONBUILD COPY` and multistage Docker builds, align your decision with your project's specific needs and complexity.

**ONBUILD COPY:**

-   Best for simpler projects with straightforward build processes.
-   Ideal for automating the copying of files into derived images.
-   Works seamlessly for single-stage builds, keeping Dockerfiles concise.
-   Well-suited for creating generic base images for wider use.

**Multistage Builds:**

-   Excels in complex environments with multiple stages and dependencies.
-   Offers fine-grained control over build stages and image layers.
-   Optimizes final image size by eliminating unnecessary build tools and dependencies.
-   Enhances security by reducing the attack surface.
-   Supports iterative development with efficient caching mechanisms.

In essence, if your project involves simple build steps and you want to streamline the process of inheriting files from a base image, `ONBUILD COPY` is a practical choice. However, if you're dealing with more intricate build processes, require greater control over image layers, or prioritize minimizing image size and maximizing security, multistage builds are the way to go.

Both approaches have their strengths and cater to different use cases.

If you have any more insights or experiences to share, I'd love to hear them. Feel free to reach out anytime.

Like comment: Like comment: 3 likes Like [Comment button Reply][75]

Collapse Expand



[![rouqe profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1293582%2F1fb5203a-fb01-4377-94c6-fb6c77699e59.jpg)][76]

[Jimben Honeyfield][77]

Jimben Honeyfield

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1293582%2F1fb5203a-fb01-4377-94c6-fb6c77699e59.jpg)Jimben Honeyfield][78]

Follow

Developer, and DevOps Practitioner.

-   Joined

    Feb 22, 2024

• [Jul 10][79]

Dropdown menu

-   [Copy link][80]

-   Hide

Wow thank you for the detailed response! You answered my question before I had the chance to ask it! 😆

Like comment: Like comment: 2 likes Like [Comment button Reply][81]

Collapse Expand



[![leadsbuilds profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1072155%2Fec182b38-43d1-464f-b24d-4069a9403845.jpeg)][82]

[Wendell][83]

Wendell

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1072155%2Fec182b38-43d1-464f-b24d-4069a9403845.jpeg)Wendell][84]

Follow

-   Joined

    Apr 26, 2023

• [Jul 9][85]

Dropdown menu

-   [Copy link][86]

-   Hide

Very useful, thanks

Like comment: Like comment: 2 likes Like [Comment button Reply][87]

Collapse Expand



[![kalkwst profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][88]

[Kostas Kalafatis][89]

Kostas Kalafatis

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)Kostas Kalafatis][90]

Follow

Sitecore employee by day, gamer by night. Battling my impostor syndrome and coding useless stuff since 2008.

-   Email

    [kalafatiskwstas@gmail.com][91]

-   Location

    Athens, Greece

-   Education

    MSc in Applied Informatics

-   Work

    Senior .NET Developer at Sitecore

-   Joined

    Jan 25, 2018

• [Jul 10][92]

Dropdown menu

-   [Copy link][93]

-   Hide

Glad to hear you found the information helpful!

Like comment: Like comment: 1 like Like [Comment button Reply][94]

Collapse Expand



[![jangelodev profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F245471%2F5b9986ec-0a88-415e-9878-710f9d8fc1ec.jpg)][95]

[João Angelo][96]

João Angelo

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F245471%2F5b9986ec-0a88-415e-9878-710f9d8fc1ec.jpg)João Angelo][97]

Follow

Full Stack Developer. Transforming ideas into innovative applications with Angular, Angular Material, Nx, C#, .NET Core, HTML, CSS and OAuth2.0.

-   Location

    Brazil

-   Pronouns

    He / Him

-   Joined

    Oct 8, 2019

• [Jul 8][98]

Dropdown menu

-   [Copy link][99]

-   Hide

Hi Kostas Kalafatis,  
Top, very nice and helpful !  
Thanks for sharing.

Like comment: Like comment: 2 likes Like [Comment button Reply][100]

Collapse Expand



[![kalkwst profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][101]

[Kostas Kalafatis][102]

Kostas Kalafatis

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)Kostas Kalafatis][103]

Follow

Sitecore employee by day, gamer by night. Battling my impostor syndrome and coding useless stuff since 2008.

-   Email

    [kalafatiskwstas@gmail.com][104]

-   Location

    Athens, Greece

-   Education

    MSc in Applied Informatics

-   Work

    Senior .NET Developer at Sitecore

-   Joined

    Jan 25, 2018

• [Jul 10][105]

Dropdown menu

-   [Copy link][106]

-   Hide

Thank you for your kind words! I'm glad you found the information helpful.

Like comment: Like comment: 2 likes Like [Comment button Reply][107]

Collapse Expand



[![clabnet profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F191444%2Ff78718ad-8249-4c83-be2e-94b47d4a1eb7.jpeg)][108]

[Claudio Barca][109]

Claudio Barca

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F191444%2Ff78718ad-8249-4c83-be2e-94b47d4a1eb7.jpeg)Claudio Barca][110]

Follow

-   Joined

    Jul 7, 2019

• [Jul 8][111]

Dropdown menu

-   [Copy link][112]

-   Hide

Very nice.

Like comment: Like comment: 2 likes Like [Comment button Reply][113]

Collapse Expand



[![kalkwst profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][114]

[Kostas Kalafatis][115]

Kostas Kalafatis

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)Kostas Kalafatis][116]

Follow

Sitecore employee by day, gamer by night. Battling my impostor syndrome and coding useless stuff since 2008.

-   Email

    [kalafatiskwstas@gmail.com][117]

-   Location

    Athens, Greece

-   Education

    MSc in Applied Informatics

-   Work

    Senior .NET Developer at Sitecore

-   Joined

    Jan 25, 2018

• [Jul 10][118]

Dropdown menu

-   [Copy link][119]

-   Hide

Thank you, I'm glad you liked it!

Like comment: Like comment: 1 like Like [Comment button Reply][120]

Collapse Expand



[![adriens profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F446871%2F3e9ded5c-f368-4906-a277-35e56c9f97a7.png)][121]

[adriens][122]

adriens

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F446871%2F3e9ded5c-f368-4906-a277-35e56c9f97a7.png)adriens][123]

Follow

I love to create, connect people & things.

-   Location

    Nouméa, New-Caledonia

-   Pronouns

    He

-   Work

    Division manager at OPT-NC New Caledonia

-   Joined

    Aug 4, 2020

• [Jul 12][124]

Dropdown menu

-   [Copy link][125]

-   Hide

Thanks a lot for having made me discover the `HEALTHCHECK`, I did'nt know it was existing wthin docker.

Like comment: Like comment: 1 like Like [Comment button Reply][126]

Collapse Expand



[![oriroth profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1763946%2F571f84f1-bb0c-4ee5-9f73-f68604604cae.jpg)][127]

[אורי רוט][128]

אורי רוט

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1763946%2F571f84f1-bb0c-4ee5-9f73-f68604604cae.jpg)אורי רוט][129]

Follow

-   Joined

    Jul 11, 2024

• [Jul 11][130]

Dropdown menu

-   [Copy link][131]

-   Hide

Loved it!  
Please write the next one about multi layered Dockerfile

Like comment: Like comment: 1 like Like [Comment button Reply][132]

Collapse Expand



[![kalkwst profile image](https://media.dev.to/cdn-cgi/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)][133]

[Kostas Kalafatis][134]

Kostas Kalafatis

[![](https://media.dev.to/cdn-cgi/image/width=90,height=90,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F55227%2Fe91bf4bf-777b-4937-add8-f6710f9b94c8.jpg)Kostas Kalafatis][135]

Follow

Sitecore employee by day, gamer by night. Battling my impostor syndrome and coding useless stuff since 2008.

-   Email

    [kalafatiskwstas@gmail.com][136]

-   Location

    Athens, Greece

-   Education

    MSc in Applied Informatics

-   Work

    Senior .NET Developer at Sitecore

-   Joined

    Jan 25, 2018

• [Jul 11][137]

Dropdown menu

-   [Copy link][138]

-   Hide

Thank you so much for your feedback and enthusiasm! I'm really glad you enjoyed the article. I appreciate your interest in multi-layered Dockerfiles. They're indeed a powerful topic that deserves detailed exploration.

While the next article won't specifically cover multi-layered Dockerfiles, I'm planning to delve into that topic soon. If all goes as scheduled, you can expect it around August 12th. Stay tuned for more updates and thank you for your continued interest and support!

Like comment: Like comment: 2 likes Like [Comment button Reply][139]

[Code of Conduct][140] • [Report abuse][141]

For further actions, you may consider blocking this person and/or [reporting abuse][142]

## Read next

[

![skipperhoa profile image](https://media.dev.to/cdn-cgi/image/width=100,height=100,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F219700%2F3c33e606-a6fa-40f5-af19-c7e52fecac4c.jpg)

### ⚡ MyFirstApp - React Native with Expo (P24) - Code Layout Register Screen

Hòa Nguyễn Coder - Jul 4

][143]

![tolu1123 profile image](https://media.dev.to/cdn-cgi/image/width=100,height=100,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1018225%2Fbd9bf0d5-d89c-489e-b35d-374bcc04a8d0.png)

### React and JS For Begineers

tolu1123 - Jun 29

][144]

![sudhanshu_developer profile image](https://media.dev.to/cdn-cgi/image/width=100,height=100,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1702127%2Fdeaff21b-ba33-4b92-b83d-83db00f5ee35.png)

### Displaying Car Information Dynamically Using JavaScript and HTML

Sudhanshu Gaikwad - Jul 4

][145]

![insideee_dev profile image](https://media.dev.to/cdn-cgi/image/width=100,height=100,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1323597%2Fe6bc269a-fd18-4ec6-826f-c0334b593878.png)

### Customizing Your Lazyvim Setup for Personal Preferences

insideee.dev - Jun 30

][146]

[DEV Community][152] — A constructive and inclusive social network for software developers. With you every step of your journey.

-   [Home][153]
-   [Podcasts][154]
-   [Videos][155]
-   [Tags][156]
-   [DEV Help][157]
-   [Forem Shop][158]
-   [Advertise on DEV][159]
-   [DEV Challenges][160]
-   [DEV Showcase][161]
-   [About][162]
-   [Contact][163]
-   [Guides][164]
-   [Software comparisons][165]

-   [Code of Conduct][166]
-   [Privacy Policy][167]
-   [Terms of use][168]

Built on [Forem][169] — the [open source][170] software that powers [DEV][171] and other inclusive communities.

Made with love and [Ruby on Rails][172]. DEV Community © 2016 - 2024.

![DEV Community](https://media.dev.to/cdn-cgi/image/width=190,height=,fit=scale-down,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8j7kvp660rqzt99zui8e.png)

We're a place where coders share, stay up-to-date and grow their careers.

[Log in][173] [Create account][174]

![](https://dev.to/assets/sparkle-heart-5f9bee3767e18deb1bb725290cb151c25234768a0e9a2bd39370c382d02920cf.svg) ![](https://dev.to/assets/multi-unicorn-b44d6f8c23cdd00964192bedc38af3e82463978aa611b4365bd33a0f1f4f3e97.svg) ![](https://dev.to/assets/exploding-head-daceb38d627e6ae9b730f36a1e390fca556a4289d5a41abb2c35068ad3e2c4b5.svg) ![](https://dev.to/assets/raised-hands-74b2099fd66a39f2d7eed9305ee0f4553df0eb7b4f11b01b6b1b499973048fe5.svg) ![](https://dev.to/assets/fire-f60e7a582391810302117f987b22a8ef04a2fe0df7e3258a5f49332df1cec71e.svg)

[3]: https://www.algolia.com/developers/?utm_source=devto&utm_medium=referral
[7]: https://twitter.com/intent/tweet?text=%22Advanced%20Dockerfile%20Directives%22%20by%20Kostas%20Kalafatis%20%23DEVCommunity%20https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f
[8]: https://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f&title=Advanced%20Dockerfile%20Directives&summary=In%20this%20post%2C%20we%20are%20going%20to%20discuss%20more%20advanced%20Dockerfile%20directives.%20These%20directives%20can%20be...&source=DEV%20Community
[9]: https://www.reddit.com/submit?url=https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f&title=Advanced%20Dockerfile%20Directives
[10]: https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f&t=Advanced%20Dockerfile%20Directives
[11]: https://www.facebook.com/sharer.php?u=https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f
[12]: https://toot.kytta.dev/?text=https%3A%2F%2Fdev.to%2Fkalkwst%2Fadvanced-dockerfile-directives-193f
[14]: https://media.dev.to/cdn-cgi/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fud6oyinws6xha4i4f179.png
[47]: https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fynq7tnklqzp44zhosob2.png
[50]: https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fynq7tnklqzp44zhosob2.png
[51]: https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdecdrnoany4idkkrrllx.png
[63]: https://dev.to/rouqe
[64]: https://dev.to/rouqe
[66]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2geaj
[67]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2geaj
[68]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2geaj
[69]: https://dev.to/kalkwst
[70]: https://dev.to/kalkwst
[72]: mailto:kalafatiskwstas@gmail.com
[73]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gebo
[74]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gebo
[75]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gebo
[76]: https://dev.to/rouqe
[77]: https://dev.to/rouqe
[79]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gec5
[80]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gec5
[81]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gec5
[82]: https://dev.to/leadsbuilds
[83]: https://dev.to/leadsbuilds
[85]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gdoe
[86]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gdoe
[87]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gdoe
[88]: https://dev.to/kalkwst
[89]: https://dev.to/kalkwst
[91]: mailto:kalafatiskwstas@gmail.com
[92]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge65
[93]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge65
[94]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2ge65
[95]: https://dev.to/jangelodev
[96]: https://dev.to/jangelodev
[98]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gd68
[99]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gd68
[100]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gd68
[101]: https://dev.to/kalkwst
[102]: https://dev.to/kalkwst
[104]: mailto:kalafatiskwstas@gmail.com
[105]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge68
[106]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge68
[107]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2ge68
[108]: https://dev.to/clabnet
[109]: https://dev.to/clabnet
[111]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gd6l
[112]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gd6l
[113]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gd6l
[114]: https://dev.to/kalkwst
[115]: https://dev.to/kalkwst
[117]: mailto:kalafatiskwstas@gmail.com
[118]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge66
[119]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2ge66
[120]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2ge66
[121]: https://dev.to/adriens
[122]: https://dev.to/adriens
[124]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gf7e
[125]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gf7e
[126]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gf7e
[127]: https://dev.to/oriroth
[128]: https://dev.to/oriroth
[130]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gek1
[131]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gek1
[132]: #/kalkwst/advanced-dockerfile-directives-193f/comments/new/2gek1
[133]: https://dev.to/kalkwst
[134]: https://dev.to/kalkwst
[136]: mailto:kalafatiskwstas@gmail.com
[137]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gele
[138]: https://dev.to/kalkwst/advanced-dockerfile-directives-193f#comment-2gele
[169]: https://www.forem.com
[170]: https://dev.to/t/opensource
[171]: https://dev.to
[172]: https://dev.to/t/rails
