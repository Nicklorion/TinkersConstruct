# Getting Started Guide

This guide will help you get started with understanding, modifying, or extending Tinkers' Construct.

## For Different Audiences

Choose your path:

- **[Understanding the Mod](#understanding-the-mod)** - Learn how it works
- **[Creating an Addon](#creating-an-addon)** - Extend with new content
- **[Porting to Unity](#porting-to-unity)** - Recreate in C#/Unity
- **[Contributing](#contributing)** - Improve the codebase

## Understanding the Mod

### 1. Start with the Architecture

Begin by reading the architecture documentation to understand the big picture:

1. Read [Architecture Overview](./Architecture/01-Overview.md)
2. Review [Module Structure](./Architecture/02-Module-Structure.md)
3. Understand the [Design Patterns](./Architecture/README.md)

**Key Concepts to Grasp:**
- Materials provide stats to tools
- Tools are assembled from parts
- Modifiers enhance tools
- Everything is data-driven via JSON

### 2. Dive into a System

Pick one system and understand it deeply:

**Materials System** (Recommended starting point)
1. Read [Materials System Technical Docs](./Technical/01-Materials-System.md)
2. Look at material JSON files in `src/main/resources/data/tconstruct/tinkering/materials/`
3. Review `MaterialRegistry.java`
4. Trace how materials load from JSON

**Tools System**
1. Read [Tools System Technical Docs](./Technical/02-Tools-System.md)
2. Examine `ToolStack.java` to see how tool data is stored
3. Look at tool definitions in `data/tconstruct/tinkering/tool_definitions/`
4. Understand NBT data structure

### 3. Build and Run the Mod

```bash
# Clone the repository
git clone https://github.com/SlimeKnights/TinkersConstruct.git
cd TinkersConstruct

# Build the mod
./gradlew build

# Generate IDE run configurations
./gradlew genIntellijRuns  # For IntelliJ IDEA
# or
./gradlew genEclipseRuns   # For Eclipse

# Run in development
# Use the run configurations created above
```

### 4. Experiment with Data

Create custom materials and tools using datapacks:

1. Create a datapack in `run/saves/<world>/datapacks/test/`
2. Add a custom material definition
3. Test it in-game
4. Modify and iterate

**Example Custom Material:**

Create `data/test/tinkering/materials/definition/custom.json`:
```json
{
    "craftable": true,
    "tier": 2,
    "sortOrder": 200
}
```

Create `data/test/tinkering/materials/stats/tconstruct/head/custom.json`:
```json
{
    "durability": 300,
    "mining_speed": 7.0,
    "mining_tier": 2,
    "attack": 2.5
}
```

### 5. Use the Quick Reference

Keep the [Quick Reference Guide](./QUICK-REFERENCE.md) handy for:
- Common code patterns
- JSON formats
- Registry access
- NBT structures

## Creating an Addon

### Prerequisites

- Java Development Kit (JDK) 17+
- IntelliJ IDEA or Eclipse
- Basic understanding of Java and Minecraft modding
- Familiarity with Forge

### Step 1: Set Up Your Project

**build.gradle:**
```gradle
plugins {
    id 'net.minecraftforge.gradle' version '[6.0,6.0.37]'
}

repositories {
    mavenCentral()
    maven {
        name 'DVS1 Maven FS'
        url 'https://dvs1.progwml6.com/files/maven'
    }
}

dependencies {
    minecraft 'net.minecraftforge:forge:VERSION'
    
    // Tinkers' Construct
    implementation fg.deobf("slimeknights.tconstruct:TConstruct:VERSION")
    
    // Mantle (required)
    implementation fg.deobf("slimeknights.mantle:Mantle:VERSION")
}
```

### Step 2: Create Your Mod Class

```java
package com.yourname.youraddon;

import net.minecraftforge.fml.common.Mod;

@Mod("youraddon")
public class YourAddon {
    public static final String MOD_ID = "youraddon";
    
    public YourAddon() {
        // Initialize your addon
    }
}
```

### Step 3: Choose Your Approach

**Option A: Data-Only Addon** (Recommended for beginners)

Create materials, tools, and modifiers using only JSON:

```
src/main/resources/data/youraddon/
├── tinkering/
│   ├── materials/
│   │   ├── definition/custom_material.json
│   │   └── stats/tconstruct/head/custom_material.json
│   └── tool_definitions/custom_tool.json
└── recipes/
    └── tools/custom_tool.json
```

**Option B: Code + Data Addon**

Combine code for complex behavior with data for content:

1. Create custom modifiers in Java
2. Define materials in JSON
3. Register programmatically

**Example Modifier:**
```java
public class ExplosiveModifier extends Modifier 
    implements BlockAfterBreakModifierHook {
    
    @Override
    public void afterBlockBreak(IToolStackView tool, int level,
                               ToolHarvestContext context) {
        if (!context.isEffective()) return;
        
        Level world = context.getLevel();
        BlockPos pos = context.getPos();
        
        // Create explosion (size = level)
        world.explode(null, pos.getX(), pos.getY(), pos.getZ(),
                     level, Level.ExplosionInteraction.BLOCK);
    }
}
```

**Register it:**
```java
@SubscribeEvent
public void registerModifiers(RegisterEvent event) {
    event.register(TinkerRegistries.MODIFIERS.get(), helper -> {
        helper.register(new ResourceLocation("youraddon", "explosive"),
                       new ExplosiveModifier());
    });
}
```

### Step 4: Test Your Addon

1. Build your addon: `./gradlew build`
2. Run in development environment
3. Test with different material combinations
4. Verify modifier behavior
5. Check datapack reload

### Step 5: Documentation and Release

1. Document your materials and modifiers
2. Create example recipes
3. Write a user guide
4. Package for release
5. Publish on CurseForge/Modrinth

### Resources for Addon Developers

- [API Documentation](./API/README.md)
- [API Overview](./API/01-API-Overview.md)
- [Mod API Interface](./API/Mod-API/README.md)
- Tinkers' Construct source code (best examples)

## Porting to Unity

### Prerequisites

- Unity 2021.3 LTS or newer
- C# programming experience
- Understanding of Unity's component system
- Familiarity with ScriptableObjects

### Step 1: Understand the Java Implementation

Before porting, thoroughly understand the original:

1. Read [C# Unity Port Guide](./CSharp-Unity-Port/README.md)
2. Study [Porting Overview](./CSharp-Unity-Port/01-Porting-Overview.md)
3. Review Java to C# translation patterns
4. Understand Unity-specific considerations

### Step 2: Set Up Your Unity Project

```
YourGame/
├── Assets/
│   ├── Scripts/
│   │   └── TinkersConstruct/
│   │       ├── Core/
│   │       ├── Materials/
│   │       ├── Tools/
│   │       ├── Modifiers/
│   │       └── UI/
│   ├── Resources/
│   │   └── TinkersConstruct/
│   │       └── Registries/
│   └── StreamingAssets/
│       └── data/
│           ├── materials/
│           └── tools/
```

### Step 3: Start with Core Systems

**Phase 1: Data Structures**
1. Create C# classes for Material, Tool, Modifier
2. Implement serialization
3. Set up ScriptableObject registries

**Phase 2: Material System**
```csharp
[CreateAssetMenu(fileName = "New Material", 
                 menuName = "TConstruct/Material")]
public class Material : ScriptableObject {
    [SerializeField] private string materialId;
    [SerializeField] private MaterialStats headStats;
    [SerializeField] private MaterialStats handleStats;
    
    // Implementation
}
```

**Phase 3: Tool System**
```csharp
[System.Serializable]
public class ToolData {
    public string toolId;
    public List<string> materialIds;
    public ToolStats stats;
    public List<ModifierEntry> modifiers;
}
```

**Phase 4: UI System**
- Create crafting station UI
- Implement drag-and-drop
- Tool preview system
- Modifier application interface

### Step 4: Implement Modding API

Follow the standardized API in [Mod API Documentation](./API/Mod-API/README.md):

```csharp
public interface IMod {
    string ModId { get; }
    string ModName { get; }
    void OnLoad(IContentRegistry registry);
}
```

### Step 5: Testing and Optimization

1. Unit tests for core systems
2. Integration tests for tool assembly
3. Performance profiling
4. Memory optimization
5. Build testing

### Resources for Unity Developers

- [C# Unity Port Guide](./CSharp-Unity-Port/README.md)
- [Porting Overview](./CSharp-Unity-Port/01-Porting-Overview.md)
- Unity documentation
- C# language reference

## Contributing

Want to contribute to Tinkers' Construct itself?

### Getting Started

1. **Fork the repository**
2. **Create a feature branch**
3. **Make your changes**
4. **Test thoroughly**
5. **Submit a pull request**

### Areas to Contribute

- **Bug Fixes**: Fix existing issues
- **Features**: Add new modifiers, tools, materials
- **Documentation**: Improve or expand docs
- **Performance**: Optimize existing code
- **Testing**: Add unit tests

### Contribution Guidelines

1. Follow existing code style
2. Write clear commit messages
3. Add/update documentation
4. Include tests for new features
5. Ensure builds pass

### Development Workflow

```bash
# Fork and clone
git clone https://github.com/YourUsername/TinkersConstruct.git
cd TinkersConstruct

# Create feature branch
git checkout -b feature/my-feature

# Make changes and test
./gradlew build
./gradlew test

# Commit and push
git add .
git commit -m "Add: my feature"
git push origin feature/my-feature

# Create pull request on GitHub
```

### Code Review Process

1. Automated checks must pass
2. Code review by maintainers
3. Address feedback
4. Final approval and merge

## Common Pitfalls to Avoid

### For All Developers

1. **Not Reading Documentation**: Start with docs, not code
2. **Ignoring Data-Driven Design**: Use JSON when possible
3. **Skipping Testing**: Always test your changes
4. **Poor Error Handling**: Handle null and missing data
5. **Not Following Patterns**: Match existing code style

### For Addon Developers

1. **Accessing Internal APIs**: Use only public interfaces
2. **Hardcoding Values**: Use configuration and data files
3. **Circular Dependencies**: Declare dependencies properly
4. **Missing Null Checks**: Always validate registry queries
5. **Forgetting Cleanup**: Unregister content on unload

### For Unity Porters

1. **Direct Translation**: Adapt to Unity patterns, don't copy blindly
2. **Ignoring Unity Best Practices**: Use ScriptableObjects, etc.
3. **Poor Performance**: Profile and optimize for Unity
4. **Missing Serialization**: Implement proper save/load
5. **Skipping Garbage Collection**: Minimize allocations

## Next Steps

### After This Guide

1. **Explore the Codebase**: Navigate through source files
2. **Read Technical Docs**: Deep dive into systems
3. **Try Examples**: Implement example features
4. **Join Community**: Discord, GitHub discussions
5. **Start Building**: Create your first addon or port

### Recommended Reading Order

1. This guide (you are here)
2. [Architecture Overview](./Architecture/01-Overview.md)
3. [Materials System](./Technical/01-Materials-System.md)
4. [API Overview](./API/01-API-Overview.md)
5. [Quick Reference](./QUICK-REFERENCE.md)

### Get Help

- **GitHub Issues**: Technical problems
- **Discord**: Quick questions
- **Documentation**: Reference material
- **Source Code**: Ultimate truth

## Navigation

- [Architecture Documentation →](./Architecture/README.md)
- [Technical Documentation →](./Technical/README.md)
- [API Documentation →](./API/README.md)
- [C# Unity Port Guide →](./CSharp-Unity-Port/README.md)
- [Quick Reference →](./QUICK-REFERENCE.md)
- [Back to Main Documentation ↑](./README.md)
