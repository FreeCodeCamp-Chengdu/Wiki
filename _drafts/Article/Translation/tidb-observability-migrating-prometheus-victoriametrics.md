---
title: "Scaling Observability: Why TiDB Moved from Prometheus to VictoriaMetrics"
date: 2025-07-17T00:00:00.000Z
author: Jack Ma
authorURL: https://www.pingcap.com/blog/author/jma/
originalURL: https://www.pingcap.com/blog/tidb-observability-migrating-prometheus-victoriametrics/
translator: ""
reviewer: ""
---

<iframe src="https://www.googletagmanager.com/ns.html?id=GTM-TPX49SBK" height="0" width="0" style="display:none;visibility:hidden"></iframe>

<!-- more -->

<iframe src="https://www.googletagmanager.com/ns.html?id=GTM-52K4D8X" height="0" width="0" style="display:none;visibility:hidden"></iframe>

[Registration for TiDB SCaiLE 2025 is now open! Secure your spot at our annual event.Register Now][1]

[][2]

[TiDB Cloud Serverless][3]

A fully-managed, auto-scaling cloud service designed for your dynamic workloads.

[TiDB Cloud Dedicated][4]

A fully-managed cloud DBaaS for predictable performance and dedicated resources.

[TiDB Self-Managed][5]

An advanced, open source, distributed SQL database for your infrastructure.

Capabilities [Horizontal Scaling][6] [Vector Search][7]

Ecosystem [Integrations][8] [TiKV][9] [TiSpark][10] [OSS Insight][11]

[Pricing][12]

[Customer Stories][13]

Trusted and verified by innovation leaders around the world.

By Industry [Fintech][14] [eCommerce][15] [SaaS][16]

By Use Case [Enable Operational Intelligence][17] [Modernize MySQL Workloads][18]

Learn [Blog][19] [eBooks & Whitepapers][20] [Videos & Replays][21] [Developer Hub][22]

Engage [Events & Webinars][23] [Discord Community][24] [HTAP Summit][25] [Spring Launch Event][26]

PingCAP University [Courses][27] [Hands-on Labs][28] [Certifications][29]

[Trust Hub][30]

Explore how TiDB ensures the confidentiality and availability of your data.

About [Press Releases & News][31] [About Us][32] [Careers][33] [Partners][34] [Contact Us][35]

[Docs][36]

[Sign In][37] [Start for Free][38]

Products

[TiDB Cloud Serverless][39]

A fully-managed, auto-scaling cloud service designed for your dynamic workloads.

[TiDB Cloud Dedicated][40]

A fully-managed cloud DBaaS for predictable performance and dedicated resources.

[TiDB Self-Managed][41]

An advanced, open source, distributed SQL database for your infrastructure.

Capabilities

[Horizontal Scaling][42] [Vector Search][43]

Ecosystem

[Integrations][44] [TiKV][45] [TiSpark][46] [OSS Insight][47]

[Pricing][48]

Solutions

[Customer Stories][49]

Trusted and verified by innovation leaders around the world.

By Industry

[Fintech][50] [eCommerce][51] [SaaS][52]

By Use Case

[Enable Operational Intelligence][53] [Modernize MySQL Workloads][54]

Resources

Learn

[Blog][55] [eBooks & Whitepapers][56] [Videos & Replays][57] [Developer Hub][58]

Engage

[Events & Webinars][59] [Discord Community][60] [HTAP Summit][61] [Spring Launch Event][62]

PingCAP University

[Courses][63] [Hands-on Labs][64] [Certifications][65]

Company

[Trust Hub][66]

Explore how TiDB ensures the confidentiality and availability of your data.

About

[Press Releases & News][67] [About Us][68] [Careers][69] [Partners][70] [Contact Us][71]

[Docs][72]

[Sign In][73]

[Start for Free][74]

# Scaling Observability: Why TiDB Moved from Prometheus to VictoriaMetrics

[Engineering][75]

