---
title: Abstract#
date: 2024-06-02T14:34:51.726Z
authorURL: ""
originalURL: https://frostming.com/en/2021/pm-review-2021/
translator: ""
reviewer: ""
---

# Abstract[#][1]

<!-- more -->

It is 2021 and we are all using or heard of package managers in Python, among which are Pipenv and Poetry. I also built a new package manager [PDM][2] to solve similar problems. There exist some comparisons between them around the community, but this article is not going to talk about the user interface or their versatility, it is going to focus on two important aspects: performance and correctness.

# Setup[#][3]

**Pipenv**: `HEAD@275f7e151eb0aa17702215165a371df7da9ad476`

**Poetry**: `1.1.5`

**PDM**: `HEAD@d72ba5d2ee0b0305f917d8739667ba78465c5cc8`

**Python version**: `3.9.1`

# Performance[#][4]

Dependency set(in poetry format):

```
[tool.poetry.dependencies]
python = "^3.9"
requests = { git = "https://github.com/psf/requests.git", tag = "v2.25.0" }
numpy = "^1.19"

[tool.poetry.dev-dependencies]
pytest = "^5.2"
```

This includes a Git dependency and a heavy binary dependency. In the following result, the cost times are measured in seconds.

Unless explicitly given, the measurements are done against the `install` command.

## Result[#][5]

|  | Pipenv | Poetry | PDM |
| --- | --: | --: | --: |
| Clean cache, no lockfile | 98 | 150 | 58 |
| With cache, no lockfile | 117 | 66 | 28 |
| Clean cache, reuse lockfile\* | 128 | 145 | 35 |
| With cache, reuse lockfile\*\* | 145 | 50 | 16 |

**\***: Test with command `poetry add click` `pipenv install --keep-outdated click` `pdm add click` respectively.

**\*\***: Commands are the same as above.

## Performance Review[#][6]

-   Pipenv has a problematic cache system, which slows down the performance with the existence of caches
-   Poetry and PDM both benefit a lot from the caches, PDM takes even less time.
-   Pipenv uses a very different mechanism to reuse the lock file — it runs full locking first then modifies the content of the old lock file, while PDM can reuse the pinned versions in the lock file. Poetry improves a little with the lock file existing.

# Correctness[#][7]

The goal of these 3 package managers is to produce a reproducible environment and they all try their best to make it work on cross-platforms and python versions. Let’s see how it turns out.

## Python Compatibility[#][8]

The project file is designed as following(Poetry):

```
[tool.poetry.dependencies]
python = "^3.6||^2.7"
```

And PDM:

```
[project]
requires-python = ">=2.7,!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*,!=3.4.*,!=3.5.*"
```

They both reference to the same range of Python versions to support. Although Pipenv doesn’t support a Python version range constraint, I will also post its results here as a reference.

Now let’s add one dependency `pytest` from a clean project. This dependency is chosen purposely because it has a different version set to support Python 2 and Python 3.

**Result of Poetry:**

```
$ poetry add pytest
...
Using version ^6.2.2 for pytest

Updating dependencies
Resolving dependencies...

  SolverProblemError

  The current project's Python requirement (>=2.7,<3.0 || >=3.6,<4.0) is not compatible with some of the required packages Python requirement:
    - pytest requires Python >=3.6, so it will not be satisfied for Python >=2.7,<3.0

  Because no versions of pytest match >6.2.2,<7.0.0
   and pytest (6.2.2) requires Python >=3.6, pytest is forbidden.
  So, because foo depends on pytest (^6.2.2), version solving failed.
```

Lock failed as `6.2.2` was picked but it was not compatible with python 2.7. Anyhow Poetry shows a well human-readable error message telling people what happened and how to solve it.

**Result of Pipenv**

Depending on Python 3 or Python 2 is used to create the virtualenv(with `--three/--two` option), `pytest` is locked to `6.2.2` and `4.6.11` respectively. The lock file surely can’t work on both Python 2 and Python 3 environment at the same time.

**Result of PDM**

```
$ pdm add pytest
Adding packages to default dependencies: pytest
✔ 🔒 Lock successful
...

  ✔ Install pytest 4.6.11 successful

...
🎉 All complete!
```

`pytest` is correctly resolved to `4.6.11` that supports all versions within the `requires-python` contraint.

## Version Tree Searching[#][9]

