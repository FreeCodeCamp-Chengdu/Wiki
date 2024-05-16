---
title: An unbiased evaluation of environment management and packaging tools
authorURL: ""
originalURL: https://alpopkes.com/posts/python/packaging_tools/
translator: ""
reviewer: ""
---

![](/images/author/image_al.jpg)

Anna-Lena Popkes

8月24, 2023

## 动机

当我开始使用 Python 并创建我的第一个包时，我感到困惑。创建和管理一个包比我预期的要困难得多。此外，存在多种工具，我不确定该使用哪一个。我相信你们中的大多数人过去也遇到过同样的问题。Python 有大量的工具来管理虚拟环境和创建包，要理解哪一个最适合你的需求可能很难（或几乎不可能）。关于这个话题存在多个演讲和博客文章，但没有一个提供完整的概览或以结构化的方式评估这些工具。这就是本文的目的。我想给你一个真正无偏见的评价，关于现有的打包和环境管理工具。如果你宁愿观看一个演讲，可以看看 [PyCon DE 2023](https://www.youtube.com/watch?v=MsJjzVIVs6M) 或 [EuroPython 2023](https://www.youtube.com/watch?v=3-drZY3u5vo) 的录像。
<!-- more -->

## 分类

为了撰写本文，我确定了环境和软件包管理的五大重要类别：

- 环境管理（主要涉及虚拟环境）
- 软件包管理
- Python 版本管理
- 软件包构建
- 软件包发布

正如你在下面的维恩图中所看到的，存在大量的工具。有些可以做一件事（即它们是单一用途的），而其他工具可以执行多个任务（因此我称它们为多用途工具）。

![](/posts/python/figures/venn_diagram.png)

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

![](/posts/python/figures/python_version_management.png)

### pyenv

Python 有一个单一用途的工具可以让您安装和管理 Python 版本：[pyenv](https://github.com/pyenv/pyenv)！ Pyenv 易于使用。 最重要的命令如下： 

```bash
# Install specific Python version
pyenv install 3.10.4

# Switch between Python versions
pyenv shell <version> # select version just for current shell session
pyenv local <version> # automatically select version whenever you are in the current directory
pyenv global <version> # select version globally for your user account
```

## (Virtual) environment management（虚拟环境管理）

### 定义

环境管理工具可以创建和管理（虚拟）环境。

### 动机

我们为什么首先要使用环境？ 正如开头所提到的，项目有特定的要求（即它们依赖于其他包）。 通常情况下，不同的项目需要相同包的不同版本。 这会导致依赖冲突。 此外，使用 `pip install` 安装包时可能会出现问题，因为该包与您的系统范围内的 Python 安装放在一起。 其中一些问题可以通过在 `pip` 命令中使用 `--user` 标志来解决。 但是，这个选项可能不是每个人都知道，尤其是初学者。 

### 工具

许多工具允许用户创建和管理环境。 它们是：`venv、virtualenv、pipenv、conda、pdm、poetry、hatch、rye` 和 `PyFlow`。 其中只有两个是单一用途的工具：`venv` 和 `virtualenv`。 让我们更详细地了解它们。 

![](/posts/python/figures/env_management.png)

### venv

[Venv](https://docs.python.org/3/library/venv.html) 是用于创建虚拟环境的内置 Python 包。 这意味着它随 Python 一起提供，用户无需安装。 最重要的命令如下：

```bash
# Create new environment
python3 -m venv <env_name>

# Activate an environment
. <env_name>/bin/activate

# Deactivate an active environment
deactivate
```

### virtualenv

[Virtualenv](https://virtualenv.pypa.io/en/latest/) 试图改进 `venv`。 它提供了比 `venv` 更多的功能，并且速度更快、功能更强大。 最重要的命令与 `venv` 的命令类似，只是创建一个新环境更简洁：

```bash
# Create new environment
virtualenv <env_name>

# Activate an environment
. <env_name>/bin/activate

# Deactivate an active environment
deactivate
```

## Recap I - `pyproject.toml`

在讨论打包之前，我想确保你了解打包最重要的文件：`pyproject.toml`。

Python 中的打包已经走过了漫长的道路。直到 [PEP 518](https://peps.python.org/pep-0518/) `setup.py` 文件被用于打包，使用 `setuptools` 作为构建工具。[PEP 518](https://peps.python.org/pep-0518/) 引入了 `pyproject.toml` 文件的使用。因此，在创建包时，你始终需要一个 `pyproject.toml` 文件。`pyproject.toml` 用于定义项目的设置、定义元数据以及许多其他内容。如果你想查看示例，请查看 [pandas 库的 `pyproject.toml` 文件](https://github.com/pandas-dev/pandas/blob/main/pyproject.toml)。了解了 `pyproject.toml` 后，我们可以继续看看包管理。

## 包管理

### 定义
能够执行包管理的工具能够下载和安装库及其依赖项。

### 动机

为什么我们关心包？包允许我们定义模块的层次结构，并使用点语法（`from package.module import my_function`）轻松访问模块。此外，它们使得与其他开发者共享代码变得容易。由于每个包都包含一个定义其依赖项的 `pyproject.toml` 文件，其他开发者不必单独安装所需的包，而是可以直接从其 `pyproject.toml` 文件安装包。

### 工具

许多工具都可以执行包管理：`pip`、`pipx`、`pipenv`、`conda`、`pdm`、`poetry`、`rye` 和 `PyFlow`。`pip` 是 Python 社区中众所周知的专用包管理工具。
![](/posts/python/figures/package_management.png)

#### pip

Python 的标准包管理器是 [`pip`](https://pip.pypa.io/en/stable/)。它随 Python 一起提供，允许你从 PyPI 和其他索引安装软件包。主要命令（可能是 Python 开发人员学习的第一个命令之一）是 `pip install <package_name>`。当然，`pip` 还提供了许多其他选项。查看[文档](https://pip.pypa.io/en/stable/)以获取有关可用标志等的更多信息。

## Recap II - Lock file

在我们继续讨论多用途工具之前，还有一个对打包很重要的文件：锁文件。`pyproject.toml` 包含抽象依赖项，而锁文件包含具体依赖项。它记录了为项目安装的所有依赖项的确切版本（例如 `pandas==2.0.3`）。这使得项目能够在多个平台上重现。如果你以前从未见过锁文件，请查看[这个来自 `poetry` 的文件](https://github.com/python-poetry/poetry/blob/master/poetry.lock)：

![](/posts/python/figures/poetry_lock.png)

## 多用途的工具

了解了锁文件之后，我们可以开始研究执行多用途的工具。我们将从 `pipenv` 和 `conda` 开始，然后再过渡到 `poetry` 和 `pdm` 等打包工具。

### Pipenv

顾名思义，[`pipenv`](https://pipenv.pypa.io/en/latest/) 结合了 `pip` 和 `virtualenv`。正如我们在维恩图中看到的，它允许你执行虚拟环境管理和包管理：

![](/posts/python/figures/pipenv.png)

`pipenv` 引入了两个额外的文件：

-   `Pipfile`
-   `Pipfile.lock`

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

[`Conda`](https://conda.io/projects/conda/en/latest/index.html) is a general-purpose package management system. That means that it’s not limited to Python packages. Conda is a huge tool with lots of capabilities. Lot’s of tutorials and blog posts exist (for example [the official one](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-python)) so I won’t go into more detail here. However, I want to mention one thing: while it is possible to build and publish a package with `conda` I did not include the tool in the appropriate categories. That’s because packaging with `conda` works a little differently and the resulting packages will be `conda` packages.

![](/posts/python/figures/conda.png)

### Feature evaluation

Last but not least I want to present multi-purpose tools for packaging. I promised an unbiased evaluation. For this purpose I created a list of features that I consider important when comparing different tools. The features are:

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ?   |
| Does it resolve/lock dependencies?           | ?   |
| Is there a clean build/publish flow?         | ?   |
| Does it allow to use plugins?                | ?   |
| Does it support PEP 660 (editable installs)? | ?   |
| Does it support PEP 621 (project metadata)?  | ?   |

Regarding the two PEPs: Python has a lot of open and closed PEPs on packaging. For a full overview take a look at [this page](https://peps.python.org/topic/packaging/). I only included PEP 660 and PEP 621 for specific reasons:

-   [PEP 660](https://peps.python.org/pep-0660/) is about editable installs for `pyproject.toml` based builds. When you install a package using `pip` you have the option to install it in editable mode using `pip install -e package_name`. This is an important features to have when you are developing a package and want your changes to be directly reflected in your environment.
-   [PEP 621](https://peps.python.org/pep-0621/) specifies how to write a project’s core metadata in a `pyproject.toml` file. I added it because one package (spoiler: it’s `poetry`) currently does not support this PEP but uses its own way for declaring metadata.

### Flit

[Flit](https://flit.pypa.io/en/stable/) tries to create a simple way to put Python packages and modules on PyPI. It has a very specific use case: it’s meant to be used for packaging pure Python packages (that is, packages without a build step). It doesn’t care about any of the other tasks:

-   Python version management: ❌
-   Package management: ❌
-   Environment management: ❌
-   Building a package: ✅
-   Publishing a package: ✅

This is also reflected in our Venn diagram:

![](/posts/python/figures/flit.png)

#### Feature evaluation

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ❌  |
| Does it resolve/lock dependencies?           | ❌  |
| Is there a clean build/publish flow?         | ✅  |
| Does it allow to use plugins?                | ❌  |
| Does it support PEP 660 (editable installs)? | ✅  |
| Does it support PEP 621 (project metadata)?  | ✅  |

#### Main commands

```bash
# Create new pyproject.toml
flit init

# Build and publish
flit publish
```

### Poetry

[Poetry](https://python-poetry.org/) is a well known tool in the packaging world. As visible in the Venn diagram it can do everything except for Python version management:

-   Python version management: ❌
-   Package management: ✅
-   Environment management: ✅
-   Building a package: ✅
-   Publishing a package: ✅

![](/posts/python/figures/poetry.png)

Taking a look at the feature evaluation below you will see than Poetry does **not** support PEP 621. There has been an open issue about this on GitHub for about 1.5 years, but it hasn’t been integrated into the main code base (yet).

#### Feature evaluation

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ✅  |
| Does it resolve/lock dependencies?           | ✅  |
| Is there a clean build/publish flow?         | ✅  |
| Does it allow to use plugins?                | ✅  |
| Does it support PEP 660 (editable installs)? | ✅  |
| Does it support PEP 621 (project metadata)?  | ❌  |

#### Main commands

```bash
# Create directory structure and pyproject.toml
poetry new <project_name>

# Create pyproject.toml interactively
poetry init

# Install package from pyproject.toml
poetry install
```

#### Dependency management

```bash
# Add dependency
poetry add <package_name>

# Display all dependencies
poetry show --tree
```

#### Running code

```bash
# Activate virtual env
poetry shell

# Run script within virtual env
poetry run python <script_name.py>
```

#### Lock file

When installing a package for the first time, Poetry resolves all dependencies listed in your `pyproject.toml` file and downloads the latest version of the packages. Once Poetry has finished installing, it writes all the packages and the exact versions that it downloaded to a `poetry.lock` file, locking the project to those specific versions. It’s recommended to commit the lock file to your project repo so that all people working on the project are locked to the same versions of dependencies. To update your dependencies to the latest versions, use the `poetry update` command.

#### Build/publish flow

```bash
# Package code (creates `.tar.gz` and `.whl` files)
poetry build

# Publish to PyPI
poetry publish
```

### PDM

[PDM](https://pdm.fming.dev/latest/) is a relatively new package and dependency manager (started in 2019) that is strongly inspired by Poetry and PyFlow. You will notice that I’m not talking about PyFlow in this article. That’s because PyFlow is not actively developed anymore - a must in the quickly evolving landscape of packaging. Being a new(er) tool, PDM requires Python 3.7 or higher. Another difference to other tools is that PDM allows users to choose a build backend.  
PDM is the only tool (apart from PyFlow) that implements [PEP 582](https://peps.python.org/pep-0582/) on local packages, an alternative way of implementing environment management. Note that this PEP was recently rejected.

As visible in the Venn diagram, PDM sits right next to Poetry. That means that it can do everything except for Python version management:

-   Python version management: ❌
-   Package management: ✅
-   Environment management: ✅
-   Building a package: ✅
-   Publishing a package: ✅

The main commands of PDM are similar to Poetry. However, less commands exist. For example, there is no `pdm shell` or `pdm new` at the moment.

![](/posts/python/figures/pdm.png)

#### Feature evaluation

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ✅  |
| Does it resolve/lock dependencies?           | ✅  |
| Is there a clean build/publish flow?         | ✅  |
| Does it allow to use plugins?                | ✅  |
| Does it support PEP 660 (editable installs)? | ✅  |
| Does it support PEP 621 (project metadata)?  | ✅  |

#### Creating a new project

```bash
# Create pyproject.toml interactively
pdm init

# Install package from pyproject.toml
pdm install
```

#### Dependency management

```bash
# Add dependency
pdm add <package_name>

# Display all dependencies
pdm list --graph
```

#### Running code

```bash
# No pdm shell command

# Run script within env
pdm run python <script_name.py>
```

#### Lock file

The locking functionality of PDM is similar to Poetry. When installing a package for the first time, PDM resolves all dependencies listed in your `pyproject.toml` file and downloads the latest version of the packages. Once PDM has finished installing, it writes all packages and the exact versions that it downloaded to a `pdm.lock` file, locking the project to those specific versions. It’s recommended to commit the lock file to your project repo so that all people working on the project are locked to the same versions of dependencies. To update your dependencies to the latest versions, use the `pdm update` command.

#### Build/publish flow

```bash
# Package code (creates `.tar.gz` and `.whl` files)
pdm build

# Publish to PyPI
pdm publish
```

### Hatch

[Hatch](https://hatch.pypa.io/latest/) can perform the following tasks:

-   Python version management: ❌
-   Package management: ❌
-   Environment management: ✅
-   Building a package: ✅
-   Publishing a package: ✅

It should be noted that the author of Hatch promised that locking functionality will be added soon, which should also enable package management. Please make sure to check the latest version of Hatch to see if this has been implemented when you read this article.

![](/posts/python/figures/hatch.png)

#### Feature evaluation

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ❌  |
| Does it resolve/lock dependencies?           | ❌  |
| Is there a clean build/publish flow?         | ✅  |
| Does it allow to use plugins?                | ✅  |
| Does it support PEP 660 (editable installs)? | ✅  |
| Does it support PEP 621 (project metadata)?  | ✅  |

#### Creating a new project

```bash
# Create directory structure and pyproject.toml
hatch new <project_name>

# Interactive mode
hatch new -i <project_name>

# Initialize existing project / create pyproject.toml
hatch new --init
```

#### Dependency management

```python
# Packages are added manually to pyproject.toml
hatch add <package_name> # This command doesn't exist!

# Display dependencies
hatch dep show table
```

#### Running code

```bash
# Activate virtual env
hatch shell

# Run script within virtual env
hatch run python <script_name.py>
```

#### Build/publish flow

```bash
# Package code (creates `.tar.gz` and `.whl` files)
hatch build

# Publish to PyPI
hatch publish
```

#### Declarative environment management

Special about Hatch is that it allows you to configure your virtual environments within the `pyproject.toml` file. In addition it lets you define scripts specifically for an environment. And example use case for this is [code formatting](https://hatch.pypa.io/1.1/config/environment/#scripts).

### Rye

[Rye](https://rye-up.com/) was recently developed by Armin Ronacher (first release May 2023), the creator of the Flask framework. It is strongly inspired by `rustup` and `cargo`, the packaging tools of the programming language Rust. Rye is written in Rust and is able to perform all tasks in our Venn diagram:

-   Python version management: ✅
-   Package management: ✅
-   Environment management: ✅
-   Building a package: ✅
-   Publishing a package: ✅

Currently, Rye does not have a plugin interface. However, since new releases are published on a regular basis, this might be added in the future.

![](/posts/python/figures/rye.png)

#### Feature evaluation

|                                              |     |
| -------------------------------------------- | --- |
| Does the tool manage dependencies?           | ✅  |
| Does it resolve/lock dependencies?           | ✅  |
| Is there a clean build/publish flow?         | ✅  |
| Does it allow to use plugins?                | ❌  |
| Does it support PEP 660 (editable installs)? | ✅  |
| Does it support PEP 621 (project metadata)?  | ✅  |

#### Creating a new project

```bash
# Create directory structure and pyproject.toml
rye init <project_name>

# Pin a Python version
rye pin 3.10
```

#### Dependency management

```python
# Add dependency - this does not install the package!
rye add <package_name>

# Synchronize virtual envs, lock file, etc.
# This install packages and Python versions
rye sync
```

#### Running code

```bash
# Activate virtual env
rye shell

# Run script within virtual env
rye run python <script_name.py>
```

#### Build/publish flow

```bash
# Package code (creates `.tar.gz` and `.whl` files)
rye build

# Publish to PyPI
rye publish
```

### Overview

|                                              | Flit | Poetry | PDM | Hatch | Rye |
| -------------------------------------------- | ---- | ------ | --- | ----- | --- |
| Does the tool manage dependencies?           | ❌   | ✅     | ✅  | ❌    | ✅  |
| Does it resolve/lock dependencies?           | ❌   | ✅     | ✅  | ❌    | ✅  |
| Is there a clean build/publish flow?         | ✅   | ✅     | ✅  | ✅    | ✅  |
| Does it allow to use plugins?                | ❌   | ✅     | ✅  | ✅    | ❌  |
| Does it support PEP 660 (editable installs)? | ✅   | ✅     | ✅  | ✅    | ✅  |
| Does it support PEP 621 (project metadata)?  | ✅   | ❌     | ✅  | ✅    | ✅  |

## Tools that do not fit the categories

Some tools exist which don’t fit into any of my categories. These are:

-   [pip-tools](https://pip-tools.readthedocs.io/en/latest/) which helps to keep the versions of your `pip`\-based packages up-to-date.
-   [tox](https://tox.wiki/) and [nox](https://nox.thea.codes/) which are mainly used for testing but also handles virtual environments.

[Improve this page](https://github.com/zotroneneis/zotroneneis.github.io/edit/main/content/posts/python/packaging_tools.md)

---

[Next  
Principal component analysis (PCA)](/posts/machine_learning/principal_component_analysis/)

---
