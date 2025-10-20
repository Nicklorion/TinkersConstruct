# Architecture Documentation

This section provides a comprehensive overview of Tinkers' Construct's architecture, design patterns, and system organization.

## Table of Contents

1. [Overview](./01-Overview.md) - High-level architecture and design philosophy
2. [Module Structure](./02-Module-Structure.md) - Package organization and module responsibilities
3. [Initialization Flow](./03-Initialization-Flow.md) - Startup sequence and registration
4. [Core Systems](./04-Core-Systems.md) - Fundamental systems and their interactions
5. [Data Flow](./05-Data-Flow.md) - How data moves through the system
6. [Event Architecture](./06-Event-Architecture.md) - Event-driven design patterns
7. [Client-Server Architecture](./07-Client-Server.md) - Networking and synchronization
8. [Design Patterns](./08-Design-Patterns.md) - Common patterns used throughout

## Architecture Overview

Tinkers' Construct follows a modular, data-driven architecture built on top of Minecraft Forge. The system is designed with these key principles:

### Core Principles

1. **Modularity**: Features are organized into self-contained modules
2. **Data-Driven**: Game content is defined through JSON data files
3. **Extensibility**: API-first design allows easy extension
4. **Performance**: Efficient caching and lazy loading strategies
5. **Maintainability**: Clear separation of concerns

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     TConstruct Main Mod                     │
│                    (Initialization & Core)                   │
└────────────┬────────────────────────────────────────────────┘
             │
    ┌────────┴────────┐
    │                 │
┌───▼────┐      ┌────▼─────┐
│ Forge  │      │  Mantle  │
│  API   │      │  Library │
└────────┘      └──────────┘

Module Organization:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Library    │  │    Common    │  │    Shared    │
│ (Core APIs)  │  │  (Base Util) │  │ (Game Items) │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
       └────────┬────────┴─────────┬────────┘
                │                  │
       ┌────────▼────────┐  ┌─────▼──────┐
       │   Game Modules  │  │   World    │
       │  - Tools        │  │ Generation │
       │  - Tables       │  └────────────┘
       │  - Smeltery     │
       │  - Gadgets      │
       │  - Fluids       │
       └─────────────────┘
```

## Module Hierarchy

### Foundation Layer
- **Library**: Core APIs, interfaces, and abstract implementations
- **Common**: Shared utilities, base classes, and common functionality
- **Shared**: Cross-module game elements (blocks, items, effects)

### Feature Layer
- **Tools**: Tool system, modifiers, and tool-related functionality
- **Tables**: Crafting stations and tool assembly UI
- **Smeltery**: Multiblock structure and ore processing
- **Gadgets**: Utility items and blocks
- **Fluids**: Custom fluid handling and rendering
- **World**: World generation and structures

### Integration Layer
- **Plugin**: Third-party mod integration
- **Client**: Client-side rendering and UI
- **Network**: Client-server communication

## Data Loading Pipeline

```
Server Startup
     │
     ├─> Load Material Definitions (JSON)
     ├─> Load Material Stats (JSON)
     ├─> Load Material Traits (JSON)
     ├─> Load Tool Definitions (JSON)
     ├─> Load Station Layouts (JSON)
     ├─> Load Recipes (JSON)
     │
     ├─> Build Material Registry
     ├─> Build Tool Registry
     ├─> Build Modifier Registry
     │
     └─> Sync to Clients (Network Packets)
```

## Key Architectural Patterns

### 1. Registry Pattern
Most game content is registered through centralized registries:
- MaterialRegistry
- ToolDefinitionLoader
- ModifierManager
- RecipeManager (Minecraft)

### 2. Data Manager Pattern
JSON data is loaded through specialized managers:
- MaterialManager
- MaterialStatsManager
- MaterialTraitsManager
- ToolDefinitionLoader
- StationSlotLayoutLoader

### 3. NBT Data Pattern
Tool data is stored in NBT (Named Binary Tag) format:
- MaterialNBT: Material composition
- StatsNBT: Tool statistics
- ModifierNBT: Applied modifiers
- ToolStack: Complete tool data wrapper

### 4. Event-Driven Architecture
System interactions use Forge's event bus:
- MaterialsLoadedEvent
- ToolBuildEvent
- ModifierApplicationEvent
- Custom Tinkers' events

### 5. Packet-Based Synchronization
Client-server sync uses custom packets:
- UpdateMaterialsPacket
- UpdateMaterialStatsPacket
- UpdateMaterialTraitsPacket
- ToolDefinition sync packets

## System Boundaries

### Library vs. Implementation
- **Library**: Defines interfaces, base classes, and contracts
- **Implementation**: Concrete implementations in game modules

### Client vs. Server
- **Server**: Authoritative data, game logic, validation
- **Client**: Rendering, UI, client-side prediction
- **Shared**: Common code that runs on both sides

### Data vs. Code
- **Data**: JSON-defined content (materials, tools, recipes)
- **Code**: Java classes for behavior and logic
- **Hybrid**: Data references code through resource locations

## Dependencies

### External Dependencies
- **Minecraft Forge**: Core modding framework
- **Mantle**: Shared library by SlimeKnights team
- **Lombok**: Code generation for boilerplate

### Optional Dependencies (Plugins)
- JEI (Just Enough Items)
- CraftTweaker
- Immersive Engineering
- JSON Things
- Diet
- And others...

## Performance Considerations

### Caching Strategies
- Material data cached after initial load
- Tool stats computed and cached in NBT
- Modifier effects cached on tool stacks

### Lazy Loading
- Tool definitions loaded only when needed
- Client receives only necessary data
- Textures and models loaded on-demand

### Network Optimization
- Batch updates during datapack sync
- Incremental updates for changes
- Client-side caching to reduce packets

## Extension Points

The architecture provides several extension points for addons:

1. **Custom Materials**: Add new materials via datapacks
2. **Custom Tools**: Define new tool types
3. **Custom Modifiers**: Implement new modifier effects
4. **Custom Recipes**: Add new recipe types
5. **Event Handlers**: Hook into game events
6. **Stat Types**: Define new material stat types
7. **Trait Types**: Create new material traits

## Navigation

- [Next: Module Structure →](./02-Module-Structure.md)
- [Back to Main Documentation ↑](../README.md)
