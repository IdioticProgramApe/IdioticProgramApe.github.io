---
layout: default
title: Preprocessor
parent: C
nav_order: 2
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Overview

| Step         | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| preprocessor | Goes through the code and does the textual substitution      |
| compiler     | Compiles the source code                                     |
| assembler    | Creates the assembly code from the compiled code (obj files) |
| linker       | Links together a bunch of object files into a binary executable  file |
| loader       | Loads the executable from the disk into the primary memory(RAM) for execution |

- The preprocessor mainly performs 3 tasks on the code (textual operations):
  - Removes all the comments
  - Includes all of the files from various libraries that the program needs to compile
  - Expands the macro definitions
- Preprocessor Directives
  - commands used by the preprocessor, and begin with "`#`" symbol
    - must be the first nonblank character
    - should begin in the first column
  - conditional compilation commands
    - `#ifdef`, `#ifndef`, `#if`, `#elif`, `#else`, `#endif`
  - others
    - `#define`, `#include`, `#undef`, `#pragma`, `#error`
- Preprocessor Operators
  - to help in creation of macros
  - used in the context of the `#define` directive
  - common operators
    - continuation operator (`\`)
    - concatenation operators
      - (`#`) when used within a macro definition, converts a macro parameter into a string constant
      - (`##`) within a macro definition combines 2 arguments
        - permits 2 separate tokens in the definition to be joined into a single token
    - `defined()` 
      - simplifies the writing of compound expressions in certain macro directives

## Conditional Compilation

- `#ifdef` is same as `#if defined(...)`, `#ifndef` is same as `#if !defined(...)`
- most compilers also permit to **define** a name to the preprocessor when the program is compiled
  - use the special option `-D` to the compiler command
  - `gcc -D UNIX program.c` to define a macro named UNIX

```C
#include <stdio.h>

#define JUST_CHECKING   // set to 0 by default
#define LIMIT 4

int main()
{
    int i;
    int total = 0;

    for(i = 1; i <= LIMIT; ++i)
    {
        total += 2 * i * i + 1;

#ifdef JUST_CHECKING
        printf("i = %d, running total = %d\n", i, total);
#endif        
    }

    printf("Grand total = %d\n", total);

    return 0;
}
```

> can also add `-DJUST_CHECKING -DLIMIT=4` as the compiler option to get rid of the 2 `#define` lines in the code

- `#if` constant expression
  - the operand must be a constant integer expression that **does not contain** any increment (`++`), decrement (`--`), `sizeof`, pointer (`*`), address (`&`) and `cast` operators
    - can use relational and logical operators
  - can contain references to identifiers defined in the previous `#define` directives

```C
#include <stdio.h>

#define US      0
#define UK      1
#define France  2
#define Germany 3
#define Country France

int main()
{
    #if Country == US || Country == UK
    #define Greeting "Hello"
    #elif Country == France
    #define Greeting "Bonjour"
    #elif Country == Germany
    #define Greeting "Guten Tag"
    #endif

    printf("Greeting is: %s\n", Greeting);
    return 0;
}
```

## Include Guards

- to prevents multiple definitions of the same variables, functions and/or macros
- standard C headers files use `#ifndef` technique to avoid multiple inclusions
  - use the filename as the identifier, e.g.`stdio.h => _STDIO_H`

## Undefine Directive

- `#undef IDENTIFIER_NAME`: to undefine an identifier which has been previously defined 
- subsequent `#ifdef IDENTIFIER_NAME` statements will evaluate as **false**

## Other Preprocessor Directives

- `#pragma`: pragmatic information, this directive is part of the standard (c89, c99, c11)

  - used to place compiler instructions in the source code, request special behavior from the compiler

  - mostly useful for programs that are **unusually large** or that need to take advantage of the capabilities of a **particular compiler**

  - can be used

    - to control the amount of memory set aside for automatic variables
    - to set the strictness of error checking
    - to enable nonstandard language features

  - **each compiler has its own set of pragmas**

    - e.g.`#pragma c9x on` to enable C9X support

  - `#pragma` is usually followed by a single token, and there are only a limited list of supported tokens for each standard/compiler

    - need to reference the compiler documentation: 
      - `MSBuild`: [Pragma directives and the __pragma and _Pragma keywords | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/preprocessor/pragma-directives-and-the-pragma-keyword?view=msvc-170)
      - `GCC`: [Pragmas (The C Preprocessor) (gnu.org)](https://gcc.gnu.org/onlinedocs/cpp/Pragmas.html)

    | Some GCC #pragmas               | Description                                                  |
    | ------------------------------- | ------------------------------------------------------------ |
    | `#pragma GCC dependency`        | - allows to check the relative dates of the current file and another file, if the other file is more recent than the current file,  a warning is issued<br />- useful if the current file is derived from the other file, and should be regenerated<br />- e.g. `#pragma GCC dependency "/usr/include/time.h" rerun fixincludes` |
    | `#pragma GCC poison`            | - to remove an identifier completely from the program which is never used<br />- followed by a list of identifiers to poison, and if any of those identifiers appear anywhere in the source after this directive, an error will be displayed by the compiler<br />- e.g.`#pragma GCC poison printf sprintf fprintf` |
    | `#pragma GCC system_header`     | - tells the compiler to consider the rest of the current include file as a system header, and takes **no arguments**<br />- system headers are header files that come with the OS or compiler, GCC gives code found in system headers **special treatments**<br />    - all warnings are suppressed while GCC is processing a system header<br />    - macros defined in a system header are immune to a few warnings wherever they are expanded |
    | `#pragma once`                  | - specifies that the header file containing this directive is included only once even if it's included multiple times during a compilation<br />- similar to include guards, but less portable |
    | `#pragma GCC warning "message"` | - causes the preprocessor to issue a warning diagnostic with the text "message"<br />- message must be a single string literal |
    | `#pragma GCC error "message"`   | - causes the preprocessor to issue a error with the text "message"<br />- message must be a single string literal |
    | `#pragma message "message"`     | - print string as a compiler message on compilation, the message is only for a informative purpose |

    ```C
    #include <stdio.h>
    
    // can be put anywhere, these message are working in preprocessing stage before the compilation
    #pragma GCC warning "this is a warning msg"
    #pragma GCC error "this is an error"	// will fail the compilation
    #pragma message "this is a notification"
    
    int main()
    {
        return 0;
    }

- `#error`: causes the preprocessor to issue an error message that includes any text in the directive

  - error message is a sequence of characters separated by spaces
  - **do not have to** enclose the text in quote
  - message is optional

  ```c
  // throw an error msg when the current compiler is not C11
  #if __STDC_VERSION__ != 201112L
  #error Not C11	// will not fail the compilation but only throw an error msg
  #endif
  
  // or just to warn other programmers
  #error *** this code is not fully finished, fix before using ***
  ```

- other directives: 

  - `#warning`: emits a diagnostic containing the remaining tokens in the directive 
  - `#line linenum filename`: sets the `__FILE__` and `__LINE__`

