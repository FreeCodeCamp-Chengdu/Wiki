---
title: Kubernetes 2.0 会是什么样子
date: 2025-06-19T00:00:00.000Z
authorURL: ""
originalURL: https://matduggan.com/what-would-a-kubernetes-2-0-look-like/
translator: ""
reviewer: ""
---


-   [RSS 订阅][3]


在 2012-2013 年左右，我开始在系统管理员社区中大量听到一种叫做"Borg"的技术。它（显然）是 Google 内部的一种 Linux 容器系统，运行着他们所有的服务。术语有点令人困惑，集群中有"cells"的单元里包含叫做"Borglet"的东西，但基本概念开始流传出来。有"服务"的概念和"作业"的概念，应用程序可以使用服务来响应用户请求，然后使用作业来完成运行时间更长的批处理任务。

然后在 2014 年 6 月 7 日，我们收到了 Kubernetes 的第一次提交。这是一个希腊语中的"舵手"一词，在前三年里几乎没有人能正确发音。（是 koo-ber-NET-ees？还是 koo-ber-NEET-ees？还是像我们其他人一样放弃，直接叫它 k8s 吧。）

在此之后不久，Microsoft、RedHat、IBM、Docker 很快加入了 Kubernetes 社区，这使得 Kubernetes 从一个有趣的 Google 项目变成了"也许这是一个真正的产品？"2015 年 7 月 21 日，我们获得了 v1.0 版本发布，同时 CNCF 也成立了。

在第一次提交后的十年里，Kubernetes 成了我职业生涯的重要组成部分。我在家里、在工作中、在副项目中使用它——任何有意义的地方。这是一个学习曲线陡峭的工具，但它也是一个巨大的力量倍增器。我们不再在服务器级别"管理基础设施"；一切都是声明式的、可扩展的、可恢复的，并且（如果你幸运的话）是自愈的。

但这个过程并非一帆风顺。一些常见的趋势已经出现，错误或配置不当源于 Kubernetes 不够规范化。即使在十年后，我们仍然看到生态系统内部存在大量动荡，人们踩在文档完善的陷阱上。那么，既然我们现在已经知道了这些，我们可以做些什么不同的事情，使这个伟大的工具适用于更多的人和问题呢？

### k8s 做对了什么？

让我们从积极的一面开始。为什么我们现在还在讨论这个平台？

**大规模容器化**

容器作为软件开发的工具是非常有意义的。抛弃单个笔记本电脑配置的混乱，采用一个标准的、可丢弃的概念，这个概念可以在整个技术栈中通用。虽然像 Docker Compose 这样的工具允许部署一些容器，但它们很笨重，仍然需要管理员来管理很多步骤。我用部署脚本设置了一个 Compose 堆栈，该脚本会将实例从负载均衡器中移除，拉取新容器，确保它们启动，然后重新添加到负载均衡器中，很多人都这样做。

K8s 允许这个概念扩展，意味着可以将笔记本电脑上的容器部署到数千台服务器上。这种灵活性使组织能够重新审视他们的整个设计策略，放弃单体架构，采用更灵活（通常也更复杂）的微服务设计。

**低维护成本**

如果你将运维的历史视为一种从"宠物到牛命名的时间线"，我们开始于我亲切地称之为"辛普森时代"。服务器是由团队设置的裸机盒子，它们通常有一次的名称，这些名称在团队内部成为了俚语，一切都是独一无二的。服务器运行的时间越长，它积累的冗余就越多，直到重启它们都成为可怕的操作，更不用说尝试重建它们了。我称之为"辛普森时代"是因为在我当时工作的职位中，用辛普森角色命名它们的情况出奇地普遍。没有任何东西能自我修复，一切都是手动操作。

然后我们过渡到"01时代"。像 Puppet 和 Ansible 这样的工具变得普遍，服务器更加可丢弃，你开始看到像堡垒主机和其他访问控制系统成为常态。服务器并不都面向互联网，它们位于负载均衡器后面，我们放弃了可爱的名称，改用"app01"或"vpn02"这样的名称。组织设计成可以在某些时候丢失一些服务器。然而故障仍然不是自愈的，仍然需要有人 SSH 进入查看什么坏了，在工具中编写修复程序，然后在整个集群中部署。操作系统升级仍然是复杂的事情。

