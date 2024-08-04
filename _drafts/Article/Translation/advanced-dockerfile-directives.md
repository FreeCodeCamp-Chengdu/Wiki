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

The **ADD** directive in Dockerfiles functions similar to the **COPY** directive but with additional features.

```dockerfile
ADD <source> <destination>
```

The `<source>` specifies a path or **URL** to the file or directory on the local filesystem or a remote URL. The `<destination>` again specifies the path where the file or directory should be copied within the Docker image filesystem.

In the example below, we are going to use **ADD** to copy a file from the local filesystem:

```dockerfile
ADD index.html /var/www/html/index.html
```

In this example, Docker is going to copy the `index.html` file from the local filesystem (relative to the Docker build context) into `/var/www/html/index.html` within the Docker image.

In the example below, we are going to use **ADD** to copy a file from a remote URL:

```dockerfile
ADD http://example.com/test-data.csv /tmp/test-data.csv
```

Unlike **COPY**, the **ADD** directive allows specifying a URL (in this case `http://example.com/test-data.csv`) as the `<source>` parameter. Docker will download the file from the URL and copy it to the `/tmp/test-data.csv` within the Docker image.

The **ADD** directive not only copies files from the local filesystem or downloads them from URLs but also includes automatic extraction capabilities from a certain types of compressed archives. When `<source>` is a compressed archive file (e.g., `.tar`, `.tar.gz`, `.tgz`, `.bz2`, `.tbz2`, `.txz`, `.zip`), Docker will automatically extract its contents into `<destination>` within the Docker image filesystem.

For example:

```dockerfile
ADD myapp.tar.gz /opt/myapp/
```

In the example above, `myapp.tar.gz` is a compressed archive file, and Docker will automatically extract the contents of `myapp.tar.gz` into `/opt/myapp/` within the Docker image.

### [][35]Best Practices: COPY vs ADD in Dockerfiles

When writing Dockerfiles, choosing between the **COPY** and **ADD** directives is crucial for maintaining clarity, security and reliability in the image build process.

##### [][36]Clarity and Intent

**COPY** is straightforward and explicitly states that files or directories from the local filesystem are being copied into the Docker image. This clarity helps with understanding the Dockerfile's purpose and makes it easier to maintain over time.

On the other hand, **ADD** introduces additional functionalities such as downloading files from URLs and automatically extracting compressed archives. While these features can be convenient in certain scenarios , they can also obscure the original intent of simply copying files. This lack of transparency might lead to unexpected behaviors or security risks if not carefully managed.

##### [][37]Security and Predictability

Using **COPY** enhances security by avoiding potential risks associated with downloading files from arbitrary URLs. Docker images should be built using controlled, validated sources to prevent unintended or malicious content from being included. Separating the download of files from the build process and using `COPY` ensures that the Docker build environment remains secure and predictable.

##### [][38]Docker Philosophy Alignment

Docker encourages building lightweight, efficient, and predictable containerized applications. **COPY** aligns well with this philosophy by promoting simplicity and reducing the risk of unintended side effects during image builds.

### [][39]Using the WORKDIR, COPY, and ADD Directives in a Dockerfile

In this example we are going to deploy a custom HTML file to an Apache web server. We are going to use Ubuntu as our base image and install Apache on top of it. Then, we are going to copy the custom `index.html` file to the Docker image and download a Docker logo.

Create a new directory named `workdir-copy-add-example` using the `mkdir` command:

```
mkdir workdir-copy-add-example
```

---

Navigate to the newly created `workdir-copy-add-example` directory:

```
cd .\workdir-copy-add-example\
```

---

Within the `workdir-copy-add-example` directory, create a file named `index.html`. This file will be copied to the Docker image during build time. I am going to use VS Code, but feel free to use any editor you feel more comfortable with:

```
code index.html
```

---

Add the following content to the index.html file, save it, and close your editor:

```
<html>
    <body>
        <h1>
            Welcome to Docker!
        </h1>
        <img src="logo.png" height="350" width="500"/>
    </body>
</html>
```

