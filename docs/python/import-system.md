---
layout: default
title: Python Import System
parent: Python
nav_order: 6
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
## Break into Import

Overall, when import a module/package in python, there are several things happening on the background:

1. Searching for the module/package to be imported
2. Loading the found module/package
3. Bounding the loaded module/package to a namespace

The 2 following method of import `PI` from `math` module is doing the same thing:

```python
# normal method
from math import pi as PI                # ok, get the PI directly

# using importlib
import importlib
_tmp = importlib.import_module("math")   # search and load module math, and bound it to _tmp
PI = _tmp.pi                             # copy the value to PI
del _tmp                                 # destroy the _tmp namespace, PI is still valid
```

## Module Cache

Internally, python import system has a built-in caching mechanism:

```python
from math import pi as PI      # here we can only directly use PI, however, math is already loaded and cached
import sys
sys.modules["math"].cos(PI)    # output -1.0
```

`sys.modules` acts as a cache for all the imported modules/packages, it's the first place the finder will look for a module/package in case the module/package is already imported in the past.

## Singleton

We discussed about the singleton design pattern when we talked about the [singletons in meta-classes](/docs/python/meta-programming/metaclasses/#singletons), however, since python has already its own caching system which we can take advantage of to make a singleton without using meta class.

```python
# person.py
class _Person:
    def __init__(self, name):
        self.name = name
    def talk(self):
        print("I'm", self.name)
        
a = _Person("a")

# test.py
import person
person.a.talk()    # person.a is a singleton, this outputs "I'm a"
```

## Reload Modules

Again, since python has this cache system to prevent repeatedly loading the same module/package, sometimes it can generate some unexpected results especially when we are working with a interactive python interpreter.

Let's say we have a module called `Foo` and a test script associating with it:

```python
# Foo.py
name = "Foo"

# test.py
import Foo
print(Foo.name)    # outputs "Foo"
```

Later on, we found the name is not pretty enough, we decided to make it to `Bar`, and test.py remain unchanged:

```python
# Foo.py
name = "Bar"

# test.py
import Foo
print(Foo.name)    # still outputs "Foo"
```

This is because when python tried to import Foo, it found out that Foo was already imported previously (the old version), so it took the old Foo and use the old name, the solution is using `importlib.reload` method to reload the Foo module:

```python
# test.py
import importlib
importlib.reload(Foo)
print(Foo.name)    # outputs "Bar"
```

## Finders & Loaders

There are several steps when we try to import a module:

1. Python checks if the module is available in the module cache. If `sys.modules` contains the name of the module, then the module is already available, and the import process ends.
2. Python starts looking for the module using several **finders**. A finder will search for the module using a given strategy. The default finders can import **built-in** modules, **frozen** modules, and modules **on the import path**.
3. Python loads the module using a **loader**. Which loader python uses is determined by the finder that located the module and is specified in something called a **module spec**.

> from [What is a frozen Python module](https://stackoverflow.com/questions/9916432/what-is-a-frozen-python-module)
>
> Frozen modules are modules written in Python whose compiled byte-code object is incorporated into a custom-built Python interpreter by Python’s freeze utility.

[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path) controls which finders are called during the import process, here are the 3 default finders we discussed above:

```python
>>> import sys
>>> sys.meta_path 
[_frozen_importlib.BuiltinImporter,
 _frozen_importlib.FrozenImporter,
 _frozen_importlib_external.PathFinder]
```

If we delete all 3 finders in `sys.meta_path`, python will not be able to find or import any new module (not cached). The finder is a class which **must contains** a class method called `find_spec` with following signature:

```python
class SomeFinder:
    @classmethod
    def find_spec(cls, name, path, target=None):
        ...
```

where:

- `name`: the name of the module
- `path`: the value of `__path__` from the parent package, `None` if it's top-level import
- `target`: a module object that the finder may use to make a more educated guess about what spec to return

`find_spec` can terminate with 3 senarios:

- by returning `None` if it doesn't know how to find and load the module, and let other finders to figure it out (can be used as a import debugger)
- by returning a `module spec` specifying how to load the module
- by raising a `ModuleNotFoundError` to indicate that the module cannot be imported

### Debug Finder

```python
# debug_importer.py
# this module is to display all the imports when we try to import a module
import sys

class DebugFinder:
    @classmethod
    def find_spec(cls, name, path, target=None):
        print(f"importing {name!r} from {path!r}")
        return None    # can be omitted since python will return automatically None
    
sys.meta_path.insert(0, Debug)
```

This finder can help us tracking the modules which are about to be imported when we try to import a module:

```python
>>> import debug_importer.py
>>> import numpy
importing 'numpy' from None
importing 'numpy._globals' from ['c:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy']
importing 'numpy.__config__' from ['c:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy']
importing 'numpy._distributor_init' from ['c:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy']
importing 'numpy.core' from ['c:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy']
...
```

### Ban Finder

```python
# ban_importer.py
# this module is to prevent us from importing numpy and re
import sys
BANNED_MODULES = {"numpy", "re"}

class BanFinder:
    @classmethod
    def find_spec(cls, name, path, target=None):
        if name in BANNED_MODULES:
            raise ModuleNotFoundError(f"module {name!r} is banned!")

sys.meta_path.insert(0, BanFinder)
```

This finder can help us to ban some module imports, which can be handy in some situation, however, if we run the following script in run mode and debug mode using pycharm:

```python
# test.py
import ban_importer
import csv   # which references re
import numpy as np
```

the results are different:

```bash
# run mode
Traceback (most recent call last):
  File "D:/ModuleFinder/test.py", line 6, in <module>
    import csv
  File "C:\Users\ipa\Miniconda3\envs\test\lib\csv.py", line 6, in <module>
    import re
  File "D:\ModuleFinder\ban_importer.py", line 9, in find_spec
    raise ModuleNotFoundError(f"module {name!r} is banned!")
ModuleNotFoundError: module 're' is banned!

# debug mode
Traceback (most recent call last):
  File "<frozen importlib._bootstrap>", line 991, in _find_and_load
  File "<frozen importlib._bootstrap>", line 971, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 914, in _find_spec
  File "D:\ModuleFinder\ban_importer.py", line 9, in find_spec
    raise ModuleNotFoundError(f"module {name!r} is banned!")
ModuleNotFoundError: module 'numpy' is banned!
```

### Pip Finder

```python
# pip_importer.py
# this module is to auto pip install the module which cannot be found when it's imported
from importlib import util
import sys
import os
import subprocess
import inspect
import pprint

class PipFinder:
    ALREADY_PROCESSED = set()

    @classmethod
    def find_spec(cls, name, path, target=None):
        if name in cls.ALREADY_PROCESSED:
            return None
        print(f"Module {name!r} not installed. Attempting to pip install", end='...')
        cmd = f"{sys.executable} -m pip install {name}"
        try:
            subprocess.run(cmd.split(), check=True, capture_output=True)
        except subprocess.CalledProcessError:
            print("failed")
            pp = pprint.PrettyPrinter(indent=1)
            outerframes = list(filter(
                lambda frame: os.path.isfile(frame.filename),
                inspect.getouterframes(inspect.currentframe())
            ))
            pp.pprint(outerframes)
            cls.ALREADY_PROCESSED.add(name)
            return None
        print("succeeded")
        return util.find_spec(name)

sys.meta_path.append(PipFinder)
```

This finder can help us to auto install some not installed modules when we try to import them, however this importer has some drawbacks:

- some imports are written in a try block to guarantee the backwards compatibility, there is no way for a new interpreter to import that
- some module name doesn't match its package name, which can leads to an intallation failure, then a import failure

Here is an example and the output

```python
# test.py
import pip_importer
import matplotlib        # not installed
import cv2               # not installed, and package name is opencv-python
```

```bash
Module 'matplotlib' not installed. Attempting to pip install...succeeded
Module 'pickle5' not installed. Attempting to pip install...failed
[...,
 FrameInfo(frame=<frame at 0x0000020B11D5C6C0, file 'C:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy\\compat\\__init__.py', line 12, code <module>>, filename='C:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\site-packages\\numpy\\compat\\__init__.py', lineno=12, function='<module>', code_context=['from . import py3k\n'], index=0),
 ...]
Module 'org' not installed. Attempting to pip install...failed
[...,
 FrameInfo(frame=<frame at 0x0000020B12642180, file 'C:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\pickle.py', line 103, code <module>>, filename='C:\\Users\\ipa\\Miniconda3\\envs\\test\\lib\\pickle.py', lineno=103, function='<module>', code_context=['    from org.python.core import PyStringMap\n'], index=0),
 ...]
Module 'cv2' not installed. Attempting to pip install...failed
Traceback (most recent call last):
  File "D:\ModuleFinder\test.py", line 6, in <module>
    import cv2
ModuleNotFoundError: No module named 'cv2'
[...]
```

