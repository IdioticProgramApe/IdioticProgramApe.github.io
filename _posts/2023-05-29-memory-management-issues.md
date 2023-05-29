---
title: Memory Management Issues
author: ipa
date: 2023-05-29
categories: [C-Cpp Language]
tags: [coding, c, cpp, debug, memory]
mermaid: true
img_path: /memory/
---

## Overview

Manual memory management can lead to various issues if not done correctly:

- failing to allocate memory for a pointer before using it
- allocating insufficient memory that leads to overwrites
- accessing memory after it has been freed
- freeing same memory twice
- not freeing memory after it is no longer required

## Uninitialized Pointers

- A pointer holds a memory address
- If not initialized, it will contain an unknown value:
  - possibly a valid memory address, could also be a invalid address
  - difficult to validate the pointer (cannot use `nullptr` to check)
- Accessing memory address hold by an uninitialized pointer may lead to disastrous consequences
- _some compilers DO warn about uninitialized variables, but not ALL of them_ (**undefined behavior**)

> Always initialize pointers !
{: .prompt-tip }

For the uninitialized variable check at runtime with debug configuration, it can be altered where some special needs are required.

![vs basic runtime check](vs_basic_runtime_check.png){: width="392" height="256" }
_Basic Runtime Check in Visual Studio_

## Memory Overwrite

- memory overwrites are caused when insufficient memory is allocated
- data stored in the memory may overflow into surrounding area which may contains some import data to the program
  - cause undefined behavior at runtime
- memory overwrites usually lead to loss of data, or worse, program crash
- quite common with arrays and strings, especially in the manual memory management
- use STL containers instead to prevent such problems
- _heap corruption checker_

## Dangling Pointers

- after deleting memory, the address still remains in the pointer, and the pointer continues pointing to the memory which is no longer valid
- a pointer pointing to an invalid memory is dangling, it can cause 2 problems in general:
  - the same memory may be deleted again unknowingly (_double delete_: undefined behavior)
  - it can be used without knowing it's dangling
- By assigning `nullptr` after deleting a pointer can solve the previous problems:
  - if the pointer is accessed again, it will cause an access violation
  - deleting a `NULL` pointer is not an error, `delete` treats the `nullptr` as no-op. 


>Avoid dangling pointers:
>
>- assign `nullptr` immediately after deleting it
>
{: .prompt-tip }

## Memory Leaks

- memory leak is caused when memory is allocated, but not released afterwards
- the address may be lost and the memory can never be reclaimed
- if the memory leak happens with large allocations and/or repeatedly while the programming is running, the heap memory will eventually run out:
  - may cause the OS to increase the pagefile size
  - will drastically reduce the performance of the system
  - even killed by the OS
- in C++ 11, it's recommended to use **smart pointers**.

### Example

```c++
void printFile(const char* filename)
{
    std::ifstream input(filename);
    if (!input)
    {
        std::cout << "Could not load the file: " << filename << std::endl;
        return;
    }
    
    // get the file size in bytes
    input.seekg(0, input.end);
    size_t filesize = static_cast<size_t>(input.tellg());
    input.seekg(0, input.beg);

    // initialize the allocated memory
    constexpr size_t LINE_MAX_SIZE = 256;
    char* buffer = new char[filesize] {};
    char* line = new char[LINE_MAX_SIZE] {};

    while (input.getline(line, LINE_MAX_SIZE, '\n'))
    {
        strcat_s(buffer, filesize, line);
        strcat_s(buffer, filesize, "\n");
    }
    std::cout << buffer << std::endl;

    // if no memory dealloc here, it will create memory leak
    delete[] buffer;
    delete[] line;
}
```

since there is no way we can access to the local variables **buffer** and **line** inside of function **printFile**, if they don't get release themselves, the memory allocated for these 2 variables will be leaked.