This HTML code creates a basic web page that greets visitors with a large heading saying "Welcome to Docker!". Below the heading, it includes an image displayed using the `<img>` tag with the source attribute (`src="logo.png"`), indicating that it should fetch and display an image file named `logo.png`. The image is sized to be 350 pixels in height and 500 pixels in width (`height="350"` and `width="500"`).

---

Now, create a **Dockerfile** within this directory:

```
code Dockerfile
```

---

Add the following content to the `Dockerfile` file, save it, and exit:

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
WORKDIR /var/www/html/
COPY index.html .
ADD https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg ./logo.png
CMD ["ls"]
```

This Dockerfile begins by specifying `FROM ubuntu:latest`, indicating it will build upon the latest Ubuntu base image available. The subsequent `RUN apt-get update && apt-get upgrade` commands update and upgrade the package lists within the container. Following this, `apt-get install apache2 -y` installs the Apache web server using the package manager. The `WORKDIR /var/www/html/` directive sets the working directory to `/var/www/html/`, a common location for serving web content in Apache.

Within this directory, `COPY index.html .` copies a local `index.html` file from the host machine into the container. Additionally, `ADD https://upload.wikimedia.org/wikipedia/commons/4/4e/Docker_%28container_engine%29_logo.svg ./logo.png` retrieves an SVG image file from a URL and saves it locally as `logo.png` in the same directory.

Lastly, `CMD ["ls"]` specifies that upon container startup, the `ls` command will execute, displaying a listing of files and directories in `/var/www/html/`.

---

Now, build the Docker image with the tag of `workdir-copy-add`:

```shell
docker build -t workdir-copy-add .
```

You should see the following output:

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

---

Execute the `docker container run` command to start a new container from the Docker image we built previously:

```shell
docker run workdir-copy-add
```

As we can see, both `index.html` and `logo.png` are available in the `/var/www/html/` directory:

```text
index.html
logo.png
```

## [][40]The USER Directive

In Docker, by default, containers run with the root user, which has extensive privileges within the container environment. To mitigate security risks, Docker allows us to specify a non-root user using the **USER** directive in the **Dockerfile**. This directive sets the default user for the container, and all subsequent commands specified in the Dockerfile, such as **RUN**, **CMD**, and **ENTRYPOINT**, will be executed under this user's context.

Implementing the **USER** directive is considered a best practice in Docker security, aligning with the principle of least privilege. It ensures that containers operate with minimal privileges necessary for their functionality, thereby enhancing overall system security and reducing the attack surface.

The **USER** directive takes the following format:

```
USER <user>
```

In addition to the username, we can also specify the optional group name to run the Docker container:

```
USER <user>:<group>
```

You need to make sure that the `<user>` and `<group>` values are valid user and group names. Otherwise, the Docker daemon will throw an error while trying to run the container.

### [][41]Using USER Directive in the Dockerfile

In this example we are going to use the **USER** directive in the **Dockerfile** to set the default user. We will be installing the Apache web server and changing the user to **www-data**. Finally, we will execute the `whoami` command to verify the current user by printing the username.

Create a new directory named `user-example`

```shell
mkdir user-example
```

---

Navigate to the newly created `user-example directory`

```shell
cd .\user-example\
```

---

Within the `user-example` directory create a new **Dockerfile**

```
code Dockerfile
```

---

