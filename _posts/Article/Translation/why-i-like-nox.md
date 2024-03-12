---
title: Why I Like Nox
authorURL: ""
originalURL: https://hynek.me/articles/why-i-like-nox/
translator: ""
reviewer: ""
---

# Why I Like Nox

<!-- more -->

2023-01-18 2023-06-05

Ever since I got involved with open-source Python projects, _[tox][1]_ has been vital for testing packages across Python versions (and other factors). However, lately, I’ve been increasingly using _[Nox][2]_ for my projects instead. Since I’ve been asked _why_ repeatedly, I’ll sum up my thoughts.

I can’t stress enough that I don’t want to discourage anyone from using _tox_. _tox_ is amazing. Without _tox_, the Python open-source ecosystem wouldn’t be where it is. Its authors and maintainers have my eternal gratitude!

I viscerally dislike saying negative things about _tox_, but it’s impossible to explain my preference without contrasting features and behaviors.

This is not a call for abandonment of _tox_ (that I still use with many projects), but an explanation of why I prefer Nox in certain situations. Neither Nox nor _tox_ are categorically better than the other – just different.

## Configuration Formats

The most visible difference between _tox_ and Nox is that _tox_ is a [DSL][3] on top of the venerable [INI format][4] (`tox.ini`)[1][5], while Nox uses a Python file (`noxfile.py`).

In case you aren’t familiar with either one, a simple `tox.ini` like this:

```ini
[tox]
env_list = py310,py311

[testenv]
extras = tests
commands = pytest {posargs}
```

…would look like this in a `noxfile.py`:

```python
import nox

@nox.session(python=["3.10", "3.11"])
def tests(session):
    session.install(".[tests]")

    session.run("pytest", *session.posargs)
```

You may notice a difference in nomenclature: What _tox_ calls _environments_ are _sessions_ to Nox.

---

Now, if you call `tox` or `nox`, they both:

1.  Create virtual environments for Python 3.10 and Python 3.11,
2.  install the current package (`.`) along with its extra dependencies `tests` in them,
3.  and run `pytest` from each of them.

The `{posargs}` and `*session.posargs` bits allow you to pass command line arguments to the test runners. Therefore, to make _pytest_ abort after the first error, you could write `nox -- -x` or `tox -- -x`, respectively.

---

Using Python here might look like a regression to some. Aren’t we _just_ migrating from `setup.py` to `pyproject.toml` to get rid of Python-for-configuration?

Yes and no. The problem with `setup.py` is not that it’s Python, but that it runs uncontrollably on installations.

Running commands – and thus code – on demand is the _raison d’être_ for both _tox_ and Nox; the only difference is how they are defined. And since _tox_ uses an own language to define those commands, you need dedicated features in its DSL to achieve anything. Meanwhile if you want to do something in Nox, you usually just have to write some Python.

---

Admittedly, one of the dominant reasons why I like Nox is that returning to a nontrivial `tox.ini` after a longer time has become a challenge for me. Just recently, I’ve noticed that in [_environ-config_][6] the _tox_ environment that was supposed to check the test suite passes with the oldest-supported version of [_attrs_][7] doesn’t work anymore. I’ve defined it like this:

```ini
[testenv]
extras = tests
deps = oldestAttrs: attrs==17.4.0
commands = pytest {posargs}
```

But although _tox_ _does_ install _attrs_ 17.4.0 first, it overwrites it with the latest version when installing the project. Why? I never figured it out, but I’m 99.9% sure it _used_ to work[2][8]. None of the other dependencies need a newer version and it still looks correct to me.

## INI Inheritance vs. Python Functions

If you squint enough, you realize that – syntax aside – the two configuration principles are a case of **code sharing via subclassing** vs. **code sharing via functions**. In _tox_, you define a base `testenv` from which all others inherit, but can override any field. This behavior alone is already something that occasionally leaves me scratching my head.

Re-use among sub-environments (e.g. between `py37` and `py38`) in _tox_ is done using factor-dependent statements (like `oldestAttrs:` above) or substitutions like `{[testenv:py37]commands}` whose syntax I can never remember and always send me chasing for examples in my other projects.

In Nox, if you want to reuse, you write functions. There’s no other language to learn, [just an API][9]. For instance, to run the oldest and newest Python versions under [_Coverage.py_][10], the rest without[3][11], and additionally run the oldest with a pinned _attrs_ dependency, I’ve come up with the following:

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

Now, if there were other environments (like Mypy or docs), I could run only tests using `nox --tags tests`.

In terms of the number of lines, this is longer than the _tox_ equivalent. But that’s because it’s more _explicit_ and anyone with a passing understanding of Python can deduce what’s happening here – including myself, looking at it in a year. Explicit can be good, actually.

## The Power of the Snake

