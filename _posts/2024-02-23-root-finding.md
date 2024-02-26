---
title: Root Finding
author: ipa
date: 2024-02-23
categories: [Algorithm]
tags: [theory, math]
math: true
---

## Problem Description

For almost all the math problems, we have often the issue with resolving polynomial equations and find the roots, some are quite easy to get with no additional algebraic manipulations as the linear polynomial (degree 1), others are quite complicated as the degree of the equation goes up.

In this post, I will list some universal solutions which are ready to use to resolve a corresponding equation (degree 2 - 4).

## Quadratic Polynomials

In this section, the target equation can be written as follows:

$$
at^2 + bt + c = 0
$$

where $a$, $b$, $c$ are the coefficients and $a$ cannot be 0, otherwise the function will become a linear equation. 

If we substract $c$ from both sides and then dividing by $a$, this will give us:

$$
t^2 + \frac{b}{a}t = -\frac{c}{a}
$$

ASTUCE: square completion -> adding a term to make the left side of the equation become a complete square:

$$
t^2 + \frac{b}{a} + \frac{b^2}{4a^2} = -\frac{c}{a} + \frac{b^2}{4a^2}
\Longrightarrow (t + \frac{b}{2a})^2 = \frac{b^2-4ac}{4a^2}
$$

By solving this equation, we get the famous **quadratic formula**:

$$
t = \frac{-b\pm\sqrt{b^2-4ac}}{2a}
$$

where $ \Delta = b^2-4ac $ is called the **discriminant** of the polynomial and reveals how many real roots the equation has base on its positivity.

## Cubic Polynomials

A cubic equation has the form (after some operations to reduce the coefficient of cubic term to 1):

$$
t^3 + at^2 + bt + c = 0
$$

The approach we are using here is trying to eliminate the quadratic term by making the following substitution:

$$
t = x - \frac{a}{3}
$$

which gives us (after doing some simplification):

$$
x^3 + (-\frac{1}{3}a^2+b)x+(\frac{2}{27}a^3-\frac{1}{3}ab+c) = 0
$$

For the simplicity, let:

$$
\begin{align}
	p &= -\frac{1}{3}a^2+b\\
	q &= \frac{2}{27}a^3-\frac{1}{3}ab+c
\end{align}
$$

### Regular Form

The discriminant $\Delta$ of a cubic polynomial is given by (TODO):
$$
\Delta = -4p^3 - 27q^2
$$

With this discriminant have our 3 roots formula for the equation:

$$
\begin{align}
	x_1 &= \sqrt[3]{-\frac{1}{2}q + \sqrt{-\frac{1}{108}\Delta}} + \sqrt[3]{-\frac{1}{2}q - \sqrt{-\frac{1}{108}\Delta}}\\
	x_2 &= \rho\sqrt[3]{-\frac{1}{2}q + \sqrt{-\frac{1}{108}\Delta}} + \rho^2\sqrt[3]{-\frac{1}{2}q - \sqrt{-\frac{1}{108}\Delta}}\\
	x_3 &= \rho^2\sqrt[3]{-\frac{1}{2}q + \sqrt{-\frac{1}{108}\Delta}} + \rho\sqrt[3]{-\frac{1}{2}q - \sqrt{-\frac{1}{108}\Delta}}
\end{align}
$$

where $\rho$ is the **primitive cube root of unity** by, and its square is its conjugate:

$$
\begin{align}
\rho &= -\frac{1}{2} + i\frac{\sqrt{3}}{2}\\
\rho^2 &= -\frac{1}{2} - i\frac{\sqrt{3}}{2}
\end{align}
$$

### Trigonometric Form

In this section, we are using an alternative way to express the roots of the equation, firstly by giving a new set of $p'$ and $q'$:

$$
\begin{align}
	p' &= \frac{p}{3} = -\frac{1}{9}a^2+\frac{1}{3}b\\
	q' &= \frac{q}{2} = \frac{1}{27}a^3-\frac{1}{6}ab+\frac{1}{2}c
\end{align}
$$

then discriminant $\Delta$ becomes:

$$
\Delta = -108(p'^3 + q'^2) \Longrightarrow \Delta' = \frac{\Delta}{108} = -(p'^3 + q'^2)
$$

Let's introduce 2 more terms $r$ and $s$:

$$
\begin{align}
	r = \sqrt[3]{-q' + \sqrt{-\Delta'}}\\
	s = \sqrt[3]{-q' - \sqrt{-\Delta'}}
\end{align}
$$

The 3 roots can be expressed as:

$$
\begin{align}
x_1 &= r + s\\
x_2 &= \rho r + \rho^2 s\\
x_3 &= \rho^2 r + \rho s
\end{align}
$$

#### Discriminant

As with quadratic equations, the discriminant gives us information about how many REAL roots exist:

- case $\Delta' < 0$:

  - there is only 1 real solution to the cubic equation, which is the $x_1$

- case $\Delta' = 0$:

  - we have $r = s$, so there are 2 real solutions:
  
    $$
    \begin{align}
    x_1 &= 2r\\
    x_2, x_3 &= (\rho + \rho^2)r = -r
    \end{align}
    $$
  
- case $\Delta/ > 0$, which implies $p < 0$ and $q'^2 < -p'^3$:

  - there are 3 distinct real solutions, by using trigonometric manipulations, we can avoid complex numbers to get the solution representations:
  
    $$
    4\cos^3\theta - 3\cos\theta = \cos3\theta
    $$
    
  - the issued equation mentioned above can be rewritten as:
  
    $$
    8m^3\cos^3\theta + 2pm\cos\theta + q = 0
    $$
    
    where $x=2m\cos\theta$ and $m=\sqrt{-p/3}$, this formula can be further simplified as:
    
    $$
    2m^3(4\cos^3\theta-3\cos\theta) + q = 0
    $$
    
    therefore, we have:
    
    $$
    \cos3\theta = \frac{-q}{2m^3} = \frac{-q/2}{\sqrt{-p^3/27}} = \frac{-q'}{\sqrt{-p'^3}}
    $$
    
    which introduce the value of $\theta$:
    
    $$
    \theta_k = \frac{1}{3}\arccos(\frac{-q'}{\sqrt{-p'^3}}) - \frac{2\pi}{3}k
    $$
    
    where $k$ picks 0, 1, 2, responding to our 3 real solutions, we define $\theta = \theta_0$:
    
    $$
    \begin{align}
    x_1 &= 2\sqrt{-p'}\cos\theta\\
    x_2 &= 2\sqrt{-p'}\cos(\theta + \frac{2\pi}{3})\\
    x_3 &= 2\sqrt{-p'}\cos(\theta - \frac{2\pi}{3})
    \end{align}
    $$
