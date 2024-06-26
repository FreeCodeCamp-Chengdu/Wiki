---
title: 为什么我喜欢 Nox
date: 2023-01-18
updated: 2024-05-26 09:55:00
original: https://hynek.me/articles/why-i-like-nox/
authors:
  - luojiyin1987
categories:
  - Article
  - Translation
toc: true
photos:
  - https://i.ytimg.com/vi/x_0uxBiByCU/maxresdefault.jpg
---

自从我参与开源 Python 项目以来，[tox][1] 一直是跨 Python 版本（以及其他因素）测试软件包的关键。然而，最近，我越来越多地在我的项目中使用 [Nox][2]。由于我被反复问到，为什么，我将总结一下我的想法。

我再怎么强调也不为过，我不想阻止任何人使用 tox。tox 很棒。没有 tox，Python 开源生态系统就不会是现在的样子。它的作者和维护者我永远感激！

我本能地不喜欢说 tox 的坏话，但如果不对比功能和行为就无法解释我的偏好。

这不是呼吁放弃 tox（我仍然在许多项目中使用它），而是解释为什么我在某些情况下更喜欢 Nox。Nox 和 tox 都不是绝对优于另一个，只是不同而已。

<!-- more -->

## 配置格式

tox 和 Nox 之间最明显的区别是 tox 是基于古老的 [INI 格式][3] [(`tox.ini`)][4] 的 [DSL][5]，而 Nox 使用 Python 文件 (`noxfile.py`)。

如果您不熟悉其中任何一个，一个简单的 `tox.ini` 如下所示：

```ini
[tox]
env_list = py310,py311

[testenv]
extras = tests
commands = pytest {posargs}
```

在 `noxfile.py`中会是这样：

```python
import nox

@nox.session(python=["3.10", "3.11"])
def tests(session):
    session.install(".[tests]")

    session.run("pytest", *session.posargs)
```

您可能会注意到命名上的差异：tox 所称的 environments 在 Nox 中称为 sessions。

现在，如果您调用 `tox` 或 `nox`，它们都会：

1.  为 Python 3.10 和 Python 3.11 创建虚拟环境，
2.  在其中安装当前软件包 (`.`) 及其额外的依赖项 `tests`，
3.  并从每个环境中运行 `pytest`。

`{posargs}` 和 `*session.posargs` 位允许您将命令行参数传递给测试运行器。因此，要使 pytest 在第一个错误后中止，您可以分别编写 `nox -- -x` 或 `tox -- -x`。

对一些人来说，在这里使用 Python 似乎是一种倒退。我们难道不是刚刚从 `setup.py` 迁移到 `pyproject.toml` 来摆脱 Python 用于配置吗？

是也不是。`setup.py` 的问题不在于它是 Python，而在于它在安装时不受控制地运行。

按需运行命令，以及代码，是 tox 和 Nox 的存在理由；唯一的区别在于它们是如何定义的。由于 tox 使用自己的语言来定义这些命令，因此您需要在其 DSL 中使用专用功能来实现任何目标。同时，如果您想在 Nox 中做某事，您通常只需要编写一些 Python 代码。

诚然，我喜欢 Nox 的主要原因之一是，在很长一段时间后回到一个重要的 `tox.ini` 对我来说已经成为一个挑战。就在最近，我注意到在 [_environ-config_][6] 中，应该检查测试套件是否通过最旧支持版本的 [_attrs_][7] 的 tox 环境不再工作了。我像这样定义它：

```ini
[testenv]
extras = tests
deps = oldestAttrs: attrs==17.4.0
commands = pytest {posargs}
```

但是，尽管 tox 确实首先安装了 _attrs_ 17.4.0，但在安装项目时，它会用最新版本覆盖它。为什么？我从来没有弄清楚，但我 99.9% 确定它曾经[工作过][8]。其他依赖项都不需要更新的版本，在我看来它仍然是正确的。

