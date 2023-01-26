---
layout: default
title: Convex Hull
parent: Good to Know
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

## Definition

A convex hull to a set of 2-dimensional points is a polygon that enclose all those points in a minmax way (maximize the area of the polygon while minimize the length of its circumference).

## Applications

Convex hull has a wide applications in **mathematics**, **statistics**, **combinatorial optimization**, **economics**, geometric modeling, and **ethology**

## Algorithms

### Graham Scan

```text
1. select the point (x0, y0) with the lowest Y coordinate
2. sort the points (xi, yi) by the angle relative to the bottom most point (from step 1) and the horizontal 
3. iterate in sorted order, placing each point on a stack, but only if it makes a counterclockwise turn relative to the previous to the previous 2 points on the stack, pop previous points off of the stack if making a clockwise turn
```

There are 2 things to pinpoint:

- calculate the angles: $ \theta_i = \arctan(y_i - y_0, x_i - x_0) $
- determine the orientation of the turn, use the cross product of 2 vectors $ \vec{v}_1 \times \vec{v}_2 $, of which the geometry meaning is the **signed** area of parallelogram formed by these 2 vectors.
  - if area is positive, then it means $ \vec{v}_1 $ is on the right of $ \vec{v}_2 $, forming a counterclockwise turn
  - if area is negative, then it means $ \vec{v}_1 $ is on the left of $ \vec{v}_2 $, forming a clockwise turn
- handle the colinear points consistently, e.g. always from left to right (top to bottom if a vertical line)

Time complexity are mainly coming from 3 phases:

- get the bottom most point, which is $ \mathcal{O}(n) $
- sort the rest of the points based on the angles, which is $ \mathcal{O}(n \log n) $
- iterate over the points, which is also $ \mathcal{O}(n) $

overall, the complexity for Graham Scan algorithm is about $ \mathcal{O}(n \log n) $, coming from the sorting phase.

### Jarvis March

```text
1. select the point with the lowest Y coordinate, as the 1st vertex
2. select the point with smallest counterclockwise in reference to previous vertex
```

Time complexity is more dependent on the given points, it comes from the number of points forming the convex hull $ n $ and the total number of the points $ m $, thus the complexity is $ \mathcal{O}(n m) $.

2 extreme cases are, only 3 points are used to enclose the entire point set and 0 inner points, all points are on the circumference of the hull.
