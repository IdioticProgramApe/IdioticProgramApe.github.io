## Frame Time (ms) vs. Frame Rate (fps)

| Name       | Unit     | Use Case                                        |
| ---------- | -------- | ----------------------------------------------- |
| Fame Time  | ms       | describe time consumption, used in optimization |
| Frame Rate | fps / Hz | set targets i.e. 30 fps, 60 fps                 |

From the following image, we can see that:

![frametime-framerate](https://www.intel.com/content/dam/developer/articles/training/unreal-engine-4-optimization-tutorial/img1.png)

Frame rate and frame time are inversely proportional $\displaystyle\frac{1}{FPS} \cdot 1000 = MS$, by just giving out how many fps we save from an optimization, we cannot know how much time we save, save 30 fps it could be:

- 16.67 ms (30 fps -> 60 fps)
- 5.56 ms (60 fps -> 90 fps)
- 2.77 ms (90 fps -> 120 fps)