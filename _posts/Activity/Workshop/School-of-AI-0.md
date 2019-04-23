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


### 我们的主讲Ethen老师
Ethen用浅显易懂的语言和实例，像大家介绍了什么叫做 AI，机器学习，深度学习，神经网络
真的是连完全没有经验的小白都能理解呢，给Ethen老师打call
{% asset_img soai_5.jpg %}


### 专心思考的参与者们
全程无尿点，后续的提问环节也是有大量的小伙伴提出了很有质量的问题，看来大家是真的理解了呢，
唯一的遗憾可能就是因为网络不太好没有能跟着实现一遍了吧。后续补上～
{% asset_img soai_3.jpg %}


### 隐藏的角落
这位美女提出了相当棒的问题，作为一个不会编程的人，却从数学的角度给出了很棒的建议，
建议用距离正确答案的比例差来取代绝对值差，就连Ethan老师也是赞不绝口呢。
{% asset_img soai_4.jpg %}


### 下次还要来玩哦
希望这次大家玩的开心，下次还要来哦
{% asset_img soai_1.jpg %}