## INI 继承 对比 Python 函数

如果你足够仔细地观察，你会发现，抛开语法不谈——这两种配置原则分别是通过子类化共享代码 与通过函数共享代码的情况。在 tox 中，你定义了一个基础 `testenv`，所有其他环境都继承自它，但可以覆盖任何字段。仅此行为就足以让我偶尔挠头。

在 tox 中，子环境之间的重用（例如 `py37` 和 `py38` 之间）是使用因子相关的语句（如上面的 `oldestAttrs:`）或替换（如 `{[testenv:py37]commands}`）完成的，我永远记不住它们的语法，并且总是让我在其他项目中寻找例子。

在 Nox 中，如果你想重用，你写函数。没有其他语言需要学习，[只有一个 API][9]。例如，为了在 [_Coverage.py_][10] 下运行最旧和最新的 Python 版本，其余的没有，另外还要运行带有固定 _attrs_ 依赖项的最旧版本，我想出了以下内容：

```python
OLDEST = "3.7"

def _cov(session):
    session.run("coverage", "run", "-m", "pytest", *session.posargs)

@nox.session(python=[OLDEST, "3.11"], tags=["tests"])
def tests_cov(session):
    session.install(".[tests]")

    _cov(session)

@nox.session(python=OLDEST, tags=["tests"])
def tests_oldestAttrs(session):
    session.install(".[tests]", "attrs==17.4.0")

    _cov(session)

@nox.session(python=["3.7", "3.8", "3.9", "3.10"], tags=["tests"])
def tests(session):
    session.install(".[tests]")

    session.run("pytest", *session.posargs)
```

现在，如果有其他环境（如 Mypy 或 docs），我可以使用 `nox --tags tests` 只运行测试。

就代码行数而言，这比 tox 等效代码要长。但这是因为它更明确，任何对 Python 有一定了解的人都可以推断出这里发生了什么。包括我自己，在一年后回顾这段代码时。实际上，明确可能是件好事。

## 蟒蛇的力量

当然，由于 Python 的强大功能，Nox 比 tox 开箱即用地强大得多。最终，你将拥有整个标准库供你使用！你可以读写文件、创建临时目录、格式化字符串、发出 HTTP 请求。所有这些都不依赖于平台特性。

使用 tox，这些事情你经常需要编写一个 shell 脚本（可能在 Windows 上不起作用）并从你的 `tox.ini` 中调用它。因为 tox 不会将它的调用包装在一个 shell 中（不像 [Hatch][11]），所以你能做的事情非常有限：没有管道、没有子命令、没有输出重定向。

Nox 唯一的（纯粹是人体工程学上的）缺点是它迫使你使用 [`subprocess.run()`][12] 的非 shell 版本。这有时会导致相当野蛮的命令行：

```python
@nox.session(python="3.10")
def docs(session: nox.Session) -> None:
    session.install(".[docs]")

    for cmd in ["html", "doctest"]:
        session.run(
            # fmt: off
            "python", "-m", "sphinx",
            "-T", "-E",
            "-W", "--keep-going",
            "-b", cmd,
            "-d", "docs/_build/doctrees",
            "-D", "language=en",
            "docs",
            "docs/_build/html",
            # fmt: on
        )

    session.run("python", "-m", "doctest", "README.md")
```

但是，考虑到 shell 封装的问题（参见 [Docker][13] 或 [变量名净化][14]），这可能仍然是一个净收益。即使我不得不关闭 [Black][15] (`# fmt: off`) 以免它变得糟糕。

## 额外提示： 将 Python 版本作为一等公民，优先选择

正如 [James Bennett][16] 精辟地 [观察到的][17]，Nox 的一个很酷的特性是 Python 版本是会话的第一类选择器。而对于 tox 来说，它只是像其他任何因素一样的因素。

