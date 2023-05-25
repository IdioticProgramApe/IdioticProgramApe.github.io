---
title: Memory Introduction
author: ipa
date: 2023-05-24
categories: [C-Cpp Language]
tags: [coding, c, cpp, debug, memory]
img_path: /memory/
---

## Basics

### Process Memory Layout

![virtual address space](virtual_address_space.png){: width="284" height="393" }

- when a program is started, virtual memory is allocated and is called **virtual address space**
- **virtual address space** provides an environment for the executable to run
- the size of **virtual address space** varies on difference platforms:
  - Platform x86 (32-bit):  4GB
  - Platform x64 (64-bit): 16EB (Exa-Bytes)
- some parts of the process are used for storing data, as *global, static, local and runtime* data.

  | Section      | Purpose                                                      |
  | ------------ | ------------------------------------------------------------ |
  | Coda Segment | Program executable code, i.e. the statements in the code     |
  | Data Segment | Global & static variables in the code                        |
  | Heap         | Dynamically allocated memory                                 |
  | Stack        | Local variables, which are stored from high memory address to low memory address (stack grows from top to bottom) |

  - the scope/lifetime of the global variables is same as the scope/lifetime of the program, which can be accessed throughout the program
  - the scope of the static variable depends on where it gets declared, however the lifetime is same as the lifetime of the program
  - these 4 sections are not already put in the same location in the **virtual address space**, many *modern* operating systems implement a technique called **address space layout randomization**, this will make the OS to place these sections at random locations every time the program is executed.

### Pointer

- a pointer is like a variable, except that it holds **address** instead of the value

- syntax is different than a variable

    ```c
    int* p = &<variable> or dynamic memory
    ```
    {: .nolineno }
    
- a pointer can hold address of ***variable, pointer, object, function***.

### Byte Ordering

- byte ordering specifies the order in which a sequence of bytes is stored in the memory
- **big endian**: most significant byte is stored first (the first byte in the data), used by IBM Mainframe, Alpha and Sun Sparc
- **little endian**: least significant byte is stored first (the last byte in the data), used by Intel Digital

### Dynamic Memory Allocation

- dynamic memory allocation refers to memory allocation on the **heap** at runtime (instead of compile time)
- dynamic memory can be accessed throughout the program as the global variable (as long as the address is available)
- dynamic memory has to be released manually by the programmer who allocated that memory in the first place
- the reasons for using dynamic memory:
  - required size of memory may be unknown at compile time, i.e.:
    - loading data from some external source (unknown)
    - building some data structures without fixed size (arrays, etc.)
  - using dynamic memory can allow us to have more control over lifetime of data
    - lifetime of stack based data has limited by scope
    - creating larger objects (unwanted for the most of time, no need to wait to exit the scope)

#### Dynamic Memory Allocation in C

**C** provides the following functions to allocate memory at runtime:

| function/macro | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `malloc`       | allocates raw memory on the heap                             |
| `calloc`       | allocates raw memory and initializes to zero                 |
| `realloc`      | can change the size of memory block, either grow or shrink the current existing memory blocks, could be troublesome |
| `free`         | releases the memory allocated by other functions `malloc`, `calloc` and `realloc` |

Here is some code showing how to use them:

```c
// in some scope ...
int* malloc_ptr = (int*)malloc(sizeof(int));		// uninitialized memory
int* calloc_ptr = (int*)calloc(1, sizeof(int));		// allocate memory for 1 integer and initialize it with 0
free(calloc_ptr);
printf("mem size: %zu\n", _msize(malloc_ptr));
malloc_ptr = (int*)realloc(malloc_ptr, sizeof(int) * 2);	// grow the allocated memory
printf("mem size: %zu\n", _msize(malloc_ptr));
free(malloc_ptr);
```

> some extra information:
> - `_msize` is a visual studio internal function to check the size of the memory that is allocated. 
>   - for **gcc** compiler. use `malloc_usable` function instead.
> - if memory cannot be allocated, `malloc` and `calloc` will return `NULL`, need to validate the pointer after each time using these allocation methods.
> - in **C++**, the C allocation functions are avoided:
>   - the C allocation functions have a very limited memory initialization, usually need extra `memset` calls
>   - for C++ class/struct objects, C allocation functions cannot invoke:
>     - the constructors during allocation
>     - the destructor during deallocation
>   - C allocation functions cannot be customized
>   - when the allocation fails, C allocation functions return `0/NULL`
>
{: .prompt-info }
