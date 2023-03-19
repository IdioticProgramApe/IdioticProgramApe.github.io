---
layout: default
title: Vectors
parent: Physics
grand_parent: Game Engine
nav_order: 1
mathjax: true
permalink: /docs/game-engine/physics/vectors/
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

A directional line segment which has a direction and a length, represented as (x, y) in 2d Cartesian coordinate system.
A **zero vector** is (0, 0).

### Vector operations

* vector addition: $ ( x_1, y_1 ) + ( x_2, y_2 ) = ( x_1 + x_2, y_1 + y_2 ) $
* vector substraction: $ ( x_1, y_1 ) - ( x_2, y_2 ) = ( x_1 - x_2, y_1 - y_2 ) $
  * negation: $ -(x, y) = (-x, -y) $
* vector multiplication: $ (x, y) \* s = (s\*x, s\*y), \forall s \in \mathbb{R} $
* vector division: $ (x, y) / s = (x/s, y/s), \forall s \in \mathbb{R} \backslash \lbrace 0 \rbrace $
* check 2 vectors are identical: $ x_1 == x_2 $ && $ y_1 == y_2 $
* vector hashcode: [Hashing in C++ using std::hash (opengenus.org)](https://iq.opengenus.org/std-hash-cpp/)
* vector string representation: use formatted string

### More Advanced Operations

* length/L2-norm: $ \Vert\vec{v}\Vert_2 = \sqrt{v_x^2 + v_y^2} $
* distance: $ distance(\vec{v}_1, \vec{v}_2) = \Vert\vec{v}_1 - \vec{v}_2\Vert = \Vert\vec{v}_2 - \vec{v}_1\Vert $
* normalize: $ normalize(\vec{v}) = \frac{\vec{v}}{\Vert\vec{v}\Vert} $
* dot product: $ \vec{v}\_1 \cdot \vec{v}\_2 = v_{1x} \* v_{2x} + v_{1y} \* v_{2y} $
* cross product: $ \vec{v}\_1 \times \vec{v}\_2 = v_{1x} \* v_{2y} - v_{1y} \* v_{2x} $
  * should only exist in 3D or higher dimensional space
