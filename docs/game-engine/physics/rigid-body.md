---
layout: default
title: Rigid Body
parent: Physics
grand_parent: Game Engine
nav_order: 2
mathjax: true
permalink: /docs/game-engine/physics/rigid-body/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
## Properties

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

## Transformation

| space | degree of freedom | descriptors                                           |
| ----- | ----------------- | ----------------------------------------------------- |
| 2d    | 3                 | location:($x,y$), rotation:$\theta$                   |
| 3d    | 6                 | location:($x, y, z$), rotation:($\theta, \phi, \psi$) |

Transformation includes **rotation** on object's orientation and **translation** on object's position/location.

* rotation: to rotate the vector $\vec{v}_1 = (x_1, y_1)$ by angle $\alpha$ **anti-clockwise** to get $\vec{v}_2$:

  <div>$$
  \begin{pmatrix} 
  x_2 \\ y_2
  \end{pmatrix} 
  = 
  \begin{pmatrix}
  \cos\alpha & -\sin\alpha\\
  \sin\alpha & \cos\alpha
  \end{pmatrix}
  \begin{pmatrix}
  x_1 \\ y_1 
  \end{pmatrix}
  \Longrightarrow
  \begin{cases}
  x_2=\cos\alpha x_1−\sin\alpha y_1 \\
  y_2=\sin\alpha x_1+\cos\alpha y_1
  \end{cases}
  $$</div>

* translation: to translate the vector $\vec{v}$ by a vector $\vec{d}$, to get the final transformed vector $\vec{v}_t = \vec{v}_2 + \vec{d}$

> Notes:
>
> * do rotation first then translation second is not an accurate saying, should be do rotation locally, and do translatio in globally.
> * parent-child system ? (local and global coordinates to think before doing transforms)

