---
title: Understanding Q_rsqrt
author: ipa
date: 2024-01-22
categories: [Algorithm]
tags: [theory, math, coding]
math: true
img_path: /memory/
---

## Source Code

From the open source repo on github, a function called `Q_rsqrt` can be found in the file [q_math.c](https://github.com/id-Software/Quake-III-Arena/blob/master/code/game/q_math.c) as follows:

```c
/*
** float q_rsqrt( float number )
*/
float Q_rsqrt( float number )
{
	long i;
	float x2, y;
	const float threehalfs = 1.5F;

	x2 = number * 0.5F;
	y  = number;
	i  = * ( long * ) &y;                       // evil floating point bit level hacking
	i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
	y  = * ( float * ) &i;
	y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//	y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

#ifndef Q3_VM
#ifdef __linux__
	assert( !isnan(y) ); // bk010122 - FPE?
#endif
#endif
	return y;
}
```

## Prerequisites

### Floating-Point Number

In computer science, there are 2 common types of floating-point number, which is `float` and `double`, representing single precision and double precision respectively.

#### Single Precision: `Float`

We will first consider the single-precision floating numbers with **32-bit** structure showed in the following image (source: [https://en.wikipedia.org/wiki/Single-precision_floating-point_format](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)):

![float-bits](float-bits.png){: width="600" height="76" }

All the bits can be categorized into 3 groups:

- S -> sign (1 bit): $ b_{31} $, with 1 -> negative and 0 -> positive
- E -> biased exponent (8 bits): $ b_{30}...b_{23} $, with bias -127
- M -> mantissa (23 bits): $ b_{22}...b_{0} $

Overall the value of the 32-bit float can be evaluated as:

$$
\begin{align}
   F_{2} & = S \times 2^{31} + E \times 2^{23} + M\\
   F_{10} & = (-1)^S \times 2^{E-127} \times (1 + \frac{M}{2^{23}})
\end{align}
$$

#### Double Precision: `Double`

Similarly, the double-precision floating point number `double` has the same formula for the calculation with the following bit arrangement:

- S -> sign (1 bit): $ b_{63} $, with 1 -> negative and 0 -> positive
- E -> biased exponent (11 bits): $ b_{62}...b_{52} $, with bias -1023
- M -> mantissa (52 bits): $ b_{51}...b_{0} $

### Logarithmical Value Approximation

In this section, we will first calculate the logarithmical value of a regular **positive** float number ($ F_{10} $) with base of 2:

$$
\begin{equation}
  \begin{split}
    \log_2 F_{10} & = \log_2 (-1)^S \times 2^{E-127} \times (1 + \frac{M}{2^{23}})\\
                  & = (E-127) + \log_2 (1 + \frac{M}{2^{23}})\\
                  & \approx \frac{M}{2^{23}} + \mu + E - 127\\
                  & = \frac{1}{2^{23}} \times (E \times 2^{23} + M) + \mu - 127\\
                  & = \frac{1}{2^{23}} \times F_{2} + \mu - 127
  \end{split}
\end{equation}
$$

The approximation is valid only when $ \frac{M}{2^{23}} $ is around 0, according to [Taylor Expansion](https://en.wikipedia.org/wiki/Taylor_series) for the [**Mercator series**](https://en.wikipedia.org/wiki/Mercator_series), $ \mu $ is a small value to reduce the error coming from this approximation.

### Newton's Method

In numerical analysis, [Newton's Method](https://en.wikipedia.org/wiki/Newton's_method) is the most commonly method used to find a real-value function($ f(x) $)'s roots. The core mechanism for this method is recursion, given a initial value $ x_{0} $ and a tolerance value $ \epsilon $, using the following equation to get the next $ x $:

$$
\begin{equation}
  x_{n+1} = x_{n} - \frac{f(x_n)}{f'(x_n)}
\end{equation}
$$

with the condition of termination:

$$
\|\Delta x\| = \|\frac{f(x_n)}{f'(x_n)}\| < \epsilon
$$


## `Q_rsqrt` Interpretation

### `0x5f3759df` ?

According to Newton's Method, we will first to provide a initial value, which should be close to the real root value to reduce the number of recursion iterations. To do so, we can start with calculating the $ log_2 $ of $ \frac{1}{\sqrt{F_{10}}} $:

$$
\log_2 \frac{1}{\sqrt{F_{10}}} = -\frac{1}{2} \log_2 F_{10}
$$

if we let $ Z_{10} = \frac{1}{\sqrt{F_{10}}} $, we have:

$$
\begin{align}
  	\frac{1}{2^{23}} \times Z_{2} + \mu - 127 &= -\frac{1}{2} (\frac{1}{2^{23}} \times F_{2} + \mu - 127) \\
    \Longrightarrow Z_{2} &= -\frac{3}{2} \times 2^{23} \times(\mu - 127) - \frac{1}{2} \times F_{2}
\end{align}
$$


with $ \mu \approx 0.0450 $, we can have this WTF value:

$$
-\frac{3}{2} \times 2^{23} \times(\mu - 127) = \text{0x5f3759df}
$$


### Recursion

The equation we try to find its root here is defined as follows:

$$
f(x) = \frac{1}{x^2} - C, \forall  C > 0
$$

combine it with the Newton's Method equation, we have:

$$
\begin{equation}
\begin{cases}
x_{n+1} &= x_{n}(\frac{3}{2} - \frac{1}{2}Cx^2_{n})\\
x_0 &= Z_{10}
\end{cases}
\end{equation}
$$

This last equation explains what we have as the last statement in in the `Q_rsqrt` function.

## Remarks

1. The key point in this algorithm is, the $ \log_2 $ value of float $ F_{10} $ is affine itself in its binary representation $ F_2 $. From the memory perspective, the bits stay the same.
2. The trick here to convert a float value to its binary representation is using a c-style cast on it's address, and then dereference it. (ref: `reinterpret_cast`)
