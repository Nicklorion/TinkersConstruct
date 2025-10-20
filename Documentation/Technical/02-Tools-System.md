# Tools System - Technical Documentation

## Overview

The Tools System is the core of Tinkers' Construct, enabling the assembly, customization, and use of modifiable tools. This document provides deep technical details on how tools are structured, stored, calculated, and rendered.

## System Architecture

```
ToolDefinitionLoader (Data)
    ↓
ToolDefinition (Structure)
    ├─ Part Requirements
    ├─ Stat Definitions
    └─ Assembly Rules
        ↓
ToolStack (Runtime Instance)
    ├─ MaterialNBT (Materials used)
    ├─ StatsNBT (Calculated stats)
    ├─ ModifierNBT (Applied modifiers)
    └─ Persistent Data
        ↓
ToolItem (Minecraft Item)
    ├─ IModifiable interface
    ├─ Tool behavior
    └─ Rendering
```

## Core Components

### 1. Tool Definitions

Tool definitions describe the structure and requirements of a tool type.

#### ToolDefinition Class

```java
public class ToolDefinition {
    private final ResourceLocation id;
    private ToolDefinitionData data;
    
    public ResourceLocation getId() {
        return id;
    }
    
    public List<PartRequirement> getRequiredComponents() {
        return data.getParts();
    }
    
    public Map<IToolStat<?>, StatDefinition> getStats() {
        return data.getStats();
    }
}
```

#### ToolDefinitionData Structure

```java
public class ToolDefinitionData {
    private final List<PartRequirement> parts;
    private final Map<IToolStat<?>, StatDefinition> stats;
    private final List<ModifierEntry> startingModifiers;
    private final int startingSlots;
    
    // Methods for accessing data
}
```

#### JSON Format

Located at: `data/<namespace>/tinkering/tool_definitions/<tool>.json`

**Minimal Tool Definition:**
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
            "stat_type": "tconstruct:head"
        },
        "tconstruct:mining_speed": {
            "stat_type": "tconstruct:head"
        }
    }
}
```

**Full Tool Definition:**
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
        },
        {
            "name": "binding",
            "item": "tconstruct:tool_binding",
            "stat_type": "tconstruct:extra"
        }
    ],
    "stats": {
        "tconstruct:durability": {
            "stat_type": "tconstruct:head",
            "multiplier": {
                "stat_type": "tconstruct:handle",
                "stat": "durability"
            }
        },
        "tconstruct:mining_speed": {
            "stat_type": "tconstruct:head",
            "multiplier": {
                "stat_type": "tconstruct:handle",
                "stat": "miningSpeed"
            }
        },
        "tconstruct:attack_damage": {
            "stat_type": "tconstruct:head",
            "multiplier": {
                "stat_type": "tconstruct:handle",
                "stat": "attack"
            }
        },
        "tconstruct:harvest_tier": {
            "stat_type": "tconstruct:head",
            "stat": "miningTier"
        }
    },
    "starting_modifiers": [
        {
            "name": "tconstruct:offhand_attack",
            "level": 1
        }
    ],
    "starting_slots": {
        "upgrades": 3,
        "abilities": 1,
        "defense": 0,
        "souls": 0
    }
}
```

### 2. Tool Parts

Tool parts are items that represent components used in assembly.

#### IToolPart Interface

```java
public interface IToolPart {
    /** Gets the material variant for this part */
    MaterialVariantId getMaterial(ItemStack stack);
    
    /** Gets the stat type this part contributes to */
    MaterialStatsId getStatType(ItemStack stack);
    
    /** Checks if this item can be used as a tool part */
    boolean canUseMaterial(MaterialVariantId material);
}
```

#### MaterialItem Implementation

```java
public class MaterialItem extends Item implements IToolPart {
    private final MaterialStatsId statType;
    
    @Override
    public MaterialVariantId getMaterial(ItemStack stack) {
        return MaterialIdNBT.from(stack).getMaterial();
    }
    
    @Override
    public MaterialStatsId getStatType(ItemStack stack) {
        return statType;
    }
}
```

