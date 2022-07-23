---
layout: default
title: Physic
parent: Game Engine
nav_order: 2
mathjax: true
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
A **zero vector** is (0, 0).

### Basic Ops

* vector addition: $ ( x_1, y_1 ) + ( x_2, y_2 ) = ( x_1 + x_2, y_1 + y_2 ) $
* vector substraction: $ ( x_1, y_1 ) - ( x_2, y_2 ) = ( x_1 - x_2, y_1 - y_2 ) $
  * negation: $ -(x, y) = (-x, -y) $
* vector multiplication: $ (x, y) \* s = (s\*x, s\*y), \forall s \in \mathbb{R} $
* vector division: $ (x, y) / s = (x/s, y/s), \forall s \in \mathbb{R} \backslash \lbrace 0 \rbrace $
* check 2 vectors are identical: $ x_1 == x_2 $ && $ y_1 == y_2 $
* vector hashcode: [Hashing in C++ using std::hash (opengenus.org)](https://iq.opengenus.org/std-hash-cpp/)
* vector string representation: use formatted string

### More Advanced Ops

* length/L2-norm: $ \Vert\vec{v}\Vert_2 = \sqrt{v_x^2 + v_y^2} $
* distance: $ distance(\vec{v}_1, \vec{v}_2) = \Vert\vec{v}_1 - \vec{v}_2\Vert = \Vert\vec{v}_2 - \vec{v}_1\Vert $
* normalize: $ normalize(\vec{v}) = \frac{\vec{v}}{\Vert\vec{v}\Vert} $
* dot product: $ \vec{v}\_1 \cdot \vec{v}\_2 = v_{1x} \* v_{2x} + v_{1y} \* v_{2y} $
* cross product: $ \vec{v}\_1 \times \vec{v}\_2 = v_{1x} \* v_{2y} - v_{1y} \* v_{2x} $
  * should only exist in 3D or higher dimensional space

## Rigid Body

* position: $(p_x, p_y)$
* rotation: $\theta$ in 2D
  * [Euler Angles](https://en.wikipedia.org/wiki/Euler_angles) in 3D (presentation of Gimbal Lock, see [Quaternion](https://en.wikipedia.org/wiki/Quaternion))
* mass: $m$
* density: $\sigma$ in 2D (la masse surfacique)
  * $\rho$ in 3D (la masse volumique)
* area: $S$ in 2D
  * $V$ in 3D
* coefficient of restitution: $e$, see [Coefficient of restitution](https://en.wikipedia.org/wiki/Coefficient_of_restitution)
* static or movable?
  * velocity: $(v_x, v_y)$
  * rotational velocity: $\omega$ in 2D
* shape and its descriptor(s)
  * circle / sphere: radius
  * box: width, height
    * cuboid: +thickness
  * polygon / polytope: ?

## Collision

### Circle Intersection

Circle is an isotropic form, which means that the orientation doesn't need to be put into consideration when compute collision.
Only the positions of 2 circle centers matter.

![image.png](./assets/image.png)

* intersection condition: $\Vert\overrightarrow{O_1O_2}\Vert < r_{O_1} + r_{O_2}$, where the radius is denoted as $r$.
* postion update:
  * for $\bigodot O_1$: move $x$ in the direction of $\overrightarrow{O_2O_1}$
  * for $\bigodot O_2$: move $y$ in the direction of $\overrightarrow{O_1O_2}$
  * where $x + y = r_{O_1} + r_{O_2} - \Vert\overrightarrow{O_1O_2}\Vert$
  * there is one degree of freedom here of how to distribute $x$ and $y$, depends on different collision scenarios and the nature of the objects.

## References

1. [Two-Bit Coding (Let's Make a Physics Engine)](https://youtube.com/playlist?list=PLSlpr6o9vURwq3oxVZSimY8iC-cdd3kIs)
2. [Math is Fun (mathsisfun.com)](https://www.mathsisfun.com/)
3. [Classroom Resources – GeoGebra](https://www.geogebra.org/materials)
4. [Paul Falstad](https://falstad.com/) (for the Math-and-Physics-Applets)
5. [geometry - Euler angles and gimbal lock - Mathematics Stack Exchange](https://math.stackexchange.com/questions/8980/euler-angles-and-gimbal-lock)
6. [Understanding Quaternions \| 3D Game Engine Programming (3dgep.com)](https://www.3dgep.com/understanding-quaternions/)
7. [The Engineering ToolBox](https://www.engineeringtoolbox.com/index.html) (all kinds of properties for different objects)
