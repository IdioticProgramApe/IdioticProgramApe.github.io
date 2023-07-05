---
title: Tweaks in Modern C++
author: ipa
date: 2023-04-21
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

## Item 41: Consider pass by value for copyable parameters that are cheap to move and always copied

### Copy via construction

Given the following test case:

```c++
Widget w;
// lvalue
std::string name("Bart");
w.addName(name);
// rvalue
w.addName(name + "Jenne");
```

#### Overloading

```c++
class Widget 
{
public: 
    void addName(const std::string& newName) { names.push_back(newName); }
    void addName(std::string&& newName) { names.push_back(std::move(newName)); }
private:
    std::vector<std::string> names;
};
```

- Cost summary: **one copy for lvalues, one move for rvalues**

#### Using a universal reference

```c++
class Widget 
{
public:
    template<typename T>
    void addName(T&& newName) { names.push_back(std::forward<T>(newName)); }
private:
    std::vector<std::string> names;
};
```

- Cost summary: **one copy for lvalues, one move for rvalues** if we assume the arguments are passed in as std::string

#### Passing by value

```c++
class Widget 
{
public:
    void addName(std::string newName) { names.push_back(std::move(newName)); }
private:
    std::vector<std::string> names;
};
```

- **newName** must be constructed:
  - If an **lvalue** is passed, this costs a **copy** construction.
  - If an **rvalue** is passed, it costs a **move** construction.
- Cost summary: **one copy plus one move for lvalues, two moves for rvalues**
- choose this method when:
  - it can avoid the function overloading
  - consider pass by value only for **copyable** parameters, for the move-only object, it requires 2 moves not 1.
  - pass by value is worth considering only for parameters that are **cheap to move**, cheap relative to copy
  - consider pass by value only for parameters that are **always copied**, to avoid the input argument is constructed for nothing.

### Copy via assignment

the extra cost of pass by value for functions that copy a parameter using assignment depends on:

- the type being passed
- the ratio of lvalue to rvalue arguments
- whether the type uses dynamically allocated memory
- if use dynamiclly allocated memory:
  - the implementation of that type's assignment operators
  - the likelihood that the memory associated with the assignment target is at least **as large as** the memory associated with the assignment source
- specail cases:
  - std::array
  - std::string with SSO occurs

### Slicing problem

