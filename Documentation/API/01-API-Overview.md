# API Overview - Tinkers' Construct

## Introduction

The Tinkers' Construct API is designed to enable addon developers to extend and integrate with the mod. This document provides an overview of the API structure, stability guarantees, and common usage patterns.

## API Scope

### Public API

The public API consists of classes and interfaces explicitly designed for external use. These are primarily located in:

```
slimeknights.tconstruct.library.*
```

**Stability Promise:**
- Public API maintains backward compatibility within major versions
- Deprecations announced at least one minor version in advance
- Breaking changes only in major version updates

### Internal API

Internal APIs are implementation details that may change:
- Classes in `.internal.` packages
- Classes marked with `@VisibleForTesting`
- Undocumented methods
- Private and package-private members

**Warning:** Using internal APIs may break your addon without notice.

## Core API Components

### 1. Materials API

**Purpose:** Register and query materials

**Key Interfaces:**
- `IMaterial` - Material definition
- `IMaterialStats` - Material statistics
- `IMaterialRegistry` - Material access

**Example Usage:**
```java
// Get a material
IMaterialRegistry registry = MaterialRegistry.getInstance();
IMaterial iron = registry.getMaterial(new MaterialId("tconstruct", "iron"));

// Get material stats
HeadMaterialStats headStats = (HeadMaterialStats) 
    registry.getMaterialStats(iron.getId(), HeadMaterialStats.ID);

// Get material traits
List<ModifierEntry> traits = 
    registry.getTraits(iron.getId(), HeadMaterialStats.ID);
```

**Registration (Datapack):**
```json
// data/yourmod/tinkering/materials/definition/custom.json
{
    "craftable": true,
    "tier": 2,
    "sortOrder": 250
}

// data/yourmod/tinkering/materials/stats/tconstruct/head/custom.json
{
    "durability": 300,
    "mining_speed": 7.0,
    "mining_tier": 2,
    "attack": 2.5
}
```

### 2. Tools API

**Purpose:** Create custom tools and query tool data

**Key Interfaces:**
- `IModifiable` - Items that can have modifiers
- `IToolStackView` - Read-only tool access
- `ToolDefinition` - Tool structure definition
- `IToolStat<T>` - Tool statistic type

**Example Usage:**
```java
// Read tool data
IToolStackView tool = ToolStack.from(itemStack);
float durability = tool.getStats().getFloat(ToolStats.DURABILITY);
MaterialNBT materials = tool.getMaterials();

// Check if item is a tool
if (itemStack.getItem() instanceof IModifiable) {
    IModifiable modifiable = (IModifiable) itemStack.getItem();
    ToolDefinition definition = modifiable.getToolDefinition();
}

// Get tool stats
ToolStats stats = tool.getStats();
int damage = tool.getDamage();
int maxDurability = stats.getInt(ToolStats.DURABILITY);
```

**Custom Tool Definition (JSON):**
```json
// data/yourmod/tinkering/tool_definitions/custom_tool.json
{
    "parts": [
        {
            "name": "head",
            "item": "yourmod:custom_head",
            "stat_type": "tconstruct:head"
        },
        {
            "name": "handle",
            "item": "tconstruct:tool_handle",
            "stat_type": "tconstruct:handle"
        }
    ],
    "stats": {
        "tconstruct:durability": {
            "stat_type": "tconstruct:head"
        },
        "tconstruct:attack_damage": {
            "stat_type": "tconstruct:head"
        },
        "tconstruct:mining_speed": {
            "stat_type": "tconstruct:head"
        }
    }
}
```

### 3. Modifiers API

**Purpose:** Create custom tool modifiers

**Key Classes:**
- `Modifier` - Base modifier class
- `ModifierEntry` - Modifier with level
- `ModifierHook<T>` - Hook interface for modifier effects

**Example - Simple Modifier:**
```java
public class MyModifier extends Modifier {
    @Override
    public Component getDisplayName(int level) {
        return Component.translatable(getTranslationKey());
    }
    
    @Override
    public int getPriority() {
        return 100; // Higher = applied later
    }
}
```

**Example - Modifier with Hooks:**
```java
public class DamageModifier extends Modifier implements DamageModifierHook {
    
    @Override
    public float getDamageBonus(IToolStackView tool, int level, 
                                 ToolAttackContext context, 
                                 float baseDamage, float damage) {
        // Add bonus damage per level
        return damage + level;
    }
}
```

**Modifier Registration:**
```java
@SubscribeEvent
public void registerModifiers(RegisterEvent event) {
    event.register(TinkerRegistries.MODIFIERS.get(), helper -> {
        helper.register(new ResourceLocation("yourmod", "custom"), 
                       new MyModifier());
    });
}
```