Of course, Nox is a lot more powerful than _tox_ out-of-the-box, courtesy of Python. In the end, you’ve got the whole standard library at your disposal! You can read and write files, create temporary directories, format strings, make HTTP requests, … all without relying on platform features.

With _tox_, these are things that you often need to write a shell script (that probably doesn’t work on Windows) and call it from your `tox.ini`. Because _tox_ doesn’t wrap its calls in a shell (unlike, say [Hatch][12]), you’re pretty limited in what you can do: no pipes, no subcommands, no output redirection.

The only (purely ergonomic) downside of Nox is that it forces you to use the non-shell version of [`subprocess.run()`][13]. This can sometimes lead to rather brutalist command lines:

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

But given the problems of shell-wrapping (c.f. [Docker][14] or [variable name sanitization][15]), this is probably a net positive nevertheless. Even if I had to switch [Black][16] off (`# fmt: off`) so it doesn’t get _too_ gross.

## Bonus Tip: Python Versions as First-Class Selectors

As [James Bennett][17] astutely [observed][18], one cool feature of Nox is that Python versions are first-class selectors for sessions – while for _tox_ it’s just a factor like any other.

That means that you can call `nox --python 3.10` and _all_ sessions that are marked for Python 3.10 run. That’s _super_ useful in CI where you don’t need to map from [`setup-python`][19]’s version numbers (“3.11”) to _tox_’s environments (`py311` – either by hand or using one of [_tox-gh_][20] or [_tox-gh-actions_][21]). For instance with GitHub Actions, you could write:

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

The new-ish `allow-prereleases: true` line allows you to install pre-release versions like as of writing `3.12.0b1`.

---

_tox_ 4 has the concept of selecting single factors using `-f`; therefore you can run all your 3.10 environments using `tox -f py310`. The version numbers don’t match that well, but [can be made][22] except for PyPy. I use the following step:

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

However, it’s _not_ the same! `tox -f py310` will only run environments starting with `py310` (e.g. `py310` or `py310-foo`), not everything that is defined to use Python 3.10 (e.g. an environment called `docs` that sets `base_python = py310`).

## Conclusion

Again, this article is _**not**_ a call to abandon _tox_ and move all your projects to Nox – I haven’t done that myself and I don’t plan to. Neither Nox nor _tox_ are categorically better than the other, but if the mentioned issues resonate with you, there’s an option!

And if you stay on _tox_, I’ve [written how to make it 75% faster][23] when running locally.

---

1.  [At least it’s not YAML.][24] [↩︎][25]
    
2.  I _did_ try to pin `tox<4` to no avail. [↩︎][26]
    
3.  Running code under Coverage slows it down – sometimes _considerably_. [↩︎][27]
    

Found this helpful? Please consider [supporting my work][28]!

Want more content like this? Here's my free, low-volume, non-creepy [_Hynek Did Something_ newsletter][29]! It allows me to share my content directly with you and add extra context:

---

[![Hynek Schlawack](https://static.hynek.me/img/avatar_w200h200.jpg)][30]

## Hynek Schlawack

Code Bohemian in ❤️ with Python 🐍, Go 🐹, and DevOps 🔧. [blogger][31] 📝, [speaker][32] 📢, PSF [fellow][33] 🏆, [big city beach bum][34] 🏄🏻, substance over flash 🧠.

Is my content helpful and/or enjoyable to you? Please consider [supporting me][35]! Every bit helps to motivate me in creating more.

[1]: https://tox.wiki/
[2]: https://nox.thea.codes/
[3]: https://en.wikipedia.org/wiki/Domain-specific_language
[4]: https://en.wikipedia.org/wiki/INI_file
[5]: #fn:1
[6]: https://github.com/hynek/environ-config
[7]: https://www.attrs.org/
[8]: #fn:2
[9]: https://nox.thea.codes/en/stable/config.html
[10]: https://coverage.readthedocs.io/
[11]: #fn:3
[12]: https://github.com/pypa/hatch/issues/701
[13]: https://docs.python.org/3/library/subprocess.html#subprocess.run
[14]: /articles/docker-signals/
[15]: /articles/macos-dyld-env/
[16]: https://black.readthedocs.io/
[17]: https://www.b-list.org
[18]: https://lobste.rs/s/983b6p/why_i_like_nox#c_zrnxmp
[19]: https://github.com/actions/setup-python
[20]: https://pypi.org/project/tox-gh/
[21]: https://pypi.org/project/tox-gh-actions/
[22]: https://fosstodon.org/@adamchainz/109717334764977122
[23]: ../turbo-charge-tox/
[24]: https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell
[25]: #fnref:1
[26]: #fnref:2
[27]: #fnref:3
[28]: /say-thanks/
[29]: https://buttondown.email/hynek
[30]: /about/
[31]: /articles/
[32]: /talks/
[33]: https://www.python.org/psf/fellows-roster/
[34]: https://www.instagram.com/p/CCWlkRRqjXP/
[35]: /say-thanks/