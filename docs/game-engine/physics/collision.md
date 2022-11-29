---
layout: default
title: Collision
parent: Physics
grand_parent: Game Engine
nav_order: 3
mathjax: true
permalink: /docs/game-engine/physics/collision/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Circle

Circle is an isotropic form, which means that the orientation doesn't need to be put into consideration when compute collision.
Only the positions of 2 circle centers matter.

![image.png](D:/Github/IdioticProgramApe.github.io/docs/game-engine/physics/assets/image.png)

* intersection condition: $\Vert\overrightarrow{O_1O_2}\Vert < r_{O_1} + r_{O_2}$, where the radius is denoted as $r$.
* postion update:
  * for $\bigodot O_1$: move $x$ in the direction of $\overrightarrow{O_2O_1}$
  * for $\bigodot O_2$: move $y$ in the direction of $\overrightarrow{O_1O_2}$
  * where $x + y = r_{O_1} + r_{O_2} - \Vert\overrightarrow{O_1O_2}\Vert$
  * there is one degree of freedom here of how to distribute $x$ and $y$, depends on different collision scenarios and the nature of the objects.

## Convex Polygon

Since the box isn't isotropic form, it has 3 degrees of freedom in 2d space.

<details close>
<summary>Separating Axis Theorem (SAT)</summary>
The objective of this theorem is to help finding the normal and overlap depth of the collision: normal and depth

<ul>
<li>for each edge in polygon A</li>
  <ul>
  <li>find the normal vector (cross product)</li>
  <li>for each polygon</li>
    <ul>
    <li>get the min value and max value of its vertex projections (dot product)</li>
    </ul>
  <li>compare the min value of polygon A and max value of polygon B, and vice versa to detect intersection</li>
    <ul>
    <li>if no intersection -> continue</li>
    <li>if found</li>
      <ul>
      <li>depth: the min value of absolute value of min(A) - max(B) and max(A) - min(B)</li>
      <li>normal: normal of current edge (remember to normalize)</li>
      </ul>
    </ul>
  </ul>
<li>do the same for polygon B</li>
<li>output normal and depth</li>
</ul>
</details>

### Box 

TODO

