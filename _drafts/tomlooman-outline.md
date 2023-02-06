# Tom Looman's Outline

site: https://courses.tomlooman.com/

## Lecture2 - Project Start & Version Control

- UE C++ naming convention
- `UCameraComponent` and `USpringArmComponent` in a customized `ACharacter` pawn
  - `CreateDefaultSubobject` to new a component
  - `SetupAttachment` to build up parent-children relation
  - `UPROPERTY`: `VisibleAnywhere`
- `SetupPlayerInputComponent` in `ACharacter`
  - `UInputComponent::BindAxis` and `UInputComponent::BindAction`
  - `ACharacter::AddMovementInput`
    - `AActor::GetActorForwardVector` and `AActor::GetActorRightVector`
  - `APawn::AddControllerYawInput` and `APawn::AddControllerPitchInput`
- Github repo setup for an UE project

## Lecture3 - Gameplay, Collision, and Physics

- Directions in UE: `X` - Forward, `Y` - Right, `Z` - Up
- `ACharacter`:
  - (Player)Controller: `APlayerController` and `AAIController`
    - rotations for the controller not for the pawn
    - in Character blueprint ***Pawn*** section:
      - if checked `Use Controller Rotation Yaw`, the pawn itself will use controller's rotation.
      - in C++, it's `bUseControllerRotationYaw`
    - to get the controller rotation, use `GetControllerRotation()` (this is not the rotation of the pawn that the controller currently possessed)
      - if used to control character's forward movement, use `UKismetMathLibrary::GetForwardVector(FRotator)`, need to set **pitch** and **roll** to 0
      - if used to control character's movement to right, use `UKismetMathLibrary::GetRightVector(FRotator)`, need to set **pitch** and **roll** to 0
  - `USpringArmComponent`:
    - in Character blueprint, sprint arm component ***Camera Setting*** section:
      - check `Use Pawn Control Rotation`, otherwise the camera won't move with the mouse
      - in C++, it's `bUsePawnControlRotation`
  - `UCharacterMovementComponent`:
    - in Character blueprint, character movement component ***Character Movement (Rotation)*** section:
      - check `Orient Rotation to Movement`, 
      - in C++, it's `bOrientRotationToMovement`
        - to get the movement component, use `GetCharacterMovement()`
- PIE settings: `Play` -> `Advanced Settings` -> `Play in Editor` -> `Game Gets Mouse Control`, change to `true` to let game automatically gain the control over mouse instead of click it once we hit the PIE
- Editor preferences: `General - Loading & Saving` -> `startup` -> `Load Level at Startup`, to set the default map  
- A customized actor as projectile:
  - `USphereComponent`: as basic collider
  - `UProjectileMovementComponent`: which provides a velocity to the actor
    - `InitialiSpeed`
    - `bRotationFollowsVelocity`
    - `bInitialVelocityInLocalSpace`
    - `ProjectileGravityScale`: 0 to disable the gravity
  - `UParticleSystemComponent`: visuals
  - should be launched through the character, in **character class**, need to ask to spawn a projectile:
    - the spawning is always done through the world, use `UWorld::SpawnActor<T>(SpawnClass, SpawnTransform, SpawnParams)`
      - the `SpawnClass` could be any subclass of this projectile base class. `TSubclassOf<T>`
        - `UPROPERTY`: `EditAnywhere`
      - the `SpawnTransform`is the place where this object should be spawned in the world, i.e. spawned from character's hand:
        - use `USkeletalMesh`'s `Skeleton` section to locate the hand socket, `Muzzle01`.
        - use function `USkeletalMesh::GetSocketLocation(SocketName)` to get the correct location on the mesh
        - `GetMesh` is used to get the character's skeletal mesh
      - the `SpawnParams` is of type `FActorSpawnParameters`:
        - `SpawnCollisionHandlingOverride`: collision check method before spawning
  - Collisions

 