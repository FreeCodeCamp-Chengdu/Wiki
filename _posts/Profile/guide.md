---
title: 社区参与指南
date: 2019-03-30 03:09:32
categories:
  - Profile
tags:
  - community
  - guide
authors:
  - TechQuery

toc: true
slidehtml: true
---

> 《公益技术社区的术与道》

---

## 术

---

### 线下技术社区的优势

主要在于**线下技术活动**

---

### 线下技术社区的成就

主要以**一次次活动**来体现

---

### 个人、公司对社区的贡献

也主要以**一次次活动**为单位

<!-- more -->

---

### 一个活动是怎样的？

首先体现在它的**宣传文案**上

---

### 一篇宣传文案

可以是一篇**博客文章**

```yaml
title: # 文章主题（翻译成英文后设为本文件的文件名）
date: # 开篇日期
updated: # 修订日期
categories:
  # 分类名应为一个首字母大写的英文单词
  # 每级分类对应一级目录，写在本文件的文件名前面
tags:
  # 与内容相关的更多标签
toc: true
authors:
  # 原作者们的 GitHub 账号
```

---

### 一篇博客文章

又可以附带**自定义元数据**

```yaml
title: # 活动主题（翻译成英文后设为本文件的文件名）
date: # 发起日期
categories:
  - Activity
  # 二级分类：Salon、Workshop 或 Conference
tags:
  # online 或 offline
  # 与内容相关的更多标签
toc: true

# Activity meta
description: # 活动简介
start: # 活动开始时间
end: # 活动结束时间
address: # 线下活动地址（市州、区县、路街、楼栋）
links:
  报名: # https://jinshuju.net/f/xxxxxx
mentors:
  # 讲师、教练们的 GitHub 账号
workers:
  # 组织者、志愿者们的 GitHub 账号
```

---

### 一次活动的反馈

可直接在活动博文下方进行

- 建议：社会化评论框
- 捐助：支付二维码

---

### 一次活动的总结

文字可以就近写在同一篇文章中

现场照片、演示文稿也可放在同级同名文件夹，并用附加元数据关联

```yaml
photos:
  -  # 第一张为活动封面图
  -  # 第二张开始为活动结束后上传的现场照片
files:
  -  # 活动结束后上传的 PDF、PPT 等非纯文本文件
```

---

### 社区贡献统计

基于现代**静态网站生成器**的二次开发能力实现

https://fcc-cd.tk/activity/

https://fcc-cd.tk/community/

---

## 道

---

### 团队的运作模式

负责人**自上而下**的领导

---

### 社区的运作模式

社群**自下而上**的生长

---

### 开源软件运动

**现代社群**的最成功探索

---

### 类 GitHub 平台

**技术型社区**的最佳协作工具

---

#### 版本控制

记录成长点滴，杜绝胡乱删改

---

#### 开源

政务透明、财务公开

---

#### Issue, Milestone & Project

任务安排、缺陷管理、进度跟踪

---

#### Fork & Pull request

人人皆可发起、参与活动

---

### 公益技术社区

就成了一个**开源文档项目**

---

开源运动的一切**最佳实践**、**基础设施**

皆可为**线下社群**所用

---

#### 社区技术工具的开发

不但本身可以**开源项目**的模式运作

也可同时做成**工作坊**活动

https://fcc-cd.tk/activity/workshop/hexo-web-app/

https://fcc-cd.tk/activity/workshop/nodejs-web-crawler/

---

Enjoy your Community!