![](https://static.pingcap.com/files/2025/07/17081832/1523679041566-150x150.jpeg)

[Jack Ma][76]

Professional Services Engineer

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

From the outset, Prometheus has served as a go-to tool for real-time performance metric collection, storage, querying, and observability in [TiDB][77]. Additionally, TiDB supports offline diagnostics using [**TiDB Clinic**][78], which allows for replaying collected metrics to investigate historical issues.

![](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

_Figure 1. How TiDB Clinic provides offline diagnostics._

However, as deployments scaled, so did the challenges of using Prometheus. This blog explores those growing pains and why we ultimately transitioned to [VictoriaMetrics][79], a high-performance, open-source time series database and monitoring solution.

## TiDB Observability: The Limitations of Prometheus at Scale

[Pinterest][80] is one of PingCAP’s largest enterprise customers, operating a TiDB cluster with over 700 nodes handling 700K+ QPS. However, the team began to encounter monitoring issues. During diagnostic sessions with TiDB Clinic, Prometheus consistently crashed, escalating operational burdens and delaying incident resolution.

<iframe width="1280" height="720" src="//www.youtube.com/embed/mvmw4NhJko4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen=""></iframe>

### Scalability Issues

During PingCAP’s work with the Pinterest team, they observed frequent **out of memory (OOM)** crashes in Prometheus, even on a high-end i4i.24xlarge instance (96 cores, 768GB RAM). Query failures and long restart times significantly impacted their ability to diagnose issues effectively.

![](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

_Figure 2. A diagram representing Pinterest’s OOM crash using Prometheus._

#### OOM (Out of Memory) Problem

When executing large queries, Prometheus frequently ran OOM, leading to crashes. The restart process posed additional challenges:

-   **Long Recovery Time**: After an OOM crash, Prometheus needed to replay the Write-Ahead Log (WAL), a process that could take at least 40 minutes, sometimes failing altogether.
-   **Recurring OOMs**: In some cases, the WAL replay triggered another OOM, preventing Prometheus from recovering.
-   **Extended Downtime & Data Loss**: This instability led to prolonged monitoring outages and potential metric loss, making it unreliable for mission-critical clusters.

The below logs show crash recovery duration in a test environment with 400 nodes.

WAL start at 22:52:07

![](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

WAL finished at 23:34:32, spent 42mins

![](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

These limitations highlighted the need for a more scalable and resilient monitoring solution.

#### Query Performance

For large queries, we had to limit the time range to 15 minutes; otherwise, the query would either become slow or fail entirely.

A similar issue occurred with clinic data collection. When attempting to retrieve 1 hour of metrics, the query would run for 40 minutes before eventually failing due to OOM. 

### Total Cost of Ownership (TCO)

With Prometheus, we had to allocate a large monitoring instance (i4i.24xlarge, 96 cores, 768GB RAM), yet it still struggled with stability and performance.

## Why TiDB Switched to VictoriaMetrics for Enhanced Observability

To meet the evolving needs of our internal teams and cloud customers, we evaluated alternative time-series backends and ultimately migrated to VictoriaMetrics. Below are key reasons behind the switch and the concrete improvements that followed.

### 1\. Better Resource Utilization

After migrating to VictoriaMetrics, we observed a significant reduction in resource consumption:

-   **CPU usage below 50%**
-   **Memory usage remained under 35%**, eliminating OOM crashes.
-   **Stable performance**, even under heavy query loads.

### 2\. Improved Query Performance

-   Large queries that previously worked only for a 15-minute window in Prometheus can now span hours in VictoriaMetrics.
-   Faster query execution, making troubleshooting and historical analysis more efficient.
-   However, some of the largest queries still face challenges and cannot fully execute, indicating room for further optimization.

### 3\. Lower Resource Consumption and Improved TCO

After switching to VictoriaMetrics, Pinterest significantly reduced its resource consumption while improving stability. Additionally, better storage efficiency helped lower disk usage, making monitoring more cost-effective.

Overall, VictoriaMetrics provided greater stability, efficiency, and scalability, making it a more reliable solution for monitoring TiDB.

After validating the improvements with Pinterest, the team there successfully migrated to VictoriaMetrics.

## TiDB Observability: Performance Test Results

To evaluate VictoriaMetrics’ impact, we conducted tests on Pinterest’s cluster using different configurations. The results showed that VictoriaMetrics significantly reduced resource usage and improved query performance, as shown in the below table.

<table class="has-fixed-layout"><tbody><tr><td><strong>Metrics</strong></td><td><strong>Duration</strong></td><td><strong>Performance</strong></td></tr><tr><td>Prom – KV Request</td><td>last 15mins</td><td>Failed</td></tr><tr><td>VM – KV Request – Default</td><td>last 15mins</td><td>Failed 30s</td></tr><tr><td>VM – KV Request – Tuned</td><td>last 15mins</td><td>Failed 1mins</td></tr><tr><td>VM – KV Request – Release</td><td>last 15mins</td><td>Success 3mins</td></tr><tr><td>Prom – 99% grpc request duration</td><td>Last 30mins</td><td>Failed 26s</td></tr><tr><td>Prom – 99% grpc request duration</td><td>Last 1h</td><td>Failed 15s</td></tr><tr><td>VM – 99% grpc request duration – default</td><td>Last 1h</td><td>Success 8.4s</td></tr><tr><td>VM – 99% grpc request duration – Tuned</td><td>Last 1h</td><td>Success 7.4s</td></tr><tr><td>VM – 99% grpc request duration – Release</td><td>Last 1h</td><td>Success 6.5s</td></tr></tbody></table>

### Key Takeaways

1.  **Prometheus consistently failed on KV requests**, while VictoriaMetrics showed significant improvements, especially in the Release configuration (success in 3 mins).
2.  **Prometheus struggled with gRPC request duration**, failing even for a 30-minute or 1-hour query.
3.  **VictoriaMetrics significantly improved gRPC query performance**, reducing execution time from 8.4s (Default) to 6.5s (Release).
4.  Different VictoriaMetrics configurations (Default, Tuned, Release) adjusted parameters like maxQueryDuration and maxSeries, impacting performance and success rates.

## A Smooth Migration Strategy

Given that the data retention period was 10 days, we took a gradual migration approach to minimize risks and ensure a smooth transition from Prometheus to VictoriaMetrics.

### **Step 1: Parallel Deployment for Observation**

-   Based on our testing results, we found that scraping metrics had a minimal impact on the TiDB cluster**.**
-   Instead of an immediate switch, we opted to run Prometheus and VictoriaMetrics in parallel**,** allowing us to observe performance and stability without disrupting existing monitoring workflows.

### **Step 2: Validation & Monitoring**

-   During this phase, we compared the data accuracy, query performance, and system stability between Prometheus and VictoriaMetrics**.**
-   Engineers continuously monitored VictoriaMetrics’ resource utilization, query response times, and failure rates to ensure reliability**.**

### **Step 3: Final Cutover**

-   Once we gathered sufficient monitoring data and confirmed that everything was running smoothly, we fully switched to VictoriaMetrics.
-   At this point, Prometheus was shut down, completing the migration without downtime or data loss.

This progressive migration strategy allowed us to ensure a stable transition while avoiding the risks of an abrupt switch.

### Configuration and Integration Considerations

Migrating from Prometheus to VictoriaMetrics required adjustments across several key components, including scrape configurations, discovery files, startup scripts, Grafana dashboards, and clinic integration.

#### **Scrape Configuration & Discovery Files**

-   VictoriaMetrics maintains compatibility with most Prometheus setups.
-   We were able to reuse existing scrape configuration files and discovery files with minor modifications to handle incompatible parameters.

#### **Startup Script Testing**

We tested three different VictoriaMetrics startup configurations to balance query performance, resource limits, and stability**.** We ended up choosing a tuned configuration approach.

##### **1.  Default Configuration (Baseline Setup)**

-   Basic setup with minimal tuning
-   Configured for 10-day retention and max scrape size of 400MB

`docker run -it -v {PATH}/victoria-metrics-data:/victoria-metrics-data \`

    `--network host -p 8428:8428 victoriametrics/victoria-metrics:v1.106.1 \`

    `-retentionPeriod=10d \`

    `-promscrape.config=/victoria-metrics-data/vm.config \`

    `-promscrape.maxScrapeSize=400MB`

##### **2.  Tuned Configuration (Slight Limits Increase, Final Choice ✅)**

-   Increased query duration and series limits to handle larger queries.

`docker run -it -v {PATH}/victoria-metrics-data:/victoria-metrics-data \`

    `--network host -p 8428:8428 victoriametrics/victoria-metrics:v1.106.1 \`

    `-search.maxSeries=5000000 \`

    `-search.maxLabelsAPISeries=5000000 \`

    `-search.maxQueryDuration=1m \`

    `-promscrape.config=/victoria-metrics-data/vm.config \`

    `-promscrape.maxScrapeSize=400MB \`

    `-search.maxSamplesPerQuery=1000000000 \`

    `-search.logSlowQueryDuration=30s \`

    `-retentionPeriod=10d`

##### **3.  Release Configuration (Aggressive Limits, Not Chosen)**

-   Further increased limits to support more complex and long-running queries.

`docker run -it -v /mnt/docker/overlay2/victoria-metrics-data:/victoria-metrics-data \`

    `--network host -v /var/lib/normandie:/var/lib/normandie:ro,rslave \`

    `-p 8428:8428 victoriametrics/victoria-metrics:v1.106.1 \`

    `-search.maxSeries=50000000 \`

    `-search.maxLabelsAPISeries=50000000 \`

    `-search.maxQueryDuration=10m \`

    `-promscrape.config=/victoria-metrics-data/vm.config \`

    `-promscrape.maxScrapeSize=400MB \`

    `-search.maxSamplesPerQuery=10000000000 \`

    `-search.logSlowQueryDuration=30s \`

    `-retentionPeriod=10d`

#### **Clinic Command for Diagnostics**

To collect TiDB clinic diagnostics, we adjusted the command to use VictoriaMetrics as the Prometheus replacement:

`tiup diag util metricdump --name {cluster_name} --pd={PD_URL}:{PD_PORT} \`

    `--prometheus="{VICTORIA_URL}:8428" --from "-1h" --to "-0h"`

#### **Grafana Dashboard Adjustments**

-   We configured Grafana to use VictoriaMetrics as the Prometheus data source.
-   Required modifications to each dashboard to default to the new VictoriaMetrics data source.

![](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

## Final Thoughts: The Future of TiDB Observability

Observability remains a crucial pillar of maintaining a healthy, high-performance TiDB cluster. With that said, the move to VictoriaMetrics has greatly improved scalability, reliability, and efficiency.

Looking ahead, further enhancements could include:

-   Optimizing for ultra-large queries.
-   Exploring long-term storage solutions for historical metric retention beyond 10 days.
-   Building a unified monitoring platform to improve resource usage and user experience.

VictoriaMetrics has proven to be a powerful foundation for TiDB observability at scale.

If you have any questions about TiDB’s approach to monitoring and observability, please feel free to connect with us on [Twitter][81], [LinkedIn][82], or through our [Slack Channel][83]. 

[Database Monitoring][84][Distributed SQL][85][Observability][86][TiDB][87][VictoriaMetrics][88]

  

Experience modern data infrastructure firsthand.

[Try TiDB Serverless][89]

Share:[][90][][91][][92]

## Related Resources

[

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Solution

##### 5 Reasons Why Amazon Aurora Fails to Scale Large, Complex Workloads



][93][

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Product

##### From Hypothesis to Validation: How User Research Shaped TiDB Cloud’s New Navigation



][94][

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Solution

##### The FinTech Scalability Crisis: How Distributed SQL Unlocks Innovation with Zero Downtime Operations



][95]

[

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Solution

##### 5 Reasons Why Amazon Aurora Fails to Scale Large, Complex Workloads



][96]

[

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Product

##### From Hypothesis to Validation: How User Research Shaped TiDB Cloud’s New Navigation



][97]

[

![tidb_feature_1800x600 (1)](data://image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAAABCAQAAACC0sM2AAAADElEQVR42mNkGCYAAAGSAAIVQ4IOAAAAAElFTkSuQmCC)

Solution

##### The FinTech Scalability Crisis: How Distributed SQL Unlocks Innovation with Zero Downtime Operations



][98]

[View All][99]

### Have questions? Let us know how we can help.

[Contact Us][100]

## TiDB Cloud Dedicated

A fully-managed cloud DBaaS for predictable workloads

[Sign Up][101] [Learn More][102]

## TiDB Cloud Serverless

A fully-managed cloud DBaaS for auto-scaling workloads

[Start for Free][103] [Learn More][104]

English

[中文][105] [日本語][106]

Products

[TiDB Cloud Serverless][107] [TiDB Cloud Dedicated][108] [TiDB Self-Managed][109] [Pricing][110]

English

[中文][111] [日本語][112]

Ecosystem

[Integrations][113] [TiKV][114] [TiSpark][115] [OSS Insight][116]

Resources

[Blog][117] [Articles][118] [Events & Webinars][119] [HTAP Summit][120] [Docs][121] [Developer Guide][122] [FAQs][123] [Support][124]

Company

[About Us][125] [News][126] [Careers][127] [Contact Us][128] [Partners][129] [Trust Hub][130] [Security][131] [Release Support][132] [Brand Guidelines][133]

Stay Connected

Sign up to receive periodic updates and  
feature releases to your email.

[][134][][135][][136][][137][][138][][139][][140]

© 2025 PingCAP. All Rights Reserved.

[Privacy Policy][141] [Legal][142]

     

[1]: /tidb-scaile-summit/
[2]: https://www.pingcap.com "TiDB"
[3]: /tidb-cloud-serverless/
[4]: /tidb-cloud-dedicated/
[5]: /tidb-self-managed/
[6]: /horizontal-scaling-vs-vertical-scaling/
[7]: /ai/
[8]: https://www.pingcap.com/integrations/
[9]: https://github.com/tikv/tikv
[10]: https://github.com/pingcap/tispark
[11]: https://ossinsight.io/
[12]: /pricing/
[13]: /customers/
[14]: /solutions/fintech/
[15]: /solutions/e-commerce/
[16]: /solutions/saas/
[17]: https://www.pingcap.com/solutions/enable-operational-intelligence/
[18]: /solutions/modernize-mysql-workloads/
[19]: /blog/
[20]: /ebook-whitepaper/
[21]: /videos/
[22]: https://www.pingcap.com/developers/
[23]: /event/
[24]: https://discord.com/invite/vYU9h56kAX
[25]: /htap-summit/
[26]: /tidb-spring-launch-25/
[27]: https://www.pingcap.com/education/
[28]: /education/tidb-hands-on-labs/
[29]: https://www.pingcap.com/education/certification/
[30]: /trust-hub/
[31]: /press-releases-news/
[32]: /about-us/
[33]: https://www.pingcap.com/careers/
[34]: https://www.pingcap.com/partners/
[35]: https://www.pingcap.com/contact-us/
[36]: https://docs.pingcap.com/
[37]: https://tidbcloud.com/signin
[38]: https://tidbcloud.com/free-trial/
[39]: /tidb-cloud-serverless/
[40]: /tidb-cloud-dedicated/
[41]: /tidb-self-managed/
[42]: /horizontal-scaling-vs-vertical-scaling/
[43]: /ai/
[44]: https://www.pingcap.com/integrations/
[45]: https://github.com/tikv/tikv
[46]: https://github.com/pingcap/tispark
[47]: https://ossinsight.io/
[48]: /pricing/
[49]: /customers/
[50]: /solutions/fintech/
[51]: /solutions/e-commerce/
[52]: /solutions/saas/
[53]: https://www.pingcap.com/solutions/enable-operational-intelligence/
[54]: /solutions/modernize-mysql-workloads/
[55]: /blog/
[56]: /ebook-whitepaper/
[57]: /videos/
[58]: https://www.pingcap.com/developers/
[59]: /event/
[60]: https://discord.com/invite/vYU9h56kAX
[61]: /htap-summit/
[62]: /tidb-spring-launch-25/
[63]: https://www.pingcap.com/education/
[64]: /education/tidb-hands-on-labs/
[65]: https://www.pingcap.com/education/certification/
[66]: /trust-hub/
[67]: /press-releases-news/
[68]: /about-us/
[69]: https://www.pingcap.com/careers/
[70]: https://www.pingcap.com/partners/
[71]: https://www.pingcap.com/contact-us/
[72]: https://docs.pingcap.com/
[73]: https://tidbcloud.com/signin
[74]: https://tidbcloud.com/free-trial/
[75]: https://www.pingcap.com/blog/?category=engineering
[76]: https://www.pingcap.com/blog/author/jma/
[77]: https://www.pingcap.com/tidb-self-managed/
[78]: https://docs.pingcap.com/tidb/stable/clinic-introduction/
[79]: https://github.com/VictoriaMetrics/VictoriaMetrics
[80]: https://www.pingcap.com/blog/why-pinterest-modernized-graph-service-distributed-sql/
[81]: https://twitter.com/PingCAP
[82]: https://www.linkedin.com/company/pingcap/mycompany/
[83]: https://slack.tidb.io/invite?team=tidb-community&channel=everyone&ref=pingcap&__hstc=86493575.a1fea8c8486aaa74956704cbebabab43.1749809256999.1752759849602.1752764486336.91&__hssc=86493575.7.1752764486336&__hsfp=1923652471
[84]: /blog?tag=database-monitoring
[85]: /blog?tag=distributed-sql
[86]: /blog?tag=observability
[87]: /blog?tag=tidb
[88]: /blog?tag=victoriametrics
[89]: https://tidbcloud.com/free-trial/
[90]: https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.pingcap.com%2Fblog%2Ftidb-observability-migrating-prometheus-victoriametrics%2F
[91]: https://twitter.com/intent/tweet?text=Scaling%20Observability%3A%20Why%20TiDB%20Moved%20from%20Prometheus%20to%20VictoriaMetrics%20%40PingCAP%20https%3A%2F%2Fwww.pingcap.com%2Fblog%2Ftidb-observability-migrating-prometheus-victoriametrics%2F
[92]: https://www.linkedin.com/shareArticle?url=https%3A%2F%2Fwww.pingcap.com%2Fblog%2Ftidb-observability-migrating-prometheus-victoriametrics%2F&title=Scaling%20Observability%3A%20Why%20TiDB%20Moved%20from%20Prometheus%20to%20VictoriaMetrics
[93]: https://www.pingcap.com/blog/amazon-aurora-fails-scale-large-complex-workloads/
[94]: https://www.pingcap.com/blog/tidb-cloud-user-research-new-console-navigation/
[95]: https://www.pingcap.com/blog/fintech-scalability-crisis-how-distributed-sql-unlocks-innovation-zero-downtime/
[96]: https://www.pingcap.com/blog/amazon-aurora-fails-scale-large-complex-workloads/
[97]: https://www.pingcap.com/blog/tidb-cloud-user-research-new-console-navigation/
[98]: https://www.pingcap.com/blog/fintech-scalability-crisis-how-distributed-sql-unlocks-innovation-zero-downtime/
[99]: https://www.pingcap.com/blog/
[100]: https://www.pingcap.com/contact-us/
[101]: https://tidbcloud.com/signup/?signup_source=pingcap-en-dedicated
[102]: https://www.pingcap.com/tidb-cloud-dedicated/
[103]: https://tidbcloud.com/free-trial/
[104]: https://www.pingcap.com/tidb-cloud-serverless/
[105]: https://cn.pingcap.com/
[106]: https://pingcap.co.jp/
[107]: https://www.pingcap.com/tidb-cloud-serverless/
[108]: https://www.pingcap.com/tidb-cloud-dedicated/
[109]: https://www.pingcap.com/tidb-self-managed/
[110]: https://www.pingcap.com/pricing/
[111]: https://cn.pingcap.com/
[112]: https://pingcap.co.jp/
[113]: https://www.pingcap.com/integrations/
[114]: https://github.com/tikv/tikv
[115]: https://github.com/pingcap/tispark
[116]: https://ossinsight.io/
[117]: https://www.pingcap.com/blog/
[118]: /article/
[119]: /event
[120]: /htap-summit/
[121]: https://docs.pingcap.com
[122]: https://docs.pingcap.com/tidb/stable/dev-guide-overview
[123]: https://www.pingcap.com/faqs/
[124]: https://tidb.support.pingcap.com/
[125]: https://www.pingcap.com/about-us/
[126]: /press-releases-news
[127]: https://www.pingcap.com/careers/
[128]: https://www.pingcap.com/contact-us/
[129]: https://www.pingcap.com/partners/
[130]: https://www.pingcap.com/trust-hub/
[131]: https://www.pingcap.com/security/
[132]: https://www.pingcap.com/tidb-release-support-policy/
[133]: /tidb-brand-guidelines/
[134]: https://github.com/pingcap
[135]: https://twitter.com/PingCAP
[136]: https://linkedin.com/company/pingcap
[137]: https://facebook.com/pingcap2015
[138]: https://slack.tidb.io/invite?team=tidb-community&channel=everyone&ref=pingcap
[139]: https://discord.com/invite/vYU9h56kAX
[140]: https://www.youtube.com/PingCAP
[141]: https://www.pingcap.com/privacy-policy/
[142]: /legal/