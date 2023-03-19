---
layout: default
title: Texture Streaming
parent: Unreal
grand_parent: Game Engine
nav_order: 1
mathjax: true
permalink: /docs/game-engine/unreal/texture-streaming/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## What is Texture Streaming & Why Use it

The texture streaming in UE4 is a technique that can reduce the bandwidth and improve the performance by downgrading the quality of the texture and using larger granularity mips (which requires less memory occupancy).

Start with the mipmaps, it is usually used to solve 2 issues:

- The flickering issue, the uv interpolation is determined by how many pixels are used for the render target of a primitive. However, because in most of the cases, one pixel is mapping to multiple texels, which can cause some floating-point precision issue for uv coordinates. When we tried to move the camera, one pixel on the screen could adopt difference texel which make this flickering.
- The performance issue, sometimes, the texels only need some pixels from the render target, it still needs to load the entire texture into the GPU memory, which increases the load for the bandwidth, even more, it could reduce the hit rate in the texture buffer.

When we use texture mipmaps, it will require more disk space for the textures, and it stills load the high res mips into the GPU memory, even those mips are not used.

The texture streaming technique is developed based on the mipmaps, on top of that, adds some dynamic calculation to decide which texture should use which mip given the runtime camera position. Taking OpenGL as an example to showcase how the GPU calculate the level of the mips:

<div>$$
d = 
\begin{cases}
level_{base}, & \lambda\leqslant0 \\
nearest(\lambda), & \lambda>0,level_{base}+\lambda\leqslant q+\frac{1}{2} \\
q, & \lambda>0,level_{base}+\lambda>q+\frac{1}{2}
\end{cases}
$$</div>

where

<div>$$
nearest(\lambda) = 
\begin{cases}
\lceil level_{base}+\lambda+\frac{1}{2}\rceil-1, & preferred \\
\lfloor level_{base}+\lambda+\frac{1}{2}\rfloor, & alternative
\end{cases}
$$</div>

- $ \lambda $ is the scaling factor, when it's less than 0, it means the texture is already upscaled, in this case we should take the highest res mip, which is the base mip

- $ level_{base} $ is 0 by default, however this value can be changed through the renderer API, for example, in order to streaming only mips 1-4, we need to do the following configurations in OpenGL:

  - `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_BASE_LEVEL, 1)`: set the $ level_{base} $ to 1
  - `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAX_LEVEL, 4)`: set the $ level_{max} $ to 4

- $ q $ is defined as:

  <div>$$
  p = \lfloor \log_2(maxsize) \rfloor + level_{base}\\
  q = \min(p, level_{max})
  $$</div>

## Sampling Scale

From texture perspective, the number of mips of a texture positively correlated to its resolution.

The density of the texel in the screen space can be changed by:

- texel's count
- texture's resolution

In UE4, the texel count can be modified through `FMaterialTextureInfo::SamplingScale`, if `utiling` is different from `vtiling`, then the final tiling will take the smaller one. The call stack of this function is as follows:

```text
FMaterialTextureInfo::SamplingScale
└─── FMeshRenderer::RenderMaterialTexCoordScales
    └─── UMaterialInterface::SetTextureStreamingData
        └───UMaterialInterface::TextureStreamingData
            └───<...>
```

## Local UV Densities

In order to render a texture on the screen, it needs to be applied to a mesh first. Since the dimension of the mesh will change the texel density in screen space, UE4 will get a Local UV density using the local coordinates. 