我们现在处于"UUID时代"。服务器的存在是为了运行容器，它们是完全可丢弃的概念。没有人关心特定操作系统版本支持多长时间，你只需要烘焙一个新的 AMI 并替换整台机器。K8s 并不是唯一实现这一点的技术，但它是加速这一点的技术。现在，使用 SSH 密钥连接到底层服务器解决问题的堡垒服务器的想法更多地被视为"破窗"解决方案。几乎所有的解决方案都是"销毁那个节点，让 k8s 根据需要重新组织，创建一个新节点"。

很多对我的职业生涯至关重要的 Linux 技能现在很大程度上只是有了就好，而不是必须具备。对此你可以感到高兴或悲伤，我当然经常在这两种情绪之间切换，但这只是事实。

**运行作业**

K8s 作业系统并不完美，但它比多年来工作中极其常见的"雪花 cron01 盒子"要好得多。按照 cron 计划运行或从消息队列运行，现在可以可靠地将作业放入队列，让它们运行，如果它们不工作就重新启动，然后继续你的生活。

这不仅将人类从耗时且无聊的任务中解放出来，而且它也是更有效地利用资源。你仍然为队列中的每个项目启动一个 pod，但你的团队在"pod"概念内部有很大的灵活性，可以决定他们需要运行什么以及如何运行。对于很多人来说，这确实是一个生活质量改善，包括我自己，他们只需要能够轻松地后台任务而不必再考虑它们。

**服务发现和负载均衡**

硬编码的 IP 地址存在于应用程序中作为请求应该路由到何处的模板，这多年来一直是一个困扰我的诅咒。如果你幸运的话，这些依赖不是基于 IP 地址，而是实际上是 DNS 条目，你可以在不协调百万应用程序部署的情况下更改 DNS 条目背后的内容。

K8s 允许使用简单的 DNS 名称来调用其他服务。它消除了整类错误和麻烦，并将整个事情简化下来。使用 Service API，你有一个稳定的、长期存在的 IP 地址和主机名，你可以将内容指向它而不必考虑任何底层概念。你甚至有像 ExternalName 这样的概念，允许你像集群内部一样处理外部服务。

## 我会在 Kubernetes 2.0 中加入什么？

### 用 HCL 替代 YAML

YAML 的吸引力在于它既不是 JSON 也不是 XML，这就像说你的新车很棒，因为它既不是马也不是独轮车。它在 k8s 中演示效果更好，在仓库中看起来更漂亮，并且有作为简单文件格式的_错觉_。实际上，YAML 对于我们试图用 k8s 做的事情来说太过复杂，而且它不够安全。缩进容易出错，文件扩展性不好（你真的不想要一个超长的 YAML 文件），调试可能很烦人。YAML 在其规范中概述了_太多_细微的行为。

我仍然记得第一次看到挪威问题时，不相信我所看到的情况。对于那些幸运到不需要处理这个问题的人来说，YAML 中的挪威问题是当 'NO' 被解释为 false 时。想象一下向你的挪威同事解释，他们的整个国家在你的配置文件中被评估为 false。再加上缺少引号导致的意外数字，这样的例子不胜枚举。有更好的文章解释为什么 YAML 很疯狂，比我能写的要好得多：[https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell][4]

**为什么选择 HCL？**

HCL 已经是 Terraform 的格式，所以至少我们只需要讨厌一种配置语言而不是两种。它是强类型的，具有显式类型。已经有良好的验证机制。它专门设计用来完成我们要求 YAML 做的工作，而且阅读起来并不难得多。它具有人们已经在使用的内置函数，这将允许我们从 YAML 工作流中移除一些第三方工具。

我敢打赌，今天 30% 的 Kubernetes 集群_已经_通过 Terraform 用 HCL 进行管理。我们不需要 Terraform 部分来获得优秀配置语言的很多好处。

