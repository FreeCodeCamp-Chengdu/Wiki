---
title: "Claude Code：代理式编码的最佳实践"
date: 2025-04-18T00:00:00.000Z
authorURL: ""
originalURL: https://www.anthropic.com/engineering/claude-code-best-practices
translator: ""
reviewer: ""
---


<img src="https://www-cdn.anthropic.com/images/4zrzovbb/website/6295100fcf8952bed666ba69536c581af87eef15-2554x2554.svg" alt="Claude Code 标志拼图" width="360">

# Claude Code：代理式编码的最佳实践

Claude Code 是一款面向代理式编码（agentic coding）的命令行工具。本文整理了在不同代码库、语言与环境中实践 Claude Code 时总结出的技巧与经验。

我们最近刚刚 [发布了 Claude Code][9]。它起源于一个研究项目，旨在帮助 Anthropic 的工程师与研究人员把 Claude 更自然地融入日常开发流程。

Claude Code 刻意保持低抽象、少约束，尽可能提供接近原生模型的操作能力，而不强制某种工作流。这让它成为一款灵活、可定制、可脚本化且安全的强力工具。不过强大的自由度也带来学习曲线——尤其对新接触代理式编码工具的工程师而言，在形成自己的最佳实践之前需要一段探索期。

本文总结了我们在内部团队与外部用户中观察到的高效模式。它们不是铁律，而是可以借鉴的起点。欢迎你大胆尝试、持续迭代，找到最适合自己的方法！

_想了解更详细的信息？请查阅 [claude.ai/code][10] 上的完整文档，其中涵盖本文提到的所有功能，并提供更多示例、实现细节与进阶技巧。_

## 1. 自定义你的工作环境

Claude Code 会在生成提示时自动收集上下文。虽然有用，却也会消耗时间和 token。通过调整工作环境，你可以让这个过程更高效。

### a. 创建 `CLAUDE.md`

`CLAUDE.md` 是一个特殊文件，只要你开启会话 Claude 就会自动读取，非常适合记录：

-   常用 bash 命令
-   核心文件与工具函数
-   代码风格指南
-   测试步骤
-   仓库约定（如分支命名、合并 vs. rebase 等）
-   开发环境配置（例如是否使用 pyenv、可用编译器）
-   项目特有的异常或注意事项
-   其他希望 Claude 长期记住的信息

`CLAUDE.md` 没有固定格式，建议保持简洁、易读，例如：

```plain
# Bash 命令
- npm run build：构建项目
- npm run typecheck：运行类型检查

# 代码风格
- 使用 ES Modules（import/export），不要 CommonJS（require）
- 能解构导入时尽量写成 import { foo } from 'bar'

# 工作流
- 一轮改动完成后务必运行类型检查
- 优先运行单个测试用例而非整套测试，以兼顾性能
```

`CLAUDE.md` 可以放在多个地方：

-   **仓库根目录**（最常见），命名为 `CLAUDE.md` 并提交到 git，以便跨会话、跨团队共享；或命名为 `CLAUDE.local.md` 并加入 `.gitignore`
-   **运行 `claude` 的父目录**，适合多模块仓库。例如你在 `root/foo` 下运行 `claude`，可以在 `root/CLAUDE.md` 与 `root/foo/CLAUDE.md` 各放一份内容，Claude 会自动合并
-   **运行目录的子目录**，当 Claude 处理子目录中的文件时，会按需加载对应的 `CLAUDE.md`
-   **用户主目录**（`~/.claude/CLAUDE.md`），对所有会话生效

运行 `/init` 命令时，Claude 还会自动为你生成一份 `CLAUDE.md`。

### b. 打磨 `CLAUDE.md`

`CLAUDE.md` 会成为 Claude 提示的一部分，应像常用提示词一样反复调优。一个常见误区是不断堆砌内容却不验证效果。花点时间做实验，找出最能让模型听话的写法。

你可以手动更新 `CLAUDE.md`，也可以在会话中按 `#` 键，让 Claude 自动把指令写入对应的文件。很多工程师会在编码时频繁使用 `#` 记录命令、文件、风格指南，然后将这些变更一并提交，方便团队共享。

在 Anthropic，我们偶尔会把 `CLAUDE.md` 丢进 [prompt improver][11]，并通过加入 “IMPORTANT” 或 “YOU MUST” 等强调语提升模型遵循度。

