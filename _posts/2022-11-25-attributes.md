---
title: Attributes
author: ipa
date: 2022-11-25
categories: [Python Language]
tags: [coding, python, meta-programming]
img_path: /python/meta-programming/
---

## Regular Attributes

```python
class Foo:
    integer = 1
   	a_list = []
    foo = lambda x: x
    
f1 = Foo()
f2 = Foo()

# set an attribute
f1.integer = 4
print(f1.integer)   # output 4
print(f2.integer)   # output 1
print(Foo.a_list)   # output 1
print(f1.__dict__)  # output {'integer': 4}
print(f2.__dict__)  # output {}

# modify an attribute
f1.a_list.append(10)
print(f1.a_list)    # output [10]
print(f2.a_list)    # output [10]
print(Foo.a_list)   # output [10]
print(f1.__dict__)  # output {}
print(f2.__dict__)  # output {}

# call a method
f1.foo(3)           # TypeError
f1.foo()            # <__main__.Foo object at ...>
Foo.foo = lambda self, x: x
f2.foo(3)           # output 3
Foo.foo(2, 3)       # output 3
```

## Inheritance

```python
class Bar(Foo):
    subfield = 'sub'
    
b = Bar()
b.z = 1
print(b.__dict__)    # output {'z': 1}
print(Bar.__dict__)  # output mappingproxy({..., 'subfield': 'sub', '__doc__': None})
print(Foo.__dict__)  # output mappingproxy({..., 'integer': 1, 'a_list': [10], 'foo': <function <lambda> at ...>, ...})
```

In python, every class contains the elements in their `__dict__`:

```text
>>> object.__dict__
mappingproxy({'__repr__': <slot wrapper '__repr__' of 'object' objects>,
              '__hash__': <slot wrapper '__hash__' of 'object' objects>,
              '__str__': <slot wrapper '__str__' of 'object' objects>,
              '__getattribute__': <slot wrapper '__getattribute__' of 'object' objects>,
              '__setattr__': <slot wrapper '__setattr__' of 'object' objects>,
              '__delattr__': <slot wrapper '__delattr__' of 'object' objects>,
              '__lt__': <slot wrapper '__lt__' of 'object' objects>,
              '__le__': <slot wrapper '__le__' of 'object' objects>,
              '__eq__': <slot wrapper '__eq__' of 'object' objects>,
              '__ne__': <slot wrapper '__ne__' of 'object' objects>,
              '__gt__': <slot wrapper '__gt__' of 'object' objects>,
              '__ge__': <slot wrapper '__ge__' of 'object' objects>,
              '__init__': <slot wrapper '__init__' of 'object' objects>,
              '__new__': <function object.__new__(*args, **kwargs)>,
              '__reduce_ex__': <method '__reduce_ex__' of 'object' objects>,
              '__reduce__': <method '__reduce__' of 'object' objects>,
              '__subclasshook__': <method '__subclasshook__' of 'object' objects>,
              '__init_subclass__': <method '__init_subclass__' of 'object' objects>,
              '__format__': <method '__format__' of 'object' objects>,
              '__sizeof__': <method '__sizeof__' of 'object' objects>,
              '__dir__': <method '__dir__' of 'object' objects>,
              '__class__': <attribute '__class__' of 'object' objects>,
              '__doc__': 'The base class of the class hierarchy.\n\nWhen called, it accepts no arguments and returns a new featureless\ninstance that has no instance attributes and cannot be given any.\n'})
```

And, everything an instance can access can be known from `dir` function (using reflection):

```text
>>> dir(b)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a_list', 'foo', 'integer', 'subfield']
```

### Takeaway

1. Attribute **access** follows the order as:
   1. instance `__dict__`
   2. class `__dict__`
   3. mro's `__dict__`
2. Attribute **set**:
   1. **only** on the instance

## `hasattr`, `getattr`, `setattr`

```python
print(hasattr(f1, 'z'))  # output False
print(hasattr(b, 'z'))   # output True
print(getattr(b, 'z'))   # output 1
setattr(f1, 'z', 2)
print(f1.__dict__)       # output {'z': 2}
print(getattr(f1, 'z'))  # output 2
```

`__getattr__` is fallback in case attribute not found normally, here is an example:

```python
class InfiniteAttr:
    def __getattr__(self, name):
        setattr(self, name, InfiniteAttr())
        return getattr(self, name)
```

it can create an `InfiniteAttr` instance once we try to access an attribute that its outer instance doesn't have this attribute. 

class `Mock` does something very similar to `InifiniteAttr`, [`Mock`](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock) is used for unit testing to be able to represent tested object.

### Immutable Class

#### Trial 1

```python
class Immutable1:
    def __init__(self, x):
        self.x = x
    def __setattr__(self, name, val):
        raise AttributeError(
            f"Cannot set {name}, {type(self).__name__} is immutable"
        )
```

With this implementation, the program will crash with `AttributeError` instantly when we try to instantiate from `Immutable1`, since we are trying to call `__setattr__` when execute `self.x = x`

#### Trial 2

```python
class Immutable1:
    def __init__(self, x):
        self.x = x
    def __setattr__(self, name, val):
        if hasattr(self, name):
            raise AttributeError(
                f"Cannot set {name}, {type(self).__name__} is immutable"
            )
        else:
            setattr(self, name, val)
```

With this implementation, we will be stuck in a infinite loop of `setattr`, and program will crash because of stack overflow (`RecursionError`).

#### Trial 3

```python
class Immutable1:
    def __init__(self, x):
        self.x = x
    def __setattr__(self, name, val):
        if hasattr(self, name):
            raise AttributeError(
                f"Cannot set {name}, {type(self).__name__} is immutable"
            )
        else:
            self.__dict__[name] = val
```

This implementation can partially achieve the immunity, however, it cannot prevent from user to change value of an attribute by modifying `__dict__`.

A proper way to create a immutable class is to inherit from some immutable data structure like [`NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple) or regular `tuple`
