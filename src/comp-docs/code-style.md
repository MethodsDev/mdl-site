# Python Code Style

This is an opinionated guide to code and packaging style for MDL tools written in Python. The goal of these rules is to make our code easier to read, maintain, and develop.

The rules can be summarized thusly:

 1. Code organized into an installable package with the [src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)
 1. [`ruff`](https://beta.ruff.rs/docs/) for linting with `["E", "F", "I", "A"]` selected
 1. [`black`](https://black.readthedocs.io/en/stable/index.html) for formatting, with default options
 1. [`pre-commit`](https://pre-commit.com/) to run the above before every commit, so PRs are always correctly formatted
 1. [`pyproject.toml`](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/) to specify package info, as well as config for tools (`pre-commit` still needs its own file)
 1. Dependencies go in `requirements.txt`, developer tools go in `dev-requirements.txt`. Both are used directly in the `pyproject.toml` file (_i.e._ do not repeat the list of requirements in that file)

For an example of this setup, check out the configuration of [isoscelles](https://github.com/MethodsDev/isoscelles)
