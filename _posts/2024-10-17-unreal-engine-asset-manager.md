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
- Need set scan options in AssetManager Settings
- Identifier uses shot name of asset by default

### Examples

#### Loading Assets At Startup

If we already know some `FPrimaryAssetType`, i.e. `TArray<FPrimaryAssetType>` that we would like to load from the beginning of the game, we can trigger a delegate from `BeginPlay` event:

```c++
void UExampleGameInstance::BeginPlay()
{
    Super::BeginPlay();
    InitializeItems();  // in charge of loading the asset for the game
}

void UExampleGameInstance::InitializeItems()
{
    UAssetManager& Manager = UAssetManager::Get();
    
    for(const FPrimaryAssetType& ItemTypeToLoad : ItemsTypesToLoad)
    {
        TArray<FPrimaryAssetId> PrimaryAssetIdList;
        Manager.GetPrimaryAssetIdList(ItemTypeToLoad, PrimaryAssetIdList);
        
        UAsyncActionLoadPrimaryAssetList* Action = UAsyncActionLoadPrimaryAssetList::AsyncLoadPrimaryAssetList(this, PrimaryAssetList, {});
        Action.AddDynamic(this, &ThisClass::OnAssetAsyncLoadComplete);
    }
    
    // add add another delegate to notify the whole loading is done
    // for example...
    OnInitializeItemsComplete.Broadcast();
}

// this function MUST be decorated by UFUNCTION() (to be used in AddDynamic)
// in blueprint would be easier to do
void UExampleGameInstance::OnOneAssetTypeAsyncLoadComplete(const TArray<UObject*>& LoadedAssets)
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