Add the following content to your **Dockerfile**, save it and close the editor:

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
USER www-data
CMD ["whoami"]
```

This Dockerfile starts with the latest Ubuntu base image and updates system packages before installing Apache web server (`apache2`). It enhances security by switching to the `www-data` user, commonly used for web servers, to minimize potential vulnerabilities. The `CMD ["whoami"]` directive ensures that when the container starts, it displays the current user (`www-data`), demonstrating a secure setup suitable for hosting web applications in a Docker environment.

---

Build the Docker image:

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

---

Now, execute the `docker container run` command to start a new container from the Docker image that we built in the previous step:

```shell
docker container run user
```

And the output should display `www-data` as the current user associated with the Docker container:

```
www-data
```

## [][42]The VOLUME Directive

In Docker, containers are designed to encapsulate applications and their dependencies in a portable and lightweight manner. However, by default, any data generated or modified within a Docker container's filesystem is ephemeral, meaning it exists only for the duration of the container's runtime. When a container is deleted or replaced, this data is lost, which poses challenges for applications that require persistent storage, such as databases or file storage systems.

To address this challenge, Docker introduced the concept of volumes. Volumes provide a way to persist data independently of the container lifecycle. They act as a bridge between the Docker container and the host machine, ensuring that data stored within volumes persists even when containers are stopped, removed, or replaced. This makes volumes essential for applications that need to maintain stateful information across container instances, such as storing databases, configuration files, or application logs.

When you define a volume in a Dockerfile using the `VOLUME` directive, Docker creates a managed directory within the container’s filesystem. This directory serves as the mount point for the volume. Crucially, Docker also establishes a corresponding directory on the host machine, where the actual data for the volume is stored. This mapping ensures that any changes made to files within the volume from within the container are immediately synchronized with the mapped directory on the host machine, and vice versa.

Volumes in Docker support various types, including named volumes and host-mounted volumes. Named volumes are created and managed by Docker, offering more control and flexibility over volume lifecycle and storage management. Host-mounted volumes, on the other hand, allow you to directly mount a directory from the host filesystem into the container, providing straightforward access to host resources.

The **VOLUME** directive generally takes a JSON array as a parameter:

```dockerfile
VOLUME ["path/to/volume"]
```

Or, we can specify a plain string with multiple paths:

```dockerfile
VOLUME /path/to/volume1 /path/to/volume2
```

We can use the `docker container inspect <container>` command to view the volumes available in a container. The output JSON of the docker container inspect command will print the volume information similar to the following:

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

### [][43]Using the VOLUME Directive in the Dockerfile

In this example, we are going to setup a Docker container to run the Apache web server. However, we don't want to lose the Apache log files in case of a Docker container failure. As a solution, we are going to persist the log files by mounting the Apache log path to the underlying Docker host.

Create a new directory named `volume-example`

```shell
mkdir volume-example
```

---

Navigate to the newly created `volume-example` directory

```shell
cd volume-example
```

---

Within the `volume-example` directory create a new **Dockerfile**

```shell
code Dockerfile
```

---

Add the following to the **Dockerfile**, save it, and exit

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade
RUN apt-get install apache2 -y
VOLUME ["/var/log/apache2"]
```

This Dockerfile starts by using the latest version of Ubuntu as the base image and ensures it is up to date by running `apt-get update` and `apt-get upgrade` to update all installed packages. It then installs Apache HTTP Server (`apache2`) using `apt-get install apache2 -y`. The `VOLUME ["/var/log/apache2"]` directive defines a Docker volume at `/var/log/apache2`, which is where Apache typically stores its log files.

---

Now, let's build the Docker image:

```shell
docker build -t volume .
```

And the output should be as follows:

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

---

Execute the `docker container run` command to start a new container from the previously built image. Note that you also need to use the `--interactive` and `--tty` flags to open an interactive bash session so that you can execute commands from the bash shell of the container. Also, you need to use the `--name` flag to define the container name as `volume-container`

```shell
docker container run --interactive --tty --name volume-container volume /bin/bash
```

Your bash shell will be opened as follows:

```
root@8aa0f5fb8a6d:/#
```

---

Navigate to the `/var/log/apache2` directory

```
root@8aa0f5fb8a6d:/# cd /var/log/apache2/
```

This will produce the following output:

```
root@8aa0f5fb8a6d:/var/log/apache2#
```

---

Now, list the available files in the directory

```
root@8aa0f5fb8a6d:/var/log/apache2# ls -l
```

The output should be as follows

```
total 0
-rw-r----- 1 root adm 0 Jun 20 13:42 access.log
-rw-r----- 1 root adm 0 Jun 20 13:42 error.log
-rw-r----- 1 root adm 0 Jun 20 13:42 other_vhosts_access.log
```

These are the log files created by Apache while running the process. The same files should be available once you check the host mount of this volume.

---

Exit the container to check the host filesystem

```
root@8aa0f5fb8a6d:/var/log/apache2# exit
```

