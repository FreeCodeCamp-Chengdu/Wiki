---
layout: activity_launch
title: 发起活动
---

## 多媒体文件上传

图片、视频、PDF、PPT 等**二进制文件**上传必须在[本地编辑][1]！

## Front-matter 规范

（点右下角按钮复制模板）

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
partners:
  # 合办方、场地方、赞助方

# Attachment meta
photos:
  -  # 第一张为活动封面图
  -  # 第二张开始为活动结束后上传的现场照片
files:
  -  # 活动结束后上传的 PDF、PPT 等非纯文本文件
```

[1]: https://github.com/FreeCodeCamp-Chengdu/Wiki#readme