A dependency resolver should be able to search for other candidates when the current one causes version conflicts.

Let’s take the example from [Poetry’s README][10]:

**Result of Pipenv**

```
$ pipenv install oslo.utils==1.4.0
...
There are incompatible versions in the resolved dependencies:
  pbr!=0.7,<1.0,>=0.6 (from oslo.utils==1.4.0->-r C:\Users\FROSTM~1\AppData\Local\Temp\pipenvkbeeio2trequirements\pipenv-0zsj0laj-constraints.txt (line 3))
  pbr!=2.1.0,>=2.0.0 (from oslo.i18n==5.0.1->oslo.utils==1.4.0->-r C:\Users\FROSTM~1\AppData\Local\Temp\pipenvkbeeio2trequirements\pipenv-0zsj0laj-constraints.txt (line 3))
```

Unable to resolve\* since Pipenv failed to search for lower versions of `oslo.i18n` to find one that is compatible with `pbr<1.0`

**\***: Be aware that Pipenv’s strategy is “lock after install”, so the incompatible package will be installed into the environment before the lock failure is reported.

**Result of Poetry**

As illustrated in the README, poetry successfully resolves with `oslo.i18n==2.1.0` . It searches along the candidate list of `oslo.i18n` and discard those that bring conflicts.

**Result of PDM**

```
$ pdm add oslo.utils==1.4.0
...
  ✔ Install oslo.i18n 2.1.0 successful
...
```

Similarly, PDM also locks successfully with the same version of `oslo.i18n`

## Environment Marker Propagation[#][11]

To make a cross-platform project template, we often need to define some platform-specific dependencies and those also may have their subdependencies. These platform-specific dependencies may not be able to build successfully on the source system. We don’t want these dependencies and subdependencies to be installed on the systems that do not match the requirements.

Let’s start by adding a dependency `gevent; os_name == "posix"`, which has several subdependencies. Commands are run from a Windows computer.

**Result of Pipenv(Pipfile.lock)**

```
{
    "_meta": {
        "hash": {
            "sha256": "9ce84144d1fc47c173581ae74c51ffbe28ac242b1fe0e1642915e803f15d3063"
        },
        "pipfile-spec": 6,
        "requires": {},
        "sources": [
            {
                "name": "pypi",
                "url": "https://pypi.org/simple",
                "verify_ssl": true
            }
        ]
    },
    "default": {
        "gevent": {
            "hashes": [
                ...
            ],
            "markers": "os_name == 'posix'",
            "version": "==21.1.2"
        }
    },
    "develop": {}
}
```

Only `gevent` is resolved with the marker in the lock file and Pipenv stops trying to find its children dependencies when the marker doesn’t match the current system. This means if you use this Pipfile.lock to deploy on the target Linux server, some significant dependencies WILL NOT be installed!

**Result of Poetry**

`gevent` together with `greenlet`, `cffi`, `pycparser`, `zope.event`, `zope.interface` are pinned in the lock file and don’t get installed to the environment, which is the expected behavior. But I didn’t figure out how to add a requirement with markers from the CLI and I have to manually write it in `pyproject.toml`. Further, if I run `poetry add pycparser` after that, `pycparser` can be installed correctly.

**Result of PDM**

The same result as Poetry, except that in `pdm.lock`, children dependencies also have the marker `os_name == "posix"` so that installers won’t have to search the dependency tree to see whether a single package should be installed.

# Conclusion[#][12]

On the performance perspective, Pipenv doesn’t play well due to its design choice that it integrates with other third-party tools and libraries instead of building its own. Pipenv can only wrap, combine, and do a little improvement on those upstream libraries. Moreover, Pipenv doesn’t meet the goal of reproducible environment as well. It can produce a determinsitic installation setup on the source system but it is not a good idea to deploy to a different system without a careful check.

On contrast, Poetry and PDM are both doing great on performance and correctness, PDM is even better especially on the time cost and compatible dependency resolving. If you do not know this tool yet, [start now][13].

[1]: #abstract
[2]: https://frostming.com/2021/01-22/introducing-pdm/
[3]: #setup
[4]: #performance
[5]: #result
[6]: #performance-review
[7]: #correctness
[8]: #python-compatibility
[9]: #version-tree-searching
[10]: https://github.com/python-poetry/poetry#dependency-resolution
[11]: #environment-marker-propagation
[12]: #conclusion
[13]: https://pdm-project.org