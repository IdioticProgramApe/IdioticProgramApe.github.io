---
title: Dynamic Memory Allocation in C++ (II)
author: ipa
date: 2023-05-25
categories: [C-Cpp Language]
tags: [coding, c, cpp, debug, memory]
mermaid: true
img_path: /memory/
---

## Placement new

- placement new is the form of `new` allocates memory at the address specified by the user
- placement new is quite handy for creating objects at designated memory locations:
  - embedded devices where objects may be stored at known locations
  - **memory pooling**: a memory pool is a large chunk of memory where certain areas are used by different objects, i.e. memory pages, to avoid fragments in the heap
- if an object is created using placement new, the following cases might be needed, depends on how memory was allocated:
  - the destructor may need to be called explicitly to delete the object
  - `delete` may need to be called explicitly to delete the object

- STL uses placement new in `std::shared_ptr`, container allocators, etc. 

>Syntax for placement new
>
>```text
>Type* ptr = new (<memory address>) Type{};
>```
>{: .nolineno }
>
{: .prompt-tip }

### Examples

#### Placement new on stack

Given a such **Example** class:

```c++
struct Example
{
    Example() { std::cout << "Example()" << std::endl; }
    Exmaple(int data) : m_data(data) { std::cout << "Example(int)" << std::endl; }
    ~Example() { std::cout << "~Example()" << std::endl; }
    
    Example& operator=(const Example& other) 
    {
        std::cout << "operator=(const Example&)" << std::endl;
        m_data = other.m_data;
        returm *this;
    }
private:
    int m_data;
};
```

As we want to create a such Example object on the **stackBuf**, the following code will work:

```c++
#include <iostream>
#include <new>

int main()
{
    char stackBuf[4];    // allocate some memory on stack
    Example* ptr = new(stackBuf) Example{};
    ptr->~Example();
    return 0;
}
```

Furthermore, if we want to create an array of Example object without knowing their initial values at compile time:

```c++ 
#include <iostream>
#include <new>

int main()
{
    constexpr int SIZE = 5;
    
    // use operator new to allocate memory first and use placement new to initialize the memory
    Example* ptr = static_cast<Example*>(operator new(SIZE * sizeof(Example)));
    for(int i = 0; i < SIZE; ++i)
    {
        new(ptr + i) Exmaple{ i * i };
    }
    
    // destroy the objects then deallocate memory
    for(int i = 0; i < SIZE; ++i)
    {
        ptr[i]->~Example();
    }
    operator delete(ptr);
    
    return 0;
}
```

#### A simple String class

This sized string class is designed as shown below:

![sized string](sized_string.png)
_string class_

```c++
class String
{
public:
    String(const char* str)
    {
        void* ptr = operator new(SIZE + strlen(str) + 1);
        m_pBuffer = static_cast<char*>(new(ptr) size_t{ strlen(str) });
        strcpy_s(m_pBuffer + SIZE, strlen(str) + 1, str);
    }
    ~String() { operator delete(m_pBuffer); }

    size_t getLength() const { return *static_cast<size_t*>(m_pBuffer); }
    const char* getString() const { return m_pBuffer + SIZE; }

private:
    const static size_t SIZE = sizeof(size_t);
    char* m_pBuffer; // preserve 4bytes for the length
};
```

## Operator new & delete Functions

### Operator new Functions

| Expression                       | Function                                                   |
| -------------------------------- | ---------------------------------------------------------- |
| `T* p = new T;`                  | `void* operator new(std::size_t)`                          |
| `T* p = new(std::nothrow) T;`    | `void* operator new(std::size_t, const std::nothrow_t&)`   |
| `T* p = new(address) T;`         | `void* operator new(std::size_t, void*)`                   |
| `T* p = new(args...) T;`         | `void* operator new(std::size_t count, args...)`           |
| `T* p = new T[n];`               | `void* operator new[](std::size_t)`                        |
| `T* p = new(std::nothrow) T[n];` | `void* operator new[](std::size_t, const std::nothrow_t&)` |
| `T* p = new(address) T[n];`      | `void* operator new[](std::size_t, void*)`                 |
| `T* p = new(args...) T[n];`      | `void* operator new[](std::size_t count, args...)`         |

### Operator delete Functions

| Expression    | Function                                               |
| ------------- | ------------------------------------------------------ |
| `delete p;`   | `void operator delete(void*)`                          |
| --            | `void operator delete(void*, const std::nothrow_t&)`   |
| --            | `void operator delete(void*, void*)`                   |
| --            | `void operator delete(void*, args...)`                 |
| `delete[] p;` | `void operator delete(void*)`                          |
| --            | `void operator delete[](void*, const std::nothrow_t&)` |
| --            | `void operator delete[](void*, void*)`                 |
| --            | `void operator delete[](void*, args...)`               |

## malloc vs. new

| malloc                                            | new                                    |
| ------------------------------------------------- | -------------------------------------- |
| function                                          | operator                               |
| requires size during allocation                   | size is ascertained from the type      |
| cannot initialize memory                          | can initialize memory                  |
| cannot call constructors                          | can call constructors                  |
| returns void pointer that needs to be type casted | return correct type of pointer         |
| cannot be customized                              | can be customized through overloading  |
| `malloc`, `calloc` and `realloc`                  | has different forms                    |
| return `NULL` on failure                          | throws exception on failure by default |