![Claude Code 工具白名单示意图](https://www.anthropic.com//_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F6961243cc6409e41ba93895faded4f4bc1772366-1600x1231.png&w=3840&q=75)

### c. 维护 Claude 的工具白名单

默认情况下，只要操作可能修改系统（写文件、执行大量 bash 命令、调用 MCP 工具等），Claude Code 就会先征求许可。这种保守策略是为了安全。你可以把可信工具加入白名单，或允许那些容易撤销的潜在风险操作（如编辑文件、`git commit`）。

管理白名单有四种方式：

-   **在提示时选择 “Always allow”**
-   **启动会话后运行 `/permissions`** 添加或移除条目，例如把 `Edit`、`Bash(git commit:*)`、`mcp__puppeteer__puppeteer_navigate` 加入允许列表
-   **手动编辑 `.claude/settings.json` 或 `~/.claude.json`**（推荐将前者纳入版本控制，便于团队共享）
-   **通过 `--allowedTools` 参数** 为单次会话指定权限

### d. 如果连接 GitHub，请安装 gh CLI

Claude 能使用 `gh` CLI 创建 Issue、打开 PR、读取评论等。即使没有安装 `gh`，它也能回退到 GitHub API 或 MCP server（若已配置）。

## 2. 为 Claude 提供更多工具

Claude 可以直接访问你的 shell 环境，你完全可以像配置自己那样给它准备脚本和函数；它也能通过 MCP、REST API 使用更复杂的工具。

### a. 配合 bash 工具

Claude Code 会继承你的 bash 环境，自然能使用所有工具。虽然 Claude 对常见的 Unix 命令和 `gh` 很熟，但对你自制的脚本并不了解，需要你提前说明：

1.  告诉 Claude 工具名称并附上使用示例
2.  提醒它运行 `--help` 查看自带文档
3.  在 `CLAUDE.md` 中记录常用工具，方便随时参考

### b. 结合 MCP

Claude Code 既是 MCP server，也是 MCP client。作为客户端时，可以通过三种方式连接任意数量的 MCP server：

-   **项目级配置**：仅在该目录运行 Claude Code 时可用
-   **全局配置**：在所有项目中可用
-   **仓库内的 `.mcp.json`**：任何拉下代码的人都能即开即用。例如把 Puppeteer、Sentry server 写进 `.mcp.json`，团队成员直接获得同样的工具集

排查 MCP 配置问题时，建议用 `--mcp-debug` 启动 Claude 以获取更多日志。

### c. 自定义斜杠命令

对于需要重复执行的流程（调试循环、日志分析等），可以把提示模板放在 `.claude/commands` 目录下的 Markdown 文件里。输入 `/` 时即可从菜单中选择这些自定义命令，把它们提交到 git 后团队也能共享。

自定义命令可以使用特殊关键字 `$ARGUMENTS`，用于接收命令调用时的参数。

例如，下面这个命令可以自动拉取并修复 GitHub Issue：

```plain
请分析并修复编号为 $ARGUMENTS 的 GitHub Issue。

执行步骤：

1. 使用 `gh issue view` 获取 Issue 详情
2. 理解其中描述的问题
3. 在代码库中搜索相关文件
4. 实施必要的修改以修复问题
5. 编写并运行测试验证修复
6. 确保通过所有 lint 与类型检查
7. 撰写清晰的提交信息
8. 推送并创建 PR

所有与 GitHub 相关的操作都请使用 GitHub CLI (`gh`)。
```

把上述内容保存到 `.claude/commands/fix-github-issue.md`，它就会以 `/project:fix-github-issue` 命令出现。例如运行 `/project:fix-github-issue 1234`，Claude 就会尝试修复第 1234 号 Issue。同样，你也可以把个人常用命令放进 `~/.claude/commands` 中，让它们在所有会话里都可用。

## 3. 试试这些常见工作流

Claude Code 不会强制任何工作流，你可以自由组合。但在这种自由度之下，我们也总结了几种实践中最成功的模式，值得当作默认模板。

### a. 探索 → 规划 → 编码 → 提交

适用场景广泛：

1.  **先让 Claude 阅读相关文件、图片或 URL**。可以给它模糊提示（“看看负责日志的文件”）或直接给文件名（“读取 logging.py”），但务必强调“暂时不要写代码”。
    1.  尤其在复杂问题上，鼓励它使用子代理（subagents）去验证细节或挖掘疑问，既能保留主上下文，也能保持效率。
2.  **请 Claude 制定解决方案**。用 “think / think hard / think harder / ultrathink” 等关键词能触发更长的思考时间，逐级增加推理预算。
    1.  如果方案可行，可以让 Claude 生成一个文档或 GitHub Issue，把计划记录下来，方便实现阶段出现偏差时回滚。
3.  **再让 Claude 实现方案**，同时提醒它实现过程中自查是否偏离计划。
4.  **最后让 Claude 提交并创建 PR**，必要时更新 README、changelog 等文档。

前两步尤为重要——跳过它们，Claude 往往直接动手写代码。对需要深度思考的问题，先调研再规划会好得多。

### b. 测试先行，迭代直到通过

当需求可以通过单元、集成或端到端测试验证时，这套流程非常高效，也是 Anthropic 工程师的常见选择：

1.  **让 Claude 根据预期输入/输出编写测试**。明确说明正在做 TDD，避免它为了让测试通过而提前写实现，即便该功能尚不存在。
2.  **要求它运行测试并确认失败**。强调“此阶段不要写实现”非常关键。
3.  **满意后提交测试**。
4.  **再让 Claude 编写通过测试的实现**，并强调不得修改测试。提示它“持续迭代直到全部通过”。通常需要几轮：写代码、跑测试、调整、再跑。
    1.  可让独立子代理验证实现是否对测试过拟合。
5.  **确认满意后提交实现代码**。

给 Claude 一个明确目标（比如测试、视觉稿等），它就能有的放矢地迭代。

### c. 编码 → 截图 → 对比 → 迭代

和 TDD 类似，但目标换成视觉稿：

1.  **让 Claude 拥有截图能力**，例如使用 [Puppeteer MCP server][12]、[iOS simulator MCP server][13]，或手动复制/粘贴截图。
2.  **提供设计稿**，可粘贴、拖拽或直接提供图片路径。
3.  **让 Claude 实现设计并截图回看**，根据对比不断迭代。
4.  **满意后提交**。

和人类一样，Claude 在迭代几次后通常会有更好结果。确保它能看见自己的输出。

![Safe yolo mode](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F6ea59a36fe82c2b300bceaf3b880a4b4852c552d-1600x1143.png&w=3840&q=75)

### d. Safe YOLO 模式

如果不想每一步都审批，可以使用 `claude --dangerously-skip-permissions`，跳过所有权限确认，让 Claude 不受打扰地工作，适合修复 lint 或生成样板代码等任务。

但请警惕风险——一旦放权，Claude 可能执行任意命令，甚至因提示注入导致数据泄露。最安全的做法是在无网络的容器里运行，可参考 [Docker Dev Container 示例][14]。

### e. 代码库问答

刚接手新项目时，把 Claude 当作结对伙伴。任何你会问同事的问题，也都可以问 Claude，它会主动搜索代码库来回答：

-   日志系统如何工作？
-   如何新增一个 API endpoint？
-   `foo.rs` 第 134 行的 `async move { ... }` 是干什么的？
-   `CustomerOnboardingFlowImpl` 覆盖了哪些边界情况？
-   为什么 333 行调用 `foo()` 而不是 `bar()`？
-   `baz.py` 第 334 行在 Java 里的等价写法是什么？

在 Anthropic，这几乎成了新人入职的标准流程，大幅缩短适应期，也减轻了其他工程师的负担。无需特殊提示，直接发问即可。

![Use Claude to interact with git](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fa08ea13c2359aac0eceacebf2e15f81e8e8ec8d2-1600x1278.png&w=3840&q=75)

### f. 让 Claude 操控 git

Claude 能高效处理大量 git 操作。许多 Anthropic 工程师已经把 90% 以上的 git 交互交给它：

-   **搜索 git 历史**：例如 “v1.2.3 包含了哪些变更？”、“谁负责这个功能？”、“这个 API 为什么这样设计？”——提示 Claude 去翻历史记录即可。
-   **撰写提交信息**：它会自动查看当前 diff 和近期历史，写出有上下文的 commit message。
-   **处理复杂 git 操作**：如回滚文件、解决 rebase 冲突、比较/拼接补丁等。

### g. 让 Claude 接管 GitHub 任务

Claude Code 也能处理多种 GitHub 交互：

-   **创建 PR**：它认识 “pr” 这种简写，会根据 diff 与上下文生成合适的提交信息。
-   **一键解决简单代码评审评论**：告诉它“修复 PR 上的评论”即可，必要时附加说明，完成后会推送到 PR 分支。
-   **修复失败的构建或 lint 警告**。
-   **批量整理 Issue**：例如让 Claude 循环遍历所有 open Issue 进行归类或打标签。

这样无需死记 `gh` 命令就能自动化日常工作。

### h. 协作编辑 Jupyter Notebook

Anthropic 的研究员和数据科学家常用 Claude Code 来读写 Jupyter Notebook。Claude 能解读 Notebook 的输出（包括图像），非常适合快速探索与互动。推荐在 VS Code 中同时打开 Claude Code 和 `.ipynb` 文件，方便来回切换。

在把 Notebook 发给同事前，也可以让 Claude “美化” 一下，尤其是强调“让图表更好看”，通常能获得更适合演示的结果。


## 5. 使用无界面模式自动化基础设施

Claude Code 提供 [headless mode（无界面模式）][15]，可在 CI、pre-commit 钩子、构建脚本等非交互场景中运行。使用 `-p` 传入提示即可启用，并可结合 `--output-format stream-json` 获取流式 JSON 输出。

请注意：无界面模式不会在会话之间保留状态，每次都需重新启动。

### a. 用 Claude 归类 Issue

无界面模式可以与 GitHub 事件配合，例如每当仓库新建 Issue 就触发自动化。公开的 [Claude Code 仓库][16] 就利用这一模式在 Issue 创建时自动阅读并打标签。

### b. 当作“主观 linter”

Claude Code 能提供超越传统 lint 的 [主观代码审查][17]，捕捉错别字、陈旧注释、误导性的函数或变量名等问题。

## 6. 多开 Claude，进一步提效

除了单实例使用，最强大的方式之一是并行运行多个 Claude。

### a. 一台写代码，另一台做验证

一种简单有效的分工是让一个 Claude 写代码，另一个负责评审或测试，类似多人协作：

1.  让第一个 Claude 写代码；
2.  运行 `/clear` 或在另一终端再开一个 Claude；
3.  让第二个 Claude 审查前者的工作；
4.  视情况再开第三个 Claude（或再次 `/clear`），阅读代码与审查意见；
5.  按反馈修改代码。

同样的思路也适用于测试：一个写测试，另一个写实现。你甚至可以让多个 Claude 通过不同的草稿文件互相留言。

### b. 多个仓库副本

与其等待 Claude 完成一个阶段再切换，许多 Anthropic 工程师更喜欢开多个 git 副本并行推进：

1.  **在不同目录创建 3～4 个仓库副本**
2.  **分别在不同终端标签页打开**
3.  **在每个副本里启动 Claude 并分配任务**
4.  **轮询各终端以审批权限或查看进度**

### c. 使用 git worktree

git worktree 是更轻量的多副本方案，适合互不影响的任务。每个 worktree 有独立工作目录，但共享 Git 历史：

1.  **创建 worktree**：`git worktree add ../project-feature-a feature-a`
2.  **在该 worktree 中启动 Claude**：`cd ../project-feature-a && claude`
3.  **需要更多 worktree 时重复上述步骤**

实用技巧：

-   统一命名规则
-   每个 worktree 对应一个终端标签页
-   macOS 的 iTerm2 用户可以[设置通知][18]，当 Claude 需要你时会提醒
-   不同 worktree 使用不同 IDE 窗口
-   任务完成后运行 `git worktree remove ...` 清理

### d. 搭配自定义脚本运行无界面模式

`claude -p`（无界面模式）可以把 Claude Code 整合进自动化流程，同时沿用所有内置工具与系统提示。常见模式有两种：

1. **扇出式批量处理**：适合大规模迁移或分析（例如处理上百份日志或上千个 CSV）。

    1.  让 Claude 写脚本生成任务列表，例如需要从框架 A 迁到框架 B 的 2000 个文件。
    2.  遍历任务列表，对每项调用 `claude -p "<任务>" --allowedTools ...`，如 `claude -p "migrate foo.py from React to Vue. When finished you MUST output OK if successful, otherwise FAIL." --allowedTools Edit Bash(git commit:*)`
    3.  多跑几轮，根据结果微调提示词。

2. **流水线集成**：把 Claude 嵌入既有数据/处理管线。

    1.  执行 `claude -p "<提示>" --json | your_command`，后续命令可以直接消费 JSON（可选）。
    2.  就这么简单！结构化输出有助于自动化解析。

这两种场景下，`--verbose` 对调试非常有用，但生产环境建议关闭以保持输出干净。

你还有哪些 Claude Code 使用技巧？欢迎在社交平台 @AnthropicAI，分享你的作品！

## 致谢

本文由 Boris Cherny 撰写，内容汇集了 Claude Code 用户社区的最佳实践，他们的创意与流程持续激励着我们。特别感谢 Daisy Hollman、Ashwin Bhat、Cat Wu、Sid Bidasaria、Cal Rueb、Nodir Turakulov、Barry Zhang、Drew Hodun 以及许多其他 Anthropic 工程师，是他们的经验让这些建议更加务实可行。
<img src="https://www-cdn.anthropic.com/images/4zrzovbb/website/43abe7e54b56a891e74a8542944dfbd33f07f49c-1000x1000.svg" alt="Interlocking puzzle piece with complex geometric shape and detailed surface texture" width="360">

[想了解更多？欢迎查看相关课程。][19]

## 4. 优化工作流

以下建议适用于所有场景：

### a. 指令要具体

指令越明确，Claude Code 首次成功的概率越高。开局说明白，后面就少绕路。

示例：

| 不够好                                           | 更好的示例                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| add tests for foo.py                             | 针对 foo.py 再写一个测试用例，覆盖“用户已登出”的边界情况，并且不要使用 mock。                                                                                                                                                                                                                                                                                                       |
| why does ExecutionFactory have such a weird api? | 查阅 ExecutionFactory 的 git 历史，概括它的 API 是如何逐步演化成现在这样的。                                                                                                                                                                                                                                                                                                       |
| add a calendar widget                            | 先看看首页现有的挂件实现，理解模式以及代码与接口如何拆分。HotDogWidget.php 是一个好例子。按照该模式实现一个新的日历挂件，让用户能选择月份并前后翻页切换年份，且只使用当前代码库已有的依赖，从零开始实现。                                                                                                                                                                             |

Claude 能理解意图，但无法读心。明确的指令才能让结果符合预期。

![给 Claude 图片](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F75e1b57a0b696e7aafeca1ed5fa6ba7c601a5953-1360x1126.png&w=3840&q=75)

### b. 提供视觉参考

Claude 在处理图片和图表时很擅长，提供方式包括：

-   **粘贴截图**（macOS 可用 `cmd+ctrl+shift+4` 截到剪贴板，再用 `ctrl+v` 粘贴。注意不是 `cmd+v`，远程环境无效）
-   **拖拽图片** 到输入框
-   **提供图片路径**

这在依据设计稿实现 UI 或分析可视化图表时尤为重要。即便没有图像，也可以直接告诉 Claude “结果要好看”，帮助它把握方向。

![指明要查看的文件](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F7372868757dd17b6f2d3fef98d499d7991d89800-1450x1164.png&w=3840&q=75)

### c. 点名具体文件

用 Tab 补全可以快速引用仓库中的文件或目录，帮助 Claude 精准找到要查看或修改的内容。

![提供 URL](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fe071de707f209bbaa7f16b593cc7ed0739875dae-1306x1088.png&w=3840&q=75)

### d. 附上 URL

在提示中粘贴具体链接，让 Claude 自行抓取阅读。对于常访问的域名（如 docs.foo.com），可通过 `/permissions` 提前添加到允许列表，避免重复确认。

### e. 及时纠偏

虽然自动接受模式（Shift+Tab）能让 Claude 自主工作，但大多数情况下你作为积极合作者效果更好。除了开场解释清楚任务，也可以随时纠偏。

常用的 4 种方式：

-   **先让 Claude 制定计划**，并声明“计划确认前不要写代码”
-   **在任何阶段按 Esc 中断**（无论思考、调用工具或编辑文件），保留上下文后重新指示
-   **双击 Esc 回到历史消息**，编辑上一步提示并换个方向再试
-   **让 Claude 撤销更改**，常与第二步配合使用

即便 Claude 偶尔“一次命中”，善用这些工具通常能更快得到满意答案。

### f. 用 `/clear` 保持上下文干净

会话时间长了，Claude 的上下文会塞进一堆无关内容，既占空间又干扰思路。任务切换时记得借助 `/clear` 重置。

### g. 大任务使用清单和草稿

对于多步骤或需要穷举的任务（大规模迁移、批量修复 lint、复杂构建脚本等），让 Claude 在 Markdown 文件或 GitHub Issue 中维护清单与草稿能大大提升效率。

例如批量修复 lint 错误：

1.  **让 Claude 运行 lint 命令**，把所有错误（含文件名和行号）写成 Markdown 清单
2.  **让它逐条处理并勾选**，修一个、验一个，再进行下一个

### h. 把数据喂给 Claude

常见方式包括：

-   **复制粘贴** 到提示中（最常用）
-   **管道输入**（如 `cat foo.txt | claude`），适合日志、CSV、海量数据
-   **让 Claude 自取**（bash 命令、MCP 工具、自定义斜杠命令等）
-   **让 Claude 读取文件或 URL**（同样支持图片）

实际上往往会组合使用。比如先把日志通过管道传进去，再让 Claude 用额外工具补充上下文以定位问题。
