# Technical Documentation

This section provides deep technical documentation of all major systems in Tinkers' Construct. Each document explains how a system works internally, its data structures, algorithms, and implementation details.

## Table of Contents

### Core Systems

1. [Materials System](./01-Materials-System.md)
   - Material definitions and IDs
   - Material stats (durability, mining speed, damage, etc.)
   - Material traits (special abilities)
   - Material variants and rendering
   - JSON data format
   - Registry and loading

2. [Tools System](./02-Tools-System.md)
   - Tool definitions and structure
   - Tool parts and assembly
   - NBT data storage
   - Tool stats calculation
   - Durability and damage
   - Tool rendering

3. [Modifiers System](./03-Modifiers-System.md)
   - Modifier types and categories
   - Modifier application and levels
   - Modifier hooks and effects
   - Slot management
   - Modifier recipes
   - Persistence and NBT

4. [Smeltery System](./04-Smeltery-System.md)
   - Multiblock structure
   - Structure validation
   - Fluid management
   - Melting recipes
   - Alloying recipes
   - Fuel consumption
   - Casting system

5. [Tables System](./05-Tables-System.md)
   - Crafting Station
   - Part Builder
   - Tinker Station
   - Modifier Worktable
   - Tinkers' Anvil
   - UI layout system
   - Recipe integration

### Data and Networking

6. [Data System](./06-Data-System.md)
   - JSON loading pipeline
   - Datapack integration
   - Resource reload handling
   - Validation and error handling
   - Data inheritance and overrides

7. [Network System](./07-Network-System.md)
   - Client-server architecture
   - Packet design and structure
   - Data synchronization
   - Bulk sync on join
   - Incremental updates
   - Packet optimization

8. [Recipe System](./08-Recipe-System.md)
   - Custom recipe types
   - Recipe serialization
   - Recipe matching
   - Result calculation
   - JEI integration

### Advanced Topics

9. [NBT Data Structures](./09-NBT-Structures.md)
   - Tool NBT format
   - Material NBT
   - Stats NBT
   - Modifier NBT
   - Persistent vs volatile data
   - Versioning and migration

10. [Event System](./10-Event-System.md)
    - Forge event integration
    - Custom Tinkers' events
    - Event priority and handling
    - Tool events
    - Material events
    - Modifier events

11. [Rendering System](./11-Rendering-System.md)
    - Dynamic tool models
    - Material textures
    - Modifier overlays
    - Fluid rendering
    - Particle effects
    - Client-side optimization

12. [Statistics and Calculations](./12-Stats-Calculations.md)
    - Base stat calculation
    - Material contribution
    - Modifier effects
    - Multipliers and bonuses
    - Conditional stats
    - Stat caching

## Documentation Purpose

Each technical document aims to answer:
- **How does it work?** - Internal mechanics and algorithms
- **What are the components?** - Classes, interfaces, and data structures
- **How is data stored?** - NBT and JSON formats
- **How does it integrate?** - Interactions with other systems
- **What are the edge cases?** - Special handling and validation

## Code Examples

Technical documentation includes:
- Code snippets showing key implementations
- Data format examples (JSON, NBT)
- Sequence diagrams for complex flows
- State diagrams where applicable
- Performance considerations

## Reading Guide

### For Implementation Understanding
1. Start with the system you're interested in
2. Follow cross-references to related systems
3. Reference Architecture docs for high-level context
4. Check code examples for concrete understanding

### For Debugging
1. Identify which system is involved
2. Understand the data flow
3. Check validation and error handling
4. Review event hooks that might interfere

### For Extension
1. Understand the base implementation
2. Identify extension points
3. Review similar implementations
4. Check API documentation for interfaces

## Relationship to Other Documentation

- **Architecture**: High-level design → Technical: Implementation details
- **Technical**: How it works → API: How to use it
- **Technical**: Java implementation → C# Port: C# implementation

## Navigation

- [Start with Materials System →](./01-Materials-System.md)
- [Back to Main Documentation ↑](../README.md)