唯一的缺点是 HCL 比 YAML 稍微冗长一些，而且它的 Mozilla Public License 2.0 (MPL-2.0) 需要仔细的法律审查才能集成到像 Kubernetes 这样的 Apache 2.0 项目中。然而，考虑到它带来的使用体验改进，这些缺点都是可以接受的。

**为什么 HCL 更好**

让我们看一个简单的 YAML 文件。

```yaml
# YAML 没有强制类型
replicas: "3"  # 字符串而不是整数
resources:
  limits:
    memory: 512  # 缺少单位后缀
  requests:
    cpu: 0.5m    # CPU 单位错误（应该是 500m）
```

即使在最基本的例子中，到处都是陷阱。HCL 和类型系统会捕获所有这些问题。

```yaml
replicas = 3  # 明确指定为整数

resources {
  limits {
    memory = "512Mi"  # 内存值使用字符串
  }
  requests {
    cpu = 0.5  # CPU 值使用数字
  }
}
```

拿一个像这样的 YAML 文件，你的 k8s 仓库中可能有 6000 个这样的文件。现在看看不需要外部工具的 HCL。

```
# 需要外部工具或模板来处理动态值

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # 无法轻松生成或转换值
  DATABASE_URL: "postgres://user:password@db:5432/mydb"
  API_KEY: "static-key-value"
  TIMESTAMP: "2023-06-18T00:00:00Z"  # Hard-coded timestamp
```

```yaml
resource "kubernetes_config_map" "app_config" {
  metadata {
    name = "app-config"
  }
  
  data = {
    DATABASE_URL = "postgres://${var.db_user}:${var.db_password}@${var.db_host}:${var.db_port}/${var.db_name}"
    API_KEY      = var.api_key != "" ? var.api_key : random_string.api_key.result
    TIMESTAMP    = timestamp()
  }
}

resource "random_string" "api_key" {
  length  = 32
  special = false
}
```

这是这个改变带来的所有好处。

1.  **类型安全**：在部署前防止类型相关错误
2.  **变量和引用**：减少重复并提高可维护性
3.  **函数和表达式**：启用动态配置生成
4.  **条件逻辑**：支持特定环境的配置
5.  **循环和迭代**：简化重复配置
6.  **更好的注释**：改进文档和可读性
7.  **错误处理**：使错误更容易识别和修复
8.  **模块化**：启用配置组件的重用
9.  **验证**：防止无效配置
10.  **数据转换**：支持复杂的数据操作  
    

### 允许 etcd 替换

我知道，我是第 10,000 个写这个的人。Etcd 做得很好，但它是唯一适合这个工作的工具，这有点疯狂。对于较小的集群或较小的硬件配置，它在集群类型中占用了大量资源，而你永远不会达到让它值得的节点数量。现在 k8s 和 etcd 之间的关系也很奇怪，k8s 基本上是剩下的唯一 etcd 客户。

我建议的是将 [kine][5] 的工作正式化。为了项目的长期健康，能够插入更多后端是有意义的，添加这种抽象意味着（应该）更容易在未来换入新的/不同的后端，并且还允许根据我部署的硬件进行更具体的调整。

