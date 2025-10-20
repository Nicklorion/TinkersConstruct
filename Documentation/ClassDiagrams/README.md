# Class Diagrams - Forge & Force

This directory contains comprehensive UML class diagrams for all major systems in the Forge & Force crafting game (derived from Tinkers' Construct).

## Overview

The diagrams are organized by system and show:
- Class hierarchies and relationships
- Key properties and methods
- Interfaces and abstract classes
- Data flow between components
- Design patterns used

## Diagram Index

### Core Systems

1. **[Material System](./01-Material-System.md)** - Material definitions, stats, traits, and registry
2. **[Tool System](./02-Tool-System.md)** - Tool definitions, parts, assembly, and NBT data
3. **[Modifier System](./03-Modifier-System.md)** - Modifier application, effects, and hooks
4. **[Smeltery System](./04-Smeltery-System.md)** - Forge/smeltery multiblock, recipes, and processing
5. **[Recipe System](./05-Recipe-System.md)** - Recipe types, serialization, and processing
6. **[Data Loading System](./06-Data-Loading-System.md)** - JSON loading, synchronization, and validation

### Supporting Systems

7. **[Registry System](./07-Registry-System.md)** - Content registration and lookup
8. **[Network System](./08-Network-System.md)** - Client-server communication
9. **[UI System](./09-UI-System.md)** - Crafting stations and menus
10. **[Temperature & Force System](./10-Temperature-Force-System.md)** - Core progression mechanics

### Integration

11. **[Overall Architecture](./11-Overall-Architecture.md)** - High-level system integration
12. **[C# Class Mappings](./12-CSharp-Mappings.md)** - Java to C# translations

## How to Read the Diagrams

### UML Notation Used

**Class Structure:**
```
┌──────────────────────┐
│   ClassName          │ ← Class name
├──────────────────────┤
│ - privateField       │ ← Fields (- private, + public, # protected)
│ + publicField        │
├──────────────────────┤
│ + publicMethod()     │ ← Methods
│ - privateMethod()    │
└──────────────────────┘
```

**Relationships:**
- `───▷` : Inheritance (extends)
- `---▷` : Implementation (implements)
- `────>` : Association (uses/has)
- `◆────>` : Composition (owns/contains)
- `◇────>` : Aggregation (has reference to)

**Stereotypes:**
- `<<interface>>` : Interface
- `<<abstract>>` : Abstract class
- `<<enum>>` : Enumeration
- `<<singleton>>` : Singleton pattern
- `<<static>>` : Static utility class

### Example Diagram

```
        <<interface>>
      ┌───────────────┐
      │   IMaterial   │
      ├───────────────┤
      │ + getId()     │
      │ + getTier()   │
      └───────────────┘
              △
              │ implements
              │
      ┌───────────────┐
      │   Material    │
      ├───────────────┤
      │ - id: string  │
      │ - tier: int   │
      ├───────────────┤
      │ + getId()     │
      │ + getTier()   │
      └───────────────┘
```

## Design Patterns Used

The system uses several well-established design patterns:

### 1. **Registry Pattern**
- Centralized storage and lookup of content
- Used for: Materials, Tools, Modifiers, Recipes
- Example: `MaterialRegistry`, `ToolDefinitionRegistry`

### 2. **Factory Pattern**
- Creates complex objects with multiple steps
- Used for: Tool assembly, Recipe creation
- Example: `ToolStackFactory`, `RecipeFactory`

### 3. **Builder Pattern**
- Constructs objects step-by-step
- Used for: Tool stats, Modifier application
- Example: `ToolStatsBuilder`, `ModifierBuilder`

### 4. **Observer Pattern** (Event System)
- Notifies listeners of changes
- Used for: Tool damage, Modifier effects, Crafting events
- Example: `ToolDamageEvent`, `ModifierAppliedEvent`

### 5. **Strategy Pattern**
- Swappable algorithms
- Used for: Modifier effects, Recipe matching
- Example: `ModifierHook`, `RecipeMatcher`

### 6. **Composite Pattern**
- Tree structures with uniform interface
- Used for: Tool parts, Multiblock structures
- Example: `ToolPart`, `MultiblockComponent`

### 7. **Singleton Pattern**
- Single instance of critical systems
- Used for: Registries, Managers
- Example: `MaterialRegistry.getInstance()`

### 8. **Data Transfer Object (DTO)**
- Carry data between processes
- Used for: Network packets, NBT data
- Example: `MaterialData`, `ToolData`

## Conversion to C#

All diagrams include notes on C# conversion:
- Property vs. field usage
- Interface naming conventions (IInterface)
- Event system differences
- Unity-specific patterns (ScriptableObject, MonoBehaviour)

See [C# Class Mappings](./12-CSharp-Mappings.md) for detailed conversion guide.

## Usage Notes

### For Understanding the System
1. Start with [Overall Architecture](./11-Overall-Architecture.md)
2. Deep-dive into specific systems as needed
3. Reference [C# Mappings](./12-CSharp-Mappings.md) for implementation

### For Implementation
1. Implement interfaces first
2. Create base classes and abstract classes
3. Implement concrete classes
4. Add registries and managers
5. Connect systems with events and data flow

### For Testing
Each class diagram includes:
- Key behaviors to test
- Edge cases to consider
- Integration points to validate

---

**Related Documentation:**
- [Game Design Document](../GameDesign/GDD-01-Core-Concept.md)
- [Technical Design Document](../TechnicalDesign/TDD-01-Overview.md)
- [C# Implementation Guide](../CSharp-Unity-Port/01-Porting-Overview.md)
