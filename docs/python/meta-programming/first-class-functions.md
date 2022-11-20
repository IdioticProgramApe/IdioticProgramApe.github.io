---
layout: default
title: First-Class Functions
parent: Meta-Programming
grand_parent: Python
nav_order: 3
permalink: /docs/python/meta-programming/first-class-functions/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Functions ? Objects !

Since in python everything is object, therefore functions make no exception, we can:

- assign functions to variables
- call methods on functions
- pass functions into functions
- return functions from functions

```python
from statistics import mean

some_func = mean
some_func([1, 2, 3])
some_func = sum
some_func([1, 2, 3])

some_func.__eq__(sum)
some_func.__str__()

def mean_or_sum(num):
    assert num >= 0, "only support positive number"
    if num == 0:
        return mean
    return sum

mean_or_sum(0)([1, 2, 3])
mean_or_sum(1)([1, 2, 3])
```

`=` variable assignment in python is basically by reference. 

## Application

### State Machine

```text
init ---> inc <---> output ---> end
```

```python
from time import sleep
global count, state

def init():
    print("init")
    global count, state
    count = 0
    state = inc

def inc():
    print("inc")
    global count, state
    count += 1
    state = output

def output():
    global state, count
    print(f"outputting {count}")
    if count >= 5:
        state = end
    else:
        state = inc

def end():
    print("end")
    raise StopIteration

def iterate():
    global state
    state = init
    while True:
        try:
            state()
            sleep(1)
        except StopIteration:
            break
            
iterate()
```

> notice here in the snippet there is a lot of `global` which means variable `state` and `count` are in the global namespace, a more appropriate approach would be put this into a class, since `self` is shared between class methods

  ## Pipeline

```python
class Processer:
    def __init__(self, x):
        self.x = x
        self.steps = [
            self.add_one,
            self.add_two,
            self.subtract_one,
            self.add_one,
        ]
    def add_one(self):
        self.x += 1
    def add_two(self):
        self.x += 2
    def subtract_one(self):
        self.x -= 1
    def process(self):
        for step in self.steps:
            step()
        return self.x
    
class OneLessProcessor(Processor):
    def __init__(self, x):
        super().__init__(x)
        self.steps.append(self.substract_one)
```

## `__call__` Method

`__call__` is a class magic method, can turn any class instance into a regular function/callable object.

```python
# regular function
def reverse(in_string):
    out = ''
    for c in in_string:
        out = c + out
    return out

reverse('abcd')
type(reverse)  # function
type(reverse.__call__)  # method wrapper
type(reverse.__call__.__call__)  # method wrapper, infinite loop...

# use `__call__`, it's become a functor
class Reverse:
    def __call__(self, in_string):
        out = ''
        for c in in_string:
            out = c + out
        return out
    
another_reverse = Reverse()
another_reverse('abcd')
type(another_reverse)  # Reverse
type(another_reverse.__call__)  # method
type(Reverse.__call__)  # function
```

### Function State

- since `__call__` takes in `self`, we can use `self` for functional state
- functions with internal memory

```python
class inc:
    def __init__(self):
        self.num = 0
    def __call__(self):
        self.num += 1
        return self.num
```

### Cached Function

If already calculated a result, don't recalculate it again, cache it instead

```python
class FibSolver:
    def __init__(self):
        self.memory = {0: 0,  1: 1}
    def __call__(self, num):
        if num in self.memory:
            return self.memory[num]
        self.memory[num] = self(num - 1) + self(num - 2)
        return self.memory[num]

fib = FibSolver()
print(', '.join(str(fib(i)) for i in range(18)))
```