### 4. Recipe API

**Purpose:** Create custom recipe types for crafting

**Key Classes:**
- `IModifyingRecipe` - Recipes that modify tools
- `IDisplayModifyingRecipe` - Recipes shown in JEI
- `ITinkerStationRecipe` - Tinker station recipes

**Example - Custom Recipe Type:**
```java
public class CustomRecipe implements ITinkerStationRecipe {
    private final ResourceLocation id;
    private final Ingredient ingredient;
    private final ItemStack result;
    
    @Override
    public boolean matches(ITinkerStationInventory inv, Level world) {
        return ingredient.test(inv.getTinkerableStack());
    }
    
    @Override
    public ItemStack assemble(ITinkerStationInventory inv, 
                             RegistryAccess access) {
        return result.copy();
    }
}
```

### 5. Data Loading API

**Purpose:** Load custom JSON data

**Key Interfaces:**
- `IJsonPredicate` - Conditional logic in JSON
- `GenericLoader<T>` - Generic JSON loader
- `IGenericLoader<T>` - Loader interface

**Example - Custom Predicate:**
```java
public record CustomPredicate(String value) implements IJsonPredicate<String> {
    public static final GenericLoader<CustomPredicate> LOADER = 
        new GenericLoader<>(
            new ResourceLocation("yourmod", "custom_predicate"),
            CustomPredicate.class
        );
    
    @Override
    public boolean test(String input) {
        return input.equals(value);
    }
    
    @Override
    public IGenericLoader<? extends IJsonPredicate<String>> getLoader() {
        return LOADER;
    }
}
```

## Common Patterns

### Pattern 1: Registry Access

All major content uses registry pattern:

```java
// Materials
IMaterialRegistry materials = MaterialRegistry.getInstance();

// Tools
ToolDefinitionLoader tools = ToolDefinitionLoader.getInstance();

// Modifiers (Forge registry)
Registry<Modifier> modifiers = TinkerRegistries.MODIFIERS.get();
```

### Pattern 2: Event Handling

Hook into Tinkers' events:

```java
@SubscribeEvent
public void onMaterialsLoaded(MaterialsLoadedEvent event) {
    // Materials are now available
    IMaterialRegistry registry = event.getRegistry();
}

@SubscribeEvent
public void onToolBuild(ToolEvent.OnToolBuild event) {
    // Tool is being built
    IToolStackView tool = event.getTool();
}
```

### Pattern 3: NBT Data Access

Read tool data safely:

```java
// Read-only access
IToolStackView tool = ToolStack.from(itemStack);
MaterialNBT materials = tool.getMaterials();
StatsNBT stats = tool.getStats();
ModifierNBT modifiers = tool.getModifiers();

// Modify tool (creates copy)
ToolStack mutableTool = ToolStack.copyFrom(itemStack);
mutableTool.setDamage(0); // Repair
mutableTool.addModifier(modifierId, 1);
// Write back to itemstack
itemStack.setTag(mutableTool.createTag());
```

### Pattern 4: Modifier Hooks

Implement behavior through hooks:

```java
public class MyModifier extends Modifier 
    implements MiningSpeedModifierHook, DurabilityModifierHook {
    
    @Override
    public void onBreakBlock(IToolStackView tool, int level, 
                            ToolHarvestContext context) {
        // Called when block is broken
    }
    
    @Override
    public int getDurabilityDamage(IToolStackView tool, int level, 
                                   int baseDamage) {
        // Modify durability cost
        return baseDamage - level; // Reduce damage per level
    }
}
```

## Addon Structure

### Recommended Project Structure

```
yourmod/
├── src/main/
│   ├── java/com/yourname/yourmod/
│   │   ├── YourMod.java
│   │   ├── modifiers/
│   │   │   ├── YourModifiers.java (registration)
│   │   │   └── CustomModifier.java
│   │   ├── tools/
│   │   │   └── CustomToolItem.java
│   │   └── data/
│   │       └── DataGenerators.java
│   └── resources/
│       ├── META-INF/
│       │   └── mods.toml
│       ├── assets/yourmod/
│       │   ├── lang/en_us.json
│       │   └── textures/
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

### Build Configuration

**build.gradle:**
```gradle
dependencies {
    minecraft 'net.minecraftforge:forge:VERSION'
    
    // Tinkers' Construct
    implementation fg.deobf("slimeknights.tconstruct:TConstruct:VERSION")
    
    // Mantle (required dependency)
    implementation fg.deobf("slimeknights.mantle:Mantle:VERSION")
}

