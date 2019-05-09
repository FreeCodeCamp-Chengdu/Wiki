---
title: "AI 学院工作坊 #1"
date: 2019-05-09 13:45:30
categories:
  - Activity
  - Workshop
tags:
  - offline
  - cooperation
  - AI
authors:
  - Akagilnc
toc: true

description: 这次我们会通过AI的图像识别技术来尝试对我们的复仇者联盟的英雄们做识别
start: 2019-5-11 14:00:00
end: 2019-5-11 17:00:00
address: 武侯区天益南巷创客家众创空间
links:
  报名: http://chengdu-ai.mikecrm.com/zoIQvRq
mentors:
  - Akagilnc
workers:
  - Akagilnc
  - TechQuery
  - HillChen3
sponsors:
  - CDCKplus
---

> [AI 学院][1]与 **FCC 成都社区**联合出品

大家复联都看了么？每个英雄都认识的清清楚楚了吧？

可你知道，AI也能认识英雄么?

今天，我們要教 AI 做**图像识别**，

介绍我们的超级英雄给 AI　认识吧。

—— 在这里，数位来自成都的　AI　科技爱好者期待与你相遇！

## 参会须知

- 目标群体：软件开发者、会一点点**程序设计**的人、AI 从业者
- 免费！**自带电脑！！！提前做好课前准备(见下文)！！！**

{% asset_img poster.png %}

<!-- more -->

## 主要流程

1.  Python 环境和深度学习开发依赖的安装
2.  图像处理相关概念介绍
3.  图像识别的python实现
4.  实现**漫威英雄识别**

## 课前准备

请提前执行以下命令，安装好**开发环境**（[操作图解][2]）

### Windows

```powershell
choco install -y python miniconda3
```

### Mac OS X

```shell
brew install python
brew cask install miniconda
```

### Conda 虚拟环境 与 python 依赖包

```shell
conda create -n deeplearning python=3.6.5 matplotlib -y
conda-env list
conda activate deeplearning

pip install scikit-image opencv-contrib-python numpy tensorflow==1.11.0 keras -i https://pypi.doubanio.com/simple/
```



