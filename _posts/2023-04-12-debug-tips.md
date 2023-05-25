---
title: Debug Tips
author: ipa
date: 2023-04-12
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp, debug]
---

## Check Deduced Types

### Error Message

An effective way to get a compiler to show a type it has deduced is to use that type in a way that leads to compilation problems. This is happening **during the compile-time**.

A helper template class:

```c++
template<typename T>
class TypeDisplayer;
```

For example, there are 2 variables we want to know their types, we could inject these 2 lines into the code and compile it:

```c++
TypeDisplayer<decltype(x)> xType;
TypeDisplayer<decltype(y)> yType;
```

One compiler could output some error messages as:

```text
error: aggregate 'TypeDisplayer<int> xType' has incomplete type and cannot be defined
error: aggregate 'TypeDisplayer<const int *> yType' has incomplete type and cannot be defined
```

now we know **x** is a `int` and **y** is a pointer to a `const int`. 

### Runtime Output

#### Use Standard Functions

Print the types **during the runtime** by using `typeid` and `std::type_info::name`.

For example, we can have something like:

```c++
std::cout << typeid(x).name() << '\n';
std::cout << typeid(y).name() << '\n';
```

Function `std::type_info::name` will output a C-style string, and it's not guaranteed to return anything sensible nor reliable, but somethign could be helpful.

#### Use Boost

The output of the following code:

```c++
#include <boost/type_index.hpp>

template<typename T>
void f(const T& param)
{
    using boost::typeindex::type_id_with_cvr;
    
    // show T
    std::cout << "T = " << type_id_with_cvr<T>().pretty_name() << '\n';
    
    // show param's type
    std::cout << "param = " << type_id_with_cvr<decltype(param)>().pretty_name() << '\n';
}

std::vector<Widget> createVec();

const auto vw = createVec();

if (!vw.empty()) {
    f(&vw[0]);
}
```

With **MSBuild** compiler, the code above outputs:

```text
T = class Widget const *
param = class Widget const * const &
```

