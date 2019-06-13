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

description: 微信机器人入门工作坊
start: 2019-06-16 13:30:00
end: 2019-06-16 17:00:00
address: 成都市高新区世纪城路1029号天华社区乡愁故事馆
mentors:
  - Akagilnc
  - TechQuery
workers:
  - HillChen3
  - demongodYY
partners:
  - CDTH-StoryEvents
---

> 2019/6/16 13:30 ~ 17:30
>
> 成都市高新区世纪城路 1029 号天华社区乡愁故事馆

## 基本流程

跟着教练一起，实现一个简单的微信机器人

- [x] 自动通过好友请求
- [x] 发送信息给指定好友
- [x] 发送信息给特定群体
- [x] 拉人进群

## 你的收获

- [x] 收获志同道合的小伙伴，锻炼你的思维、动手能力和表达能力
- [x] 了解并使用学会做自己的微信机器人

## 参与须知

- 活动免费，**自带电脑**，和你的**激情**！
- **对小白友好**，欢迎任何对 coding 感兴趣的小白们参与!

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
如果提示权限问题，请在前面加上`sudo`

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
npm init es-pack ~/Desktop/WeChat-robot
cd ~/Desktop/WeChat-robot

npm uninstall amd-bundle

npm install -D \
    @babel/cli \
    @babel/core \
    @babel/plugin-transform-runtime

npm install wechaty @babel/runtime
```

[1]: ../hexo-web-app/#%E3%80%90%E9%99%84-0%E3%80%91Windows-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%9B%BE%E8%A7%A3