这意味着你可以调用 `nox --python 3.10`，所有标记为 Python 3.10 的会话都会运行。这在 CI 中非常有用，你不需要将 [`setup-python`][18] 的版本号（“3.11”）映射到 tox 的环境（`py311`——无论是手动还是使用 [_tox-gh_][19] 或 [_tox-gh-actions_][20]）。例如，使用 GitHub Actions，你可以编写：

```yaml
jobs:
  tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          allow-prereleases: true

      - name: Setup & run Nox
        run: |
          python -Im pip install nox
          python -Im nox --python ${{ matrix.python-version }}
```

新的 `allow-prereleases: true` 行允许你安装预发布版本，例如撰写本文时的 `3.12.0b1`。

---

tox 4 引入了使用 `-f` 选择单个因素的概念；因此，你可以使用 `tox -f py310` 运行所有 3.10 环境。版本号并不完全匹配，但 [可以进行调整][21]，PyPy 除外。我使用以下步骤：

```yaml
- name: Setup & run tox
  run: |
    V=${{ matrix.python-version }}

    if [[ "$V" = pypy-* ]]; then
      V=pypy3
    else
      V=py$(echo $V | tr -d .)
    fi

    python -Im pip install tox
    python -Im tox run -f $V
```

然而，这并不相同！`tox -f py310` 只会运行以 `py310` 开头的环境（例如 `py310` 或 `py310-foo`），而不是所有定义为使用 Python 3.10 的环境（例如名为 `docs` 的环境，它设置了 `base_python = py310`）。

## 结语

再次强调，这篇文章不是呼吁放弃 tox 并将所有项目迁移到 Nox。我自己没有这样做，也不打算这样做。Nox 和 tox 都没有绝对的优劣之分，但如果提到的问题引起了你的共鸣，那么你还有另一种选择！

如果你继续使用 tox，我已经[写了一篇关于如何在本地运行时使其速度提高 75% 的文章][22]。

[![Hynek Schlawack](https://static.hynek.me/img/avatar_w200h200.jpg)][23]

## Hynek Schlawack

一个热爱 Python 🐍、Go 🐹 和 DevOps 🔧 的代码波西米亚人。[博主][24] 📝，[演讲者][25] 📢，PSF [成员][26] 🏆，[大城市的海滩流浪汉][27] 🏄🏻，注重实质而非浮华 🧠。

我的内容对您有帮助或乐趣吗？请考虑[支持我][28]！每一份支持都能激励我创造更多内容。

[1]: https://tox.wiki/
[2]: https://nox.thea.codes/
[3]: https://en.wikipedia.org/wiki/INI_file
[4]: https://tox.wiki/
[5]: https://en.wikipedia.org/wiki/Domain-specific_language
[6]: https://github.com/hynek/environ-config
[7]: https://www.attrs.org/
[8]: https://nox.thea.codes/
[9]: https://nox.thea.codes/en/stable/config.html
[10]: https://coverage.readthedocs.io/
[11]: https://github.com/pypa/hatch/issues/701
[12]: https://docs.python.org/3/library/subprocess.html#subprocess.run
[13]: https://hynek.me/articles/docker-signals/
[14]: https://hynek.me/articles/macos-dyld-env/
[15]: https://black.readthedocs.io/
[16]: https://www.b-list.org/
[17]: https://lobste.rs/s/983b6p/why_i_like_nox#c_zrnxmp
[18]: https://github.com/actions/setup-python
[19]: https://pypi.org/project/tox-gh/
[20]: https://pypi.org/project/tox-gh-actions/
[21]: https://fosstodon.org/@adamchainz/109717334764977122
[22]: https://hynek.me/articles/turbo-charge-tox/
[23]: https://hynek.me/about/
[24]: https://hynek.me/articles/
[25]: https://hynek.me/talks/
[26]: https://www.python.org/psf/fellows-roster/
[27]: https://www.instagram.com/p/CCWlkRRqjXP/
[28]: https://hynek.me/say-thanks/
