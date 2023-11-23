---
title: Unreal Engine Materials (Shaders)
author: ipa
date: 2023-11-22
categories: [Unreal Engine]
tags: [ue, ue-editor]
img_path: /unrealengine/shaders/
---

## Concepts

A **shader** is a piece of code that controls the color of each pixel on the screen and usually runs on the graphics processor. Nowadays:

- game engines using advanced PBR lighting and rendering

  - shader is just doing the surface properties and engine does the lighting properties

- lighting model is more locked down

- shaders now feed the G-buffer, including color, normal, roughness, metallic

- most engines have a built-in shader editor

- shaders control material properties of 2 catergories:

  | Surface Properties | Light Properties        |
  | ------------------ | ----------------------- |
  | Base Color         | Diffuse Light           |
  | Reflectance        | Light Reflections       |
  | Surface Normal     | Shadows                 |
  | Metallicity        | Ambient Occlusion       |
  | Roughness          | Environment Reflections |
  |                    | Ambient Light           |

## Basics of PBR

PBR, aka Physically Based Rendering is a rendering technique commonly used in most game engine:

- previous rendering tried to imitate the appearance of lighting
- PBR is imitating the actual behavior of lighting, makes graphics look more real
- PBR has 2 major components: **light properties** and **surface properties**

### Light Categories

|          | Incoming Light                                               | Outgoing Light                                               |          |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| Direct   | ![Direct Light](pbr_light_categories_1.png){: height="150" } | ![Specular Light](pbr_light_categories_3.png){: height="150" } | Specular |
| Indirect | ![Indirect Light](pbr_light_categories_2.png){: height="150" } | ![Diffuse Light](pbr_light_categories_4.png){: height="150" } | Diffuse  |

if we combine the incoming and outgoing light behaviors together, we end up with 4 different main light surface interactions in computer graphics:

- **Direct Diffuse**: light coming directly from the source and getting scattered in all directions by the surface
- **Direct Specular**: light coming directly from the source and being reflected in a focus direction
- **Indirect Diffuse**: light coming from the ambient environment and getting scattered in all directions by the surface
  - computed with the engine's global illumination (**GI**) solution - often rendered offline and baked into light maps (most complex)
- **Indirect Specular**: light coming from the ambient environment and being reflected in a focus direction
  - computed using **reflection probes**, **planar reflections**, **SSR**, or **ray tracing** (fairly complex)

### Surface Categories

#### Base Color

- Defines the diffuse color of the surface
- Real-world materials are not darker than 20 or brighter than 240 [sRGB](https://en.wikipedia.org/wiki/SRGB)
- Rough surfaces have a higher minimum ~50 sRGB
- Out of range values will not light correctly, it's critical to stay in range.

#### Normal

- Defines the shape of the surface
- Each pixel represents a vector that indicates the direction that the surface is facing
- When the mesh is perfectly flat, a normal map can make it appear bumpy
- Used to add a level of surface shape detail that's impossible to achieve with triangles
- Should never be hand-authored since they represent **vector** data not the color data

#### Specular

- A multiplier for the direct and indirect specular lighting
- Defines the reflectivity when looking straight at the surface
- Non-metal surface reflect about **4%** of light, the majority objects in the world:
  - 0.5 represents 4% reflection
  - 1.0 represents 8% reflection - too high for most objects
  - tip: a crevice map multiplied by 0.5 makes a good specular map
- At glancing angles, all surfaces are 100% reflective. Fresnel term built into the engine
  - The 90-degree complement to the angle of incidence is called the **grazing angle** or **glancing angle**


#### Roughness

- Represents the roughness of the surface at a microscopic scale
- Controls the "focus" of the reflections
- **White is rough** and **black is smooth**
  - rough = blurry, diffused reflections
  - smooth = tight, sharp reflections
- Roughness map is purely artistic, up to designers to decide some interesting looking

#### Metallic

- metallic blends between a metal shader and a non-metal shader
- Base color becomes specular instead of diffuse, Metal diffuse is black
  - with base color, the specular range is up to 100%, most metal are between 60 and 100% reflective
- Use real-world measurements for metal color values and keep them bright
- **Specular input is ignored when metallic is 1**
- Grayscale values are weird, best to use solid white and black:
  - metallic maps only consist of black and white color
  - when metallic is white, be sure to use correct base color values for metal
  - all metals are 180 sRGB or brighter

## References

- Video: [UE4 Material Editor - Shader Creation](https://www.youtube.com/playlist?list=PL78XDi0TS4lFlOVKsNC6LR4sCQhetKJqs)