---

Inspect the `volume-container` to view the mount information

```
docker container inspect volume-container
```

Under the `Mounts` key, you will be able to see the information relating to the mount

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

---

Inspect the volume with the `docker volume inspect <volume_name>` command. You can find the `<volume_name>` in the `Name` field of the previous output

```
docker volume inspect 50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8
```

You should get an output similar to the following

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

---

List the files available in the host file path. The host file path can be identified with the `Mountpoint` field of the previous output

```shell
ls -l /var/lib/docker/volumes/50d3a5abf34535fbd3a347cbd6c74acf87a7aa533494360e661c73bbdf34b3e8/_data
```

## [][44]The EXPOSE Directive

The **EXPOSE** directive in Docker serves to indicate to Docker that a container will be listening on specific ports during its runtime. This declaration is primarily informative and does not actually publish the ports to the host system or make them accessible from outside the container by default. Instead, it documents which ports are intended to be used for inter-container communication or network services within the Docker environment.

The **EXPOSE** directive supports both TCP and UDP protocols, allowing flexibility in how ports are exposed for various networking requirements. This directive is a precursor to the `-p` or `-P` options used during container runtime to actually map these exposed ports to ports on the host machine, enabling external access if required.

The **EXPOSE** directive has the following format:

```
EXPOSE <port>
```

## [][45]The HEALTHCHECK Directive

A health check is a crucial mechanism that designed to assess the operational health of containers. It provides a means to verify if applications running within Docker containers are functioning properly. Without a specified health check, Docker lacks the capability to autonomously determine the health status of a container. This becomes especially critical in production environments where reliability and uptime are paramount.

The **HEALTHCHECK** directive in Docker allows developers to define custom health checks, typically in the form of commands or scripts, that periodically inspect the container's state and report back on its health. This directive ensures proactive monitoring and helps Docker orchestration tools make informed decisions about container lifecycle management based on health status.

There can be only one **HEALTHCHECK** directive in a **Dockerfile**. If there is more than one **HEALTHCHECK** directive, only the last one will take effect.

For example, we can use the following directive to ensure that the container can receive traffic on the `http://localhost/` endpoint:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

The exit code at the end of the preceding command is used to specify the health status of a container, `0` and `1` are valid values for this field. `0` means a healthy container, `1` means an unhealthy container.

When using the **HEALTHCHECK** directive in Docker, it's possible to configure additional parameters beyond the basic command to tailor how health checks are performed:

-   **`--interval`**: Specifies the frequency at which health checks are executed, with a default interval of 30 seconds.
-   **`--timeout`**: Defines the maximum time allowed for a health check command to complete successfully. If no successful response is received within this duration, the health check is marked as failed. The default timeout is also set to 30 seconds.
-   **`--start-period`**: Specifies the initial delay before Docker starts executing the first health check. This parameter allows the container some time to initialize before health checks begin, with a default start period of 0 seconds.
-   **`--retries`**: Defines the number of consecutive failed health checks allowed before Docker considers the container as unhealthy. By default, Docker allows up to 3 retries.

In the following example, the default values of **HEALTHCHECK** are overridden, by providing custom values:

```dockerfile
HEALTHCHECK \
    --interval=1m \
    --timeout=2s \
    --start-period=2m \
    --retries=3 \
    CMD curl -f http://localhost/ || exit 1

```

### [][46]Using EXPOSE and HEALTHCHECK Directives in the Dockerfile

We are going to dockerize the Apache web server to access the Apache home page from the web browser. Additionally, we are going to configure health checks to determine the health status of the Apache web server.

Create a new directory named `expose-heathcheck-example`

```shell
mkdir expose-healthcheck-example
```

---

Navigate to the newly created `expose-healthcheck-example` directory

```
cd .\expose-healthcheck-example\
```

---

Create a **Dockerfile** and add the following content

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade

RUN apt-get install apache2 curl -y

HEALTHCHECK CMD curl -f http://localhost/ || exit 1

