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