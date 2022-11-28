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

In python to define a class static method, we will need to use a built-in `staticmethod` decorator, which make the method:

- can be called on instances
- can be called on class
- without `self` being passed in (as for the function signature)

```python
class ClsWithStatic:
    @staticmethod
    def foo(a, b):
        return a + b

c = ClsWithStatic()
print(c.foo(1, 2))              # output 3
print(ClsWithStatic.foo(1, 2))  # output 3
```

### Implementations

The first implementation is simple, just stripping `self` from the argument list:

```python
def naive_static(func):
    def wrapper(self, *args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

The second implementation uses the descriptor, `__get__`:

```python
class good_static:
    def __init__(self, func):
        self.func = func
    def __get__(self, obj, type=None):
        return self.func
```

With a test case:

```python
class ClsWithStatic:
    @naive_static
    def foo(a, b):
        return a + b
    
    @good_static
    def bar(a, b):
        return a + b
    
c = ClsWithStatic()
# test foo
print(c.foo(1, 2))              # output 3
print(ClsWithStatic.foo(1, 2))  # TypeError, missing b's value
# test bar
print(c.bar(1, 2))              # output 3
print(ClsWithStatic.bar(1, 2))  # TypeError, missing b's value
```

Using `good_static` decorator is equivalent to:

```python
class ClsWithStatic:
    @naive_static
    def foo(a, b):
        return a + b
    
    def bar(a, b):
        return a + b
    
    bar = good_static(bar)  # to rebind the method
```

## `__set__`

`__set__` is used to change how to change things in python:

```python
class Setter:
    def __set__(self, obj, value):
        print(f"setting {self} on {obj} to {value}")
        obj.__dict__[self.name] = value
    def __set_name__(self, owner, name):
        self.name = name
        
class TestSetter:
    x = Setter()
    def __init__(self, x):
        self.x = x
```

With the definition of `Setter` and `TestSetter` above, we can get the following results:

```python
>>> t = TestSetter(3)
setting <__main__.Setter object at 0x000001655B052610> on <__main__.TestSetter object at 0x000001655B079A30> to 3
>>> t.x = 4
setting <__main__.Setter object at 0x000001655B052610> on <__main__.TestSetter object at 0x000001655B079A30> to 4
>>> TestSetter.x
<__main__.Setter object at 0x000001655B052610>
```

The execution order is:

1. `x = Setter()`
2. `__set_name__(self, owner, name)`: 
   1. `self: <__main__.Setter object at 0x00000250312B4F40>`
   2. `name: 'x'`
   3. `owner: <__main__.Setter object at 0x00000250312B4F40>`
3. `self.x = x`
4. `__set__(self, obj, value)`

One example:

```python
class TypeChecked:
    def __init__(self, type, optional=False):
        self.type = type
    def __set__(self, obj, value):
        if not isinstance(value, self.type):
            raise TypeError(f"{self.name} must be of type {self.type}")
        obj.__dict__[self.name] = value
    def __set_name__(self, owner, name):
        self.name = name

class DescriptPerson:
    name = TypeChecked(str)
    age = TypeChecked(int)
```

here is some test code:

```python
>>> p = DescriptPerson()
>>> p.name = "P"
>>> p.age = 10
>>> p.age = '10'
TypeError: age must be of type <class 'int'>
```

Combine it with some meta class to auto initialize the member variables, check [work with globals and locals](../reflection/#work-with-globals-and-locals) for more details on `exec`:

```python
class AutoInit(type):
    def __new__(meta, name, bases, attrs):
        fields = [key for key, value in attrs.items() if isinstance(value, TypeChecked)]
        args = ", ".join(f for f in fields)
        sets = "".join(f"\n    self.{f} = {f}" for f in fields)
        code = f"def __init__(self, {args}): {sets}"
        exec(code, globals(), attrs)
        return super().__new__(meta, name, bases, attrs)

class DescriptPerson(metaclass=AutoInit):
    name = TypeChecked(str)
    age = TypeChecked(int)
```

This is similar to the implementation of [`dataclass`](https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass) and [`field`](https://docs.python.org/3/library/dataclasses.html#dataclasses.field) in module `dataclasses`.

