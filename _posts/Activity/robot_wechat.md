
---
title: "微信机器人入门 Python & JavaScript"
date: 2019-06-16
categories:
  - Activity
  - Workshop
tags:
  - offline
  - wechat_robot
  - Python
  - JavaScript
toc: true

# Activity meta
description: 微信机器人入门工作坊
start: 2019-06-16 13:30:00
end: 2019-06-16 17:00:00
address: 成都市高新区世纪城路1029号天华社区乡愁故事馆
mentors:
  - Akagilnc
workers:
  - HillChen3
  - TechQuery
partners:
  - CDTH-StoryEvents
---



> 2019/6/16 13:30 ~ 17:30
>
> 成都市高新区世纪城路1029号天华社区乡愁故事馆

## 基本流程
跟着教练一起，实现一个简单的微信机器人
自动通过好友请求
发送信息给指定好友
发送信息给特定群体
拉人进群

## 你的收获

- [x] 收获志同道合的小伙伴，锻炼你的思维、动手能力和表达能力。
- [x] 了解并使用学会做自己的微信机器人。

## 课前准备

建议学员提前执行以下命令，安装好**开发环境**（[操作图解][2]）

## 安装包管理器
### Windows
```cmd
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

```

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

### Mac OS X
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
## 安装开发环境
### Windows

```powershell
choco install -y git python vscode nodejs googlechrome
```

### Mac OS X

```shell
brew install python nodejs
brew cask install sourcetree visual-studio-code google-chrome
```

### 虚拟环境

```shell

pip install pipenv -i https://pypi.doubanio.com/simple/
```
### Mac 或已经装过python2的 Windows环境需要修改命令为

```shell

pip3 install pipenv -i https://pypi.doubanio.com/simple/
```

### 依赖包

```shell

pipenv install itchat
```


## 参与须知

活动免费，**自带电脑**，和你的**激情**！

## 需要有编程基础么？

对小白友好，欢迎任何对 coding 感兴趣的小白们参与。


