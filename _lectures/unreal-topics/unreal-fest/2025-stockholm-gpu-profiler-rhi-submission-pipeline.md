# New GPU Profiler & RHI Submission Pipeline

Ref: [New GPU Profiler and RHI Submission Pipeline \| Unreal Fest Stockholm 2025](https://www.youtube.com/watch?v=vnbARZHccpQ)

[TOC]

## GPU Queue

Can be regarded analogically as hyper-threads on CPU as queue on GPU.

```mermaid
stateDiagram-v2
	classDef movement font-style:italic,font-weight:bold

	direction LR
    [*] --> GraphicsQueue
    GraphicsQueue: Graphics Queue
    state GraphicsQueue {
    	direction LR
        [*] --> Graphics0
        Graphics0 --> SFenceA
        SFenceA --> GCompute0
        GCompute0 --> Graphics1
        Graphics1 --> WFenceB
        WFenceB --> Graphics2
        Graphics2 --> GCompute1
        GCompute1 --> Graphics3
        Graphics3 --> [*]
        
        Graphics0: Graphics
        Graphics1: Graphics
        Graphics2: Graphics
        Graphics3: Graphics
        GCompute0: Compute
        GCompute1: Compute
    }

    [*] --> ComputeQueue
    ComputeQueue: Compute Queue
    state ComputeQueue {
    	direction LR
        [*] --> WFenceA
        WFenceA --> Compute0
        Compute0 --> Compute1
        Compute1 --> Compute2
        Compute2 --> SFenceB
        SFenceB --> Compute3
        Compute3 --> Compute4
        Compute4 --> [*]
        
        Compute0: Compute
        Compute1: Compute
        Compute2: Compute
        Compute3: Compute
        Compute4: Compute
    }
    
    SFenceA --> WFenceA: **Sync**
    SFenceA: ***Signal Fence A***
    WFenceA: ***Wait for Fence A***
    
    SFenceB --> WFenceB: **Sync**
    SFenceB: ***Signal Fence B***
    WFenceB: ***Wait for Fence B***
```

Fences on GPU could be regarded analogically as locks on CPU

## GPU Profiler

We have 3 components in Unreal Engine:

- `stat GPU`
- `ProfileGPU` command
- Unreal Insights

They are heavily based on RHI breadcrumbs[^1], and sharing the unified data source

[^1]: TODO

### Stat GPU

Taking `BasePass` stat from stat GPU list:

```c++
// BasePassRendering.cpp
DEFINE_GPU_DRAWCALL_STAT(Basepass);

void FDeferredShadingSceneRenderer::RenderBasePassInternal(...)
{
    // ...
    if (bRenderLightmapDensity || ViewFamily.UseDebugViewPS()) { ... }
    else
    {
        SCOPE_CYCLE_COUNTER(STAT_BasePassDrawTime);
        RDG_EVENT_SCOPE_STAT(GraphBuilder, Basepass, "BasePass");
        RDG_GPU_STAT_SCOPE(GraphBuilder, Basepass);
        
        // ...
    }
    // ...
}
```

This gpu stat is defined in the code and linked to specific RDG event scopes and RHI breadcrumb scopes.

#### UE5.6

Stat GPU becomes Multi-GPU aware and Multi-Queue aware, aka, one stat page per GPU Queue.

Commands updated:

- `stat GPU[n]_[Queue][m]`: `stat GPU0_Graphics0`, `stat GPU0_Compute0`, `stat GPU0_Copy0`
- `stat gpu` (consider as a shortcut to toggle all of them on/off)

The stats get also expanded with extra counters:

- `Busy`: the same as `stat gpu` prior to UE5.6, time spent on doing useful work as running shaders
- `Wait`: new in UE5.6, track the time used for synchronization (fencing) inter-queues
  - generally, graphics queue should be kept busy, minimize the wait time on this queue
  - the fence latency can make queue idle for a short amount of time
- `Idle`: new in UE5.6, the time GPU is idling without doing any work
  - large number might indicate that the game is CPU bound

#### Concurrency

Queues are concurrent, don't sum the busy time up to get an total GPU time, use `stat unit` instead.

### ProfileGPU

This is for **single frame** GPU trace. 

`ProfileGPU` on a packaged build emits the results to log, it's also driven by RHI breadcrumbs which includes individual RDG pass names, and same as stat GPU, it's also muti-GPU / multi-Queue aware:

- better to disable async compute (`r.RDG.AsyncCompute 0`) if in a process of optimizing a particular render pass or a shader

A list of console variables for profile gpu:

| CVar                          | CVar                           |
| ----------------------------- | ------------------------------ |
| r.profilegpu.sort             | r.profilegpu.showemptyqueues   |
| r.profilegpu.root             | r.profilegpu.showstats         |
| r.profilegpu.thresholdpercent | r.profilegpu.showpercentcolumn |
| r.profilegpu.unicodeoutput    | r.profilegpu.showinclusive     |
| r.profilegpu.showleafevents   | r.profilegpu.showexclusive     |
| r.profilegpu.showheader       | r.profilegpu.showui            |

### Unreal Insights

Used to get a detailed GPU timing information

- GPU & CPU work on the same timeline, for a better context
- one row per GPU queue

Insights use the regions to highlight the queue status, whether they are working (region `GpuWork`), waiting (region `GpuWait`) or completely idling (no region at all).

Similar to TaskGraph in CPU tracks, on GPU, insights now has the Fence arrows to indicate the dependencies between GPU queues (RDG handles)

- check the fence number to verify the synchronization situation

#### RHI Submission Thread

A new thread lies in between RHI thread and driver, which prepares work for submission to:

- batches work to minimize driver / GPU overload
- resolves fences

There is a marker `RHI_SubmitToGPU` marker on RHI thread which will connect to RHI submission thread.

On RHI submission thread, the `F<GraphicsAPI>Queue::ExecuteCommandLists` will trigger the work on GPU graphics queue.

#### RHI Interrupt Thread

Another thread called RHI interrupt thread which:

- monitors for GPU work completion
- signals the rest of the engine
- handles all GPU queues

An example: 

1. GPU queue completes a work such as Occlusion Query work, it signals a fence
2. RHI interrupt thread catches that and re-signals to the TaskGraph (which marker `InterruptQueue_Process`/`SetEventOnCompletion`)
3. A worker gets woken up and continues the `OcclusionCullPipe` work.

## Renderer Pipeline

The RHI submission pipeline is shipped in UE5.5

### RHI API

#### High-Level Types

| Category     | Types                                                        |
| ------------ | ------------------------------------------------------------ |
| Command List | - `FRHICommandListBase`<br />  - `FRHIComputeCommandList`<br />  - `FRHICommandList`<br />  - `FRHICommandListImmediate` |
| Interfaces   | - `IRHIComputeContext` / `IRHICommandContext`<br />- `IRHIPlatformCommandList` |

RHI Contexts is used for translating the rendering commands into API-specific calls (Dx12, Metal, Vulkan...)

#### High-Level Overview

Renderer or RDG creates `FRHICommandList` instances, one per recording thread and they represent CPU timeline

One **RHICmdList** can have multiple RHI Contexts:

- `RHICmdList.SwitchPipeline(...);`
- `RHICmdList.EnqueueLambdaMultiPipe(...);`

All RHICmdLists submitted via `FRHICommandListImmediate` (an singleton owned by render thread):

- `ImmCmdList.QueueAsyncCommandListSubmit({...});`
- `ImmCmdList.ImmediateFlush(...);`

All RHICmdLists are recorded on render thread and translated on rhi thread (a replay of rhi commands, calling into the rhi context and generating the actual GPU commands).

#### Switching Pipelines

|         `IRHICommandContext`<br />(Graphics Context)         |                      `FRHICommandList`                       |      `IRHIComputeCommandContext`<br />(Compute Context)      |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `Draw(...);`<br />`Draw(...);`<br />`Draw(...);`<br />`Dispatch(...);` | `SwitchPipeline(ERHIPipeline::Graphics);`<br />`Draw(...);`<br />`Draw(...);`<br />`Draw(...);`<br />`Dispatch(...);` |                                                              |
|                                                              | `SwitchPipeline(ERHIPipeline::AsyncCompute);`<br />`Dispatch(...);`<br />`Dispatch(...);` |            `Dispatch(...);`<br />`Dispatch(...);`            |
|              `Dispatch(...);`<br />`Draw(...);`              | `SwitchPipeline(ERHIPipeline::Graphics);`<br />`Dispatch(...);`<br />`Draw(...);` |                                                              |
|                                                              | `SwitchPipeline(ERHIPipeline::AsyncCompute);`<br />`Dispatch(...);`<br />`Dispatch(...);`<br />`Dispatch(...);` | `Dispatch(...);`<br />`Dispatch(...);`<br />`Dispatch(...);` |

RHICmdList direct the proper context to translate the commands.

### From Render to RHI

Some concepts:

- ***<u>Dispatch</u>***: This is a task graph task which is used to group command lists together into translate chains to achieve parallelism<br />
- ***<u>Tranlate chain</u>***: this is a task graph task which is responsible for replaying the commands recorded in RHI command list into RHI contexts.
- ***<u>RHI translation</u>***: this is the process that calls into the platform RHI implementation to generate actual GPU specific command lists.

| Graph                                                        | Notes                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![render-to-rhi-0](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-0.png) | - The render thread creates 3 new RHI command lists A, B, C<br />- Then it starts a set of tasks to record work into these command lists, each recording task runs concurrently (render graph parallel pass execution)<br />- When the command lists have been fully recorded, the recording task will call `FinishRecording`, this function signals the commandlist is completed and ready for dispatch |
| ![render-to-rhi-1](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-1.png) | - Meanwhile, the render thread  has continued on, and has called `EnqueueAsyncCommandListSubmit`, passing the 3 command lists. The order of A, B, C is significant as this is the submission order<br />- `EnqueueAsyncCommandListSubmit` will lauch a new task called dispatch task. - It starts by creating a state structure to track the progress of the submission |
| ![render-to-rhi-2](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-2.png) | - After creating the state structure, the dispatch task awaits completion of the first command list in submission order. <br />- This waiting is resolved once the recording task is called `FinishRecording`, at this point, A is ready for dispatch.<br />- Since A is the first command list in the submission, the dispatch thread starts a new translate chain.<br />- The translate chain acquires the relevant RHI context by calling `RHIGetCommandContext` and then execute command lists.<br />- the contexts acquired are stored on the chain for later use.<br />- A gets deleted once it’s done execution |
| ![render-to-rhi-3](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-3.png) | - While the translation of A is on going, the record task of B has finished.<br />- The dispatch task has been waiting for B to be completed and once it has, B will be dispatched<br />- A and B are 2 short command lists, the dispatch task decided to append B to the first translation chain.<br />- The context of the translation chain acquired while translating A are reused for B, once B is executed, it’ll be deleted |
| ![render-to-rhi-4](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-4.png) | - Dispatch task move forward, waiting for C to be completed, this time it decides to avoid appending C to the first translate chain to achieve better parallelism. It issues a close command to chain 1. <br />- This causes chain 1 to finialize the context that it acquired once all translation in that chain have completed.<br />- Finalizing an RHI context converts it into an `IRHIPlatformCommandList` pointer which contains all the platform specific GPU work generated by that context ready for submission<br />- `IRHIPlatformCommandList`  pointers are stored on chain for later<br />- Chain 2 is started by a dispatch task, as before, contexts are acquired, C is executed then deleted. |
| ![render-to-rhi-5](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-5.png) | - Render thread has continued on and called `ImmediateFlush` on the immediate command list<br />- This function flushes all prior work and will eventually result in submission of work to the GPU by appending a submit command to the end of the dispatch task<br />- The submit command runs after all prior command list dispatches have completed |
| ![render-to-rhi-6](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-6.png) |                                                              |
| ![render-to-rhi-7](https://cdn.jsdelivr.net/gh/idioticprogramape/cdn/_lectures/_unreal_fest/_new_gpu_profiler/render-to-rhi-7.png) |                                                              |

