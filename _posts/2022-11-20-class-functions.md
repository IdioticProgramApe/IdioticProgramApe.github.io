---
title: Class Functions
author: ipa
date: 2022-11-20
categories: [Python Language]
tags: [coding, python, meta-programming]
img_path: /python/meta-programming/
---

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

### Pipeline

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

### Recursive Callability

```python
class recursion:
    def __call__(self):
        return self
    
func = recursion()
func()()()()()()()...   # infinite calls :p
```

## Lambdas

Lambda functions have no name and simple (one line code)

```python
def add5(num):
    return num + 5

lambda_add5 = lambda num: num + 5
```

### Map

`map` is a built-in function to apply a function to evert element of a sequence: `list`, `tuple`, `range`, etc

The function that `map` takes in should return the mapped value

```python
', '.join(range(10))  # crash!
', '.join(map(str, range(10)))
```

`map` is use a lazy implementation, it will not execute the code until the result is specifically required:

```python
map(print, range(2))    # will return a map object
list(map(print, range(2)))   # will print 0 and 1, and a list of [None, None]
```

### Sorting

`sorted` function is used to sort some objects by certain attributes

```python
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
points = [Point(0, 1), Point(1, 0), Point(-4, 1), Point(100, -2)]
sorted(points)  # will be sorted by x value
sorted(points, key=lambda p: p.y)  # will be sorted by y value

import math
sorted(points, key=lambda p: math.hypot(*p))  # sort the points based on the hypotenuse value
```

### Filter

`filter` will take 2 arguments: a predicate and a sequence

```python
array = list(range(25))

# want to get the even numbers that less than 10
out = []
for i in array:
    if i % 2 == 0:
        if len(str(i)) < 2:
            out.append(i)

# use `filter` to achieve the same thing 
list(filter(lambda i: len(str(i)) < 2, filter(lambda i: i % 2 == 0, array)))
```

### Reduce

`reduce` is used to reduce a sequence to a single value by applying the following algorithm:

- `f(...f(f(s[0], s[1]), s[2])..., s[n]) -> value`
- in other words, repeatedly apply function to a sequence to get a single value at the end, the function should take in 2 params and return one value

```python
from functools import reduce
# the same as 0 + 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9
reduce(lambda a, b: a + b, range(10))
```
