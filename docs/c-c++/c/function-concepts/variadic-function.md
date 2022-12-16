---
layout: default
title: Variadic Function
parent: C
grand_parent: C/C++
nav_order: 4
permalink: /docs/c-c++/c/variadic-function/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Basics

### Creation Steps

To create a variadic function by following the up coming steps:

1. function signature rules: 

   | Function signature                    | Valid ? |
   | ------------------------------------- | ------- |
   | `void func(int n, ...)`               | Yes     |
   | `int func(const char* s, int k, ...)` | Yes     |
   | `char func(char c1, ..., char c2)`    | No      |
   | `double func(...)`                    | No      |

2. get the pointer pointing to the first optional argument in this argument list.

3. repeatedly retrieve the value of the optional variables, like an iterator.

4. clean up, reset `va_list` pointer to `NULL`

### Example

```c
#include <stdio.h>
#include <stdarg.h>   // provide va_list, va_start(), va_arg(), va_end()... macros

// forward declaration
double average(double v1, double v2, ...);

int main()
{
    double v1 = 10.5, v2 = 2.5;
    int num1 = 6, num2 = 5;
    long num3 = 12l, num4 = 20l;
	
    // all doubles
    printf("Average = %.2lf\n", average(v1, 3.5, v2, 4.5, 0.0));
    printf("Average = %.2lf\n", average(1.0, 2.0, 0.0));
    printf("Average = %.2lf\n", average((double)num2, v2, (double)num1, (double)num3, (double)num4, 0.0));

    return 0;
}

// implementation
double average(double v1, double v2, ...)
{
    // pointer for variable argument list, including v1 and v2
    va_list pArgs;
    
    double sum = v1 + v2;
    double value = 0.0;
    int count = 2;
    
    // varidic list process scope
    {
     	va_start(pArgs, v2);  // initialize pArgs
        while((value = va_arg(pArgs, double)) != 0.0) // get next optional argument value
        {
            sum += value;
            ++count;
        }
        va_end(pArgs);   	  // reset pArgs, clean up
	}

    return sum / count;
}
```

### Summary

- There must be at least one fixed parameter
- Must call `va_start()` to initialize the value of the variable argument list pointer in the variadic function:
  - this pointer must be declared as type `va_list`
- There needs to be a mechanism to determine the type of each argument
  - either there can be a default type assumed or there can be a parameter that allows the argument type to be determined
    - for example, in the `average()` function, we could have an extra fixed argument that would have the value 0 if the variable arguments were `double` and 1 if they were `long`
    - in c++, can use templates
- Must have a way to determine when the list of arguments if exhausted
  - for example, the last argument in the variable argument list could have a fixed value called a **sentinel value** that can be detected because it's different from all the others
  - OR the first argument could specify the count of the number of arguments in total or in the variable part of the argument list. 
- Must call `va_end()` before exit a function with a variable number of arguments
  - if failed to do so, the function would not work properly

## va_copy

### Example

```c
#include <stdio.h>
#include <stdarg.h>
#include <math.h>  // need add -lm to link math lib when use gcc to compile

double sample_stddev(int count, ...);

int main()
{
    printf("%f\n", sample_stddev(4, 25.0, 27.3, 26.9, 25.7));

    return 0;
}


double sample_stddev(int count, ...)
{
    double sum = 0.0;
	
    // initialize variable argument list
    va_list pArgs;
    va_start(pArgs, count);
	
    // after initialization, then copy the list pointer to a new pointer
    va_list pArgs_copy;
    va_copy(pArgs_copy, pArgs);
	
    // looping once for getting the mean value
    for(int i = 0; i < count; ++i)
    {
        double value = va_arg(pArgs, double);
        sum += value;
    }
	
    // reset once finish the usage
    va_end(pArgs);

    double mean = sum / count;
    double sum_sq_diff = 0.0;
	
    // looping twice for calculating the squared deviation
    for(int i = 0; i < count; ++i)
    {
        double value = va_arg(pArgs_copy, double);
        sum_sq_diff += pow(value - mean, 2);
    }
	
    // reset the copied pointer
    va_end(pArgs_copy);

    return sqrt(sum_sq_diff / count);
}
```

