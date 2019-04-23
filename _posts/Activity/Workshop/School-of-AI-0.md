---
title: "AI 学院工作坊 #0"
date: 2019-04-17 17:54:30
categories:
  - Activity
  - Workshop
tags:
  - offline
  - cooperation
  - AI

description: 动手打造你的神经网络
start: 2019-4-20 14:00:00
end: 2019-4-20 17:00:00
address: 成都市高新区世纪城路1029号天华社区乡愁故事馆
mentors:
  - hchen13
workers:
  - Akagilnc
  - TechQuery
sponsors:
  - CDTH-StoryEvents
---

> [AI 学院][1]与 **FCC 成都社区**联合出品

一次开放的 AI **深度学习**体验，
一场解码 AI 更多可能性的讨论。

—— 在这里，数位来自成都的热爱 AI 科技的 Deans 期待与你相遇！

## 参会须知

- 目标群体：软件开发者、会一点点**程序设计**的人、AI 从业者
- 免费！自带电脑！！！

{% asset_img poster.jpg %}

<!-- more -->

## 主要流程

1.  AI 学院简介
2.  AI 简介
3.  Python 环境和深度学习开发依赖的安装
4.  实现一个简单的**深度神经网络**

## 课前准备

建议学员提前执行以下命令，安装好**开发环境**（[操作图解][2]）

### Windows

```powershell
choco install -y git tortoisegit python vscode miniconda3
```

### Mac OS X

```shell
brew install python
brew cask install sourcetree visual-studio-code miniconda 
```

### Conda 虚拟环境 与 python 依赖包

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
conda create -n deeplearning python=3.6.5 -y
conda-env list
conda activate deeplearning

# 中国教育网用户切换镜像
# pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install matplotlib numpy pandas tensorflow==1.11.0 keras -i https://pypi.doubanio.com/simple/
```

## 特别鸣谢

[![天华社区乡愁故事馆](../../../sponsor/cdth-storyevents/CDTH-SE-logo.png)](../../../sponsor/cdth-storyevents/ "点击查看详情")

天华社区乡愁故事馆是由桂溪街道天华社区居委会主办，桂溪街道办事处支持，爱有戏社会工作服务中心运营的公益性社区营造项目。

[1]: https://www.theschool.ai/
[2]: ../hexo-web-app/#%E3%80%90%E9%99%84-0%E3%80%91Windows-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%9B%BE%E8%A7%A3


## 活动回顾


# 我们的主讲Ethen老师
{% asset_img soai_5.jpg %}


# 专心思考的参与者们
{% asset_img soai_3.jpg %}


# 隐藏的角落
{% asset_img soai_4.jpg %}


# 下次还要来玩哦
{% asset_img soai_1.jpg %}
