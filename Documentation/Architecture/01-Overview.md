# Overview - Tinkers' Construct Architecture

## Introduction

Tinkers' Construct is a comprehensive tool crafting and modification system for Minecraft. This document provides a high-level architectural overview of the entire system, its design philosophy, and core concepts.

## Design Philosophy

### 1. Player Agency and Customization
The entire system is built around giving players complete control over their tools. Every design decision supports:
- Material selection with meaningful tradeoffs
- Progressive tool enhancement through modifiers
- Visual customization through material rendering
- Strategic choices in tool assembly

### 2. Data-Driven Design
Rather than hardcoding game content, Tinkers' Construct uses JSON datapacks:
- **Materials**: Defined in JSON with stats and traits
- **Tools**: Tool definitions specify parts and assembly rules
- **Recipes**: All crafting uses JSON recipe definitions
- **Modifiers**: Effects and requirements defined in data
- **Station Layouts**: UI slot arrangements in JSON

This enables:
- Easy content addition without code changes
- Datapack support for custom content
- Clean separation of data and logic
- Better testability

### 3. Component-Based Tool Assembly
Tools are not single items but assemblies of components:
```
Tool = Definition + Materials + Modifiers
```

Each component contributes:
- **Tool Definition**: Structure (head, handle, extra)
- **Materials**: Stats (durability, mining speed, damage)
- **Modifiers**: Special abilities and enhancements

### 4. Progressive Enhancement
Tools start basic and improve over time:
1. **Craft**: Assemble tool from parts at crafting station
2. **Use**: Tool gains experience and durability
3. **Repair**: Restore durability at crafting station
4. **Modify**: Add modifiers at modifier worktable
5. **Upgrade**: Apply material upgrades and swaps

## System Architecture

### Core Systems Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    GAME CONTENT LAYER                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Materials│  │   Tools  │  │ Modifiers│  │  Recipes │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
└───────┼─────────────┼─────────────┼─────────────┼──────────┘
        │             │             │             │
┌───────▼─────────────▼─────────────▼─────────────▼──────────┐
│                   DATA MANAGEMENT LAYER                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Material   │  │     Tool     │  │   Modifier   │      │
│  │   Registry   │  │ DefinitionLdr│  │   Manager    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
└─────────┼──────────────────┼──────────────────┼─────────────┘
          │                  │                  │
┌─────────▼──────────────────▼──────────────────▼─────────────┐
│                    RUNTIME LAYER                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ToolStack │  │   NBT    │  │  Events  │  │ Network  │   │
│  │  (Live)  │  │(Persist) │  │ (Hooks)  │  │  (Sync)  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└──────────────────────────────────────────────────────────────┘
          │                  │                  │
┌─────────▼──────────────────▼──────────────────▼─────────────┐
│                   MINECRAFT FORGE LAYER                      │
│  Items | Blocks | Entities | Recipes | Commands | etc.      │
└──────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

#### 1. Game Content Layer
- Defines what exists in the game
- Materials: wood, iron, diamond, etc.
- Tools: pickaxe, sword, bow, etc.
- Modifiers: sharpness, knockback, silk touch, etc.
- Recipes: how to craft everything

#### 2. Data Management Layer
- Loads JSON data from datapacks
- Validates and parses data
- Builds runtime registries
- Syncs data to clients
- Handles datapack reloads

#### 3. Runtime Layer
- Active game logic
- Tool instances with live data
- Event handling and hooks
- Client-server communication
- State persistence via NBT

#### 4. Minecraft Forge Layer
- Integration with Minecraft
- Forge API usage
- Event bus connections
- Registry management

## Core Concepts

### Materials

Materials are the building blocks of tools. Each material has:
- **Identity**: Unique ID and display name
- **Stats**: Numeric properties per stat type (durability, mining speed, etc.)
- **Traits**: Special abilities granted to tools
- **Rendering**: Color and texture information

Materials are composable - a tool can use different materials for different parts.

### Tools

Tools are defined by:
- **Definition**: What parts are needed and how they combine
- **Parts**: Individual components (head, handle, binding, etc.)
- **Assembly**: Which materials were used for each part
- **State**: Current durability, modifiers, etc.

Tools are instances, not types. Each pickaxe can be unique.

### Modifiers

Modifiers enhance tools with special abilities:
- **Levels**: Most modifiers have multiple levels
- **Requirements**: May require specific materials or other modifiers
- **Effects**: Alter tool behavior (mining speed, damage, etc.)
- **Slots**: Limited by modifier slots on the tool
- **Persistence**: Stored in tool NBT data

### NBT Data Structure

