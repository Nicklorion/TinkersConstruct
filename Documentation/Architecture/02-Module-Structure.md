# Module Structure - Tinkers' Construct

## Package Organization

This document provides a comprehensive breakdown of every major package in Tinkers' Construct, its purpose, key classes, and responsibilities.

## Root Package: `slimeknights.tconstruct`

### `TConstruct.java`
Main mod class - handles initialization, module registration, and lifecycle.

**Key Responsibilities:**
- Mod initialization
- Module registration (Tools, Tables, Smeltery, etc.)
- Event bus setup
- Plugin/integration loading
- Data generation coordination

## Foundation Modules

### 1. Library Module (`library/`)

The library module contains all core APIs, interfaces, and abstract implementations. This is the contract layer that the rest of the mod builds upon.

#### Key Sub-Packages:

**`library/materials/`** - Material System Foundation
- `MaterialRegistry.java`: Central registry for all materials
- `definition/`: Material definitions and IDs
  - `IMaterial.java`: Material interface
  - `MaterialId.java`: Material identifier
  - `MaterialManager.java`: Loads material definitions from JSON
  - `MaterialVariantId.java`: Material variants (e.g., different colors)
- `stats/`: Material statistics
  - `IMaterialStats.java`: Stats interface
  - `MaterialStatsId.java`: Stat type identifier
  - `MaterialStatsManager.java`: Loads stats from JSON
  - `BaseMaterialStats.java`: Base implementation
- `traits/`: Material traits
  - `MaterialTraitsManager.java`: Loads traits from JSON
  - Trait definitions and application logic

**`library/tools/`** - Tool System Foundation
- `definition/`: Tool structure definitions
  - `ToolDefinition.java`: Defines a tool type
  - `ToolDefinitionLoader.java`: Loads tool definitions from JSON
  - `ToolDefinitionData.java`: Data structure for tool definitions
  - `PartRequirement.java`: Required parts for tools
- `item/`: Tool item implementations
  - `IModifiable.java`: Interface for modifiable items
  - `ToolItem.java`: Base class for tools
  - `ModifiableItem.java`: Items that can have modifiers
- `nbt/`: NBT data structures
  - `ToolStack.java`: Complete tool data wrapper
  - `MaterialNBT.java`: Material composition data
  - `StatsNBT.java`: Tool statistics data
  - `ModifierNBT.java`: Applied modifiers data
  - `IToolStackView.java`: Read-only tool view
- `stat/`: Tool statistics
  - `IToolStat.java`: Tool stat definition
  - `ToolStats.java`: Standard tool stats registry
  - `ModifierStatsBuilder.java`: Stat calculation with modifiers
- `part/`: Tool parts
  - `IToolPart.java`: Interface for tool parts
  - `MaterialItem.java`: Items made from materials
- `layout/`: UI layout for crafting stations
  - `StationSlotLayout.java`: Slot arrangement
  - `StationSlotLayoutLoader.java`: Loads layouts from JSON
  - `Patterns.java`: Part patterns for crafting
- `helper/`: Helper utilities for tools
  - `ToolBuildHandler.java`: Tool assembly logic
  - `ToolDamageUtil.java`: Durability management
  - `ToolHarvestLogic.java`: Mining/harvesting logic
  - `ModifierUtil.java`: Modifier application helpers

**`library/modifiers/`** - Modifier System
- `Modifier.java`: Base modifier class
- `ModifierId.java`: Modifier identifier
- `ModifierManager.java`: Loads and manages modifiers
- `ModifierEntry.java`: Modifier with level
- `hook/`: Modifier hooks for events
  - `HarvestEnchantmentsModifierHook.java`
  - `ConditionalStatModifierHook.java`
  - `DisplayNameModifierHook.java`
  - Many more specialized hooks...

**`library/recipe/`** - Recipe System
- `TinkerRecipeTypes.java`: Custom recipe types
- `worktable/`: Modifier worktable recipes
- `casting/`: Casting table/basin recipes
- `melting/`: Melting recipes
- `alloying/`: Alloy smelting recipes
- `partbuilder/`: Part builder recipes
- `tinkerstation/`: Tinker station recipes

**`library/client/`** - Client-Side APIs
- `model/`: Model rendering
  - `ToolModel.java`: Dynamic tool model
  - `MaterialModel.java`: Material-based models
  - `FluidContainerModel.java`: Fluid rendering
- `materials/`: Material rendering
  - `MaterialRenderInfo.java`: Material colors and textures
  - `MaterialRenderInfoLoader.java`: Loads render data
- `modifiers/`: Modifier rendering
  - `ModifierModelManager.java`: Manages modifier models
- `book/`: In-game manual
  - Book content and formatting