EXPOSE 80

ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]
```

This Dockerfile starts by pulling the latest Ubuntu base image and updating it. It then installs Apache web server and curl using `apt-get`. The `HEALTHCHECK` directive is set to run a health check command (`curl -f http://localhost/ || exit 1`), ensuring the container's health based on localhost connectivity. Port 80 is exposed to allow external access to Apache. Finally, the container is configured to run Apache in foreground mode using `ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]`, ensuring it stays active and responsive as the main process. This setup enables hosting a web server accessible via port 80 within the Docker environment.

---

Build the image

```shell
docker image build -t expose-healthcheck-example .
```

You should get an output similar to the following:

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

---

Execute the `docker container run` command to start a new container. You are going to use the `-p` flag to redirect port `80` of the host to port `8080` of the container. Additionally, you are going to use the `--name` flag to specify the container name as `expose-healthcheck-container`, and the `-d` flag to run the container in detach mode

```shell
docker container run -p 8080:80 --name expose-healthcheck-container -d expose-healthcheck-example
```

---

List the running containers with the `docker container list` command

```shell
docker container list
```

In the output, you will see that the `STATUS` of `expose-healthcheck-container` is healthy

```text
CONTAINER ID   IMAGE                        COMMAND                  CREATED              STATUS                        PORTS                            NAMES
3ff16b11275c   expose-healthcheck-example   "apache2ctl -D FOREG…"   About a minute ago   Up About a minute (healthy)   80/tcp, 0.0.0.0:8080->8080/tcp   expose-healthcheck-container
```

---

Now, you should be able to view the Apache home page. Navigate to the `http://127.0.0.1:8080` endpoint from your browser

[![Image description](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fynq7tnklqzp44zhosob2.png)][47]

## [][48]The ONBUILD Directive

The ONBUILD directive in Dockerfiles facilitates the creation of reusable base images intended for subsequent image builds. It allows developers to define instructions that will be triggered only when another Docker image uses the current image as its base. For instance, you could construct a Docker image containing all necessary prerequisites and configurations required to run an application.

By applying the ONBUILD directive within this "prerequisite" image, specific instructions can be deferred until the image is employed as a parent in another Dockerfile. These deferred instructions are not executed during the build process of the current Dockerfile but are instead inherited and executed when building the child image. This approach streamlines the process of setting up environments and ensures that common dependencies and configurations are consistently applied across multiple projects or applications derived from the base image.

The **ONBUILD** directive takes the following format

```dockerfile
ONBUILD <instruction>
```

As an example, imagine that we have the following **ONBUILD** instruction in the **Dockerfile** of a custom base image

```dockerfile
ONBUILD ENTRYPOINT ["echo", "Running an ONBUILD Directive"]
```

The `Running an ONBUILD Directive` value will not be printed if we create a Docker container from our custom base image, but will be printed if we use it as a base for another Docker image.

### [][49]Using the ONBUILD Directive in a Dockerfile

In this example, we are going to build a parent image with an Apache web server and use the **ONBUILD** directive to copy HTML files.

Create a new directory named `onbuild-parent-example`

```shell
mkdir onbuild-parent-example
```

---

Navigate to the newly created `onbuild-parent-example` directory:

```shell
cd .\onbuild-parent-example\
```

---

Create a new **Dockerfile** and add the following content

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade

RUN apt-get install apache2 -y

ONBUILD COPY *.html /var/www/html

EXPOSE 80

ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]
```

This Dockerfile begins by using the latest Ubuntu base image. It updates and upgrades the system packages, then installs the Apache web server. The ONBUILD directive specifies that any child images built from this Dockerfile will automatically copy all HTML files from the build context to the `/var/www/html` directory within the container. Port 80 is exposed to allow incoming traffic to the Apache server. Finally, the `ENTRYPOINT` command configures the container to run Apache in foreground mode, ensuring it remains active and responsive as the primary process. This setup enables the container to serve web content via Apache on port 80.

---

Now, build the Docker image:

```shell
docker image build -t onbuild-parent-example .
```

The output should be as follows:

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

---

Execute the `docker container run` command to start a new container from the Docker image built in the previous step:

```
docker container run -p 8080:80 --name onbuild-parent-container -d onbuild-parent-example
```

---

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
