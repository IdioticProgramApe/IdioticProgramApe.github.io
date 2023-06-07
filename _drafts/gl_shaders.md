## GLSL

### Built-in Keywords

| Keyword                | Note                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `gl_Position`          | as the output of the vertex shader, give the final position information |
| `#version`             | setup opengl compatibility version, as `#version 330`, at least opengl3.3 |
| `layout (location=0)`  | coming from the vertex array, possibly including the position info, uv info and maybe more |
| `vec2`, `vec3`, `vec4` | vector in 2, 3, 4 dimensions                                 |
| `mat2`, `mat3`, `mat4` | matrix in 2x2, 3x3, 4x4 shape                                |
| `in`                   | as the input of the shader                                   |
| `out`                  | as the output of the shader                                  |
| `uniform`              | act as the static variable in the rendering pipeline, syntax: `uniform <type> <var_name>` |
| `sampler2D`            | texture2d sampler type, used to pick the color information from the bounded texture unit and the given texture coordinates |
| `struct`               | define a customized data type                                |

### Built-in Functions

| Function    | Note                                                         |
| ----------- | ------------------------------------------------------------ |
| `clamp`     | clamp the value(s) so that the value(s) fall in the demanded range: `vec (*clamp)(vec, min, max)` |
| `texture`   | function to get the color information: `vec4 (*texture)(sampler2D, vec2)` |
| `inverse`   | get the inversed matrix of the given matrix: `mat (*inverse)(mat)` |
| `transpose` | get the transposed matrix of the give matrix: `mat (*transpose)(mat)` |
| `reflect`   | get the reflected vector of the incident vector: `vec (*reflect)(vec incident, vec normal)` |
| `dot`       | get the dot product result from 2 vectors: `float (*dot)(vec, vec)` |
| `max`       | get the max value from the given values: `float (*max)(float, float, ...)` |

### Useful Techniques

| Name              | Method                                                       |
| ----------------- | ------------------------------------------------------------ |
| Model Matrix      | usually composed of translate, rotate and rotate matrices    |
| Normal Matrix     | the inverse transpose of Model Matrix                        |
| View Matrix       | usually composed of View Rotation and View Position matrices |
| Projection Matrix | see `glm::perspective` and `glm::ortho`                      |