repositories {
    maven {
        name 'DVS1 Maven FS'
        url 'https://dvs1.progwml6.com/files/maven'
    }
}
```

### Mod Metadata

**mods.toml:**
```toml
modId = "yourmod"
version = "${file.jarVersion}"
displayName = "Your Tinkers Addon"
description = "Description of your addon"

[[dependencies.yourmod]]
    modId = "tconstruct"
    mandatory = true
    versionRange = "[VERSION,)"
    ordering = "AFTER"
    side = "BOTH"

[[dependencies.yourmod]]
    modId = "mantle"
    mandatory = true
    versionRange = "[VERSION,)"
    ordering = "AFTER"
    side = "BOTH"
```

## API Best Practices

### 1. Use Data When Possible

Prefer JSON data over code:
- Materials: Always use JSON
- Tool definitions: Always use JSON
- Simple modifiers: Can use JSON
- Complex logic: Requires code

### 2. Check for Null

Always null-check registry queries:
```java
IMaterial material = registry.getMaterial(id);
if (material == null) {
    // Handle missing material
    return;
}
```

### 3. Respect API Boundaries

Don't:
- Access internal classes
- Reflect into private fields
- Modify registries after init
- Assume implementation details

Do:
- Use public interfaces
- Follow event ordering
- Register during proper events
- Check API documentation

### 4. Handle Compatibility

Support multiple versions:
```java
// Check if feature exists
if (ToolStats.hasStatType("yourmod:custom_stat")) {
    // Use custom stat
}

// Version checks
if (TConstruct.getAPIVersion().compareTo("2.0.0") >= 0) {
    // Use new API
} else {
    // Use old API
}
```

### 5. Test Thoroughly

Test with:
- Different Minecraft versions
- Different Forge versions
- Different TConstruct versions
- Other popular addons
- Datapack reloads
- Multiplayer scenarios

## Common Issues

### Issue 1: ClassNotFoundException
**Cause:** Missing dependency or wrong version
**Solution:** Check build.gradle dependencies

### Issue 2: Material Not Loading
**Cause:** JSON in wrong location or invalid format
**Solution:** Check datapack path and JSON syntax

### Issue 3: Modifier Not Applied
**Cause:** Not registered or missing hooks
**Solution:** Verify registration and implement required hooks

### Issue 4: Client Desync
**Cause:** Not handling client-server differences
**Solution:** Check side (@OnlyIn, DistExecutor)

### Issue 5: NBT Data Lost
**Cause:** Not copying NBT properly
**Solution:** Use ToolStack.copyFrom()

## API Examples

### Example 1: Simple Material

```json
// Material definition
{
    "craftable": true,
    "tier": 2,
    "sortOrder": 200
}

// Head stats
{
    "durability": 250,
    "mining_speed": 6.5,
    "mining_tier": 2,
    "attack": 2.0
}

// Traits
{
    "default": [
        {
            "name": "tconstruct:sturdy",
            "level": 1
        }
    ]
}
```

### Example 2: Custom Modifier

```java
public class AutoSmeltModifier extends Modifier 
    implements BlockAfterBreakModifierHook {
    
    @Override
    public void afterBlockBreak(IToolStackView tool, int level, 
                               ToolHarvestContext context) {
        if (!context.isEffective()) return;
        
        BlockState state = context.getState();
        Level world = context.getLevel();
        BlockPos pos = context.getPos();
        
        // Get smelting recipe
        List<ItemStack> drops = Block.getDrops(state, 
            (ServerLevel) world, pos, null);
        
        for (ItemStack drop : drops) {
            Optional<SmeltingRecipe> recipe = world.getRecipeManager()
                .getRecipeFor(RecipeType.SMELTING, 
                             new SimpleContainer(drop), world);
            
            if (recipe.isPresent()) {
                ItemStack smelted = recipe.get().getResultItem(
                    world.registryAccess());
                Block.popResource(world, pos, smelted.copy());
            }
        }
    }
}
```

## Support and Resources

### Documentation
- This documentation
- JavaDocs (in source code)
- Wiki: https://slimeknights.github.io/docs/
- Examples in TConstruct source

### Community
- Discord: Join SlimeKnights Discord
- GitHub: Report issues and ask questions
- CurseForge: Find other addons for examples

### Updates
- GitHub releases for API changes
- Changelog for version updates
- Deprecation notices in code

## Navigation

- [Next: Materials API →](./03-Materials-API.md)
- [Back to API Documentation ↑](./README.md)
