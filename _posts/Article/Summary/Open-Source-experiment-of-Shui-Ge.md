---
title: 水歌的开源实验
date: 2020-11-26 23:26:26
authors:
  - TechQuery
categories:
  - Article
  - Summary
tags:
toc: true
original:
thumbnail:
---

> 「[2020 中国开源年会][1]」官网开发及[成都站][2]总结

## 官网开发 —— 骑马造车

### 展厅需求与日程难题

COSCon'20 筹备伊始，组委会分派给我的首要任务是**合作伙伴云展厅**的开发，以便在疫情尚未平息之时的线上大会给各大合作伙伴一个集中宣传的窗口。

但云展厅的设计图并非一般线下大会在官网贴一屏 **logo** 那么简单，还有**文字简介**、**宣传视频**、**加群二维码**和**入选的议题与讲师**等等。面对这样的需求，必然难以用近年技术站流行的 [MarkDown 网站生成器][3]来规范而高效地实现，因为去年用 YAML 文件维护前两年的 logo 墙已经很繁琐了。

提到合作伙伴关联的议题、讲师，又让我想起参加 COSCon 2018、2019 时手拿一张*尺寸巨大、文字特小的日程表*找会场时的痛苦，与参加[台湾 COSCUP 2019][4] 手持[会议 App][5] 畅游时的体验形成鲜明对比。

<figure>
    <img src="https://opass.app/img/logo.png">
    <figcaption>OPass logo - 台湾开源会议通用 App</figcaption>
</figure>

与我一同经历上述 3 场大会的庄表伟老师（开源社 2020 年度理事长），在 COSCon'19 现场曾和我不约而同地提出“要做个**开源会议系统**”，大会结束后便以产品配开发的搭档[火速开干][6]，但又因团队缺乏精力而停摆……

<!-- more -->

### 开源软件一箭三雕

但今时不同往日，*疫情闭关*时折腾的 [Headless CMS][7] 正好派上用场。得益于 [Strapi][8] 的**可视化数据模型设计器**，分分钟完成以下**数据库表结构**设计：

|   集合类型   |     用途     |           备注           |
| :----------: | :----------: | :----------------------: |
|     User     | 外部注册用户 |        非后台用户        |
|   Activity   |     活动     |     各类小中大型活动     |
| Organization |   外部组织   | 公司、开源社区、公益组织 |
| Partnership  |   合作关系   |      活动合作、赞助      |
|   Category   |   内容分类   |  可关联于多种内容型数据  |
|   Program    |   活动环节   |    演讲、工作坊、展位    |
|   Project    |     专案     |      开源、公益项目      |
|    Place     |     场地     |        室内、户外        |

<figure>
{% asset_img Strapi-admin-0.png %}
    <figcaption>Strapi 后台管理界面（数据模型）</figcaption>
</figure>

不仅如此，运营人员用的**内容编辑界面**也自动生成了~

<figure>
{% asset_img Strapi-admin-1.png %}
    <figcaption>Strapi 后台管理界面（内容编辑）</figcaption>
</figure>

再装上官方的 **RESTful API 文档生成器**，前端工程师最喜欢的 [Swagger UI][9] 瞬间呈现！

<figure>
{% asset_img Strapi-Swagger.png %}
    <figcaption>Strapi 接口的 Swagger 界面</figcaption>
</figure>

相比之下，反而在云主机上**安装 Python PIP、Docker Compose** 和**配置 HTTPS 证书**花了更多的时间。

于是乎，我只需带着志愿者专注于 [Web 前端项目][10]就好了，而且是一次性搞定**云展厅**、**日程表**和**展位申请表**页面~

### 反思、反刍、反哺

其实办一场大会涉及的信息不止以上内容，还有会前**志愿者任务的发布、提交与评价**，会中的**议题评论**，及会后的**差旅报销**、**物资登记**等等。同时，一个 NGO 的**志愿者积分**、**财务账目**等数据也要长期统计，以便向社会公开。

但大多数组织的**信息化**还非常初级，数据散落在多个商业工具的免费版里：

- 表单系统：调查问卷、讲师征集、志愿者报名
- 表格工具：员工名册、活动日程、物资账目
- 活动平台：内容介绍、合作伙伴、观众报名

而且国内各商业平台几乎没有可用的 API，最终形成一堆**数据孤岛**，让组织成员在**数据同步与查找**的过程中*疲于奔命*……

其实，将上述工作稍加梳理，也不需要增加太多数据模型：

|   集合类型   |   用途   |          备注          |
| :----------: | :------: | :--------------------: |
|     Task     |   任务   |                        |
| Contribution | 任务贡献 |                        |
|  Evaluation  |   评价   |  贡献、活动环节的评分  |
|   Comment    | 内容评论 | 可关联于多种内容型数据 |
|   Account    |   账目   |    赞助、差旅、采购    |
|   Property   |   物资   |      设备、纪念品      |

又因为 Strapi 的**数据模型定义**自动保存在项目目录的 JSON 文件中，可以直接与代码一起提交到 Git 仓库，加之 GitHub 推出了**模板仓库**功能，包含上述通用数据模型的 Strapi 项目，就变成了一个综合的**组织管理系统**，可以发布为一个**开源后端脚手架** ——

> https://github.com/kaiyuanshe/OrgServer

再把替换 Strapi 默认**富文本编辑器**的组件独立成库，又多了一个方便的第三方插件 ——

> https://github.com/TechQuery/strapi-plugin-ckeditor

Web 前端用 MobX 对接 Strapi 规范接口的通用逻辑也可以抽象出来，做成一个**支持响应式渲染的 SDK** ——

> https://github.com/EasyWebApp/MobX-Strapi

以后再用 Strapi 开发项目的效率自然高上加高咯~

![](https://raw.githubusercontent.com/sindresorhus/awesome/main/media/mentioned-badge.svg)

最后，[Strapi 官方生态推荐列表][11]也很快[收录了上述 3 个项目][12]。

[1]: http://coscon.kaiyuanshe.cn/
[2]: /activity/conference/coscon-2020-chengdu/
[3]: /activity/workshop/hexo-web-app/
[4]: https://coscup.org/2019/
[5]: https://opass.app/
[6]: https://github.com/kaiyuanshe/ActivityHub
[7]: https://tech-query.me/development/headless-cms-strapi/
[8]: https://strapi.io/
[9]: https://swagger.io/tools/swagger-ui/
[10]: https://github.com/kaiyuanshe/PWA
[11]: https://github.com/strapi/awesome-strapi
[12]: https://github.com/strapi/awesome-strapi/pull/22/files
