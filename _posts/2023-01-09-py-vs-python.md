---
title: Py vs. Python
author: ipa
date: 2023-01-09
categories: [Python Language]
tags: [coding, python, design]
---

> TLDR:
> - `python` is the python executable
> - `py` is considered as a python launcher (picks the newest version)
> - `#!/usr/bin/env` comment is used for `py`, not for `python`
> - original answer: [What is the difference between 'py' and 'python' in the Windows terminal](https://stackoverflow.com/questions/50896496/what-is-the-difference-between-py-and-python-in-the-windows-terminal)
{: .prompt-tip }

## On Windows

`python` is the Python executable of the Python installation which you have selected as a default during the installation. This basically put the path to that version inside the PATH, so that the executable is directly available.

`py` is the *Python launcher* which is a utility that comes with Python installations on Windows. It gets installed into `C:\Windows\` so it’s available without requiring PATH modifications. The Python launcher detects what Python versions are installed on your machine and is able to automatically delegate to the right version. By default, it will use the latest Python version that is on your machine. So if you have installed 2.7, 3.5 and 3.6, running `py` will launch 3.6. You can also specify a different version by doing e.g. `py -3.5` to lauch 3.5, or `py -2` to launch the latest Python 2 version on your machine.

You can read more about the launcher [in the documentation](https://docs.python.org/3/using/windows.html#launcher). If using `py` to pip install some modules, need to be more specific about where to install these modules, Every Python installation comes with its own directory where pip modules are installed in. So if you e.g. launch IDLE for Python 3.5, you need to make sure that you also run pip with Python 3.5 (e.g. `py -3.5 -m pip install`).

## On Linux

`python` is a symlink to the default Python installation on your machine. For many Linux machines, this will only be Python 2. Even distributions that no longer come with Python 2 but only ship Python 3 will not use `python` for Python 3 since the general expectations of tools would be that `python` is a Python 2. So they may only have a `python3` symlink.

`py` usually does not exist on Linux, unless you set an alias or symlink yourself. You can check with `which python` and `which py` to see what these commands actually are.

## Anaconda

The Python version you are using is from [Anaconda](https://en.wikipedia.org/wiki/Anaconda_(Python_distribution)), which is a different Python distribution targeted at data scientists that comes bundled with quite a few things. It uses a different Python version that is separate from the official CPython releases that are available from [python.org](https://python.org/). I assume that those versions simply won’t be available through the Python launcher by default.

## Additional Information

There's a `#!/usr/bin/env python2` comment can be added at the top of Python files to tell it what version of Python to use.

`python` command line command ignores the comment. `py` parses the comment and uses the correct version.
