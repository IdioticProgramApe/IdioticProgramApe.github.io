---
title: Reflection
author: ipa
date: 2022-11-24
categories: [Python Language]
tags: [coding, python, meta-programming]
img_path: /python/meta-programming/
---

## Definition

The reflection is pure metaprogramming (not only used for object-oriented based metaprogramming), it's an ability of a program to view and modify its own structure at runtime.

> Note: Not always true, but in python, if you think something might be possible, then it probably is and there is probably a function for it.

## Application - A Runtime Debugger

The code can be found in: [metaprogramming/reflective_debugger.py](https://github.com/max7patek/metaprogramming/blob/master/misc/code_examples/reflective_debugger.py)

### Step Over

We want our code to run every time a new line is executed by python

- use `sys.settrace(func)`
  - `func` should take in `(frame, event, arg)`, check [sys.settrace](https://docs.python.org/3/library/sys.html#sys.settrace) for detailed information about the trace function
  - `event` param stores the trace function's action, and the `arg` param depends on it:

    | Event         | Description                                                  |
    | ------------- | ------------------------------------------------------------ |
    | `'call'`      | - A function is called (or some other code block entered)<br />- The global trace function is called *arg* is `None`<br />- The return value specifies the local trace function. |
    | `'line'`      | - The interpreter is about to execute a new line of code or re-execute the condition of a loop<br />- The local trace function is called; *arg* is `None`<br />- The return value specifies the new local trace function. <br />- Per-line events may be disabled for a frame by setting `f_trace_lines` to `False` on that frame. |
    | `'return'`    | - A function (or other code block) is about to return. <br />- The local trace function is called, *arg* is the value that will be returned, or `None` if the event is caused by an exception being raised. <br />- The trace function’s return value is ignored. |
    | `'exception'` | - An exception has occurred<br />- The local trace function is called, *arg* is a tuple `(exception, value, traceback)` <br />- The return value specifies the new local trace function. |
    | `'opcode'`    | - The interpreter is about to execute a new opcode<br />- The local trace function is called *arg* is `None`<br />- The return value specifies the new local trace function<br />- Per-opcode events are not emitted by default: they must be explicitly requested by setting `f_trace_opcodes` to `True` on the frame. |
  - `func` will be called every time a function a called
  - `func` can return another function
    - this returned function also takes in `(frame, event, arg)`
    - will be called every time python executes the next line
    - need not return anything

```python
import sys

def printer(*args):
    print(args)
    return printer

sys.settrace(printer)
printer("test")
```

the output is shown below, take a note that the line numbers, it shows that the function is executing line by line, and each `trace` object is composed by 3 things, (`frame`, `event`, `argument`) 

```text
(<frame at 0x000001772D2F5DD0, line 3, code printer>, 'call', None)
(<frame at 0x000001772D2F5DD0, line 4, code printer>, 'line', None)
('test',)
(<frame at 0x000001772D2F5DD0, line 5, code printer>, 'line', None)
(<frame at 0x000001772D2F5DD0, line 5, code printer>, 'return', <function printer at 0x000001772D696A60>)
```

### Global & Local Variables

We'd like to output the information about the global and local variables at each line execution:

- there are some built-in reflective functions called:
  - `globals()`
  - `locals()` 

```python
def naive(frame, event, arg):
    print(globals(), locals())

sys.settrace(naive)
printer("test")
```

the output would be as the following, note that the local variables are from our `naive` debug function instead of the function we want to debug `printer`:

```text
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x000001FC30F23DC0>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__file__': 'D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', '__cached__': None, 'sys': <module 'sys' (built-in)>, 'printer': <function printer at 0x000001FC317E6A60>, 'naive': <function naive at 0x000001FC314C9820>} {'frame': <frame at 0x000001FC31445DD0, file 'D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', line 3, code printer>, 'event': 'call', 'arg': None}
('test',)
```

### Frame

Stack frames are how the computer keeps track of the function calls:

- each time a function is called, a frame is pushed
- each time a function terminates itself (`return`), a frame is popped

```python
def inspect_frame(frame, event, arg):
    print(dir(frame))
    
sys.settrace(inspect_frame)
printer("test")
```

This time the output becomes:

```text
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'clear', 'f_back', 'f_builtins', 'f_code', 'f_globals', 'f_lasti', 'f_lineno', 'f_locals', 'f_trace', 'f_trace_lines', 'f_trace_opcodes']
('test',)
```

There are something we want to use: `f_code`, `f_globals`, `f_lineno`, `f_locals`

#### `f_globals` & `f_locals`

we can use these 2 variables to print out the `globals` and `locals` of our `printer`, now the local variable is `args` with value `'test'`

```python
def global_local_trace(frame, event, arg):
    print(frame.f_globals, frame.f_locals, sep='\n')

sys.settrace(global_local_trace)
printer("test")
```

```text
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x0000021EACE63DC0>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__file__': 'D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', '__cached__': None, 'sys': <module 'sys' (built-in)>, 'printer': <function printer at 0x0000021EAD3B6A60>, 'global_local_trace': <function global_local_trace at 0x0000021EAD099AF0>}
{'args': ('test',)}
('test',)
```

#### `f_code`

```python
def inspect_code(frame, event, arg):
    print(frame.f_code, dir(frame.f_code), sep='\n')

sys.settrace(inspect_code)
printer("test")
```

```text
<code object printer at 0x0000021BAF21F3A0, file "D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py", line 3>
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_kwonlyargcount', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_posonlyargcount', 'co_stacksize', 'co_varnames', 'replace']
('test',)
```

there are some interesting variables in `f_code`, such as `co_code`, `co_freevars`, `co_firstlineno`...

- `co_code`: this variable stores the compiled python code

### Inspection

Here is a test script for the inspection:

```python
import itertools, functools
def foo(a, b):
    c = a + b
    print(c)
def bar(*args):
    for i in itertools.combinations(args, 2):
        foo(*i)
def nop(frame, event, arg):
    return None
end = functools.partial(sys.settrace, nop)
```

#### `inspect.getsourcelines`

To get the functions source code during runtime:

```python
def get_source(f_code):
    try:
        return inspect.getsourcelines(f_code)
    except OSError:
        return None
    
sys.settrace(lambda f, e, a: print(get_source(f.f_code)))
foo(1, 2)
```

```text
(['def foo(a, b):\n', '    c = a + b\n', '    print(c)\n'], 7)
3
```

#### `inspect.getouterframes`

To get the callstack at the runtime:

```python
sys.settrace(lambda f, e, a: print(inspect.getouterframes(f)))
foo(1, 2)
```

```text
[FrameInfo(frame=<frame at 0x0000014C64825BE0, file 'D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', line 7, code foo>, filename='D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', lineno=7, function='foo', code_context=['def foo(a, b):\n'], index=0), FrameInfo(frame=<frame at 0x0000014C64027840, file 'D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', line 32, code <module>>, filename='D:/Github/ProgrammingLanguagesPlayGround/Python/Meta/test/reflection.py', lineno=32, function='<module>', code_context=['    foo(1, 2)\n'], index=0)]
3
```

### Code Execution/Evaluation

#### `exec()`

`exec()` is a built-in function which can take in a string **OR** a code object (ref: [exec](https://docs.python.org/3.8/library/functions.html#exec)):

- it will execute that passed-in code, if a string is passed in, then it will `compile()` it first, ref: [compile](https://docs.python.org/3.8/library/functions.html#compile)
- returns `None`

```python
exec('print(1+2)')  # output 3

code = compile(input(), '<string>', 'exec')  # input 'print(2+3)'
exec(code)  # output 5
```

#### `eval()`

`eval()` is also a built-in function which can take in a string **OR** a code object (ref: [eval](https://docs.python.org/3.8/library/functions.html#eval)):

- it will execute that passed-in code, if a string is passed in, then it will `compile()` it first, ref: [compile](https://docs.python.org/3.8/library/functions.html#compile)
- returns whatever the code evaluated to

```python
print(eval('1+2'))  # output 3

code = compile(input(), '<string>', 'eval')  # input '2+3'
eval(code)  # output 5
```

#### work with `globals` and `locals`

Both `exec` and `eval` may take in optional arguments: `globals` and `locals`, which determines:

- the context of evaluation
- the destination of execution

```python
eval('a + b', {'a': 1}, {'b': 2})  # output 3

global_vars = {'__builtins__': None}  # not locals
exec('a = 5', global_vars)  # global_vars becomes {'__builtins__': None, 'a': 5}
```

> This method does **not** support modifying locals and often will **not** work, undefined behavior.

```python
def set_vars(string, globals, locals):
    arr = string.split("=")
    arr = list(map(str.strip, arr))
    if len(arr) == 2:
        value = eval(arr[1], globals, locals)
        if arr[0] in locals:
            locals[arr[0]] = value
        else:
            globals[arr[0]] = value
```

We can use this function to dynamically change a variable's value by re evaluate the code and update the frame information.
