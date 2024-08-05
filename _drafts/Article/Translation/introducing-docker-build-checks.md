---
title: "Introducing Docker Build Checks: Optimize Dockerfiles with Best Practices"
date: 2024-07-30T00:00:00.000Z
author: Colin Hemmings
authorURL: https://www.docker.com/author/colin-hemmings/
originalURL: https://www.docker.com/blog/introducing-docker-build-checks/
translator: ""
reviewer: ""
---

# Introducing Docker Build Checks: Optimize Dockerfiles with Best Practices

<!-- more -->

![](https://www.docker.com/wp-content/uploads/2023/12/Colin-Hemmings.webp)

[Colin Hemmings][1]  

Today, we’re excited to announce the release of [Docker Build checks][2] with [Docker Desktop 4.33][3]. Docker Build checks help your team learn and follow best practices for building container images. When you run a [Docker Build][4], you will get a list of warnings for any check violations detected in your build. Taking a proactive approach and resolving Build warnings and issues early will save you time and headaches downstream. 

![Banner how to set up the weaviate vector database on docker](https://www.docker.com/wp-content/uploads/2023/09/banner_how-to-set-up-the-weaviate-vector-database-on-docker-1110x583.png "- Banner How To Set Up The Weaviate Vector Database On Docker")

## Why did we create Docker Build checks?

During conversations with developers, we found that many struggle to learn and follow the best practices for building container images. According to our [**2024 State of Application Development Report**][5], 35% of Docker users reported creating and editing [Dockerfiles][6] as one of the top three tasks performed. However, 55% of respondents reported that creating Dockerfiles is the most selected task they refer to support. 

Developers often don’t have the luxury of reading through the [Docker Build docs][7], making the necessary changes to get things working, and then moving on. A Docker Build might “work” when you run `docker build`, but a poorly written Dockerfiles may introduce quality issues, such as they are:

-   Hard to maintain or update
-   Contain hidden and unexpected bugs 
-   Have sub-optimal performance

In our conversations with Docker users, we heard that they want to optimize their Dockerfiles to improve build performance, aren’t aware of current best practices, and would like to be guided as they build. 

Investigating and fixing build issues wastes time. We created Docker Build checks to empower developers to write well-structured Dockerfiles from the get-go and learn from existing best practices. With Build checks, your team spends less time on build issues and more on innovation and coding.   

## Why should you use Docker Build checks? 

You want to write better Dockerfiles and save time! 

We have collected a set of best practices from the community of build experts and codified them into Docker Build tooling. You can use Docker Build checks to evaluate all stages of your local and CI workflows, including multi-stage builds and [Bake][8], and deep dive in the [Docker Desktop Builds view][9]. You can also choose which rules to skip. 

You can access Docker Build checks in the CLI and in the Docker Desktop Builds view. 

### More than just linting: Docker Build checks are powerful and fast 

Linting tools typically just evaluate the text files against a set of rules. As a native part of Docker Build, the rules in Docker Build checks are more powerful and accurate than just linting. Docker Build checks evaluate the entire build, including the arguments passed in and the base images used. These checks are quick enough to be run in real-time as you edit your Dockerfile. You can quickly evaluate a build without waiting for a full build execution. 

### Check your local builds

A good practice is to evaluate a new or updated Dockerfile before committing or sharing your changes. Running `docker build` will now give you an overview of issues and warnings in your Dockerfile.

[![Build checks 433 f1](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20545'%3E%3C/svg%3E "- Build Checks 433 F1")

![Build checks 433 f1](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f1-1110x545.png "- Build Checks 433 F1")

][10]

Figure 1: A Docker Build with four check warnings displayed.

To get more information about these specific issues, you can specify the debug flag to the Docker CLI with `docker --debug build`. This information includes the type of warning, where it occurs, and a link to more information on how to resolve it. 

[![Build checks 433 f2](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20948'%3E%3C/svg%3E "- Build Checks 433 F2")

![Build checks 433 f2](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f2-1110x948.png "- Build Checks 433 F2")

][11]

Figure 2: Build debug output for the check warnings.

### Quickly check your build

Running these checks during a build is great, but it can be time-consuming to wait for the complete build to run each time when you’re making changes or fixing issues. For this reason, we added the `--check` flag as part of the build command. 

```
# The check flag can be added anywhere as part of your build command
docker build . --check
docker build --check .
docker build --build-arg VERSION=latest --platfrom linux/arm64 . --check
```

As illustrated in the following figure, appending the flag to your existing build command will do the same full evaluation of the build configuration without executing the entire build. This faster feedback typically completes in less than a second, making for a smoother development process. 

[![Build checks 433 f3](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20938'%3E%3C/svg%3E "- Build Checks 433 F3")

![Build checks 433 f3](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f3-1110x938.png "- Build Checks 433 F3")

][12]

Figure 3: Running check of build.

### Check your CI builds

By default, running a Docker build with warnings will not cause the build to fail (return a non-zero exit code). However, to catch any regressions in your CI builds, add the following declarations to instruct the checks to generate errors. 

```
# syntax=docker/dockerfile:1
# check=error=true

FROM alpine
CMD echo "Hello, world!"
```

### Checking multi-stage builds in CI

During a build, only the specified stage/target, including its dependent, is executed. We recommend adding a stage check step in your workflow to do a complete evaluation of your Dockerfile. This is similar to how you would run automated tests before executing the full build.

If any warnings are detected, it will return a non-zero exit code, which will cause the workflow to fail, therefore catching any issues.

```
docker build --check .
```

### Checking builds in Docker Build Cloud

Of course, this also works seamlessly with [Docker Build Cloud][13], both locally and through CI. Use your [e][14]xisting cloud builders to evaluate your builds. Your team now has the combined benefit of Docker Build Cloud performance with the reassurance that the build will align with best practices. In fact, as we expand our checks, you should see even better performance from your Docker Build Cloud builds.

[![Build checks 433 f4](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20294'%3E%3C/svg%3E "- Build Checks 433 F4")

![Build checks 433 f4](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f4-1110x294.png "- Build Checks 433 F4")

][15]

Figure 4: Running checks in Docker Build Cloud.

### Configure rules

You have the flexibility to configure rules in Build checks with a skip argument. You can also specify `skip=all` or `skip=none` to toggle the rules on and off. Here’s an example of skipping the `JSONArgsRecommended` and `StageNameCasing` rules:

```
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing

FROM alpine AS BASE_STAGE
CMD echo "Hello, world!"
```

### Dive deep into Docker Desktop Builds view

In Docker Desktop Builds view, you can see the output of the build warnings. Locating the cause of warnings in Dockerfiles and understanding how to resolve them quickly is now easy.

As with build errors, warnings are shown inline with your Dockerfile when inspecting a build in Docker Desktop:

[![Build checks 433 f5](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20977'%3E%3C/svg%3E "- Build Checks 433 F5")

![Build checks 433 f5](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f5-1110x977.png "- Build Checks 433 F5")

][16]

Figure 5: Build checks warnings in Docker Desktop Builds view.

## What’s next? 

### More checks

We are excited about the new Builds checks to help you apply best practices to your Dockfiles, but this is just the start. In addition to the [current set of checks][17], we plan on adding even more to provide a more comprehensive evaluation of your builds. Further, we look forward to including custom checks and policies for your Docker builds.

### IDE integration

The earlier you identify issues in your builds, the easier and less costly it is to resolve them. We plan to integrate Build checks with your favorite IDEs so you can get real-time feedback as you type.

[![Build checks 433 f6](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20740'%3E%3C/svg%3E "- Build Checks 433 F6")

![Build checks 433 f6](https://www.docker.com/wp-content/uploads/2024/07/build-checks-433_f6-1110x740.png "- Build Checks 433 F6")

][18]

Figure 6: Check violations displaying in VS Code.

### GitHub Actions and Docker Desktop

You can already see Build checks warnings in Docker Desktop, but more detailed insights are coming soon to Docker Desktop. As you may have heard, we recently announced [Inspecting Docker Builds in GitHub Actions][19]’s beta release, and we plan to build on this new functionality to include support for investigating check warnings.

## Get started now

To get started with Docker Build checks, upgrade to [Docker Desktop 4.33][20] today and try them out with your existing Dockerfiles. Head over to [our documentation][21] for a more detailed breakdown of Build checks. 

## Learn more

-   [Authenticate and up][22][date][23] to receive your subscription level’s newest Docker Desktop features.
-   What else is [new Docker Desktop 4.33][24]? GA Releases of Docker Debug and Docker Build Checks Plus Enhanced Configuration Integrity Checks.
-   New to Docker? [Create an account][25]. 
-   Subscribe to the [Docker Newsletter][26].

[build][27], [Docker Desktop][28], [dockerfile][29]

[

][30]

#### [Docker Scout Health Scores: Security Grading for Container Images in Your Docker Hub Repo][31]

By [Tazin Progga][32]

[

][33]

#### [Docker Desktop 4.33: GA Releases of Docker Debug and Docker Build Checks Plus Enhanced Configuration Integrity Checks][34]   

By [Deanna Sparks][35] July 29, 2024

[

][36]

#### [How to Create Dockerfiles with GenAI][37] 

By [Docker Labs][38] July 29, 2024

#### Posted

Jul 29, 2024

-   [][39]
-   [][40]
-   [][41]

#### Post Tags

[build][42][Docker Desktop][43][dockerfile][44]

#### Categories

-   [Community][45]
-   [Company][46]
-   [Engineering][47]
-   [Products][48]

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