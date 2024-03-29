---
title: C Compiler
author: ipa
date: 2022-09-29
categories: [C-Cpp Language]
tags: [coding, c, cpp]
img_path: /c/
---

## GCC Compiler Options

**GCC** is a very powerful and popular C compiler for various Linux distributions

- able to use on windows because of **Cygwin**:
  - a Unix-like environment and command-line interface for windows (also **includes the bash shell**)
- able to use a front end version named **clang** on mac:
  - everything work with **gcc** should work with **clang**
- when invoked, it normally does **preprocessing**, **compilation**, **assembly** and **linking** to produce the binary:
  - some compiler options allow users to stop the process at an intermediate stage, use `gcc --help` to check the usage:
    
    | Option | Description                                       |
    | ------ | ------------------------------------------------- |
    | `-E`   | Preprocess only, do not compile, assemble or link |
    | `-S`   | Compile only, do not assemble or link             |
    | `-c`   | Compile and assemble, but do not link             |
    | `-o`   | Generate the final executable file                |
  - other options are passed on to one stage of processing:
    - some options control the preprocessor and others the compiler itself
    - other options control the assembler and linker

### Compilation Steps

![c compilation process](c_compilation_process.png){: width="430" height="461" }
_[C Compilation Process](https://codeforwin.org/c-programming/c-compilation-process)_

To get the intermediate processed files during the compilation process, need to put option `-save-temps` which prevent gcc from deleting the files:

```bash
gcc -save-temps helloworld.c -o helloworld
```
{: .nolineno }

Here are a list of file that are generated:

| File name    | Description                                               |
| ------------ | --------------------------------------------------------- |
| helloworld.i | Generated by preprocessor                                 |
| helloworld.s | Generated by compiler                                     |
| helloworld.o | Generated by assembler                                    |
| helloworld   | On linux generated by linker or helloworld.exe on windows |

### The Do's & Don'ts

- the gcc program accepts options and file names as operands
  - many options have multi-letter names, therefore multiple single-letter options **may not be grouped**, i.e. `-dr` is very different from `-d -r`
- mixing options and other arguments is allowed
- for the most part, the order of the options does not matter, however it **DOES** matter when several options of the same kind are used:
  - e.g. when link multiple libs, multiple `-L` are used, the directories are searched in the order specified
- many options have long names starting with `-f` or with `-W`

### Useful Options

| Option                                  | Description                                                  |
| --------------------------------------- | ------------------------------------------------------------ |
| `-Wall`                                 | enable all the warnings during the compilation/linking       |
| `-DXXX`                                 | define a macro during compilation process                    |
| `-E`, `-S`, `-c`, `-o`<br>`-save-temps` | see the description above, might need redirect the output to a file |
| `-g`                                    | to generate the debugging information for debugging the code |
| `-l`                                    | for linking the shared library                               |
| `-fPIC`                                 | to generate **P**osition **I**ndependent **C**ode and generate a shared library |
| `-shared`                               | used to generate a shared object **.so**                     |
| `-v`                                    | to output verbosely all the steps for compilation, can be redirected to a file |
| `-ansi`                                 | used to compile some files with ansi standard (ISO C90)      |

