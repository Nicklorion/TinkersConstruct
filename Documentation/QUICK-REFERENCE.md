# Quick Reference Guide

This guide provides quick lookups for common tasks and patterns in Tinkers' Construct.

## Table of Contents

- [Data File Locations](#data-file-locations)
- [Common Code Patterns](#common-code-patterns)
- [Registry Access](#registry-access)
- [NBT Structure](#nbt-structure)
- [Event Hooks](#event-hooks)
- [JSON Formats](#json-formats)

## Data File Locations

### Material Data
```
data/<namespace>/tinkering/materials/
├── definition/<material>.json      # Material properties
├── stats/
│   ├── tconstruct/head/<material>.json
│   ├── tconstruct/handle/<material>.json
│   └── tconstruct/extra/<material>.json
├── traits/<material>.json          # Material traits
└── render_info/<material>.json     # Rendering data
```

### Tool Data
```
data/<namespace>/tinkering/tool_definitions/
└── <tool>.json                     # Tool structure and stats
```

### Station Layouts
```
data/<namespace>/tinkering/station_layouts/
└── <layout>.json                   # UI slot arrangements
```

### Recipes
```
data/<namespace>/recipes/
├── tools/                          # Tool-related recipes
├── parts/                          # Part crafting
└── modifiers/                      # Modifier recipes
```

## Common Code Patterns

### Get Material by ID
```java
MaterialId id = new MaterialId("tconstruct", "iron");
IMaterialRegistry registry = MaterialRegistry.getInstance();
IMaterial material = registry.getMaterial(id);

if (material != null) {
    // Use material
}
```

### Get Material Stats
```java
IMaterialRegistry registry = MaterialRegistry.getInstance();
HeadMaterialStats stats = (HeadMaterialStats) 
    registry.getMaterialStats(
        new MaterialId("tconstruct", "iron"),
        HeadMaterialStats.ID
    );

if (stats != null) {
    int durability = stats.getDurability();
    float miningSpeed = stats.getMiningSpeed();
}
```

### Read Tool Data
```java
// Read-only access
IToolStackView tool = ToolStack.from(itemStack);
if (tool != null) {
    MaterialNBT materials = tool.getMaterials();
    StatsNBT stats = tool.getStats();
    ModifierNBT modifiers = tool.getModifiers();
    
    int durability = stats.getInt(ToolStats.DURABILITY);
    int damage = tool.getDamage();
}
```

### Modify Tool Data
```java
// Create mutable copy
ToolStack tool = ToolStack.copyFrom(itemStack);

// Modify
tool.setDamage(0); // Repair
tool.addModifier(new ModifierId("tconstruct", "sharpness"), 1);

// Write back
CompoundTag tag = tool.createTag();
itemStack.setTag(tag);
```

### Check if Item is Modifiable
```java
if (itemStack.getItem() instanceof IModifiable) {
    IModifiable modifiable = (IModifiable) itemStack.getItem();
    ToolDefinition definition = modifiable.getToolDefinition();
    // Use tool
}
```

### Create Custom Modifier
```java
public class MyModifier extends Modifier {
    @Override
    public Component getDisplayName(int level) {
        return Component.translatable(getTranslationKey());
    }
    
    @Override
    public int getPriority() {
        return 100;
    }
}
```

### Register Modifier
```java
@SubscribeEvent
public void registerModifiers(RegisterEvent event) {
    event.register(TinkerRegistries.MODIFIERS.get(), helper -> {
        helper.register(
            new ResourceLocation("mymod", "custom"),
            new MyModifier()
        );
    });
}
```

## Registry Access

### Material Registry
```java
IMaterialRegistry materials = MaterialRegistry.getInstance();

// Get all materials
Collection<IMaterial> allMaterials = materials.getAllMaterials();

// Get specific material
IMaterial iron = materials.getMaterial(
    new MaterialId("tconstruct", "iron"));

// Get material stats
IMaterialStats stats = materials.getMaterialStats(
    new MaterialId("tconstruct", "iron"),
    HeadMaterialStats.ID
);

// Get material traits
List<ModifierEntry> traits = materials.getTraits(
    new MaterialId("tconstruct", "iron"),
    HeadMaterialStats.ID
);
```

### Tool Definition Loader
```java
ToolDefinitionLoader loader = ToolDefinitionLoader.getInstance();

// Get all tool definitions
Collection<ToolDefinition> tools = loader.getRegisteredToolDefinitions();

// Tool definitions are accessed through items
if (item instanceof IModifiable) {
    ToolDefinition def = ((IModifiable) item).getToolDefinition();
}
```

### Modifier Registry (Forge Registry)
```java
Registry<Modifier> modifiers = TinkerRegistries.MODIFIERS.get();

// Get modifier by ID
ModifierId id = new ModifierId("tconstruct", "sharpness");
Modifier modifier = modifiers.getValue(id);

// Get all modifiers
Set<ResourceLocation> allIds = modifiers.keySet();
```

## NBT Structure

### Tool NBT
```
ItemStack.tag:
{
    "tconstruct": {
        "materials": [
            "tconstruct:iron",      // Head
            "tconstruct:wood",      // Handle
            "tconstruct:flint"      // Binding
        ],
        "stats": {
            "tconstruct:durability": 204,
            "tconstruct:mining_speed": 6.0,
            "tconstruct:attack_damage": 2.0,
            "tconstruct:harvest_tier": 2
        },
        "modifiers": {
            "modifiers": [
                {
                    "name": "tconstruct:sharpness",
                    "level": 2
                }
            ]
        },
        "damage": 0,
        "persistent": {
            "custom_data": "value"
        },
        "volatile": {
            "last_used": 1234567890
        }
    }
}
```

### Material Part NBT
```
ItemStack.tag:
{
    "tconstruct": {
        "material": "tconstruct:iron"
    }
}
```

## Event Hooks

### Subscribe to Events
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

### Modifier Hooks
```java
// Implement hook interface
public class MyModifier extends Modifier 
    implements DamageModifierHook {
    
    @Override
    public float getDamageBonus(IToolStackView tool, int level,
                                ToolAttackContext context,
                                float baseDamage, float damage) {
        return damage + level; // Add damage per level
    }
}
```

## JSON Formats

### Material Definition
```json
{
    "craftable": true,
    "tier": 2,
    "sortOrder": 150,
    "hidden": false
}
```

### Material Stats (Head)
```json
{
    "durability": 250,
    "mining_speed": 6.5,
    "mining_tier": 2,
    "attack": 2.5
}
```

### Material Stats (Handle)
```json
{
    "durability": 1.1,
    "miningSpeed": 1.0,
    "attack": 0.9
}
```

### Material Traits
```json
{
    "default": [
        {
            "name": "tconstruct:sturdy",
            "level": 1
        }
    ],
    "per_stat": {
        "tconstruct:head": [
            {
                "name": "tconstruct:jagged",
                "level": 1
            }
        ]
    }
}
```

### Tool Definition
```json
{
    "parts": [
        {
            "name": "head",
            "item": "tconstruct:pickaxe_head",
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
            "stat_type": "tconstruct:head",
            "multiplier": {
                "stat_type": "tconstruct:handle",
                "stat": "durability"
            }
        }
    }
}
```

### Station Layout
```json
{
    "main": [
        {
            "type": "tool",
            "index": 0
        }
    ],
    "left": [
        {
            "type": "input",
            "index": 0
        },
        {
            "type": "input",
            "index": 1
        }
    ]
}
```

## Stat Types

### Tool Stats
- `tconstruct:durability` - Max durability
- `tconstruct:attack_damage` - Attack damage
- `tconstruct:mining_speed` - Mining speed
- `tconstruct:harvest_tier` - Mining level
- `tconstruct:attack_speed` - Attack speed

### Material Stat Types
- `tconstruct:head` - Tool head stats
- `tconstruct:handle` - Tool handle stats
- `tconstruct:extra` - Extra part stats
- `tconstruct:bowstring` - Bowstring stats
- `tconstruct:grip` - Armor grip stats
- `tconstruct:plating` - Armor plating stats

## Modifier Slot Types
- `upgrades` - General upgrades
- `abilities` - Special abilities
- `defense` - Armor defense
- `souls` - Soul slots (special)
- `slotless` - Don't use slots

## Common Modifier Hooks

- `ModifierHook` - Base hook interface
- `DamageModifierHook` - Modify damage dealt
- `DurabilityModifierHook` - Modify durability cost
- `MiningSpeedModifierHook` - Modify mining speed
- `HarvestEnchantmentsModifierHook` - Add enchantment effects
- `BlockBreakModifierHook` - Called when block broken
- `EntityHitModifierHook` - Called when entity hit

## Useful Utility Classes

- `ToolHelper` - Tool manipulation utilities
- `ToolDamageUtil` - Durability management
- `ModifierUtil` - Modifier application helpers
- `MaterialIdNBT` - Material NBT reading/writing
- `RestrictedCompoundTag` - Safe NBT access
- `Util` - General utilities

## Navigation

- [Back to Main Documentation ↑](./README.md)
