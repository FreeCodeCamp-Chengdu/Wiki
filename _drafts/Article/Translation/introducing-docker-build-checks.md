---
title: "Introducing Docker Build Checks: Optimize Dockerfiles with Best Practices"
date: 2024-07-30T00:00:00.000Z
author: Colin Hemmings
authorURL: https://www.docker.com/author/colin-hemmings/
originalURL: https://www.docker.com/blog/introducing-docker-build-checks/
translator: ""
reviewer: ""
---

# 引入Docker构建检查:使用最佳实践优化Dockerfile


![](https://www.docker.com/wp-content/uploads/2023/12/Colin-Hemmings.webp)

[Colin Hemmings][1]  

今天,我们很高兴地宣布[Docker Desktop 4.33][3]发布了[Docker构建检查][2]。Docker构建检查帮助您的团队学习并遵循构建容器镜像的最佳实践。当您运行[Docker构建][4]时,您将获得构建中检测到的任何检查违规的警告列表。采取主动方法并及早解决构建警告和问题将为您节省下游的时间和麻烦。 

![Banner how to set up the weaviate vector database on docker](https://www.docker.com/wp-content/uploads/2023/09/banner_how-to-set-up-the-weaviate-vector-database-on-docker-1110x583.png "- Banner How To Set Up The Weaviate Vector Database On Docker")

## 我们为什么创建Docker构建检查?

在与开发人员的对话中,我们发现许多人在学习和遵循构建容器镜像的最佳实践方面存在困难。根据我们的[2024年应用程序开发状态报告][5],35%的Docker用户将创建和编辑[Dockerfiles][6]列为执行的三大任务之一。然而,55%的受访者表示创建Dockerfile是他们最常求助的任务。
开发人员通常没有时间通读[Docker构建文档][7],进行必要的更改以使其工作,然后继续前进。当您运行docker build时,Docker构建可能"有效",但编写不当的Dockerfile可能会引入质量问题,例如:

-   难以维护或更新
-   包含隐藏和意外的错误
-   性能不佳

在与Docker用户的对话中,我们听到他们希望优化Dockerfile以提高构建性能,不了解当前的最佳实践,并希望在构建过程中得到指导。
调查和修复构建问题浪费时间。我们创建了Docker构建检查,使开发人员能够从一开始就编写结构良好的Dockerfile,并从现有的最佳实践中学习。使用构建检查,您的团队可以减少在构建问题上花费的时间,而将更多时间用于创新和编码。

## 为什么应该使用Docker构建检查? 

您希望编写更好的Dockerfile并节省时间! 

我们收集了一套来自构建专家社区的最佳实践,并将其编码为Docker构建工具。您可以使用Docker构建检查来评估本地和CI工作流的所有阶段,包括多阶段构建和[Bake][8],并在[Docker Desktop Builds视图][9]中深入研究。您还可以选择跳过哪些规则。

您可以在CLI和Docker Desktop Builds视图中访问Docker构建检查。

### 不仅仅是linting:Docker构建检查功能强大且快速


Linting工具通常只评估文本文件集中的规则。作为Docker构建的一部分,Docker构建检查比linting更强大和准确。Docker构建检查评估整个构建,包括传递的参数和使用的基本镜像。这些检查足够快,可以在您编辑Dockerfile时实时运行。您可以快速评估构建,而无需等待完整的构建执行。 

### 检查您的本地构建

一个好的做法是在提交或共享更改之前评估新的或更新的Dockerfile。运行`docker build`现在会给您一个关于Dockerfile中问题和警告的概述。

![Build checks 433 f1](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f1-1110x545.png "- Build Checks 433 F1")

图1:一个带有四个检查警告的Docker构建。


要获取有关这些特定问题的更多信息,您可以指定调试标志到Docker CLI与`docker --debug build`。此信息包括警告类型、发生位置以及如何解决的链接。

![Build checks 433 f2](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f2-1110x948.png "- Build Checks 433 F2")

图2:检查警告的构建调试输出。

### 快速检查您的构建

运行这些检查时,构建是很好的,但等待完整的构建每次在您进行更改或修复问题时运行可能很耗时。为此,我们添加了`--check`标志作为构建命令的一部分。

```dockerfile
# check标志可以添加到构建命令的任何位置
docker build . --check
docker build --check .
docker build --build-arg VERSION=latest --platfrom linux/arm64 . --check
```

如图3所示,将标志附加到现有的构建命令将执行构建配置的完整评估,而无需执行完整的构建。这种更快的反馈通常在不到一秒内完成,使开发过程更加顺畅。
![Build checks 433 f3](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f3-1110x938.png "- Build Checks 433 F3")


图3:运行构建检查。

### 检查您的CI构建

默认情况下,运行带有警告的Docker构建不会导致构建失败(返回非零退出代码)。然而,为了捕获CI构建中的任何回归,添加以下声明以指示检查生成错误。

```dockerfile
# syntax=docker/dockerfile:1
# check=error=true

FROM alpine
CMD echo "Hello, world!"
```

### 检查CI中的多阶段构建

在构建过程中,仅执行指定的阶段/目标,包括其依赖项。我们建议在您的工件中添加一个阶段检查步骤,以完成对Dockerfile的完整评估。这与您在执行完整构建之前运行自动化测试的方式类似。

如果检测到任何警告,将返回非零退出代码,这将导致工作流失败,从而捕获任何问题。

```dockerfile
docker build --check .
```

### 检查Docker Build Cloud中的构建

当然,这也与[Docker Build Cloud][13]无缝协作,无论是本地还是通过CI。使用您的[e][14]xisting云构建器来评估您的构建。您的团队现在可以享受Docker Build Cloud性能和构建将符合最佳实践的保证。事实上,随着我们扩展检查,您应该看到来自Docker Build Cloud构建的更好性能。


![Build checks 433 f4](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f4-1110x294.png "- Build Checks 433 F4")

][15]

图4:在Docker Build Cloud中运行检查。

### 配置规则

您有灵活性来配置Build检查中的规则,使用skip参数。您还可以指定`skip=all`或`skip=none`来打开和关闭规则。以下是跳过`JSONArgsRecommended`和`StageNameCasing`规则的示例:

```dockerfile
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing

FROM alpine AS BASE_STAGE
CMD echo "Hello, world!"
```

### 深入了解Docker Desktop Builds视图

在Docker Desktop Builds视图中,您可以看到构建警告的输出。定位Dockerfile中警告的原因并快速了解如何解决它们现在很容易。

与构建错误一样,警告在Docker Desktop中检查构建时显示在Dockerfile中:


![Build checks 433 f5](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f5-1110x977.png "- Build Checks 433 F5")

图5:Docker Desktop Builds视图中的构建检查警告。

## 下一步是什么? 

### 更多检查

我们对新的Builds检查感到兴奋,以帮助您应用最佳实践到Dockfiles,但这只是开始。除了[当前的检查集][17],我们计划添加更多以提供更全面的构建评估。此外,我们期待包括自定义检查和策略,以用于您的Docker构建。

### IDE integration

尽早识别构建中的问题,解决问题更容易且成本更低。我们计划将Build检查与您喜欢的IDE集成,以便您可以实时反馈。

![Build checks 433 f6](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f6-1110x740.png "- Build Checks 433 F6")

图6:在VS Code中显示检查违规。

### GitHub Actions和Docker Desktop

您已经在Docker Desktop中看到Build检查警告,但更详细的见解即将到来。正如您可能已经听说,我们最近宣布了[Inspecting Docker Builds in GitHub Actions][19]的beta版本,我们计划在此新功能的基础上,包括对检查警告的支持。

## 现在开始

要开始使用Docker Build检查,升级到[Docker Desktop 4.33][20]今天并尝试使用现有的Dockerfiles。前往[我们的文档][21]了解更多关于Build检查的详细信息。 

## 了解更多

-   [进行身份验证和更新][22]以获取您订阅级别的最新Docker Desktop功能。
-   [Docker Desktop 4.33][24]还有什么新功能?Docker Debug和Docker构建检查的GA版本以及增强的配置完整性检查。
-   新到 Docker?[创建一个账户][25]。
-   订阅[Docker Newsletter][26]。

[build][27], [Docker Desktop][28], [dockerfile][29]


[1]: https://www.docker.com/author/colin-hemmings/ "Posts by Colin Hemmings"
[2]: https://docs.docker.com/build/checks/
[3]: https://www.docker.com/blog/docker-desktop-4-33/
[4]: https://docs.docker.com/build/
[5]: https://www.docker.com/blog/docker-2024-state-of-application-development-report/
[6]: https://docs.docker.com/guides/docker-concepts/building-images/writing-a-dockerfile/
[7]: https://docs.docker.com/build/
[8]: https://docs.docker.com/build/bake/
[9]: https://www.docker.com/blog/announcing-builds-view-in-docker-desktop-ga/
[10]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f1.png
[11]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f2.png
[12]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f3.png
[13]: https://www.docker.com/products/build-cloud/
[14]: https://docs.docker.com/build-cloud/setup/
[15]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f4.png
[16]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f5.png
[17]: https://docs.docker.com/reference/build-checks
[18]: https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f6.png
[19]: https://www.docker.com/blog/new-beta-feature-deep-dive-into-github-actions-docker-builds-with-docker-desktop/
[20]: https://docs.docker.com/desktop/release-notes/
[21]: https://docs.docker.com/build/checks/
[22]: https://www.docker.com/pricing/
[23]: https://www.docker.com/pricing/
[24]: https://www.docker.com/blog/docker-desktop-4-33/
[25]: https://hub.docker.com/signup?_gl=1*452i3u*_ga*MjEzNzc3Njk5MC4xNjgzNjY3NDkw*_ga_XJWPQMJYHQ*MTcwODcxNjA4Ni4zNjguMS4xNzA4NzE2MzE2LjUzLjAuMA..
[26]: https://www.docker.com/newsletter-subscription/
[27]: https://www.docker.com/blog/tag/build/
[28]: https://www.docker.com/blog/tag/docker-desktop/
[29]: https://www.docker.com/blog/tag/dockerfile/
[30]: https://www.docker.com/blog/docker-scout-health-scores-security-grading-for-container-images/
[31]: https://www.docker.com/blog/docker-scout-health-scores-security-grading-for-container-images/
[32]: https://www.docker.com/author/tazin-progga/ "Posts by Tazin Progga"
[33]: https://www.docker.com/blog/docker-desktop-4-33/
[34]: https://www.docker.com/blog/docker-desktop-4-33/
[35]: https://www.docker.com/author/deanna-sparks/ "Posts by Deanna Sparks"
[36]: https://www.docker.com/blog/how-to-create-dockerfiles-with-genai/
[37]: https://www.docker.com/blog/how-to-create-dockerfiles-with-genai/
[38]: https://www.docker.com/author/docker-labs/ "Posts by Docker Labs"
[39]: https://twitter.com/intent/tweet?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fintroducing-docker-build-checks%2F
[40]: https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fintroducing-docker-build-checks%2F
[41]: https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.docker.com%2Fblog%2Fintroducing-docker-build-checks%2F
[42]: https://www.docker.com/blog/tag/build/
[43]: https://www.docker.com/blog/tag/docker-desktop/
[44]: https://www.docker.com/blog/tag/dockerfile/
[45]: https://www.docker.com/blog/category/community-content/
[46]: https://www.docker.com/blog/category/company/
[47]: https://www.docker.com/blog/category/engineering/
[48]: https://www.docker.com/blog/category/products/