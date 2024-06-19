---
title: 对 Python 对环境管理和包管理工具的一次公正评估
authors:
  - luojiyin1987
original: https://alpopkes.com/posts/python/packaging_tools/
date: 2023-08-24
updated: 2024-05-23 00:23:00
categories:
  - Article
  - Translation
toc: true
---

![作者头像][1]

Anna-Lena Popkes

## 动机

当我开始使用 Python 并创建我的第一个包时，我感到困惑。创建和管理一个包比我预期的要困难得多。此外，存在多种工具，我不确定该使用哪一个。我相信你们中的大多数人过去也遇到过同样的问题。Python 有大量的工具来管理虚拟环境和创建包，要理解哪一个最适合你的需求可能很难（或几乎不可能）。关于这个话题存在多个演讲和博客文章，但没有一个提供完整的概览或以结构化的方式评估这些工具。这就是本文的目的。我想给你一个真正无偏见的评价，关于现有的打包和环境管理工具。如果你宁愿观看一个演讲，可以看看 [PyCon DE 2023][2] 或 [EuroPython 2023][3] 的录像。

<!-- more -->

## 分类

为了撰写本文，我确定了环境和软件包管理的五大重要类别：

- 环境管理（主要涉及虚拟环境）
- 软件包管理
- Python 版本管理
- 软件包构建
- 软件包发布

正如你在下面的维恩图中所看到的，存在大量的工具。有些可以做一件事（即它们是单一用途的），而其他工具可以执行多个任务（因此我称它们为多用途工具）。

![工具分类示意图][4]

让我们从开发者的角度来逐一介绍这些类别。假设您除了工作项目之外，还在进行个人项目。在工作中，您使用的是 Python 3.7，而您的个人项目应该使用最新的 Python 版本（目前是 3.11）。换句话说：您希望能够安装不同的 Python 版本并在它们之间切换。这就是我们的第一个类别，**Python 版本管理**的意义所在。

在您的项目中，您会使用其他软件包（例如用于数据科学的 `pandas` 或 `sklearn`）。这些是您项目的依赖项，您必须安装和管理它们（例如，在新版本发布时进行升级）。这就是**软件包管理**的意义所在。

由于不同的项目可能需要相同软件包的不同版本，因此您需要创建（和管理）虚拟环境以避免依赖项冲突。用于此目的的工具被归类在**环境管理**类别中。大多数工具使用虚拟环境，但有些工具使用另一种称为**本地软件包**的概念，我们将在后面介绍。

一旦您的代码处于适当的状态，您可能希望与其他开发人员共享它。为此，您必须先构建您的软件包（**软件包构建**），然后才能将其发布到 PyPI 或其他索引（**软件包发布**）。

接下来，我们将更详细地了解每个类别，包括简短的定义、动机和可用工具。我将更详细地介绍一些单一用途的工具，并在最后的部分单独介绍几种多用途工具。让我们从第一个类别开始：Python 版本管理。

## Python 版本管理

### 定义

能够控制 Python 版本管理的工具，允许您安装 Python 版本并轻松地在它们之间切换。

### 动机

我们为什么要使用不同的 Python 版本？ 有几个原因。 例如，您可能正在处理多个项目，其中每个项目都需要不同的 Python 版本。 或者您可能正在开发一个支持多个 Python 版本的项目，并且您想要测试所有这些版本。 除此之外，查看最新的 Python 版本提供了什么，或者测试 Python 的预发布版本是否存在错误也是一件好事。

### 工具

我们的维恩图显示了可用于 Python 版本管理的工具：`pyenv`、`conda`、`rye` 和 `PyFlow`。我们将首先看看 `pyenv`，并在单独的部分中考虑多用途工具。

![版本管理工具对比图][5]

### PyEnv

