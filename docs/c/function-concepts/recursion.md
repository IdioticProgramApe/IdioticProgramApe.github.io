---
layout: default
title: Recursion
parent: Function Concepts
grand_parent: C
nav_order: 3
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Recursion vs. Iteration

Any problem that can be solved recursively can also be solved iteratively (non-recursively using loops)

| Overview                                                     | Iteration                                                    | Recursion*                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Both iteration and recursion are based on a control structure | uses a repetition structure                                  | uses a selection structure                                   |
| Both iteration and recursion involve repetition              | explicitly uses a repetition structure                       | achieves repetition through repeated function calls (call stack) |
| Iteration and recursion each involve a termination test      | terminates when the loop-continuation condition fails        | terminates when **a base case** is recognized (very important, an infinite loop is never wanted) |
| Iteration with counter-controlled repetition and recursion each gradually approach termination | keeps modifying a counter until the counter assumes a value that makes the loop-continuation condition fail | keeps producing simpler version of the original problem until the base case is reached |
| Both iteration and recursion can occur infinitely            | an infinite loop occurs with iteration if the loop-continuation test never becomes false | infinite recursion occurs if the recursion step does not reduce the problem each time in the manner that converges on the base case |

*: A recursive approach is normally chosen in preference to an iterative approach when the recursive approach more naturally mirrors the problem

## Recursion Pros & Cons

### Pros

- Recursion sometimes offers the simplest solution to some programming problems

### Cons

- Recursive functions can rapidly exhaust a computer's memory resources
  - it repeatedly invokes the mechanism and consequently the overhead, of function calls (stored in the stack)
    - expensive in both processor time and memory space
  - each recursive call causes another copy of the function (only the function's variables) to be created (can consume considerable memory)
- Avoid using recursion in performance situations
- Recursion can be difficult to document and maintain

## Tail Recursion

- Tail recursion is the simplest form of recursion
  - the recursive call is at the end of the function, just before the return statement
  - the recursive call comes at the end
  - acts like a loop
- Tail recursive functions can be optimized by compiler
  - since the recursive call is the last statement, there is nothing left to do in the current function, so saving the current function's **stack frame** is of no use

## Example

```c
#include <stdio.h>

int factorial(int n);
void up_and_down(int n);
void recursive_print(int n);

int main()
{
    printf("======= factorial section ======\n");
    for(int j = 0; j < 8; ++j)
    {
        printf("%d! = %d\n", j, factorial(j));
    }

    printf("====== up and down section ======\n");
    up_and_down(1);

    return 0;
}

int factorial(int n)
{
    int result = 1;

    if(n <= 0) 
        return result;
    else 
        result = n * factorial(n - 1);  // put on hold the outer function

    return result;
}

void up_and_down(int n)
{
    printf("Level %d entered: n location %p\n", n, &n);

    if(n < 4)
    {
        up_and_down(n + 1);     // put on hold the outer function
    }

    printf("Level %d exited: n location %p\n", n, &n);
}

void recursive_print(int n)
{
    if(n < 0) return;
    
    printf("%d\n", n - 1);
}
```

