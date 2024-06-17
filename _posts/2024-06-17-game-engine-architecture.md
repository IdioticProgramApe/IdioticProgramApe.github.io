---
title: Game Engine Architecture
author: ipa
date: 2024-06-17
categories: [Engine Programming]
tags: [theory, design, coding]
mermaid: true
---

## Structure

This section is to briefly introduce what the game engine generally look like, in other words, these days, the game engine will probably need to have the following sections/features:

- a game application (which in form of executable, can be hot-reloadable through dlls or so depend on the target platforms)
- a editor application (which a collection of tools to help build game, and in form of executable)
- optionally a testbed (also an executable which is used to test all the engine features, usually not very structured)

```mermaid
stateDiagram-v2
	Engine: Engine (DLL, so)
	Game: Game Application (EXE)
	Code: Hot-reloadable Game Code (DLL, so)
	Editor: Editor (EXE)
	TestBed: Test Bed (EXE)
	
	Game --> Engine
	Game --> Code
    Code --> Engine
    Editor --> Engine
    TestBed --> Engine
```

## Features

Generally speaking, for a game engine, there following features should be provided (or preferred to have):

- build system (to help with generating the game build/editor executable etc...)
- low-level utilities:
  - dynamic arrays (similar to `std::vector`)
  - string handling,
  - etc...
- low-level platform layer
  - platform-wise windowing (for desktop platforms)
  - platform-wise input
  - platform-wise console interaction
  - etc...
- Logger
  - multi-threaded and performant
- File IO
  - some different file types as text, images, etc...
- Application layer
  - high-level implementation such as game loop
- Renderer/API Abstraction layer (large topic)
  - usually involves a frontend/backend system
  - backend communicates the lower level of the actual APIs
- Memory management (allocators etc...)
- Scene graph/ECS (Entity Component System)
- Profiling/Debugging utilities
- "Scripting" support via hot-reloading
- Physic system

## Architecture

```mermaid
stateDiagram-v2
	AudioLayer: Audio
	RendererLayer: Renderer
	ResourceLayer: Resource Management
	CoreLayer: Core
	PlatformLayer: Platform Layer
	
	Others --> ResourceLayer
	Others --> AudioLayer
	Others --> RendererLayer
	AudioLayer --> ResourceLayer
	RendererLayer --> ResourceLayer
	ResourceLayer --> CoreLayer
	CoreLayer --> PlatformLayer
	
	state Others{
		Animation/Timelines
		Collision/Physics
		EventSystem: Event System
		ECS: Entity Component System
		StateMachine: State Machines
		etc...
	}
	
	state AudioLayer{
		AudioFile: 2D/3D Audio
		Effects
		Playback: Playback Management
	}
	
	state RendererLayer{
		RendererBack: Renderer Backend
		RendererFront: Renderer Frontend
		
		RendererBack --> RendererFront
		RendererFront --> RendererBack
		
		state RendererBack{
            API: API Abstraction
            Material/Lighting
            Shaders
            Meshes
            Textures
        }

        state RendererFront{
            GUI
            Scenegraph
            Camera: Camera Management
            PostFX
        }
	}
	
	state ResourceLayer{
		Images
		Materials
		Meshes
		Animations
		Maps: World Maps
	}
	
	state CoreLayer {
		Core1: Core Part I
		Core2: Core Part II
	
		Core1 --> Core2
		Core2 --> Core1
	
		state Core1 {
            Logger
            Assertions
            DataStructure: Data Structures
            MemoryAllocators: Memory Allocators
            Math: Maths Library
            Config: Engine Configuration
		}
		
		state Core2 {
			Parser: Parsers (XML, CSV, JSON, etc...)
            Profiling
            AsyncFiles: Async File I/O
            Localization
            StringLib: String Library
            RNG
		}
	}
	
	state PlatformLayer {
        PlatformApi: Platform APIs (Windows/Linux/Mac...)
        ConsoleOutput: Console Output
        FileIO: File I/O
        Memory
        Renderer: Renderer API Extensions 
    }
```

It's worthy nothing that all the component listed in this diagram are in form of generic illustration, no detail provided.

There might be other important modules/elements not listed here, to be continued.

#### Note:

- RNG: Random Number Generator
