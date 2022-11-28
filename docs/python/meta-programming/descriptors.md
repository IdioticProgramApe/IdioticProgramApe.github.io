---
layout: default
title: Descriptors
parent: Meta-Programming
grand_parent: Python
nav_order: 9
permalink: /docs/python/meta-programming/descriptors/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Method Binding

```python
>>> class Person:
... def __init__(self, **kwargs):
...     self.__dict__.update(kwargs)
...        
>>> p = Person(name="P", age=10)
>>> p.talk = lambda self: print(f"I'm {self.name}")
>>> p.talk()
TypeError: <lambda>() missing argument: 'self'
>>> p.talk(p)
I'm P
>>> Person.talk = lambda self: print(f"I'm {self.name}")
>>> p.talk()
TypeError: <lambda>() missing argument: 'self'
>>> del p.talk
>>> p.talk()
I'm P
>>> p.__dict__
{'name': 'P', 'age': 10}
>>> Person.talk
<function <lambda> at ...>
>>> p.talk
<bound method <lambda> of <__main__.Person object at ...>>
```

For more details, check [Descriptor HowTo Guide — Python 3.11.0 documentation](https://docs.python.org/3/howto/descriptor.html?highlight=object __getattribute__).

In short, "For objects, the machinery is in `object.__getattribute__()` which transforms `b.x` into `type(b).__dict__[x].__get__(b, type(b))`"

```python
class BindMe:
    def __get__(self, obj, type=None):
        print(f"binding {self} to {obj}, {type=}")
        return "hello descriptors"

class Test:
    x = BindMe()
```

If we try to get the value of `x` from a instance of `Test`:

```python
>>> test = Test()
>>> test.x
binding <__main__.BindMe object at 0x00000259CA294B50> to <__main__.Test object at 0x00000259CA2946D0>, type=<class '__main__.Test'>
'hello descriptors'
```

`__get__` method doesn't define how to lookup things on instances of `BindMe`, it **overrides** how to lookup instances of `BindMe` on **other classes**. If `BindMe` is ever included in any other class, its lookup procedure includes the bind print statement, and always returns the `'hello descriptors'` 

or if we try to get the value of `x` from `Test` itself:

```python
>>> Test.x
binding <__main__.BindMe object at 0x00000259CA294B50> to None, type=<class '__main__.Test'>
```

This time we are binding the `BindMe` object to `None`, this is because `Test` is not a instance any more.

## Static Methods

