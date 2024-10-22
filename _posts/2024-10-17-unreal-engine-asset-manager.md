---
title: Unreal Engine Asset Manager
author: ipa
date: 2024-10-17
categories: [Unreal Engine]
tags: [ue]
---

## Asset Registry

The Asset Registry provides the following functions in UE:

- powers the Content Browser in the **editor**
- stores information about saved assets even if not in memory
- refreshed at editor startup or start of cook
- uses key and value map that is class specific (for single asset)
- most data is also available in a packaged game
- provides data storage for asset manager

## Asset Manager

The Asset Manager is a singleton global UObject in Unreal Engine, it's not attached to any specific game map or game mode:

- categories and queries unloaded assets using the Asset Registry
- maintains global asset loading state (memory state)
- integrates existing systems like cooking and async loading
- designed to be overridden by games

Using the Asset Manager can be helpful to resolve:

- long load times and high memory usage
  - by selectively load object during different loading states
- move from hitchy hard refs to async loaded soft refs
- complicated packing/chunking rules

Here are all the Asset Manager Components:

- Primary Asset Type: one per item **class** (descriptor)
- Primary Asset Identifier: One per item **asset**
- Primary Asset Rules: Cooking/chunking/management  rules
- Asset Manager Settings: Specifies where to load each type
- Secondary asset: Textures, meshes, etc, Anything not a primary asset

### Primary Asset Type Definition

- Use unique subclass of `UPrimaryDataAsset`, either from a data class or just in the Blueprint editor
- can also be prototyped with any asset, but need extra setup for cook
- Need set scan options in Asset Manager Settings
- Identifier uses shot name of asset by default

### Asset Manager Settings

#### Primary Asset Types To Scan

Go to `Project Settings -> Game -> Asset Manager`, the settings contains all the meta data for UE to scan a primary asset type:

```ini
[/Script/Engine.AssetManagerSettings]
@PrimaryAssetTypesToScan=PrimaryAssetType

; Note the default rule here for both PrimaryAssetLabels and Maps sets them to bIsEditorOnly=true, which means the
; AssetManager will NOT use them to pull assets into the build; they will only label the target chunks (aka pak files)
; into which assets pulled in by other sources (e.g. ProjectSettings) will be put. It also sets the CookRule=Unknown,
; which is another way of saying that they do not impact whether the assets labeled by the PrimaryAssetLabel or Map are
; cooked. If you want PrimaryAssetLabels or maps to cause assets to be cooked even if the label and its assets are otherwise
; unreferenced, then set bIsEditorOnly=false and CookRule=AlwaysCook. 

+PrimaryAssetTypesToScan=(PrimaryAssetType="Map",AssetBaseClass=/Script/Engine.World,bHasBlueprintClasses=False,bIsEditorOnly=True,Directories=((Path="/Game/Maps")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=Unknown))
+PrimaryAssetTypesToScan=(PrimaryAssetType="PrimaryAssetLabel",AssetBaseClass=/Script/Engine.PrimaryAssetLabel,bHasBlueprintClasses=False,bIsEditorOnly=True,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=Unknown))
```

More details can be found in struct `FPrimaryAssetTypeInfo`. The settings can be accessed via:

```c++
const TArray<FPrimaryAssetTypeInfo>& Infos = UAssetManager::Get().GetSettings().PrimaryAssetTypesToScan;
```

### Asset Bundle

It's more efficient to load with asset manager using asset bundles:

- load related resources only when needed, reduce the memory usage
  - support multiple game modes: menu, in-game, client, server, etc
- `AssetBundles` meta tag on soft reference inside `UPrimaryDataAsset`
- override `UpdateAssetBundleData` for complex cases
- update some with `ChangeBundleStateForPrimaryAssets`, will async load
- update everything with `ChangeBundleStateForMatchingPrimaryAssets`

## Loading Best Pratices

- Only sync load during load screens or as fast response to input (bad choice in general)
- Change bundle state and clear caches during mode switches
- Use **Streamable Handles** to sync load or keep in memory
- **Load With Delegate** vs **Preload Plus Fallback**
- Never use unsafe lambdas as delegates

