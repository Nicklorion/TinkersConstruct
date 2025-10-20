# API Documentation

This section provides comprehensive API documentation for developers creating addons, integrations, or ports of Tinkers' Construct.

## Table of Contents

### Getting Started

1. [API Overview](./01-API-Overview.md)
   - Public API scope
   - Versioning and stability
   - When to use which API
   - Common patterns

2. [Integration Guide](./02-Integration-Guide.md)
   - Setting up your addon
   - Dependencies and build setup
   - Best practices
   - Common pitfalls

### Core APIs

3. [Materials API](./03-Materials-API.md)
   - Registering materials (code vs data)
   - Creating custom stat types
   - Defining material traits
   - Material rendering
   - Material predicates

4. [Tools API](./04-Tools-API.md)
   - Creating tool definitions
   - Custom tool items
   - Tool part registration
   - Tool stat types
   - Tool capabilities
   - Tool helpers

5. [Modifiers API](./05-Modifiers-API.md)
   - Creating modifiers
   - Modifier hooks
   - Modifier recipes
   - Level-based effects
   - Conditional modifiers
   - Modifier compatibility

6. [Recipe API](./06-Recipe-API.md)
   - Custom recipe types
   - Recipe builders
   - Recipe conditions
   - Recipe serialization
   - JEI integration

### Data APIs

7. [Data Loading API](./07-Data-Loading-API.md)
   - Custom data loaders
   - JSON parsers
   - Data validation
   - Datapack integration
   - Resource reloading

8. [NBT Data API](./08-NBT-Data-API.md)
   - Reading tool data
   - Modifying tool data
   - Custom persistent data
   - Data migration
   - Validation

### Event APIs

9. [Event Hooks](./09-Event-Hooks.md)
   - Tool events
   - Material events
   - Modifier events
   - Recipe events
   - Custom event creation

10. [Modifier Hooks](./10-Modifier-Hooks.md)
    - Hook types and usage
    - Implementing hook interfaces
    - Hook priority
    - Conditional hooks
    - Common hook patterns

### Rendering APIs

11. [Rendering API](./11-Rendering-API.md)
    - Material rendering info
    - Tool model customization
    - Modifier overlays
    - Custom sprites
    - Particle effects

### Utility APIs

12. [Utility Classes](./12-Utility-Classes.md)
    - Tool helpers
    - NBT utilities
    - JSON utilities
    - Registry helpers
    - Common predicates

## API Interface Specifications

### Key Interfaces for Addons

#### Materials
```java
// Primary interfaces for material integration
interface IMaterial
interface IMaterialStats
interface MaterialVariant
interface MaterialRenderInfo
```

#### Tools
```java
// Primary interfaces for tool integration
interface IModifiable
interface IToolStackView
interface ToolDefinition
interface IToolStat
```

#### Modifiers
```java
// Primary interfaces for modifier integration
class Modifier (base class)
interface ModifierHook<T>
interface ModifierEntry
```

#### Data
```java
// Primary interfaces for data loading
interface IJsonPredicate
interface GenericLoader<T>
interface IGenericLoader<T>
```

## Addon Development Guide

### Minimal Addon Structure

```
your-addon/
├── src/main/
│   ├── java/your/package/
│   │   ├── YourAddon.java (main mod class)
│   │   ├── materials/
│   │   │   └── CustomMaterials.java
│   │   ├── modifiers/
│   │   │   └── CustomModifiers.java
│   │   └── tools/
│   │       └── CustomTools.java
│   └── resources/
│       ├── META-INF/mods.toml
│       └── data/yourmod/
│           ├── tinkering/
│           │   ├── materials/
│           │   │   ├── definition/
│           │   │   ├── stats/
│           │   │   └── traits/
│           │   └── tool_definitions/
│           └── recipes/
└── build.gradle
```

### Integration Patterns

#### 1. Data-Only Addon (Recommended)
Add content purely through datapacks:
- Materials via JSON
- Tool definitions via JSON
- Recipes via JSON
- No code required for basic additions

#### 2. Code + Data Addon
Combine code and data:
- Custom modifier logic in code
- Material definitions in JSON
- Register modifiers programmatically
- Define behavior through data

#### 3. Full Integration Addon
Deep integration:
- Custom stat types
- Custom tool types
- Custom recipe types
- Custom UI elements
- Event handlers

## Mod API Design Pattern

### Standardized Mod Loading

For Unity/C# or other platforms, the API follows this pattern:

```csharp
// Example structure for standardized mod loading
public interface IModContent
{
    string ModId { get; }
    string ModName { get; }
    Version ModVersion { get; }
    
    void RegisterMaterials(MaterialRegistry registry);
    void RegisterTools(ToolRegistry registry);
    void RegisterModifiers(ModifierRegistry registry);
    void RegisterRecipes(RecipeRegistry registry);
}

public interface IModLoader
{
    void LoadMod(IModContent mod);
    void UnloadMod(string modId);
    bool IsModLoaded(string modId);
    IModContent GetMod(string modId);
}
```

This pattern enables:
- Dynamic mod loading/unloading
- Dependency resolution
- Version checking
- Content isolation

## API Stability

### Public API
Classes and interfaces in:
- `slimeknights.tconstruct.library.` (with exceptions)
- Clearly documented as `@API`

Stability: **Stable** - Breaking changes only in major versions

### Internal API
Classes marked:
- `@VisibleForTesting`
- In `.internal.` packages
- Undocumented classes

Stability: **Unstable** - May change at any time

## Example Implementations

Each API section includes:
- Minimal working example
- Complete implementation example
- Common patterns
- Error handling
- Best practices

## API Versioning

```java
// API Version in TConstruct
public static final String API_VERSION = "1.0.0";
```

### Version Format: Major.Minor.Patch
- **Major**: Breaking API changes
- **Minor**: New features, backward compatible
- **Patch**: Bug fixes, no API changes

## Third-Party Integration

### Supported Integration Points

1. **JEI (Just Enough Items)**
   - Recipe categories
   - Recipe displays
   - Ingredient matching

2. **CraftTweaker**
   - Script integration
   - Recipe manipulation
   - Material modification

3. **Other Mods**
   - Material registration
   - Tool type extension
   - Modifier addition

## Community Addons

Example addons demonstrating API usage:
- Materials addon examples
- Tool addon examples
- Modifier addon examples

## Troubleshooting

Common issues and solutions:
- ClassNotFoundException → Check dependencies
- Missing materials → Verify datapack loading
- Sync errors → Check packet registration
- Rendering issues → Verify client-side code

## API Support

- GitHub Issues: Technical questions
- Discord: Quick questions
- Wiki: Documentation
- Source Code: Ultimate reference

## Navigation

- [Start with API Overview →](./01-API-Overview.md)
- [Back to Main Documentation ↑](../README.md)