### 3. Tool NBT Data

Tools store all their data in NBT format on the ItemStack.

#### NBT Structure

```
Tool ItemStack NBT:
{
    "tconstruct": {
        "materials": [
            "tconstruct:iron",
            "tconstruct:wood",
            "tconstruct:flint"
        ],
        "stats": {
            "tconstruct:durability": 204.0,
            "tconstruct:mining_speed": 6.0,
            "tconstruct:attack_damage": 2.0,
            "tconstruct:harvest_tier": 2
        },
        "modifiers": {
            "modifiers": [
                {
                    "name": "tconstruct:sharpness",
                    "level": 2
                },
                {
                    "name": "tconstruct:unbreaking",
                    "level": 1
                }
            ]
        },
        "damage": 50,
        "volatile": {
            "last_modified": 1234567890
        }
    }
}
```

#### ToolStack Class

Wrapper for tool data:

```java
public class ToolStack implements IToolStackView {
    private final ToolDefinition definition;
    private final MaterialNBT materials;
    private final StatsNBT stats;
    private final ModifierNBT modifiers;
    private final CompoundTag volatileData;
    private final CompoundTag persistentData;
    
    // Read from ItemStack
    public static IToolStackView from(ItemStack stack) {
        if (!(stack.getItem() instanceof IModifiable)) {
            return null;
        }
        return new ToolStack(stack);
    }
    
    // Create mutable copy
    public static ToolStack copyFrom(ItemStack stack) {
        return new ToolStack(stack);
    }
    
    // Getters
    public MaterialNBT getMaterials() { return materials; }
    public StatsNBT getStats() { return stats; }
    public ModifierNBT getModifiers() { return modifiers; }
    public int getDamage() { /* read from NBT */ }
    
    // Setters (mutable operations)
    public void setDamage(int damage) { /* write to NBT */ }
    public void addModifier(ModifierId id, int level) { /* update NBT */ }
    
    // Convert back to ItemStack NBT
    public CompoundTag createTag() { /* serialize to NBT */ }
}
```

#### MaterialNBT

Stores which materials were used:

```java
public class MaterialNBT {
    private final List<MaterialVariantId> materials;
    
    public MaterialNBT(List<MaterialVariantId> materials) {
        this.materials = materials;
    }
    
    public MaterialVariantId getMaterial(int index) {
        return materials.get(index);
    }
    
    public List<MaterialVariantId> getMaterials() {
        return materials;
    }
    
    // Serialize/deserialize
    public void write(CompoundTag tag) {
        ListTag list = new ListTag();
        for (MaterialVariantId material : materials) {
            list.add(StringTag.valueOf(material.toString()));
        }
        tag.put("materials", list);
    }
    
    public static MaterialNBT read(CompoundTag tag) {
        ListTag list = tag.getList("materials", Tag.TAG_STRING);
        List<MaterialVariantId> materials = new ArrayList<>();
        for (int i = 0; i < list.size(); i++) {
            materials.add(MaterialVariantId.parse(list.getString(i)));
        }
        return new MaterialNBT(materials);
    }
}
```

#### StatsNBT

Stores calculated tool statistics:

```java
public class StatsNBT {
    private final Map<IToolStat<?>, Object> stats;
    
    public <T> T get(IToolStat<T> stat) {
        return stat.cast(stats.get(stat));
    }
    
    public void set(IToolStat<?> stat, Object value) {
        stats.put(stat, value);
    }
    
    public int getInt(IToolStat<Integer> stat) {
        return get(stat);
    }
    
    public float getFloat(IToolStat<Float> stat) {
        return get(stat);
    }
    
    // Serialize/deserialize
    public void write(CompoundTag tag) {
        for (Map.Entry<IToolStat<?>, Object> entry : stats.entrySet()) {
            IToolStat<?> stat = entry.getKey();
            stat.write(tag, entry.getValue());
        }
    }
}
```

