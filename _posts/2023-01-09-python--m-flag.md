---
title: Python -m Flag
author: ipa
date: 2023-01-09
categories: [Python Language]
tags: [coding, python, design]
---

From [Meaning of python -m flag - Stack Overflow](https://stackoverflow.com/questions/50821312/meaning-of-python-m-flag):

> Consider the following scenario: you have three versions of Python installed:
> 
> - Python 3.7
> - Python 3.8
> - Python 3.9
> 
> Your "default" version is 3.8. It's the first one appearing in your *path*. Therefore, when you type `python3` (Linux or Mac) or `python` (Windows) in a shell you will start a 3.8 interpreter because that's the first Python executable that is found when traversing your path.
> 
> Suppose you are then starting a new project where you want to use Python 3.9. You create a [virtual environment](https://docs.python.org/3/library/venv.html) called `.venv` and activate it.
> 
> ```bash
> # On linux
> python3.9 -m venv .venv
> source .venv/bin/activate
> ```
> 
> ```bash
> # On windows
> py -3.9 -m venv .venv
> .venv\Scripts\activate
> ```
> 
> We now have the virtual environment activated using Python 3.9. Typing `python` in a shell starts the 3.9 interpreter.
> 
> **But**, how to install a module/package to this venv ? If you type 
> 
> ```bash
> pip install <some-package>
> ```
> 
> Then what version of `pip` is used? Is it the pip for the default version, i.e. Python 3.8, or the Python version within the virtual environment?
> 
> An easy way to get around that ambiguity is simply to use
> 
> ```bash
> python -m pip install <some-package>
> ```
> 
> The `-m` flag **makes sure that you are using the pip that's tied to the active Python executable**. It's good practice to always use `-m`, even if you have just one global version of Python installed from which you create virtual environments.
> 
> In a more general sense though, **when using `python -m some_module`, the `-m` flag makes Python execute `some_module` as a script**. This is stated in the [docs](https://docs.python.org/3/using/cmdline.html), What does it mean to "run as a script"?
> 
> In Python, a module `some_module` is typically imported into another Python file with an `import some_module` statement at the top of the importing file. This enables the use of functions, classes, and variables defined in `some_module` inside the importing file. To execute `some_module` as a script instead of importing it, you would define an `if __name__ == "__main__"` block inside the file. This block gets executed when running `python some_module.py` on the command line. This is useful because you **do not** want this code block to run when importing into other files, but you **do** want it to run when invoked from the command line.
> 
> For modules inside your project, this script/module construct should run as is, because Python will find the module from your working directory when running from the terminal:
> 
> ```bash
> python some_module.py
> ```
> 
> But for modules that are part of Python's Standard Library, this will not work. The example from the Python docs uses `timeit` (`pip` works the same):
> 
> ```bash
> python3 timeit -s 'print("hello")'  # 'python timeit.py ...' fails as well 
> ```
> 
> This returns the error: 
> > "python: can't open file '/home/<username>/timeit': [Errno 2] No such file or directory"
> 
> Adding the `-m` flag tells Python to look in path for `timeit.py` and execute the `if __name__ == "__main__"` clause from the file.
> 
> ```bash
> python3 -m timeit -s 'print("hello")'
> ```
>
> This works as expected.
{: .prompt-info }

More `-m` detail can be found here](https://docs.python.org/3/using/cmdline.html#cmdoption-m).

