---
title: GitHub 在线 IDE 初体验
authors:
  - Kitety
categories:
  - Article
  - Engineering
tags:
  - GitHub
  - Codespaces
original: "https://kitety.github.io/posts/73c1c21d.html"
thumbnail: "https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-16/1602840943531-image.png"
toc: true
date: 2020-10-16 17:31:27
---

之前就有听说 GitHub 将推出在线 IDE，一搜索发现很多结果。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201014212657.png">
</figure>

现在 GitHub 的在线 IDE 发布一段时间了，官方命名为：[GitHub Codespaces](https://GitHub.com/features/codespaces)（点击可以申请），今天我们就来体验一下。

<!-- more -->

## 基础体验

在这里，我就拿本人博客的[仓库](https://github.com/kitety/blog)来简单跑一下 GitHub Codespaces。

### 创建 IDE

在 Clone 的按钮选择“Open With Codespaces”

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602682721438-image.png">
     <figcaption>创建流程</figcaption>
</figure>

进入之后会列出已有的 IDE 列表，没有的话点击下面的新建就是了。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602682784043-image.png">
     <figcaption>创建流程</figcaption>
</figure>

### 进入 IDE

开始进入是在初始化，然后就是同步一些配置。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602682883142-image.png">
</figure>

我们会发现在 VS Code 的配置和插件扩展都会被同步过来（当然，前提是你本地的 VSC 和自己的 GitHub 绑定起来，并且同步配置）。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602683153389-image.png">
</figure>

同时，IDE 可以自动识别 `package.json` 安装依赖，进来就自动安装好了。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602682955735-image.png">
</figure>

点击查看日志还可以查看初始化的日志。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602683024095-image.png">
</figure>

### 基本使用

#### 预装基础环境

简单的几行代码，我们可以发现 IDE 已经预装了 `node`、`docker`、`npm`、`git`、`python` 等等基础开发环境。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602690279501-image.png">
     <figcaption>预安装基础环境</figcaption>
</figure>

#### 启动项目

首先全局安装 `Hexo`，再启动项目 `yarn d`。

```bash
yarn add global hexo
```

因为 GitHub 的环境在外面，因此安装速度还是很快的，纵享丝滑。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602690553256-image.png">
</figure>

##### 外部端口的打开

如果我们的页面需要启动本地端口，IDE 也会提示出来有外部端口。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-14/1602690647172-image.png">
     <figcaption>端口打开</figcaption>
</figure>

我们也可以在“Remote Explorer”看到全部的端口映射情况

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-15/1602694549227-image.png">
    <figcaption>Remote Explorer</figcaption>
</figure>

我们点击在浏览器打开，然后就可以看到页面了。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-15/1602694402020-image.png">
</figure>

当我们修改之后，在侧边栏至直接提交就是了，简单快捷。也不用任何的设置。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-15/1602694872590-image.png">
</figure>

## 进阶玩法

我们的项目不仅仅是前端项目，也有可能是后端 Server，这里我就用一个[后端 Server](https://GitHub.com/kitety/likeReddit) 来简单演示一下。

### 安装依赖跑起来

- 全局安装 `nodemon`
- 进入 `server` 目录安装依赖

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015152911.png">
</figure>

很明显这里报错了，原因是我们的 Server 需要连接 PostgreSQL，而我们没有安装。

### 不恰当的安装方法

找到一篇[教程](https://www.runoob.com/postgresql/linux-install-postgresql.html)，照着代码跑起来。

```bash
sudo apt-get update
sudo apt-get install postgresql postgresql-client
# 创建一个数据库超级用户 postgres
sudo -i -u postgres
```

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153252.png">
</figure>

最后，我们会卡在这里，因为我们不知道 Codespace 的密码，因此安装失败。

### Docker 出马

我们可以观察到 Codespace 已经为我们安装了 Docker，而且在现在相当流行容器化部署，上面的那种安装方式也不够优雅。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153359.png">
</figure>

因此运行命令，安装 PostgreSQL

```bash
docker run -p 5432:5432 -v /home/docker/postgresql/data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=123456 -e TZ=PRC -d --name=some-postgres postgres
//-p端口映射
//-v将数据存到宿主服务器
//-e POSTGRES_PASSWORD 密码（默认用户名postgres）
//-e TZ=PRC时区，中国
//-d后台运行
//--name容器名称
```

运行之后，找不到镜像会自动去拉取镜像

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153646.png">
</figure>

查看下状态

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153710.png">
</figure>

现在重启 server，发现已经可以连接上了。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153818.png">
</figure>

### 端口

我们可以在“Forwarded Ports”增加端口转发

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015153909.png">
</figure>

简单演示一下 Get 请求，并且是即时的，修改之后可以通过域名来访问。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015154129.png">
</figure>

顺便说一句，如果我们在代码中写好 URL 地址，就可以直接用鼠标在命令行打开对应端口，网址也会被进行替换。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015154329.png">
</figure>

### 注意

但是也需要注意，如果我们用 Postman 去请求就无法正常请求结果。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015154656.png">
</figure>

如果我们访问 `/graphql`，请求就会提示“Server cannot be reached”和一些跨域错误。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/img/20201015154819.png">
</figure>

## 总结

### 优点

俗话说，工欲善其事必先利其器。

编程更重要的是一种思想，而编码更重要的是去表达思想。

如果我们将配置环境，机器选择的的步骤省下来，让自己更加专注于思想表达，专注于编码的话，这样会让我们事半功倍。而现在 GitHub IDE 就可以看成 VSC 的网页版。如果你将 VSC 的配置同步到 GitHub 账户的话，你打开在线 IDE 的时候就会直接同步配置，你会很快上手。

除此之外，GitHub 的里面预装各种环境，让你不再苦恼于环境安装，而且所处的网络环境也很棒，各种库、配置下载起来也是很快，我想这对我们的帮助也是很大的。比如再也不用纠结 `node-sass` 下载不下等尴尬场景。

**遇到紧急的事情，一个浏览器就可以让你专注开发，这难道不香吗？**

### 不足

虽然 GitHub 在线 IDE 有很多优点，但是还是有一些不足，肯定不能和 VSC 真机比拟。比如一些接口 `/graphql`，就没有本地真机开发的那么爽。除此之外，真机的 VSC 就有很多辅助扩展。比如 `PicGo` 来实现图片上传到 GitHub 做图床，在浏览器 IDE 里面经过测试是跑不通的。

我测试的时候限制了同时启用两个 IDE，不然会提示你让你处理。

<figure>
    <img src="https://cdn.jsdelivr.net/gh/kitety/blog_img/2020-10-16/1602842378177-image.png">
</figure>

因为每个人所处的网络环境不同，不用高级姿势访问有可能会出现链接断开的情况，这倒是有点硬伤。🤣

## 结语

之前听过“阿里 云电脑”，加上现在 5G 的逐渐普及，说不定未来大家需要的只是一个显示器，可以完成学习、工作和娱乐，配置全部都在远端。听起来天方夜谭，说不定在未来就会实现。

一个新兴事物的出来，肯定引起人们的好奇和质疑。仔细想想 GitHub 被微软收购之后，先后推出了个人无限私有仓库，免费使用 GitHub Actions，再有 GitHub Codespaces。而微软也先后推出 TS、VSC 等市场举足轻重的开源项目。我所看到是开源界的发展和繁荣，也希望未来越来越好。

撒花！