### 4. Tool Stats Calculation

Stats are calculated from materials and modifiers.

#### Calculation Flow

```
1. Get Tool Definition
2. Get Material for Each Part
3. Get Material Stats for Each Part's Stat Type
4. Combine Stats Based on Definition Rules
5. Apply Material Traits as Modifiers
6. Apply User-Added Modifiers
7. Cache Final Stats in NBT
```

#### Example Calculation Code

```java
public class ToolStatsBuilder {
    
    public static StatsNBT buildStats(
            ToolDefinition definition,
            MaterialNBT materials) {
        
        StatsNBT stats = new StatsNBT();
        IMaterialRegistry matRegistry = MaterialRegistry.getInstance();
        
        // For each stat in definition
        for (Map.Entry<IToolStat<?>, StatDefinition> entry : 
             definition.getStats().entrySet()) {
            
            IToolStat<?> stat = entry.getKey();
            StatDefinition statDef = entry.getValue();
            
            // Get base value from material
            int partIndex = statDef.getPartIndex();
            MaterialVariantId material = materials.getMaterial(partIndex);
            MaterialStatsId statType = statDef.getStatType();
            
            IMaterialStats materialStats = 
                matRegistry.getMaterialStats(material.getId(), statType);
            
            if (materialStats != null) {
                Object baseValue = statDef.getValue(materialStats);
                
                // Apply multipliers if defined
                if (statDef.hasMultiplier()) {
                    float multiplier = getMultiplier(
                        statDef.getMultiplier(), materials);
                    baseValue = applyMultiplier(baseValue, multiplier);
                }
                
                stats.set(stat, baseValue);
            }
        }
        
        return stats;
    }
    
    private static float getMultiplier(
            MultiplierDefinition multDef,
            MaterialNBT materials) {
        // Get material for multiplier part
        // Get stat value from that material
        // Return as multiplier
        return 1.0f;
    }
}
```

### 5. Tool Rendering

Tools are rendered with dynamic models and textures.

#### ToolModel

```java
public class ToolModel implements BakedModel {
    private final ToolDefinition definition;
    private final Map<String, BakedModel> partModels;
    
    @Override
    public List<BakedQuad> getQuads(
            @Nullable BlockState state,
            @Nullable Direction side,
            RandomSource rand,
            ModelData data,
            @Nullable RenderType renderType) {
        
        List<BakedQuad> quads = new ArrayList<>();
        
        // Get tool data from ModelData
        ToolStack tool = data.get(ToolModel.TOOL_DATA);
        
        if (tool != null) {
            // Render each part with its material color
            List<PartRequirement> parts = definition.getRequiredComponents();
            MaterialNBT materials = tool.getMaterials();
            
            for (int i = 0; i < parts.size(); i++) {
                PartRequirement part = parts.get(i);
                MaterialVariantId material = materials.getMaterial(i);
                
                // Get part model
                BakedModel partModel = partModels.get(part.getName());
                
                // Get material color
                MaterialRenderInfo renderInfo = 
                    MaterialRenderInfoLoader.INSTANCE.getRenderInfo(material);
                int color = renderInfo.getVertexColor(i);
                
                // Add colored quads
                quads.addAll(colorQuads(partModel.getQuads(), color));
            }
        }
        
        return quads;
    }
}
```

## Tool Usage

### Mining/Harvesting

