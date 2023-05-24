---
title: Decorators
author: ipa
date: 2022-11-22
categories: [Python Language]
tags: [coding, python, meta-programming]
img_path: /python/meta-programming/
---

## Decorators? Closures?!

**Usually**, decorators is a subset of closures, take the following example, `func` is the free variable captured by the closure:

```python
def decorator(func):
    ...
    def wrapper(*args, **kwargs):
        ...
        return func(*args, **kwargs)
    ...
    return wrapper
```

A standard workflow for a decorator should be:

1. take in a function
2. wrap that function, wrapper should match taken-in function's signature
3. return the wrapper

## Simple Decorator

```python
def debug(func):
    print("decorating", func.__name__)
    def wrapper(*args, **kwargs):
        print(func.__name__, 'being called on', args, kwargs)
        return func(*args, **kwargs)
    return wrapper

def foo(a, b, c): ...
foo = debug(foo)  # get the decorated foo function
```

Careful with the unexpected recursion, in the previous example, if we decorated `print`, `print = debug(print)`, the recursion error will be raised when we call `print` after.

- solution, change the function name after the decoration

### Symbol `@`

Python provides some special syntax for the function decoration when it's defined:

```python
@debug
def foo(a, b, c): ...
```

It's syntactic sugar to replace `foo = debug(foo)`

## Parameterized Decorator

Parameterized Decorator is used when we want decorator to perform differently based on the given option/parameter.

```python
def debug_prefix(prefix):
    print("receiving prefix:", prefix)
    def debug(func):
        print("decorating", func.__name__)
        def wrapper(*args, **kwargs):
            print(func.__name__, 'being called on', args, kwargs)
            return func(*args, **kwargs)
        return wrapper
    return debug

@debug_prefix("blabla...")
def foo(a, b, c): ...
```

One more layer is needed for passing the parameters into the `wrapper`.

## Decorating Class

Class is basically functions, when we call a class, we are calling the constructor of the class

```python
import inspect

def debug_class(cls):
    print("decorating", cls.__name__)
    class wrapper(*cls.__bases__):
        pass
    # use some reflections to populate the wrapper class
    for key, val in inspect.getmembers(cls, predicate=inspect.isfunction):
        setattr(wrapper, key, debug(val))
    return wrapper
```

Some more example on `partial`

```python
from functools import partial

@partial(partial, c=10)
def foo(a, b, c):
    return a + b + c

foo(1, 2)  # output 13

@partial(partial, partial)
def bar(a, b):
    return a + b

bar(1, 2)  # output something like functools.partial(<function foo at 0x000001C8B26D2A60>, 1, 2)
bar(1, 2)()  # output 3
```

`partial(partial, partial)` in python is evaluated as a compiler-compiler thing. 

Other examples on some iterative function:

```python
# a decorator to make fib func run faster
def make_fast(func):
    mem = {}
    def wrapper(n):
        if n in mem:
            return mem[n]
        mem[n] = func(n)
        return mem[n]
    return wrapper

# another more general solution
from functools import lru_cache
make_fast = lru_cache(None)  # don't set limitation to the memoization
```

For more details about `lru cache`, which refers to least recently used cache, ref: [What is an LRU Cache?](https://medium.com/@vienchitang/what-is-an-lru-cache-3e8ad1853584)

## Application - Aspect Oriented Programming

**AOP** separate cross-cutting concerns from vertical concerns. ref: [Aspect-oriented programming - Wikipedia](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