An code example, ref: [https://stackoverflow.com/questions/274626/what-is-object-slicing](https://stackoverflow.com/questions/274626/what-is-object-slicing)

```c++
#include <iostream>

using namespace std;

// Base class
class A 
{
public:
    A() {}
    A(const A& a) 
    {
        cout << "'A' copy constructor" << endl;
    }
    virtual void run() const { cout << "I am an 'A'" << endl; }
};

// Derived class
class B : public A 
{
public:
    B() : A() {}
    B(const B& a) : A(a) 
    {
        cout << "'B' copy constructor" << endl;
    }
    void run() const override { cout << "I am a 'B'" << endl; }
};

void passByRef(const A& a) { a.run(); }
void passByVal(const A a) { a.run(); }

int main() 
{
    cout << "Call by reference" << endl;
    passByRef(B());
    
    cout << endl << "Call by copy" << endl;
    passByVal(B());
}
```

the output will be:

```text
Call by reference
I am a 'B'

Call by copy
'A' copy constructor
I am an 'A'
```
{: .nolineno }

The reason is that when a **B** object is passed in as value, it's used to contruct a branch new **A** object, then it calls the A's **run** function, whereas, pass by value is use a **A** pointer which is pointing to B object to call **run** function.

- reference can be considered as an **auto-dereferenced** pointer.

### Key points

- For **copyable**, **cheap-to-move** parameters that are **always copied**, pass by value may be **nearly** as efficient as pass by reference, it's easier to implement, and it can generate less object code
- Copying parameters via construction may be significantly more expensive than copying them via assignment, involving the memory allocation/deallocation
- Pass by value is subject to the slicing problem, so it's typically **inappropriate for base class parameter types**.

## Item 42: Consider emplacement instead of insertion

### Use conditions

- The value being added is constructed into the container, not assigned, which means the location is not occupied before the operation

  - If constructed, it can save us one temp object's construction and one destruction

- The argument type(s) being passed **differ from** the type held by the container

- The container is unlikely to reject the new value as a duplicate, this can have 2 situation:

  - container accepts duplicated elements, in STL, there are a few containers **DON'T qualify**: **std::set**, **std::map**, **std::unordered_set**, **std::unordered_map**
  - most of the values are unique, the container type therefore doesn't really matter.

  this is because if the container rejects the value, the temp object will be detroyed, we made extra effort (temp objet's contruction and destruction) for nothing.

### Special cases

#### Resource management

Given the scenario:

```c++
std::list<std::shared_ptr<Widget>> ptrs;
void killWidget(Widget* pWidget); // custom deleter
```

- use **push_back**:

  ```c++
  ptrs.push_back(std::shared_ptr<Widget>(new Widget, killWidget));  // method 1
  ptrs.push_back({ new Widget, killWidget });  // method 2
  ```

  - a temporary `std::shared_ptr<Widget>` object is constructed to hold the raw pointer resulting from "**new Widget**". Call this object **temp**
  - push_back takes temp by reference. **If** during allocation of a list node to hold a copy of temp, an out-of-memory exception gets thrown:
    - As the exception propagates out of **push_back**, **temp** is destroyed. Being the sole `std::shared_ptr` referring to the **Widget** it's managing, it automatically releases that Widget, in this case by calling killWidget. **No leaks**.

- use emplace_back:

  ```c++
  ptrs.emplace_back(new Widget, killWidget);
  ```

  - The raw pointer resulting from "**new Widget**" is perfect-forwarded to the pointer inside **emplace_back** where a list node is to be allocated. **If** that allocation fails, and an out-of-memory exception is thrown:
    - As the exception propagates out of **emplace_back**, the raw pointer that was the only way to get at the **Widget** on the heap is lost. That Widget (and any resources it owns) is leaked. **Memory leak**.

When working with **containers of resource-managing objects**, ensure that if you choose an emplacement function over its insertion counterpart, you're not paying for **improved code efficiency** with **diminished exception safety**.

> Bad practice
>
> Passing expressions like "**new Widget**" to **emplace_back** or **push_back** or most any other function. this can lead to the possibility of exception safety problems.
{: .prompt-danger }

The proper way of adding a new element into **ptrs**:

```c++
std::shared_ptr<Widget> spw(new Widget, killWidget);
ptrs.push_back(std::move(spw));
```

```c++
std::shared_ptr<Widget> spw(new Widget, killWidget);
ptrs.emplace_back(std::move(spw));
```

#### Explicit constructor

Given the scenario:

```c++
std::vector<std::regex> regexes;
```

- use **push_back**:

  ```c++
  regexes.push_back(nullptr);  // error! won't compile
  ```

  - `std::regex` constructor taking a `const char*` pointer is **explicit**, prevent `std::regex` from being constructed from a pointer-type variable. the statement above is equivalent to:

    ```c++
    {
        std::regex r = nullptr; // error! won't compile
        regexes.push_back(r);
    }
    ```

- use **emplace_back**:

  ```c++
  regexes.emplace_back(nullptr);  // no compilation errors
  ```

  - for the emplace_back, there is **no copy initialization for the std::regex, it's a direct initialization**. therefore, the explicitness of the constructor is bypassed by the perfect-forward:

    ```c++
    std::regex r2(nullptr);  // compiles
    ```

> Good practice
>
> Copy initialization is not permitted to use explicit constructors. Direct initialization is. when use an emplacement function, be especially careful to make sure the correct arguments are passed in, because even explicit constructors will be considered by compilers as they try to find a way to interpret the code as valid.
> {: .prompt-info }

### Key points

- In principle, emplacement functions should **sometimes be more efficient** than their insertion counterparts, and they should **never be less efficient**
- In practice, they're most likely to be faster when:
  - the value being added is constructed into the container, not assigned
  - the argument type(s) passed differ from the type held by the container
  - the container won't reject the value being added due to it being a duplicate
- Emplacement functions may perform type conversions that would be rejected by insertion functions, **unexpected behavior**!