我怀疑这最终会看起来像这样：[https://github.com/canonical/k8s-dqlite][6]。具有 Raft 共识的分布式 SQlite 内存数据库，几乎不需要升级工作，这将允许集群操作员对其 k8s 安装的持久层有更多的灵活性。如果你在数据中心有传统的服务器设置，etcd 资源使用不是问题，那很好！但这允许低端 k8s 有更好的体验，并且（希望）减少对 etcd 项目的依赖。

### 超越 Helm：原生包管理器

Helm 是一个临时性变通方法的完美例子，这种变通方法已经成长为永久依赖。我感谢 Helm 的维护者的所有辛勤工作，将最初的黑客马拉松项目发展成为向 k8s 集群安装软件的事实方式。在没有与 k8s 更深入集成的情况下，它已经尽可能地履行了这一角色。

尽管如此，Helm 使用起来是一场噩梦。Go 模板很难调试，通常包含复杂的逻辑，导致真正令人困惑的错误场景。你从这些场景中得到的错误消息通常是胡言乱语。Helm 不是一个很好的包系统，因为它在包系统需要做的一些基本任务上失败了，这些任务是传递依赖项和解决依赖项之间的冲突。

**我是什么意思？**

告诉我这个条件逻辑试图做什么：

```yaml
# Helm 中复杂条件逻辑的真实示例
{{- if or (and .Values.rbac.create .Values.serviceAccount.create) (and .Values.rbac.create (not .Values.serviceAccount.create) .Values.serviceAccount.name) }}
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: {{ template "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
{{- end }}
```

或者如果我向我的 chart 提供多个值文件，哪一个会获胜：

```bash
helm install myapp ./mychart -f values-dev.yaml -f values-override.yaml --set service.type=NodePort
```

好的，如果我想用 Helm chart 管理我的应用程序和所有应用程序依赖项。这是有道理的，我有一个应用程序本身依赖于其他东西，所以我想把它们放在一起。所以我在 Chart.yaml 中定义我的子图表或伞形图表。  

```yaml
dependencies:
- name: nginx
  version: "1.2.3"
  repository: "<https://example.com/charts>"
- name: memcached
  version: "1.2.3"
  repository: "<https://another.example.com/charts>"
```

但假设我有多个应用程序，完全有可能我有 2 个服务都依赖于 nginx 或其他类似的东西：  

![](https://matduggan.com/content/images/2025/06/image-2.png)

Helm 不能优雅地处理这种情况，因为模板名称是全局的，模板按字母顺序加载。基本上你需要：

-   不要多次声明对同一个图表的依赖（对于很多微服务来说很难做到）
-   如果你确实多次声明了同一个图表，必须使用完全相同的版本

问题列表不胜枚举。

-   跨命名空间安装很糟糕
-   Chart 验证过程很痛苦，没有人使用它

让我们直接访问 artifacthub 的首页：

![](https://matduggan.com/content/images/2025/06/image-3.png)

我选择 elasticsearch，因为它看起来很重要。

![](https://matduggan.com/content/images/2025/06/image-4.png)

![](https://matduggan.com/content/images/2025/06/image-5.png)

对于官方 Elastic helm chart 来说，似乎_非常糟糕_。当然 `ingress-nginx` 会是正确的，它是整个行业的绝对关键依赖。

![](https://matduggan.com/content/images/2025/06/image-6.png)

不。另外，chart 的维护者如何是"Kubernetes"，并且_仍然_没有被标记为`verified publisher`。天啊，还要多么经过验证才行。

-   Chart 搜索中没有元数据。你只能按名称和描述搜索，而不能按功能、能力或其他元数据搜索。

![](https://matduggan.com/content/images/2025/06/image-7.png)

-   Helm 不严格执行语义版本控制

```
# 具有非语义版本的 Chart.yaml

```yaml
apiVersion: v2
name: myapp
version: "v1.2-alpha" 
```

-   如果你卸载并重新安装带有 CRD 的 chart，它可能会删除这些 CRD 创建的资源。这个已经_多次_搞砸了我，而且非常不安全。

我可以再写 5000 个字，仍然无法概述所有问题。没有办法让 Helm 足够好地完成"地球上所有关键基础设施的包管理器"的任务。

#### k8s 包系统会是什么样子？

让我们称我们假设的包系统为 KubePkg，因为如果有一件事是 Kubernetes 生态系统需要的，那就是另一个带有'K'的缩写名称。我们将尝试复制 Linux 生态系统中的现有工作，同时利用 k8s 的 CRD 能力。我的想法看起来像这样：

![](https://matduggan.com/content/images/2025/06/image-8.png)

包是像 Linux 包一样的捆绑包：  

![](https://matduggan.com/content/images/2025/06/image-9.png)

有一个定义文件，它涵盖了安装东西时实际遇到的许多真实场景。

```yaml
apiVersion: kubepkg.io/v1
kind: Package
metadata:
  name: postgresql
  version: 14.5.2
spec:
  maintainer:
    name: "PostgreSQL Team"
    email: "[email protected]"
  description: "PostgreSQL database server"
  website: "https://postgresql.org"
  license: "PostgreSQL"
  
  # 具有语义版本控制的依赖项
  dependencies:
    - name: storage-provisioner
      versionConstraint: ">=1.0.0"
    - name: metrics-collector
      versionConstraint: "^2.0.0"
      optional: true
  
  # 安全上下文和要求
  security:
    requiredCapabilities: ["CHOWN", "SETGID", "SETUID"]
    securityContextConstraints:
      runAsUser: 999
      fsGroup: 999
    networkPolicies:
      - ports:
        - port: 5432
          protocol: TCP
    
  # 要创建的资源（嵌入或引用）
  resources:
    - apiVersion: v1
      kind: Service
      metadata:
        name: postgresql
      spec:
        ports:
        - port: 5432
    - apiVersion: apps/v1
      kind: StatefulSet
      metadata:
        name: postgresql
      spec:
        # StatefulSet definition
  
  # 使用 JSON Schema 的配置架构
  configurationSchema:
    type: object
    properties:
      replicas:
        type: integer
        minimum: 1
        default: 1
      persistence:
        type: object
        properties:
          size:
            type: string
            pattern: "^[0-9]+[GMK]i$"
            default: "10Gi"
  
  # 具有正确排序的生命周期钩子
  hooks:
    preInstall:
      - name: database-prerequisites
        job:
          spec:
            template:
              spec:
                containers:
                - name: init
                  image: postgres:14.5
    postInstall:
      - name: database-init
        job:
          spec:
            # Job definition
    preUpgrade:
      - name: backup
        job:
          spec:
            # Backup job definition
    postUpgrade:
      - name: verify
        job:
          spec:
            # Verification job definition
    preRemove:
      - name: final-backup
        job:
          spec:
            # Final backup job definition
  
  # 有状态应用程序的状态管理
  stateManagement:
    backupStrategy:
      type: "snapshot"  # or "dump"
      schedule: "0 2 * * *"  # Daily at 2 AM
      retention:
        count: 7
    recoveryStrategy:
      type: "pointInTime"
      verificationJob:
        spec:
          # Job to verify recovery success
    dataLocations:
      - path: "/var/lib/postgresql/data"
        volumeMount: "data"
    upgradeStrategies:
      - fromVersion: "*"
        toVersion: "*"
        strategy: "backup-restore"
      - fromVersion: "14.*.*"
        toVersion: "14.*.*"
        strategy: "in-place"
```

有一个真正的签名过程是必需的，并允许你对过程有更多的控制。  

```yaml
apiVersion: kubepkg.io/v1
kind: Repository
metadata:
  name: official-repo
spec:
  url: "https://repo.kubepkg.io/official"
  type: "OCI"  # or "HTTP"
  
  # 验证设置
  verification:
    publicKeys:
      - name: "KubePkg Official"
        keyData: |
          -----BEGIN PUBLIC KEY-----
          MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAvF4+...
          -----END PUBLIC KEY-----
    trustPolicy:
      type: "AllowList"  # or "KeyRing"
      allowedSigners:
        - "KubePkg Official"
        - "Trusted Partner"
    verificationLevel: "Strict"  # or "Warn", "None"
```

拥有一个我可以自动更新包而不需要在我这边做任何事情的东西，那该有多好啊。

```yaml
apiVersion: kubepkg.io/v1
kind: Installation
metadata:
  name: postgresql-main
  namespace: database
spec:
  packageRef:
    name: postgresql
    version: "14.5.2"
  
  # 配置值（根据架构验证）
  configuration:
    replicas: 3
    persistence:
      size: "100Gi"
    resources:
      limits:
        memory: "4Gi"
        cpu: "2"
  
  # 更新策略
  updatePolicy:
    automatic: false
    allowedVersions: "14.x.x"
    schedule: "0 2 * * 0"  # Weekly on Sunday at 2am
    approvalRequired: true
  
  # 状态管理引用
  stateRef:
    name: postgresql-main-state
    
  # 要使用的服务账户
  serviceAccountName: postgresql-installer
```

K8s 需要的是一个满足以下要求的系统：

1.  **真正的 Kubernetes 原生**：一切都是具有适当状态和事件的 Kubernetes 资源
2.  **一流的状态管理**：对有状态应用程序的内置支持
3.  **增强的安全性**：强大的签名、验证和安全扫描
4.  **声明式配置**：没有模板，只有带有架构的结构化配置
5.  **生命周期管理**：全面的生命周期钩子和升级策略
6.  **依赖解析**：类似 Linux 的依赖管理与语义版本控制
7.  **审计跟踪**：完整的更改历史记录，包括谁、什么和何时，而不是 Helm 当前提供的。
8.  **策略执行**：对组织策略和合规性的支持。
9.  **简化的用户体验**：熟悉的类似 Linux 的包管理命令。我们试图与已经工作了几十年的包系统走不同的方向，这似乎很疯狂。  
      
    

### 默认使用 IPv6

试着想象一下，在全球范围内，有多少时间和精力投入到试图解决以下三个问题中的任何一个。

1.  我需要这个集群中的这个 pod 与那个集群中的那个 pod 通信。
2.  NAT 遍历过程中某处出现问题，我需要解决它
3.  我的集群 IP 地址用完了，因为我没有考虑到你会使用多少个。记住：一个公司从 /20 子网（4,096 个地址）开始，部署 40 个节点，每个节点 30 个 pod，突然意识到他们正在接近 IP 限制。节点并不多！

我不是建议整个互联网切换到 IPv6，现在 k8s 如果你想要的话，很乐意支持仅 IPv6 和双栈方法。但我是说现在是时候切换默认值并直接使用 IPv6。你可以一次性消除大量问题集合。

-   集群内部更平坦、更简单的网络拓扑。
-   多个集群之间的区别成为组织可以选择忽略的事情，如果他们想要获得公共 IP。
-   更容易准确理解堆栈内部的流量流动。
-   内置 IPSec

这与推动全球采用 IPv6 无关，只是承认我们不再生活在一个你必须接受 IPv4 奇怪限制的世界中，在这个世界中你可能突然需要 10,000 个 IP 地址而几乎没有警告。

拥有公共 IPv6 地址的组织的好处是显而易见的，但对于云提供商和用户来说，有足够的价值，甚至公司高管可能会支持它。AWS 永远不需要尝试在 VPC 内部寻找更多的私有 IPv4 空间。这肯定值得一些东西。

### 结论

对这些想法的常见反驳是，"Kubernetes 是一个开放平台，所以社区可以构建这些解决方案。"虽然这是真的，但这个论点忽略了一个关键点：**默认是技术中最强大的力量。**核心项目定义的"快乐路径"决定了 90% 的用户将如何与之交互。如果系统默认期望签名包并提供强大、原生的管理方式，那么生态系统将采用这种方式。

我知道这是一个雄心勃勃的清单。但如果我们做梦，那就做大梦。毕竟，我们是那个认为命名技术为'Kubernetes'会流行起来的行业，而且不知何故它确实流行起来了！

我们在其他领域经常看到这种情况，比如移动开发和 Web 开发，平台评估其情况并做出_激进_的前进。这些不一定是维护者或公司_会_承担的项目，但我认为它们都是_某人_应该至少重新审视并思考"既然我们现在占全球数据中心运营的相当大比例，这样做是否值得"的想法。

问题/反馈/有什么错误？在这里找到我：[https://c.im/@matdevdug][8]

## 保持更新

[订阅 RSS 源以获取新文章发送到您的阅读器。][9]

© 2025 matduggan.com. 版权所有。

[1]: #main-content
[2]: https://matduggan.com
[3]: https://matduggan.com/rss/
[4]: https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell
[5]: https://github.com/k3s-io/kine
[6]: https://github.com/canonical/k8s-dqlite
[7]: /cdn-cgi/l/email-protection
[8]: https://c.im/@matdevdug
[9]: https://matduggan.com/rss/