### C++ Usage

#### Useful Functions

```c++
// Get the global asset manager
UAssetManager& AssetManager = UAssetManager::Get();

// Get a list of all specific item type that can be loaded
AssetManager.GetPrimaryAssetIdList(ItemType, /* OUT */AssetIdList);

// Get tag/value data for a unloaded item
AssetManager.GetPrimaryAssetData(ItemIdList[0], /* OUT */AssetDataToParse);

// Permanently load a single item
FPrimaryAssetId ItemId = FPrimaryAssetId(ItemType, ItemName);
AssetManager.LoadPrimaryAsset(ItemId, CurrentLoadState, DelegateFunction);

// From DelegateFunction
UExampleItem* ExampleItem = AssetManager.GetPrimaryAssetObject<UExampleItem>(ItemId);

// Release previously loaded item
AssetManager.UnloadPrimaryAsset(ItemId);

// Load a list of items as long as handle is alive
Handle = AssetManager.PreloadPrimaryAssets(ListOfPrimaryAssetIds, CurrentLoadState, false);
```

The `CurrentLoadState` in the previous code examples are a list of Asset Bundles.

#### Example: Loading Assets At Startup

If we already know some `FPrimaryAssetType`, i.e. `TArray<FPrimaryAssetType>` that we would like to load from the beginning of the game, we can trigger a delegate from `BeginPlay` event:

```c++
/**
 *  UClasses used in the callback function downbelow, put it here for clearity
 */
UCLASS()
class MyGame_API UMyAssetManager : public UAssetManager
{
    GENERATED_BODY()
        
public:
    /** static types for items */
    static const FPrimaryAssetType WeaponItemType;
    static const FPrimaryAssetType SkillItemType;
    
    UFUNTION()
    void OnLoadWeaponAssetCallbackFunction(FPrimaryAssetId AssetId)
    {
        UWeaponItem* WeaponItem = GetPrimaryAssetObject<UWeaponItem>(AssetId);
        
        if(IsValid(WeaponItem))
        {
            UE_LOG(LogTemp, Log, TEXT("Loaded Weapon %s"), *WeaponItem->GetName());
        }
        
        // Release previously loaded item
        UnloadPrimaryAsset(WeaponId);
    }
    
    UFUNCTION()
    void OnInitializeItemsCompleteCallback();
    
    // ... other types, declarations/definitions
};

void UMyAssetManager::OnInitializeItemsCompleteCallback()
{
    // get the global asset manager
    UMyAssetManager& AssetManager = UMyAssetManager::Get();
    
    // get a list of all item i.e. weapon that can be loaded
    TArray<FPrimaryAssetId> WeaponIdList;
    AssetManager.GetPrimaryAssetIdList(WeaponItemType, WeaponIdList);
    
    for(const FPrimaryAssetId& WeaponId : WeaponIdList)
    {
        // get tag/value data for an unloaded weapon
        FAssetData AssetDataToParse;
        AssetManager.GetPrimaryAssetData(WeaponId, AssetDataToParse);
        
        FName QueryExample;
        AssetDataToParse.GetTagValue(GET_MEMBER_NAME_CHECKED(UMyAssetItem, ExampleRegistryTag), QueryExample);
        
        UE_LOG(LogTemp, Log, TEXT("Read ExampleRegistryTag %s from Weapon %s"), *QueryExample.ToString(), *AssetDataToParse.AssetName.ToString());
    }
    
    // permanently load a single item (Asset bundle)
    TArray<FName> CurrentLoadState;
    CurrentLoadState.Add(FName("Game"));
    
    FName WeaponName = TEXT("Weapon_Bo_1");
    FPrimaryAssetId WeaponId = FPrimaryAssetId(WeaponItemType, WeaponName);
    AssetManager.LoadPrimaryAsset(WeaponId, CurrentLoadState, FStreamableDelegate::CreateUObject(&AssetManager, &UMyAssetManager::OnLoadWeaponAssetCallbackFunction, WeaponId))
    
    TArray<FPrimaryAssetId> ListOfPrimaryAssetIds;
    AssetManager.GetPrimaryAssetIdList(SkillItemType, ListOfPrimaryAssetIds);
    
    // load a list of items as long as handle is alive
    TSharedPtr<FStreamableHandle> Handle = AssetManager.PreloadPrimaryAssets(ListOfPrimaryAssetIds, CurrentLoadState, false);
    
    // store the handle to an object which can define its life cycle
    // ...
}

UCLASS(Abstract, BlueprintType)
class MyGame_API UMyAssetItem : public UPrimaryDataAsset
{
    GENERATED_BODY()
        
public:
    /** some fields, ctor, dtor, etc... */
    
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Item, meta=(AssetBundles="Menu"))
    TSoftObjectPtr<UTexture> DisplayTexture;
    
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Item, meta=(AssetBundles="Game"))
    TSoftObjectPtr<UTexture> HitSound;
    
    // this is a meta data which is allowed to be queried later on
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Item, AssetRegistrySearchable)
    FName ExampleRegistryTag;
    
    /** the rest of the declarations/definitions */
};

void UExampleGameInstance::BeginPlay()
{
    Super::BeginPlay();
    
    // to load the persistent assets for the entire game
    InitializeItems();  
}

void UExampleGameInstance::InitializeItems()
{
    UMyAssetManager& AssetManager = UMyAssetManager::Get();
    
    for(const FPrimaryAssetType& ItemTypeToLoad : ItemsTypesToLoad)
    {
        TArray<FPrimaryAssetId> PrimaryAssetIdList;
        Manager.GetPrimaryAssetIdList(ItemTypeToLoad, PrimaryAssetIdList);
        
        // all the async action can take this class as an example
        // quite classic way of doing things in UE
        UAsyncActionLoadPrimaryAssetList* Action = UAsyncActionLoadPrimaryAssetList::AsyncLoadPrimaryAssetList(this, PrimaryAssetList, {});
        Action.AddDynamic(this, &ThisClass::OnAssetTypeAsyncLoadCompleteCallback);
    }
    
    // add add another delegate to notify the whole loading is done
    // for example... will trigger UMyAssetManager::OnInitializeItemsCompleteCallback
    OnInitializeItemsComplete.Broadcast();
}

// this function MUST be decorated by UFUNCTION() (to be used in AddDynamic)
// in blueprint would be easier to do
void UExampleGameInstance::OnAssetTypeAsyncLoadCompleteCallback(const TArray<UObject*>& LoadedAssets)
{
    for(const UObject* LoadedAsset : LoadedAssets)
    {
        if(IsValid(LoadedAsset))
        {
             UE_LOG(LogTemp, Display, TEXT("'%s' is loaded"), LoadedAsset->GetName());
        }
    }
}
```

### Cooking

Asset Manager is the primary game interface with cooking:

- override `ModifyCook` function directly
- override `FinishInitialLoading` to load other things before cook, anything in memory will be cooked
- set `CookRules` to stop prototype assets from cooking
  - via `GetPackageCookRule`, `PrimaryAssetLabels` or directly in config

### Chunking

#### Definition

Chunking is a process that assign individual assets to different pak/iostore/staged files

- one chunk identifier per stage file, 0 is default (highest priority)
- enabled with `bGenerateChunks` packaging setting
- create hierarchy with `ChunkDependencyInfo`
- needed for platform-specific features like language-specific downloads
- works with `ChunkDownloader` plugin

#### Chunk Assignment & Cook Rules

- Chunk/CookRules apply to all "managed" secondary assets
  - higher priority applied first: ***modding idea*** ?
  - assign directly in `AssetRules` type config: check the asset manager settings
  - set `CustomPrimaryAssetRules` in config: check the asset manager settings
  - create `PrimaryAssetLabels` to recursively set
  - override `ShouldSetManager` to change global rules, can be complicated