Tools store all their data in NBT (Named Binary Tag):
```
ToolStack NBT:
├── Materials: [head_material, handle_material, ...]
├── Stats: {durability, mining_level, mining_speed, ...}
├── Modifiers: [{id, level}, ...]
├── Persistent Data: {custom_name, ...}
└── Volatile Data: {current_damage, ...}
```

## Modular Architecture

### Module Organization

The codebase is organized into modules, each with specific responsibilities:

1. **Library Module** (`slimeknights.tconstruct.library`)
   - Core APIs and interfaces
   - Abstract base classes
   - Common utilities
   - Data structures

2. **Tools Module** (`slimeknights.tconstruct.tools`)
   - Tool implementations
   - Modifier implementations
   - Tool-specific logic
   - Tool statistics

3. **Tables Module** (`slimeknights.tconstruct.tables`)
   - Crafting stations (Part Builder, Tinker Station, etc.)
   - UI and menus
   - Station-specific recipes
   - Block entities

4. **Smeltery Module** (`slimeknights.tconstruct.smeltery`)
   - Multiblock structure
   - Melting and alloy recipes
   - Casting recipes
   - Smeltery controller logic

5. **Shared Module** (`slimeknights.tconstruct.shared`)
   - Common blocks and items
   - Shared effects and attributes
   - Cross-module functionality

6. **Common Module** (`slimeknights.tconstruct.common`)
   - Base utilities
   - Configuration
   - Data providers
   - Network infrastructure

### Module Dependencies

```
     Library (Foundation)
         ↑
         │
    ┌────┴────┐
    │         │
  Common   Shared
    │         │
    └────┬────┘
         │
    ┌────┴────┬────────┬──────────┐
    │         │        │          │
  Tools   Tables  Smeltery    Gadgets
```

## Initialization Flow

### Startup Sequence

1. **Mod Construction** (`TConstruct` constructor)
   - Initialize configuration
   - Register modules
   - Setup event listeners
   - Initialize registries

2. **Registration Phase** (Forge events)
   - Register blocks
   - Register items
   - Register block entities
   - Register recipes
   - Register modifiers

3. **Common Setup** (`FMLCommonSetupEvent`)
   - Initialize network
   - Register capabilities
   - Setup integrations
   - Configure modules

4. **Data Loading** (Server start)
   - Load material definitions
   - Load material stats
   - Load material traits
   - Load tool definitions
   - Load recipes

5. **Client Sync** (Player join)
   - Sync materials to client
   - Sync stats to client
   - Sync traits to client
   - Sync tool definitions

## Extension Architecture

### Plugin System

Tinkers' Construct supports plugins/addons through:

1. **Public API**: Well-defined interfaces in library module
2. **Events**: Forge event bus for hooks
3. **Registries**: Add custom content to registries
4. **Datapacks**: JSON-based content addition
5. **Integration API**: Dedicated integration points

### Integration Points

Addons can:
- Register new materials (code or data)
- Create custom tool types
- Add new modifiers
- Define new stat types
- Add new traits
- Integrate with other mods

## Data Flow Patterns

### Read Path (Tool Usage)
```
Player Uses Tool
    ↓
Get ToolStack from ItemStack
    ↓
Read NBT Data
    ↓
Query Material Registry
    ↓
Calculate Stats
    ↓
Apply Modifier Effects
    ↓
Execute Tool Action
```

### Write Path (Tool Modification)
```
Player Modifies Tool
    ↓
Validate Modification
    ↓
Calculate New Stats
    ↓
Update NBT Data
    ↓
Trigger Events
    ↓
Sync to Client (if needed)
```

### Data Sync Path
```
Server Loads Data (JSON)
    ↓
Build Registries
    ↓
Create Sync Packets
    ↓
Send to Clients
    ↓
Client Updates Registries
```

## Performance Design

### Optimization Strategies

1. **Caching**
   - Material data cached after load
   - Tool stats cached in NBT
   - Modifier lookups cached

2. **Lazy Loading**
   - Data loaded only when needed
   - Client receives minimal necessary data
   - Textures loaded on-demand

3. **Batch Processing**
   - Data sent in batches during sync
   - Recipe lookups optimized
   - Event handling batched

4. **Efficient Storage**
   - NBT data compressed
   - Only changed data synced
   - Smart delta updates

## Error Handling

### Validation Layers

1. **Data Validation**: JSON schemas and parsing
2. **Runtime Validation**: Null checks and error handling
3. **User Input Validation**: Recipe and crafting validation
4. **Network Validation**: Packet integrity checks

### Fallback Strategies

- Missing materials → Use default material
- Invalid tools → Clear and rebuild
- Network errors → Request re-sync
- Data errors → Log and skip

## Navigation

- [Next: Module Structure →](./02-Module-Structure.md)
- [Back to Architecture Index ↑](./README.md)
