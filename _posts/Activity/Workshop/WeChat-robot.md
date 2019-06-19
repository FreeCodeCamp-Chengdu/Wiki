---
title: 微信机器人入门实战 Python & JavaScript
date: 2019-06-16 10:57:00
categories:
  - Activity
  - Workshop
tags:
  - offline
  - WeChat
  - robot
  - Python
  - JavaScript
toc: true
thumbnail: mentees1.jpg

description: 微信机器人入门工作坊
start: 2019-06-16 13:30:00
end: 2019-06-16 17:00:00
address: 成都市高新区天益南巷18号创客大使馆
links:
  报名: https://jinshuju.net/f/Yxtuy1
mentors:
  - Akagilnc
  - TechQuery
workers:
  - HillChen3
  - demongodYY
partners:
  - CD-CKplus
---

> 2019 年 6 月 16 日 13:30 ~ 17:30
>
> 成都市高新区天益南巷 18 号[创客大使馆](/partner/cd-ckplus/)

## 基本流程

跟着教练一起，实现一个简单的微信机器人

- [x] 自动通过好友请求
- [x] 发送信息给指定好友
- [x] 发送信息给特定群体
- [x] 拉人进群

## 你的收获

- [x] 收获志同道合的小伙伴，锻炼你的思维、动手能力和表达能力
- [x] 学会用 Python 或 JavaScript 做自己的微信机器人

## 参与须知

- **活动免费**，**自带电脑**和你的**激情**！
- **对小白友好**，欢迎任何对 coding 感兴趣的小伙伴参与!

<!-- more -->

## 课前准备

建议学员提前执行以下命令，安装好**开发环境**（[操作图解][1]）

### 安装包管理器

#### Windows

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force;
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

#### Mac OS X

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果提示权限问题，请在前面加上 `sudo`

### 安装开发环境

#### Windows

```powershell
choco install -y git python nodejs vscode googlechrome

pip install pipenv -i https://pypi.doubanio.com/simple/
# 或
pip3 install pipenv -i https://pypi.doubanio.com/simple/
```

#### Mac OS X

```shell
brew install python nodejs
brew cask install sourcetree visual-studio-code google-chrome

pip install pipenv -i https://pypi.doubanio.com/simple/
# 或
pip3 install pipenv -i https://pypi.doubanio.com/simple/
```

### 项目初始化

#### Python

```shell
mkdir ~/Desktop/WeChat-robot
cd ~/Desktop/WeChat-robot

pipenv install itchat
```

#### Node.JS

```shell
mkdir ~/Desktop/WeChat-robot
cd ~/Desktop/WeChat-robot

npm init -y
npm install wechaty wechaty-puppet-puppeteer puppeteer
```

## 参考资料

1. [WeChaty 官方文档](https://docs.chatie.io/v/zh/)
2. [WeChaty 规范示例](https://github.com/wechaty/wechaty-getting-started/blob/master/README-zh.md)

[1]: ../hexo-web-app/#%E3%80%90%E9%99%84-0%E3%80%91Windows-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%9B%BE%E8%A7%A3

## 活动回顾

### 即将开始

还没有开始呢，**水歌**大佬就已经迫不及待的指点迷津了

{% asset_img water_brother.jpeg %}

#### 破冰时间

大家各自介绍了自己，希望你们在活动中交到好朋友哦
{% asset_img ice_break.jpeg %}

### 导师风采

学习**真**是一件**开心**的事情！
{% asset_img mentors.jpeg %}

#### 活动现场

我们先从**登录**和**自动回复**消息入手
{% asset_img mentees1.jpeg %}

然后给**特定好友**发消息，再给**群组好友**发消息
{% asset_img mentees2.jpeg %}

最后学会了如何自动**通过好友请求**，并且把这个好友拉入**指定的群**
{% asset_img mentees3.jpeg %}

#### 完结撒花

大家都动手敲了一个下午，收获满满吧。下次还要来哦
{% asset_img the_end.jpeg %}

### 反馈教训

事后我们做了反馈调查，小伙伴们提出了一些超棒的建议

- 对于`pipenv`和虚拟环境的认识不够，可以做一些说明
- 现场环境网络比较差
- 椅子质量有点差（摔）
- 活动内容本身
  - 节奏希望再紧凑一些
  - 除了按照导师教的做之外，能有一些小挑战，即学习，模仿，思考题的模式
  - 现场采用结对编程的模式，让大家更有互动感，也可以规避一些环境安装的问题
  - 活动完后可以给学员一个发表感受的机会

我们非常感谢这些很棒的建议，并且会应用到下一次工作坊，敬请期待～
