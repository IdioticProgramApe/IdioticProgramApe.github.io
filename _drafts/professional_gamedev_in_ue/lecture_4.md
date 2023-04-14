## Lecture4 - Interfaces & Collision Queries

- C++ interfaces, inherited from Unreal Interface class, used to extend some functionalities to a class
- A default Interface class should have 2 parts:
    - A `UObject` class inherited from `UInterface` base class, as a boilerplate in unreal, shouldn't be modified
    - A new class whose name starts with **I** where all the interface functions should be declared (no implementation)
    - To check if a class has implementation for one interface, use:
      - `SomeInstance->Implements<USomeInterface>()`: this function will return true if found a implementation
      - note: use **U** instead of **I**
    - To call the implemented function, use:
      - `ISomeInterface::Execute_SomeFunction(SomeInstance, ...)`: the function name will be prefixed with **Execute_**, the first parameter will be the caller object, the original parameters follow after.
- Best practices for an interaction function, it'd be good to have:
    - an instigator: the actor who start the interaction

- `UFUNCTION(...)`: 
    - `BlueprintImplementableEvent`: used when the decorated function's implementation only exists in the blueprint
    - `BlueprintNativeEvent`: has implementation in C++ with postfix **_Implementation** for the function name, can be override in blueprint
        - for the children classes, need to declare function name post fixed with **_Implementation**

- Customized actor component:
    - i.e. to detect all the surroundings and return one which can be interacted with
    - the way to detect object in unreal is to use tracing and collision:`UWorld::LineTraceSingleByObjectType(...)`, it will need:
        - `FHitResult`: this holds the tracing results
            - `FHitResult.Actor`: the actor the line hits
            - `FHitResult.Component`: the component within that actor the line hits
            - `FHisResult.ImpactPoint`: the world location of the hit point

        - line's start and end locations
        - `FCollisionObjectQueryParams`: this parameter holds all the object types we want to query

- `AActor`:
    - `GetActorEyesViewPoint(...)`: give the actor's view location and view rotation
    - this function is overridden by some other classes, as `APawn`, `AController`, etc.


- Debug helper functions (ref *DrawDebugHelper.h*):
  - `DrawDebugLine(...)`: will draw a line in the current world with specified start/end points, color, thickness, lifetime, etc.
- 