Python 有一个单一用途的工具可以让您安装和管理 Python 版本：[pyenv](https://github.com/pyenv/pyenv)！ PyEnv 易于使用。 最重要的命令如下：

```bash
# Install specific Python version
pyenv install 3.10.4

# Switch between Python versions
pyenv shell <version> # select version just for current shell session
pyenv local <version> # automatically select version whenever you are in the current directory
pyenv global <version> # select version globally for your user account
```

## Virtual environment management（虚拟环境管理）

### 定义

环境管理工具可以创建和管理（虚拟）环境。

### 动机

我们为什么首先要使用环境？ 正如开头所提到的，项目有特定的要求（即它们依赖于其他包）。 通常情况下，不同的项目需要相同包的不同版本。 这会导致依赖冲突。 此外，使用 `pip install` 安装包时可能会出现问题，因为该包与您的系统范围内的 Python 安装放在一起。 其中一些问题可以通过在 `pip` 命令中使用 `--user` 标志来解决。 但是，这个选项可能不是每个人都知道，尤其是初学者。

### 工具

许多工具允许用户创建和管理环境。 它们是：`venv、virtualenv、pipenv、conda、pdm、poetry、hatch、rye` 和 `PyFlow`。 其中只有两个是单一用途的工具：`venv` 和 `virtualenv`。 让我们更详细地了解它们。

![管理环境工具对比图][6]

### VEnv

[VEnv][7] 是用于创建虚拟环境的内置 Python 包。 这意味着它随 Python 一起提供，用户无需安装。 最重要的命令如下：

```bash
# Create new environment
python3 -m venv <env_name>

# Activate an environment
. <env_name>/bin/activate

# Deactivate an active environment
deactivate
```

### VirtualEnv

[VirtualEnv][8] 试图改进 `venv`。 它提供了比 `venv` 更多的功能，并且速度更快、功能更强大。 最重要的命令与 `venv` 的命令类似，只是创建一个新环境更简洁：

```bash
# Create new environment
virtualenv <env_name>

# Activate an environment
. <env_name>/bin/activate

# Deactivate an active environment
deactivate
```

## 回顾 1 - `pyproject.toml`

在讨论打包之前，我想确保你了解打包最重要的文件：`pyproject.toml`。

Python 中的打包已经走过了漫长的道路。直到 [PEP 518][9] `setup.py` 文件被用于打包，使用 `setuptools` 作为构建工具。[PEP 518][9] 引入了 `pyproject.toml` 文件的使用。因此，在创建包时，你始终需要一个 `pyproject.toml` 文件。`pyproject.toml` 用于定义项目的设置、定义元数据以及许多其他内容。如果你想查看示例，请查看 [pandas 库的 `pyproject.toml` 文件][10]。了解了 `pyproject.toml` 后，我们可以继续看看包管理。

## 包管理

### 定义

能够执行包管理的工具能够下载和安装库及其依赖项。

### 动机

为什么我们关心包？包允许我们定义模块的层次结构，并使用点语法（`from package.module import my_function`）轻松访问模块。此外，它们使得与其他开发者共享代码变得容易。由于每个包都包含一个定义其依赖项的 `pyproject.toml` 文件，其他开发者不必单独安装所需的包，而是可以直接从其 `pyproject.toml` 文件安装包。

### 工具

许多工具都可以执行包管理：`pip`、`pipx`、`pipenv`、`conda`、`pdm`、`poetry`、`rye` 和 `PyFlow`。`pip` 是 Python 社区中众所周知的专用包管理工具。
![包管理工具对比][11]

#### PIP

Python 的标准包管理器是 [`pip`][12]。它随 Python 一起提供，允许你从 PyPI 和其他索引安装软件包。主要命令（可能是 Python 开发人员学习的第一个命令之一）是 `pip install <package_name>`。当然，`pip` 还提供了许多其他选项。查看[文档][12]以获取有关可用标志等的更多信息。

## 回顾 2 - Lock file

在我们继续讨论多用途工具之前，还有一个对打包很重要的文件：锁文件。`pyproject.toml` 包含抽象依赖项，而锁文件包含具体依赖项。它记录了为项目安装的所有依赖项的确切版本（例如 `pandas==2.0.3`）。这使得项目能够在多个平台上重现。如果你以前从未见过锁文件，请查看[这个来自 `poetry` 的文件][13]：

![poetry_lock 文件][14]

## 多用途的工具

了解了锁文件之后，我们可以开始研究执行多用途的工具。我们将从 `pipenv` 和 `conda` 开始，然后再过渡到 `poetry` 和 `pdm` 等打包工具。

### PipEnv

顾名思义，[`pipenv`][15] 结合了 `pip` 和 `virtualenv`。正如我们在维恩图中看到的，它允许你执行虚拟环境管理和包管理：

![pipenv 文件][16]

`pipenv` 引入了两个额外的文件：

- `Pipfile`
- `Pipfile.lock`

`Pipfile` 是一个 TOML 文件（类似于 `pyproject.toml`），用于定义项目依赖项。当开发者调用 `pipenv` 命令（如 `pipenv install`）时，它由开发者管理。`Pipfile.lock` 允许确定性构建。它消除了对 `requirements.txt` 文件的需求，并通过锁定操作自动管理。

最重要的 `pipenv` 命令是：

```bash
# Install package
pipenv install <package_name>

# Run Python script within virtual env
pipenv run <script_name.py>

# Activate virtual env
pipenv shell
```

### Conda

[`Conda`][17] 是一个通用的包管理系统。这意味着它不局限于 Python 包。Conda 是一个功能强大的工具，拥有许多功能。有许多教程和博客文章（例如[官方教程][18]），因此我在这里不再赘述。但是，我想提一点：虽然可以使用 `conda` 构建和发布包，但我没有将该工具包含在相应的类别中。这是因为使用 `conda` 打包的工作方式略有不同，生成的包将是 `conda` 包。

![conda][19]

### 功能评估

最后但同样重要的是，我想介绍一些用于打包的多功能工具。我承诺进行公正的评估。为此，我创建了一个我认为在比较不同工具时很重要的功能列表。这些功能包括：

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ?   |
| 它是否解析/锁定依赖项？            | ?   |
| 是否有干净的构建/发布流程？        | ?   |
| 它是否允许使用插件？               | ?   |
| 它是否支持 PEP 660（可编辑安装）？ | ?   |
| 它是否支持 PEP 621（项目元数据）？ | ?   |

关于这两个 PEP：Python 在打包方面有很多开放和关闭的 PEP。有关完整概述，请查看[此页面][20]。我仅出于特定原因包含了 PEP 660 和 PEP 621：

- [PEP 660][21] 是关于基于 `pyproject.toml` 构建的可编辑安装。当你使用 `pip` 安装软件包时，你可以选择使用 `pip install -e package_name` 以可编辑模式安装它。当你正在开发一个软件包并希望你的更改直接反映在你的环境中时，这是一个非常重要的功能。
- [PEP 621][22] 指定了如何在 `pyproject.toml` 文件中写入项目的核心元数据。我添加它是为了说明，目前有一个软件包（剧透：它是 `poetry`）不支持此 PEP，而是使用自己的方式声明元数据。

### Flit

[Flit][23] 试图创建一个将 Python 包和模块放到 PyPI 上的简单方法。它有一个非常具体的用例：它旨在用于打包纯 Python 包（即没有构建步骤的包）。它不关心任何其他任务：

- Python 版本管理：❌
- 包管理：❌
- 环境管理：❌
- 构建包：✅
- 发布包：✅

这也反映在我们的维恩图中：

![flit][24]

#### 功能评估

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ❌  |
| 它是否解析/锁定依赖项？            | ❌  |
| 是否有干净的构建/发布流程？        | ✅  |
| 它是否允许使用插件？               | ❌  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ✅  |

#### 主要命令

```bash
# Create new pyproject.toml
flit init

# Build and publish
flit publish
```

### Poetry

[Poetry][25] 是打包界中一个众所周知的工具。正如维恩图所示，除了 Python 版本管理之外，它可以做任何事情：

- Python 版本管理：❌
- 包管理：✅
- 环境管理：✅
- 构建包：✅
- 发布包：✅

![poetry][26]

查看下面的功能评估，您会发现 Poetry **不**支持 PEP 621。在 GitHub 上有一个关于此的开放问题大约 1.5 年了，但它还没有被集成到主代码库中（还）。

#### 功能评估

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ✅  |
| 它是否解析/锁定依赖项？            | ✅  |
| 是否有干净的构建/发布流程？        | ✅  |
| 它是否允许使用插件？               | ✅  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ❌  |

#### 主要命令

```bash
# Create directory structure and pyproject.toml
poetry new <project_name>

# Create pyproject.toml interactively
poetry init

# Install package from pyproject.toml
poetry install
```

#### 依赖管理

```bash
# Add dependency
poetry add <package_name>

# Display all dependencies
poetry show --tree
```

#### 运行代码

```bash
# Activate virtual env
poetry shell

# Run script within virtual env
poetry run python <script_name.py>
```

#### Lock file

首次安装软件包时，Poetry 会解析 `pyproject.toml` 文件中列出的所有依赖项，并下载软件包的最新版本。Poetry 完成安装后，它会将所有软件包及其下载的确切版本写入 `poetry.lock` 文件，将项目锁定到这些特定版本。建议将锁定文件提交到项目仓库，以便所有参与项目的人员都锁定到相同版本的依赖项。要将依赖项更新到最新版本，请使用 `poetry update` 命令。

#### Build/publish flow（构建/发布流程）

```bash
# Package code (creates `.tar.gz` and `.whl` files)
poetry build

# Publish to PyPI
poetry publish
```

### PDM

[PDM][27] 是一个相对较新的包和依赖管理器（始于 2019 年），它很大程度上受到 Poetry 和 PyFlow 的启发。您会注意到，我在这篇文章中没有谈论 PyFlow。这是因为 PyFlow 不再积极开发——在快速发展的打包领域中，这是一个必须的条件。作为一个更新的工具，PDM 需要 Python 3.7 或更高版本。与其他工具的另一个区别是 PDM 允许用户选择构建后端。

PDM 是唯一一个（除了 PyFlow）在本地包上实现 [PEP 582][28] 的工具，这是一种实现环境管理的替代方法。请注意，此 PEP 最近被拒绝。

如维恩图所示，PDM 位于 Poetry 旁边。这意味着它可以做除 Python 版本管理之外的所有事情：

- Python 版本管理：❌
- 包管理：✅
- 环境管理：✅
- 构建包：✅
- 发布包：✅

PDM 的主要命令与 Poetry 类似。但是，存在的命令较少。例如，目前没有 `pdm shell` 或 `pdm new`。

![][29]

#### 功能评估

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ✅  |
| 它是否解析/锁定依赖项？            | ✅  |
| 是否有干净的构建/发布流程？        | ✅  |
| 它是否允许使用插件？               | ✅  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ✅  |

#### 创建新项目

```bash
# Create pyproject.toml interactively
pdm init

# Install package from pyproject.toml
pdm install
```

#### 依赖管理

```bash
# Add dependency
pdm add <package_name>

# Display all dependencies
pdm list --graph
```

#### 运行代码

```bash
# No pdm shell command

# Run script within env
pdm run python <script_name.py>
```

#### Lock file

PDM 的锁定功能类似于 Poetry。首次安装软件包时，PDM 会解析 `pyproject.toml` 文件中列出的所有依赖项，并下载软件包的最新版本。PDM 完成安装后，会将所有软件包及其下载的确切版本写入 `pdm.lock` 文件，将项目锁定到这些特定版本。建议将锁定文件提交到项目仓库，以便所有参与项目的人员都锁定到相同版本的依赖项。要将依赖项更新到最新版本，请使用 `pdm update` 命令。

#### Build/publish flow（构建/发布流程）

```bash
# Package code (creates `.tar.gz` and `.whl` files)
pdm build

# Publish to PyPI
pdm publish
```

### Hatch

[Hatch][30] 可以执行以下任务：

- Python 版本管理：❌
- 软件包管理：❌
- 环境管理：✅
- 构建软件包：✅
- 发布软件包：✅

需要注意的是，Hatch 的作者承诺很快就会添加锁定功能，这也应该能够实现软件包管理。 在阅读本文时，请务必查看最新版本的 Hatch，以了解是否已实现此功能。

![hatch][31]

#### 功能评估

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ❌  |
| 它是否解析/锁定依赖项？            | ❌  |
| 是否有干净的构建/发布流程？        | ✅  |
| 它是否允许使用插件？               | ✅  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ✅  |

#### 创建一个新项目

```bash
# Create directory structure and pyproject.toml
hatch new <project_name>

# Interactive mode
hatch new -i <project_name>

# Initialize existing project / create pyproject.toml
hatch new --init
```

#### 依赖管理

```python
# Packages are added manually to pyproject.toml
hatch add <package_name> # This command doesn't exist!

# Display dependencies
hatch dep show table
```

#### 运行代码

```bash
# Activate virtual env
hatch shell

# Run script within virtual env
hatch run python <script_name.py>
```

#### Build/publish flow（构建/发布流程）

```bash
# Package code (creates `.tar.gz` and `.whl` files)
hatch build

# Publish to PyPI
hatch publish
```

#### Declarative environment management（声明式环境管理）

Hatch 的特别之处在于它允许您在 `pyproject.toml` 文件中配置您的虚拟环境。此外，它还允许您专门为某个环境定义脚本。这方面的一个示例用例是 [代码格式化][32]。

### Rye

[Rye][33] 是由 Flask 框架的创建者 Armin Ronacher 最近开发的（第一个版本于 2023 年 5 月发布）。它的灵感很大程度上来自于 `rustup` 和 `cargo`，这两个都是 Rust 编程语言的打包工具。Rye 是用 Rust 编写的，并且能够执行我们维恩图中的所有任务：

- Python 版本管理：✅
- 软件包管理：✅
- 环境管理：✅
- 构建软件包：✅
- 发布软件包：✅

目前，Rye 没有插件接口。但是，由于新版本会定期发布，因此将来可能会添加此功能。

![rye][34]

#### 功能评估

|                                    |     |
| ---------------------------------- | --- |
| 该工具是否管理依赖项？             | ✅  |
| 它是否解析/锁定依赖项？            | ✅  |
| 是否有干净的构建/发布流程？        | ✅  |
| 它是否允许使用插件？               | ❌  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ✅  |

#### 创建一个新项目

```bash
# Create directory structure and pyproject.toml
rye init <project_name>

# Pin a Python version
rye pin 3.10
```

#### 依赖管理

```python
# Add dependency - this does not install the package!
rye add <package_name>

# Synchronize virtual envs, lock file, etc.
# This install packages and Python versions
rye sync
```

#### 运行代码

```bash
# Activate virtual env
rye shell

# Run script within virtual env
rye run python <script_name.py>
```

#### Build/publish flow（构建/发布流程）

```bash
# Package code (creates `.tar.gz` and `.whl` files)
rye build

# Publish to PyPI
rye publish
```

### 概述

|                                    | Flit | Poetry | PDM | Hatch | Rye |
| ---------------------------------- | ---- | ------ | --- | ----- | --- |
| 该工具是否管理依赖项？             | ❌   | ✅     | ✅  | ❌    | ✅  |
| 它是否解析/锁定依赖项？            | ❌   | ✅     | ✅  | ❌    | ✅  |
| 是否有干净的构建/发布流程？        | ✅   | ✅     | ✅  | ✅    | ✅  |
| 它是否允许使用插件？               | ❌   | ✅     | ✅  | ✅    | ❌  |
| 它是否支持 PEP 660（可编辑安装）？ | ✅   | ✅     | ✅  | ✅    | ✅  |
| 它是否支持 PEP 621（项目元数据）？ | ✅   | ❌     | ✅  | ✅    | ✅  |

## 不属于这些类别的工具

有些工具不属于我的任何类别。它们是：

- [pip-tools][35] 帮助您保持基于 `pip` 的软件包版本最新。
- [tox][36] 和 [nox][37] 主要用于测试，但也处理虚拟环境。

[给本博文提供建议][38]

[1]: https://alpopkes.com/images/author/image_al_hu6aa485cfd64a98f54b411e3a0324b396_178285_120x120_fit_q75_box.jpg
[2]: https://www.youtube.com/watch?v=MsJjzVIVs6M
[3]: https://www.youtube.com/watch?v=3-drZY3u5vo
[4]: https://alpopkes.com/posts/python/figures/venn_diagram.png
[5]: https://alpopkes.com/posts/python/figures/python_version_management.png
[6]: https://alpopkes.com/posts/python/figures/env_management.png
[7]: https://docs.python.org/3/library/venv.html
[8]: https://virtualenv.pypa.io/en/latest
[9]: https://peps.python.org/pep-0518
[10]: https://github.com/pandas-dev/pandas/blob/main/pyproject.toml
[11]: https://alpopkes.com/posts/python/figures/package_management.png
[12]: https://pip.pypa.io/en/stable
[13]: https://github.com/python-poetry/poetry/blob/master/poetry.lock
[14]: https://alpopkes.com/posts/python/figures/poetry_lock.png
[15]: https://pipenv.pypa.io/en/latest
[16]: https://alpopkes.com/posts/python/figures/pipenv.png
[17]: https://conda.io/projects/conda/en/latest/index.html
[18]: https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-python
[19]: https://alpopkes.com/posts/python/figures/conda.png
[20]: https://peps.python.org/topic/packaging/
[21]: https://peps.python.org/pep-0660/
[22]: https://peps.python.org/pep-0621/
[23]: https://flit.pypa.io/en/stable/
[24]: https://alpopkes.com/posts/python/figures/flit.png
[25]: https://python-poetry.org/
[26]: https://alpopkes.com/posts/python/figures/poetry.png
[27]: https://pdm.fming.dev/latest/
[28]: https://peps.python.org/pep-0582
[29]: https://alpopkes.com/posts/python/figures/pdm.png
[30]: https://hatch.pypa.io/latest
[31]: https://alpopkes.com/posts/python/figures/hatch.png
[32]: https://hatch.pypa.io/1.1/config/environment/#scripts
[33]: https://rye-up.com
[34]: https://alpopkes.com/posts/python/figures/rye.png
[35]: https://pip-tools.readthedocs.io/en/latest
[36]: https://tox.wiki
[37]: https://nox.thea.codes
[38]: https://github.com/zotroneneis/zotroneneis.github.io/edit/main/content/posts/python/packaging_tools.md
