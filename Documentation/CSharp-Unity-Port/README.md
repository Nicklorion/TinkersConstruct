# C# Unity Port Guide

This comprehensive guide is specifically designed for developers porting Tinkers' Construct to Unity using C#. It provides detailed implementation strategies, API design patterns, and Unity-specific considerations.

## Table of Contents

### Getting Started

1. [Porting Overview](./01-Porting-Overview.md)
   - Scope and goals
   - Java to C# translation strategy
   - Unity integration approach
   - Development roadmap
   - Resource requirements

2. [Unity Architecture](./02-Unity-Architecture.md)
   - Unity vs Forge comparison
   - Component-based design in Unity
   - ScriptableObject usage
   - Prefab system integration
   - Scene management

3. [C# API Design](./03-CSharp-API-Design.md)
   - Namespace organization
   - Interface design
   - Abstract base classes
   - Event system (C# events vs Forge events)
   - Generic programming patterns

### Core System Ports

4. [Materials System Port](./04-Materials-System-Port.md)
   - Material ScriptableObjects
   - Material stat implementation
   - Material trait system
   - Material registry in Unity
   - Material rendering with shaders

5. [Tools System Port](./05-Tools-System-Port.md)
   - Tool item representation
   - Tool assembly in Unity
   - Tool data serialization
   - Tool stats calculation
   - Tool prefabs and instances

6. [Modifiers System Port](./06-Modifiers-System-Port.md)
   - Modifier ScriptableObjects
   - Modifier application pipeline
   - Modifier hooks as delegates/events
   - Modifier UI in Unity
   - Modifier persistence

7. [UI System Port](./07-UI-System-Port.md)
   - Unity UI vs IMGUI
   - Crafting station UI
   - Inventory integration
   - Tool assembly interface
   - Modifier application interface
   - Drag-and-drop functionality

### Data Management

8. [Data System Port](./08-Data-System-Port.md)
   - JSON loading in Unity
   - ScriptableObject generation
   - Addressables system integration
   - Resource streaming
   - Hot reloading during development

9. [Save System](./09-Save-System.md)
   - Tool data serialization
   - Player inventory persistence
   - World state saving
   - Save file format
   - Migration and versioning

10. [Networking (Multiplayer)](./10-Networking-Port.md)
    - Unity Netcode vs Forge networking
    - Tool data synchronization
    - Material registry sync
    - RPC design
    - Authoritative server model

### Unity-Specific Implementation

11. [Rendering in Unity](./11-Rendering-Unity.md)
    - Material system (Unity Materials vs TC Materials)
    - Shader-based material colors
    - Tool model generation
    - Mesh combination
    - Texture atlasing
    - Particle systems

12. [Performance Optimization](./12-Performance-Unity.md)
    - Object pooling
    - Lazy loading strategies
    - Coroutine usage
    - Memory management
    - GC optimization
    - Profiling tools

13. [Physics Integration](./13-Physics-Unity.md)
    - Tool durability from physics
    - Mining/harvesting mechanics
    - Projectile simulation
    - Collision detection
    - Ragdoll integration

### Modding API for Unity

14. [Unity Modding API](./14-Unity-Modding-API.md)
    - Mod structure and organization
    - Mod loading system
    - Assembly loading
    - Mod isolation and sandboxing
    - Dependency resolution
    - Version compatibility

15. [Mod Content Registration](./15-Mod-Registration.md)
    - Material registration API
    - Tool registration API
    - Modifier registration API
    - Recipe registration API
    - Asset bundle integration

16. [Mod Events and Hooks](./16-Mod-Events-Hooks.md)
    - C# event system
    - Custom event bus
    - Hook priority system
    - Async event handling
    - Event debugging

### Data Formats

17. [JSON Data Formats](./17-JSON-Formats.md)
    - Material definition JSON
    - Tool definition JSON
    - Modifier definition JSON
    - Recipe JSON
    - Schema definitions
    - Validation

18. [Asset Bundle Format](./18-Asset-Bundles.md)
    - Bundle structure
    - Texture packing
    - Model formats
    - Audio clips
    - Prefab bundling
    - Version management

### Advanced Topics

19. [Shader Programming](./19-Shader-Programming.md)
    - Material color shaders
    - Tool rendering shaders
    - Particle effect shaders
    - Outline shaders
    - Custom lighting

20. [Editor Tools](./20-Editor-Tools.md)
    - Material editor window
    - Tool preview editor
    - Modifier testing tools
    - JSON validator
    - Asset import pipeline

## Java to C# Translation Guide

### Language Feature Mapping

| Java Feature | C# Equivalent | Notes |
|--------------|---------------|-------|
| `interface` | `interface` | Similar, C# supports default implementations |
| `abstract class` | `abstract class` | Nearly identical |
| `@Override` | `override` | Keyword instead of annotation |
| `@Nullable` | `?` (nullable types) | C# has built-in nullable reference types |
| `Optional<T>` | `T?` or nullable | C# nullables are cleaner |
| `Stream` API | LINQ | C# LINQ is more powerful |
| `HashMap<K,V>` | `Dictionary<K,V>` | Similar performance |
| `List<T>` | `List<T>` | Nearly identical API |
| Generics | Generics | C# generics are reified (better) |
| Lambda | Lambda | Similar syntax |
| `static` initialization | `static` constructor | Different syntax, same concept |
| Package | Namespace | Different keyword, same concept |
| `enum` | `enum` | C# enums are integral types |
| Annotations | Attributes | Different syntax: `[Attribute]` |
| Try-with-resources | `using` statement | Automatic disposal |

### Common Patterns Translation

#### Singleton Pattern
```java
// Java
public class Registry {
    private static Registry INSTANCE;
    public static Registry getInstance() {
        return INSTANCE;
    }
}
```

```csharp
// C# - Modern approach
public class Registry {
    private static Registry _instance;
    public static Registry Instance => _instance ??= new Registry();
}

// Or Unity approach with ScriptableObject
public class Registry : ScriptableObject {
    private static Registry _instance;
    public static Registry Instance {
        get {
            if (_instance == null) {
                _instance = Resources.Load<Registry>("Registry");
            }
            return _instance;
        }
    }
}
```

#### Builder Pattern
```java
// Java
public class MaterialBuilder {
    private MaterialId id;
    
    public MaterialBuilder id(MaterialId id) {
        this.id = id;
        return this;
    }
    
    public Material build() {
        return new Material(id);
    }
}
```

```csharp
// C#
public class MaterialBuilder {
    private MaterialId _id;
    
    public MaterialBuilder WithId(MaterialId id) {
        _id = id;
        return this;
    }
    
    public Material Build() => new Material(_id);
}
```

## Unity-Specific Considerations

### 1. ScriptableObjects vs Static Registries

**Java/Forge Approach:**
```java
public static final MaterialRegistry INSTANCE = new MaterialRegistry();
```

**Unity Approach:**
```csharp
// Create as asset in project
[CreateAssetMenu(fileName = "MaterialRegistry", menuName = "TConstruct/Material Registry")]
public class MaterialRegistry : ScriptableObject {
    [SerializeField] private List<Material> materials = new List<Material>();
    
    // Access via Resources or singleton pattern
}
```

**Benefits:**
- Visible in Inspector
- Serialized with project
- Easy to modify in editor
- No initialization order issues

### 2. Prefabs vs Procedural Generation

**Java:** Tools generated entirely in code

**Unity:** Hybrid approach
- Base tool prefab
- Runtime material application
- Component-based assembly

### 3. Component Architecture

Unity's GameObject-Component model maps well to TC's modular design:

```csharp
// Tool as GameObject with components
public class Tool : MonoBehaviour {
    private ToolDefinition definition;
    private ToolMaterials materials;
    private ToolModifiers modifiers;
    private ToolStats stats;
}

// Or more Unity-like
public class ToolMaterialComponent : MonoBehaviour { }
public class ToolModifierComponent : MonoBehaviour { }
public class ToolStatsComponent : MonoBehaviour { }
```

### 4. Event System

**Forge Events:**
```java
@SubscribeEvent
public void onToolBuild(ToolBuildEvent event) {
    // Handle event
}
```

**Unity C# Events:**
```csharp
// Define event
public class ToolEvents {
    public static event Action<Tool> OnToolBuilt;
    
    public static void FireToolBuilt(Tool tool) {
        OnToolBuilt?.Invoke(tool);
    }
}

// Subscribe
void Start() {
    ToolEvents.OnToolBuilt += HandleToolBuilt;
}

void HandleToolBuilt(Tool tool) {
    // Handle event
}
```

### 5. Resource Loading

**Forge:** Datapacks loaded from files

**Unity Options:**
1. **Resources folder** (simple, but all loaded)
2. **Addressables** (complex, but streaming)
3. **Asset Bundles** (for mods)
4. **ScriptableObjects** (built-in assets)

```csharp
// Addressables example
using UnityEngine.AddressableAssets;

public class MaterialLoader {
    public async Task<Material> LoadMaterial(string id) {
        var handle = Addressables.LoadAssetAsync<MaterialData>(id);
        await handle.Task;
        return handle.Result.ToMaterial();
    }
}
```

## Modding API Architecture for Unity

### Mod Loading System

```csharp
public interface IMod {
    string ModId { get; }
    string ModName { get; }
    Version Version { get; }
    
    void OnLoad();
    void RegisterContent(ContentRegistry registry);
    void OnUnload();
}

public class ModLoader : MonoBehaviour {
    private Dictionary<string, IMod> loadedMods = new Dictionary<string, IMod>();
    
    public void LoadMod(string assemblyPath) {
        // Load assembly
        var assembly = Assembly.LoadFrom(assemblyPath);
        
        // Find mod class
        var modType = assembly.GetTypes()
            .FirstOrDefault(t => typeof(IMod).IsAssignableFrom(t));
            
        if (modType != null) {
            var mod = (IMod)Activator.CreateInstance(modType);
            mod.OnLoad();
            mod.RegisterContent(ContentRegistry.Instance);
            loadedMods[mod.ModId] = mod;
        }
    }
}
```

### Content Registration API

```csharp
public class ContentRegistry {
    public static ContentRegistry Instance { get; private set; }
    
    private MaterialRegistry materials;
    private ToolRegistry tools;
    private ModifierRegistry modifiers;
    
    public void RegisterMaterial(MaterialData material) {
        materials.Register(material);
    }
    
    public void RegisterTool(ToolDefinition tool) {
        tools.Register(tool);
    }
    
    public void RegisterModifier(Modifier modifier) {
        modifiers.Register(modifier);
    }
}
```

### Mod Isolation

```csharp
public class ModContext {
    public string ModId { get; }
    private List<string> registeredMaterials = new List<string>();
    private List<string> registeredTools = new List<string>();
    
    public void TrackMaterial(string id) {
        registeredMaterials.Add(id);
    }
    
    public void UnregisterAll() {
        foreach (var id in registeredMaterials) {
            MaterialRegistry.Instance.Unregister(id);
        }
        // ... etc
    }
}
```

## Development Workflow

### 1. Setup Phase
- Install Unity (LTS version recommended)
- Set up project structure
- Import dependencies
- Configure build settings

### 2. Foundation Phase
- Port core data structures
- Implement registries
- Create base ScriptableObjects
- Set up JSON loading

### 3. Feature Phase
- Port materials system
- Port tools system
- Port modifiers system
- Port UI system

### 4. Polish Phase
- Rendering and visuals
- Performance optimization
- Testing and debugging
- Documentation

### 5. Modding Phase
- Design mod API
- Implement mod loader
- Create example mods
- Write mod documentation

## Testing Strategy

### Unit Tests
```csharp
using NUnit.Framework;

[TestFixture]
public class MaterialTests {
    [Test]
    public void TestMaterialStats() {
        var material = new Material("test");
        material.SetStat(StatType.Durability, 100);
        Assert.AreEqual(100, material.GetStat(StatType.Durability));
    }
}
```

### Integration Tests
- Test tool assembly
- Test modifier application
- Test data loading
- Test networking

### Playtest
- Manual testing in Unity
- Performance profiling
- User experience testing

## Performance Targets

- **FPS**: 60+ in normal gameplay
- **Tool crafting**: < 100ms
- **Material registry load**: < 1s
- **Modifier application**: < 50ms
- **Save/Load**: < 2s for full inventory

## Common Pitfalls

1. **Over-use of MonoBehaviour** - Not everything needs to be a component
2. **Synchronous loading** - Use async for large data sets
3. **Not using object pooling** - Important for particles and projectiles
4. **Ignoring garbage collection** - Minimize allocations in hot paths
5. **Missing null checks** - Unity's "fake null" can be tricky
6. **Not testing in builds** - Editor behavior differs from builds

## Resources

- Unity Documentation
- C# Language Reference
- .NET API Documentation
- Unity Forums
- This Documentation (Java implementation reference)

## Navigation

- [Start with Porting Overview →](./01-Porting-Overview.md)
- [Back to Main Documentation ↑](../README.md)