**`library/json/`** - JSON Utilities
- `predicate/`: Conditional logic
  - `IJsonPredicate.java`: Base predicate interface
  - `entity/`: Entity predicates
  - `item/`: Item predicates
  - `modifier/`: Modifier predicates
- `variable/`: Variable systems for data
- `serializer/`: Custom JSON serializers
- `field/`: JSON field utilities

**`library/events/`** - Event System
- `TinkerToolEvent.java`: Tool-related events
- `MaterialsLoadedEvent.java`: Material loading event
- `teleport/`: Teleportation events

**`library/utils/`** - Shared Utilities
- `Util.java`: General utilities
- `NBTTags.java`: NBT tag constants
- `RestrictedCompoundTag.java`: Safe NBT access
- `ToolHelper.java`: Tool manipulation helpers

### 2. Common Module (`common/`)

Shared infrastructure used across all modules.

#### Key Sub-Packages:

**`common/config/`** - Configuration
- `Config.java`: Main configuration
- Forge config specs

**`common/network/`** - Networking
- `TinkerNetwork.java`: Network setup
- Packet definitions for client-server sync

**`common/data/`** - Data Generation
- Data providers for resources
- Tag providers
- Recipe providers
- Loot table providers
- Advancement providers

**`common/recipe/`** - Common Recipes
- `RecipeResult.java`: Recipe output wrapper
- Shared recipe logic

**`common/registration/`** - Registration Helpers
- `CastItemObject.java`: Cast item registration
- Registration utilities

### 3. Shared Module (`shared/`)

Cross-module game content.

#### Key Sub-Packages:

**`shared/block/`** - Common Blocks
- `SlimeBlock.java`: Slime blocks
- `ClearStainedGlassBlock.java`: Glass variants

**`shared/item/`** - Common Items
- Shared items used across modules

**`shared/TinkerMaterials.java`** - Materials Module
- Material registrations

**`shared/TinkerCommons.java`** - Commons Module
- Common registrations

**`shared/TinkerClient.java`** - Client Setup
- Client-side initialization

## Feature Modules

### 4. Tools Module (`tools/`)

Implementation of the tool system.

#### Key Sub-Packages:

**`tools/item/`** - Tool Items
- `ModifiableArmorItem.java`: Armor pieces
- `ModifiableSwordItem.java`: Swords
- `ModifiableItem.java`: Generic modifiable items
- Specific tool implementations (pickaxe, axe, shovel, etc.)

**`tools/stats/`** - Stat Implementations
- `HeadMaterialStats.java`: Tool head stats
- `HandleMaterialStats.java`: Handle stats
- `ExtraMaterialStats.java`: Extra part stats
- `BowstringMaterialStats.java`: Ranged weapon stats
- `GripMaterialStats.java`: Armor grip stats
- `PlatingMaterialStats.java`: Armor plating stats

**`tools/modifiers/`** - Modifier Implementations
- `ability/`: Active ability modifiers
- `armor/`: Armor-specific modifiers
- `effect/`: Effect-applying modifiers
- `traits/`: Material trait implementations
- `upgrades/`: Tool upgrade modifiers
- `slotless/`: Modifiers that don't use slots

**`tools/data/`** - Tool Data Generation
- `material/`: Material data generation
- `sprite/`: Sprite generation
- Modifier data generation

**`tools/TinkerModifiers.java`** - Modifier Registry Module
- Registers all modifiers

**`tools/TinkerTools.java`** - Tool Registry Module
- Registers all tools

**`tools/TinkerToolParts.java`** - Part Registry Module
- Registers all tool parts

### 5. Tables Module (`tables/`)

Crafting stations and UI.

#### Key Sub-Packages:

**`tables/block/`** - Station Blocks
- `CraftingStationBlock.java`: Basic crafting station
- `TinkerStationBlock.java`: Tinker station
- `PartBuilderBlock.java`: Part builder
- `TinkersAnvilBlock.java`: Tinkers' anvil
- `ModifierWorktableBlock.java`: Modifier application table

**`tables/block/entity/`** - Station Block Entities
- `table/`: Table block entities
  - `CraftingStationBlockEntity.java`
  - `TinkerStationBlockEntity.java`
  - `PartBuilderBlockEntity.java`
  - `ModifierWorktableBlockEntity.java`
- `chest/`: Chest block entities
  - `TinkersChestBlockEntity.java`
  - `PartChestBlockEntity.java`
  - `CastChestBlockEntity.java`

**`tables/menu/`** - UI Menus
- Container/menu implementations for each station
- Slot management
- Crafting logic integration

**`tables/client/`** - Client Rendering
- Screen implementations
- UI rendering

