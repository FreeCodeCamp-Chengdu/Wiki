---
title: I Didn't Need Kubernetes, and You Probably Don't Either
date: 2024-12-02T04:13:25.239Z
authorURL: ""
originalURL: https://benhouston3d.com/blog/why-i-left-kubernetes-for-google-cloud-run
translator: ""
reviewer: ""
---

# I Didn't Need Kubernetes, and You Probably Don't Either

<!-- more -->

By [Ben Houston][1], 2024-11-05

_(This blog post was discussed on YCombinator's Hacker News [here][2].)_

Kubernetes often represents the ultimate solution for container orchestration, but my experience has led me to leave it behind in favor of a simpler, cost-effective solution using [Google Cloud Run][3]. This transition has made my infrastructure projects easier to manage, more scalable, and significantly cheaper. Here’s why I made this choice and how Cloud Run offers a better fit for my needs going forward.

## How I ended up on Kubernetes

First, let's quickly look at how we ended up on Kubernetes. We has launched [Clara.io (now sunset)][4], an online 3D editor and rendering platform, in 2013. We cost optimized the platform by using bare metal machines from [OVH][5] for both its primary servers, DBs and job workers. While it worked, bare metal machines were a source of potential failures and we did have some over the years. Luckily we had a redundant setup so our users never noticed. But it was a massive amount of work to provision, monitor and maintain.

So when we looked to remake the platform for [Threekit.com][6], an enterprise focused re-imaging of Clara.io, we looked to switch to a managed compute setup. At the time in 2018, Kubernetes was just emerging as the industry solution as Azure, AWS and Docker converged on it in late 2017.

## Why I Left Kubernetes

At first, Kubernetes seemed like the right approach for managing services at scale. But as we lived with it over the years since 2018, it became apparent that the complexity and costs were significant. Here’s a breakdown of the challenges we encountered:

1.  **Cost Overruns**: Kubernetes comes with substantial infrastructure costs that go beyond DevOps and management time. The high cost arises from needing to provision a bare-bones cluster with redundant management nodes. Moreover, Kubernetes’s slow autoscaling meant I had to over-provision services to ensure availability, paying for unused resources rather than scaling based on demand.
    
2.  **Difficulty Managing Large Job Volumes**: Handling a high volume of jobs on Kubernetes is tedious. Both the built-in scheduler and Argo introduced limitations and complexity, often failing to scale well under load or just being really complex.
    
3.  **Complexity Overload**: Kubernetes is feature-rich, yet these “enterprise” capabilities turned even simple tasks into protracted processes. The result was added complexity without substantial benefits, making it more of an obstacle than a solution. If you have Kubernetes, you probably need at least one dedicated Kubernetes dev-ops engineer, if not a couple.
    

Overall, Kubernetes proved difficult to provision, expensive to maintain, and time-consuming to manage. For companies or individual projects seeking simplicity and cost-effectiveness, it may not be the right choice. But it did get rid of the need for us to keep track of whether the hardware in our machines was breaking down and our manual provision processing.

## Embracing Cloud Run for a Simpler Setup

Google Cloud Run offers a streamlined alternative to Kubernetes. Here’s how my new setup works:

### The Setup

My infrastructure is now centered around **Docker containers**, with some running as auto-scaling services and others as tasks for long-running jobs. Google Cloud Run handles container deployment, scaling, downtime management, and running/retrying jobs eliminating many of the challenges I faced with Kubernetes.

### Why Cloud Run Is Ideal

1.  **Cost Efficiency**: Cloud Run charges only for the CPU and memory used during requests, making it highly cost-effective. For example, this personal project of mine, [Web3D Survey][7] sees 500,000 hits monthly and costs just **$4/month** for hosting, even though it is running all the time. They key is that Cloud Run charges based on the fraction of the CPU you use per second. The ability to scale to zero also means no charges for idle services.
2.  **Fast, Reliable Autoscaling**: Cloud Run scales in a few seconds, unlike Kubernetes, where scaling often took minutes. This quick scaling lets me handle surges reliably without over-provisioning.
3.  **No Kubernetes Management Overhead**: Cloud Run, built atop Google’s Borg, avoids the need for Kubernetes cluster management, simplifying deployments and cutting costs.
4.  **Simple Async Tasks**: Cloud Run Tasks allows me to execute up to 10,000 tasks per job, track their results with auto-retries with no need to manage job-running infrastructure or individual machines — a refreshing alternative to the complexity running this on Kubernetes was.

## The Kubernetes Lock-In Trap

One overlooked downside of Kubernetes is **cluster lock-in**. Once you start using Kubernetes-specific features, it becomes challenging to leverage resources outside the cluster. Integrating services across data centers or using dedicated resources in a colocation setup adds significant complexity. Kubernetes demands you stay within its ecosystem, effectively binding you to the infrastructure that supports it, making even simple expansions or migrations into costly, complex endeavors.

## Addressing Common Concerns

Several questions arose from others when I shared my experience online, so here’s a deeper look into how this setup works:

-   **Orchestration**: I use GitHub Actions CI to handle CI/CD, leveraging workflows with dependencies and matrix strategies for builds across multiple services which only deploy if all tests/builds are successful.
-   **Storage**: Managed databases or Cloud Storage cover shared data needs, eliminating the need to manage my own disks.
-   **Inter-Service Communication**: For async communication, I leverage pub-sub messaging. When direct connections are absolutely necessary, each service has a dedicated domain-name.
-   **Security**: Cloud Run supports internal-only services, and for public services, I secure routes using JWT.

## Debunking Misconceptions

### “Aren’t You Afraid of Being Locked into GCP?”

Switching cloud providers, such as AWS, wouldn’t be overly complex since my setup is Docker-based, and migration would take about a week. In practice, few companies switch providers unless politics are involved, as the differences between major cloud services are minimal.

### “Cloud Run Is Just Managed Kubernetes, Right?”

Technically, Cloud Run uses Knative interfaces but runs on top of Google’s Borg, rather than Kubernetes. But it could have been implemented on top of Kubernetes and I think similar PaaS systems are implemented on Kubernetes. The key is that I am not using Kubernetes or Borg, I am using a simplified PaaS system that has an opinionated and simple interface that fits my needs.

## Remaining Workflow Pains

There are some pains I have with my current Cloud Run development workflow:

### Arbitrary Management of Services Names

I do find that I need to manage my service names both locally and in the server in a unified way with clear configuration. Basically an abstraction layer. I know Kubernetes has one and I do miss that.

### Lack of Cloud Run Task Emulation

There is no solution for running Cloud Run Tasks locally that I have found. I would like it to be able to just run an arbitrary command line locally in place of a docker container in a locally emulated Cloud Run Tasks environment that captures log output, trackings runs, etc. This would simplify development of tasks without having to build and deploy docker containers.

## My New Stack and Workflow

My infrastructure is built around a reliable stack. You can view a simplistic prototype setup and practices in this [TypeScript monorepo template][8]. Cloud Run enables me to deploy this stack efficiently, without the complexity or cost overheads Kubernetes brought.

## Final Thoughts

For my projects, Cloud Run offers the perfect blend of **cost savings, speed, scalability, and simplicity**. While Kubernetes may suit some large enterprises, for agile projects where simplicity and efficiency matter, the managed environment of Cloud Run is transformative. If your infrastructure goals include reduced DevOps overhead, predictable costs, and responsive scaling, Cloud Run may be the solution you’ve been looking for.

(PS. Yes, I am sure there was another library/extension I should have added to Kubernetes to enable feature X, Y or Z, but this really is just further increasing the complexity. I don't want to have to exist inside of world of Kubernetes-specific lingo and techniques if they are not giving me benefits.)

_This post was inspired by [this discussion on Hacker News on 04/11/2025][9]_

[1]: /about
[2]: https://news.ycombinator.com/item?id=42252336
[3]: https://cloud.google.com/run
[4]: https://clara.io
[5]: https://www.ovhcloud.com/
[6]: https://threekit.com
[7]: https://web3dsurvey.com
[8]: https://github.com/bhouston/template-typescript-monorepo
[9]: https://news.ycombinator.com/item?id=42041917