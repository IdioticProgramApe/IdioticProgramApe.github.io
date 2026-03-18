## Basics of Frame Pacing

Frame rate describe how many frame rendered in one second in game, which is an average number, frame pacing on the other hand describes how consistent the frames are.

Having a good frame rate doesn't mean have a smooth user feeling experience.

| Console                                 | Note                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| `stat unitgraph`                        | in game performance monitor                                  |
| `stat raw`                              | enable unfiltered stats on unit graph                        |
| `t.targetframetimethreshold <ms>`       | adjust the unit graph y-axis as well as the target frame time |
| `t.unacceptableframetimethreshold <ms>` | beyond which the performance value will be red               |

Some common causes affecting the frame pacing:

- running slow code on lower frequencies, using timers or lower frequency ticks
- waiting on expensive ops, not using async ops:
  - physics line traces: async line trace -> act on the results at the beginning of the next frame
- creating too many GPU resources in one frame could stall the game
  - streaming, add too many lights (especially cast shadows)
- (DX12) JIT compiling PSOs
- running garbage collecting (GC)
  - avoid gc, i.e. object pooling (memory overhead)
- spawning/streaming too many object at once