**`tables/recipe/`** - Station Recipes
- `TinkerStationRepairRecipe.java`: Tool repair
- `TinkerStationPartSwapping.java`: Part replacement
- Crafting integration

**`tables/TinkerTables.java`** - Tables Module
- Registers all tables and related content

### 6. Smeltery Module (`smeltery/`)

Multiblock smeltery and ore processing.

#### Key Sub-Packages:

**`smeltery/block/`** - Smeltery Blocks
- `controller/`: Smeltery controller
- `component/`: Smeltery components (drains, ducts, etc.)
- Smeltery structure blocks

**`smeltery/block/entity/`** - Smeltery Block Entities
- `controller/`: Controller logic
- `module/`: Modular components
- Tank management
- Fuel consumption
- Alloy creation

**`smeltery/client/`** - Smeltery Rendering
- Fluid rendering in structure
- Controller UI

**`smeltery/data/`** - Smeltery Data
- Recipe data generation
- Fluid data

**`smeltery/TinkerSmeltery.java`** - Smeltery Module
- Registers smeltery content

### 7. Fluids Module (`fluids/`)

Custom fluid handling.

#### Key Sub-Packages:

**`fluids/fluids/`** - Fluid Definitions
- Custom fluid implementations
- Molten metals
- Special fluids

**`fluids/block/`** - Fluid Blocks
- Fluid block implementations

**`fluids/item/`** - Fluid Containers
- Buckets and containers

**`fluids/TinkerFluids.java`** - Fluids Module
- Registers all fluids

### 8. Gadgets Module (`gadgets/`)

Utility items and blocks.

#### Key Sub-Packages:

**`gadgets/item/`** - Gadget Items
- Slimeslings
- Fancy items
- Utility tools

**`gadgets/entity/`** - Gadget Entities
- Projectiles
- Special entities

**`gadgets/TinkerGadgets.java`** - Gadgets Module
- Registers gadgets

### 9. World Module (`world/`)

World generation and structures.

#### Key Sub-Packages:

**`world/worldgen/`** - World Generation
- Island generation
- Ore generation
- Tree generation

**`world/block/`** - World Blocks
- Ores
- Special blocks
- Trees

**`world/TinkerWorld.java`** - World Module
- Registers world content

**`world/TinkerStructures.java`** - Structures Module
- Registers structures

## Integration Modules

### 10. Plugin Module (`plugin/`)

Third-party mod integrations.

#### Integrations:

**`plugin/jei/`** - Just Enough Items (JEI)
- Recipe displays
- Category registrations

**`plugin/jsonthings/`** - JSON Things
- Dynamic content integration

**`plugin/craftingtweaks/`** - Crafting Tweaks
- Crafting enhancement integration

**`plugin/DietPlugin.java`** - Diet Mod
**`plugin/ImmersiveEngineeringPlugin.java`** - Immersive Engineering
**`plugin/DummmmmmyPlugin.java`** - Dummmmmmy (target dummy)

## Module Interaction Map

```
                    TConstruct (Main)
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    Library           Common             Shared
        │                 │                 │
        └────────┬────────┴────────┬────────┘
                 │                 │
    ┌────────────┼────────┬────────┼────────┐
    │            │        │        │        │
  Tools      Tables  Smeltery  Gadgets  Fluids
    │            │        │        │        │
    └────────────┴────────┴────────┴────────┘
                          │
                       Plugin
```

## Module Dependencies

### Dependency Rules

1. **Library** has no dependencies on other modules
2. **Common** depends only on Library
3. **Shared** depends on Library and Common
4. **Feature modules** depend on Library, Common, and Shared
5. **Plugin** depends on all modules as needed

### Import Rules

- Library exports public APIs
- Implementation modules import from Library
- Cross-module imports should be minimal
- Plugin imports are flexible for integration

## File Organization Within Modules

Standard structure:
```
module/
├── ModuleName.java (Registration class)
├── block/ (Blocks)
├── item/ (Items)
├── block/entity/ (Block entities)
├── menu/ (Containers/Menus)
├── recipe/ (Recipes)
├── client/ (Client-only code)
│   ├── screen/ (UI screens)
│   ├── render/ (Rendering)
│   └── model/ (Models)
├── data/ (Data generation)
├── network/ (Network packets)
└── logic/ (Business logic)
```

## Module Lifecycle

1. **Construction**: Module class instantiated
2. **Registration**: Register content to Forge
3. **Setup**: Common and client setup
4. **Data Load**: Load JSON data
5. **Runtime**: Active gameplay

## Navigation

- [Previous: Overview ←](./01-Overview.md)
- [Next: Initialization Flow →](./03-Initialization-Flow.md)
- [Back to Architecture Index ↑](./README.md)