```java
public class ToolItem extends TieredItem implements IModifiable {
    
    @Override
    public boolean mineBlock(ItemStack stack, Level world, 
                            BlockState state, BlockPos pos, 
                            LivingEntity entity) {
        
        ToolStack tool = ToolStack.from(stack);
        
        if (tool != null && !world.isClientSide) {
            // Create harvest context
            ToolHarvestContext context = new ToolHarvestContext(
                world, entity, state, pos, /* ... */);
            
            // Fire before break event
            ToolEvents.BEFORE_BLOCK_BREAK.forEach(
                tool, hook -> hook.beforeBlockBreak(tool, context));
            
            // Calculate durability cost
            int damage = getDurabilityDamage(tool, state);
            
            // Damage tool
            ToolDamageUtil.damage(tool, damage, entity, stack);
            
            // Fire after break event
            ToolEvents.AFTER_BLOCK_BREAK.forEach(
                tool, hook -> hook.afterBlockBreak(tool, context));
        }
        
        return true;
    }
    
    private int getDurabilityDamage(ToolStack tool, BlockState state) {
        int baseDamage = 1;
        
        // Modifiers can affect durability cost
        for (ModifierEntry entry : tool.getModifiers().getModifiers()) {
            Modifier modifier = entry.getModifier();
            if (modifier instanceof DurabilityModifierHook) {
                baseDamage = ((DurabilityModifierHook) modifier)
                    .getDurabilityDamage(tool, entry.getLevel(), baseDamage);
            }
        }
        
        return baseDamage;
    }
}
```

## Tool Assembly

### Assembly at Tinker Station

```java
public class TinkerStationRecipe implements ITinkerStationRecipe {
    
    @Override
    public ItemStack assemble(ITinkerStationInventory inv,
                             RegistryAccess access) {
        
        ToolDefinition definition = getToolDefinition();
        List<PartRequirement> parts = definition.getRequiredComponents();
        
        // Extract materials from parts
        List<MaterialVariantId> materials = new ArrayList<>();
        for (int i = 0; i < parts.size(); i++) {
            ItemStack partStack = inv.getItem(i);
            if (partStack.getItem() instanceof IToolPart) {
                IToolPart part = (IToolPart) partStack.getItem();
                materials.add(part.getMaterial(partStack));
            }
        }
        
        // Build tool
        return buildTool(definition, materials);
    }
    
    private ItemStack buildTool(ToolDefinition definition,
                               List<MaterialVariantId> materials) {
        
        // Create base item
        ItemStack result = new ItemStack(definition.getItem());
        
        // Create tool data
        MaterialNBT materialNBT = new MaterialNBT(materials);
        StatsNBT stats = ToolStatsBuilder.buildStats(
            definition, materialNBT);
        ModifierNBT modifiers = new ModifierNBT();
        
        // Add material traits
        addMaterialTraits(modifiers, definition, materials);
        
        // Add starting modifiers
        addStartingModifiers(modifiers, definition);
        
        // Write to NBT
        CompoundTag tag = new CompoundTag();
        materialNBT.write(tag);
        stats.write(tag);
        modifiers.write(tag);
        
        result.getOrCreateTag().put("tconstruct", tag);
        
        return result;
    }
}
```

## Performance Considerations

### Caching
- Tool stats cached in NBT (not recalculated each use)
- Material lookups cached
- Render models cached
- Modifier lists cached

### Lazy Evaluation
- Stats only calculated when tool is built
- Modifiers only applied when changed
- Rendering data loaded on-demand

### Memory Usage
- Tool NBT: ~200-500 bytes per tool
- In-memory cache minimal
- Models shared across instances

## Events

### Tool Events

```java
// Before tool is built
TinkerToolEvent.OnToolBuild

// After tool is built
TinkerToolEvent.ToolBuilt

// Before block broken
ToolEvents.BEFORE_BLOCK_BREAK

// After block broken
ToolEvents.AFTER_BLOCK_BREAK

// Before entity hit
ToolEvents.BEFORE_ENTITY_HIT

// Tool damaged
ToolEvents.TOOL_DAMAGE

// Tool repaired
ToolEvents.TOOL_REPAIR
```

## Navigation

- [Previous: Materials System ←](./01-Materials-System.md)
- [Next: Modifiers System →](./03-Modifiers-System.md)
- [Back to Technical Documentation ↑](./README.md)
