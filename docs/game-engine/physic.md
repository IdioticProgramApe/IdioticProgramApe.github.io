---
layout: default
title: Physic
parent: Game Engine
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

## Vector

A directional line segment which has a direction and a length, represented as (x, y) in 2d Cartesian coordinate system.

A **zero vector** is $(0, 0)$.

### Basic Ops

* vector addtion: $( x_1, y_1 ) + ( x_2, y_2 ) = ( x_1 + x_2, y_1 + y_2 )$
* vector subtraction: $( x_1, y_1 ) - ( x_2, y_2 ) = ( x_1 - x_2, y_1 - y_2 )$
  * negation: $-(x, y) = (-x, -y)$
* vector multiplication: $(x, y) * s = (s*x, s*y), \forall s \in \mathbb{R}$
* vector division: $(x, y) / s = (x/s, y/s), \forall s \in \mathbb{R} \backslash \{0\}$
* check 2 vectors are identical: $x_1 == x_2$ && $y_1 == y_2$
* vector hashcode: [Hashing in C++ using std::hash (opengenus.org)](https://iq.opengenus.org/std-hash-cpp/)
* vector string representation: use formatted string

### More Advanced Ops

* length (L2 norm): $||\vec{v}||_2 = \sqrt{v_x^2 + v_y^2}$
* distance:$distance(\vec{v}_1, \vec{v}_2) = ||\vec{v}_1 - \vec{v}_2|| = ||\vec{v}_2 - \vec{v}_1||$
* normalize: $normalize(\vec{v}) = \frac{\vec{v}}{||\vec{v}||}$
* dot product (scalar):$\vec{v}_1 \cdot \vec{v}_2 = v_{1x} * v_{2x} + v_{1y} * v_{2y}$
* cross product (vectorial): $\vec{v}_1 \times \vec{v}_2 = v_{1x} * v_{2y} - v_{1y} * v_{2x}$

> technically, cross product only exists in 3D space, if in 2D, the value itself is the component on the z direction.

## References

1. [Two-Bit Coding (Let's Make a Physics Engine)](https://youtube.com/playlist?list=PLSlpr6o9vURwq3oxVZSimY8iC-cdd3kIs)
2. [Math is Fun (mathsisfun.com)](https://www.mathsisfun.com/)
3. [Classroom Resources – GeoGebra](https://www.geogebra.org/materials)
4. [Paul Falstad](https://falstad.com/) (for the Math-and-Physics-Applets)
