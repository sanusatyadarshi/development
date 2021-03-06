# Python Standards

- [Style](#style)
- [Linting](#linting)
- [Imports](#imports)
- [Dependency support/testing matrix](#dependency-supporttesting-matrix)
  - [Considerations](#considerations)
  - [Specifying dependencies in `requirements.txt` or `setup.py`](#specifying-dependencies-in-requirementstxt-or-setuppy)

## Style

All of our Python code should match the [PEP8 style guide](https://www.python.org/dev/peps/pep-0008/) and the [Django coding style guidelines](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/).

We follow [PEP8's line length](https://www.python.org/dev/peps/pep-0008/#maximum-line-length) instead of Django's for consistency with our other development standards. We follow [Django's recommended import ordering in Django projects](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/#imports).

We use [Black](http://black.readthedocs.io) to standardize formatting of our Python code and make linting easier. We override the default Black line length to 79 to match PEP8. We also exclude generated files like migrations. We provide a sample [`pyproject.toml` file containing line length and exclude configuration for Black](../pyproject.toml).

## Linting

For linting we recommend [flake8](http://flake8.pycqa.org/en/latest/), and provide a [sample flake8 configuration](../.flake8).
Exceptions to PEP8 are found in that [sample flake8 configuration](../.flake8), and include:

- `E731`, we allow assignment of `lambda` expressions
- `W503`, we allow line breaks after binary operators
- `W504`, we allow line breaks before binary operators

On the whole, generated Python files like Django migrations can be safely ignored.

The flake8 and configuration can go into a `tox.ini` file under a `[flake8]` header.

It is highly recommended to configure flake8 linting in your editor of choice to use the configuration file above or one provided in the project you're working on.

## Imports

See [Imports in PEP8](https://www.python.org/dev/peps/pep-0008/#imports) and [Imports in Django Coding Style](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/#imports).

We use [isort](https://github.com/timothycrosley/isort) to lint imports to comply with both style guidelines, and provide a [sample isort configuration](../.isort.cfg). This configuration attempts to enforce the following conventions:

- Multiple-line `from` imports become grouped within parenthesis with one imported name per-line and a trailing comma for the last imported name.
- Django imports are included as their own section between the standard library and third-party imports.
- For Wagtail-specific projects like [cfgov-refresh](https://github.com/cfpb/cfgov-refresh), Wagtail imports are included as their own section between Django and third-party imports.

The isort configuration can go into a `tox.ini` file under an `[isort]` header.

## Dependency support/testing matrix

We have two general types of Python projects:

- Applications that have a running instance, e.g., [cfgov-refresh](https://github.com/cfpb/cfgov-refresh)
- Libraries that provide some functionality independent of applications

We use a rubric like the one below to determine which versions of Python and major dependencies should be supported. Assuming the project has Django and Wagtail as its major dependencies, our recommendation would be this:

|| Library | Application |
|--|--|--|
|Oldest Python 3 supported by Django LTS | ✔️| |
|Current production Python |  | ✔️ |
|Latest Python | ✔️ | ✔️ |
|Latest Django LTS version | ✔️ | ✔️ |
|Latest Django version |  ✔️ | ✔️ |
|Latest Wagtail LTS version | ✔️ | ✔️ |
|Latest Wagtail version |  ✔️ | ✔️ |

Our applications should support the version of Python 3 we currently run in production and the latest version. For example, [cfgov-refresh](https://github.com/cfpb/cfgov-refresh) should support version 3.6 and 3.8 at time of writing.

Our libraries should support the oldest version of Python 3 that the library's major dependencies supports, and the latest version of Python. For example, [Django 2.2 supports Python 3.5 and up](https://docs.djangoproject.com/en/2.2/releases/2.2/#python-compatibility), so our Django and Wagtail libraries that support Django 2.2 should do the same.

"Major dependencies" are the frameworks upon which our applications and libraries are built. Examples are Django and Wagtail. Both our applications and our libraries should support the LTS (long-term support) version of that dependency we currently run in production, and the latest version (LTS or not) of that dependency.

### Considerations

We recommend using [tox](https://tox.readthedocs.io/en/latest/) for matrix testing against all the versions listed above, and we provide a [sample tox configuration](../tox.ini) that tests against both our current production the latest versions Python and Django.

When dealing with support for multiple versions of Python or major dependencies, we prefer to [use feature detection instead of version detection](https://docs.python.org/3/howto/pyporting.html#use-feature-detection-instead-of-version-detection) for incompatibilities/breaking changes that may exist between versions.

### Specifying dependencies in `requirements.txt` or `setup.py`

The difference between pinned versions of dependencies in `requirements.txt`  and dependencies specified in `setup.py` is frequently confusing and often misunderstood. The difference is between *concrete* and *abstract* dependencies, which is most easily understood in terms of applications (intended to be deployed and run), which specify *concrete* dependencies, and libraries (reusable code that may be included in applications), which specify *abstract* dependencies.

[Donald Stufft has an excelent article with a more in-depth discussion of this difference](https://caremad.io/posts/2013/07/setup-vs-requirement/). For our purposes, we use both requirements files and requirements specified in `setup.py` depending on the circumstance of the dependency.

Our applications should use requirements files with pinned versions. For example, [cfgov-refresh has several requirements files](https://github.com/cfpb/cfgov-refresh/tree/master/requirements) that pin all relevant versions for deployment, testing, local execution, and other specific needs. Some of these requirements files inherit from each other (e.g., [`base.txt` uses the `-r libraries.txt` directive](https://github.com/cfpb/cfgov-refresh/blob/master/requirements/base.txt#L3) to include the requirements from that file).

Our libraries should specify dependencies with supported version ranges ([and those ranges should be tested with tox](../tox.ini)) in `setup.py` using [the appropriate setuptools `_requires` keyword](https://setuptools.readthedocs.io/en/latest/setuptools.html#declaring-dependencies). For example, our [wagtail-sharing library](https://github.com/cfpb/wagtail-sharing) declares install-time requirements with `install_requires` and testing requirements with `extras_require={'testing': testing_extras}`.
