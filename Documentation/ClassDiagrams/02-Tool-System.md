# Class Diagram: Tool System

## Overview

The Tool System manages tool definitions, tool instances, parts assembly, NBT data persistence, and stat calculations. Tools are composed of multiple parts, each with a material, and can have modifiers applied.

---

## Core Tool Classes

```
           <<interface>>
        ┌───────────────────┐
        │  IToolStackView   │
        ├───────────────────┤
        │ + getItem()       │
        │ + getDefinition() │
        │ + getMaterials()  │
        │ + getStats()      │
        │ + getModifiers()  │
        │ + getDamage()     │
        └───────────────────┘
                △
                │ implements
                │
        ┌───────────────────┐
        │    ToolStack      │
        ├───────────────────┤
        │ - item: ItemStack │
        │ - definition: ID  │
        │ - materials: List │
        │ - stats: StatsNBT │
        │ - modifiers: List │
        │ - damage: int     │
        ├───────────────────┤
        │ + ToolStack()     │
        │ + from()          │
        │ + getItem()       │
        │ + copy()          │
        │ + setDamage()     │
        │ + addModifier()   │
        └───────────────────┘
                ◇
                │ contains
                │
        ┌───────────────────┐
        │  ToolDefinition   │
        ├───────────────────┤
        │ - id: ResourceLoc │
        │ - item: Item      │
        │ - parts: List     │
        │ - stats: List     │
        │ - traits: List    │
        │ - modules: List   │
        ├───────────────────┤
        │ + getId()         │
        │ + getParts()      │
        │ + getStats()      │
        │ + buildTool()     │
        └───────────────────┘
```

---

## Tool Parts System

```
              <<interface>>
           ┌──────────────────┐
           │   IToolPart      │
           ├──────────────────┤
           │ + getId()        │
           │ + getStatType()  │
           │ + getCost()      │
           └──────────────────┘
                   △
                   │ implements
                   │
           ┌──────────────────┐
           │   ToolPart       │
           ├──────────────────┤
           │ - id: ResourceLoc│
           │ - statType: ID   │
           │ - cost: int      │
           │ - item: Item     │
           ├──────────────────┤
           │ + getId()        │
           │ + getStatType()  │
           │ + getCost()      │
           │ + getItem()      │
           └──────────────────┘

        Concrete Part Types:
        
    ┌─────────────┐  ┌──────────────┐  ┌─────────────┐
    │  PickaxeHead│  │ HandlePart   │  │ BindingPart │
    ├─────────────┤  ├──────────────┤  ├─────────────┤
    │ statType:   │  │ statType:    │  │ statType:   │
    │  "head"     │  │  "handle"    │  │  "extra"    │
    ├─────────────┤  ├──────────────┤  ├─────────────┤
    │ + build()   │  │ + build()    │  │ + build()   │
    └─────────────┘  └──────────────┘  └─────────────┘
```

---

## Tool Stats System

```
              <<interface>>
           ┌──────────────────┐
           │   IToolStat      │
           ├──────────────────┤
           │ + getId()        │
           │ + getDefault()   │
           │ + getValue()     │
           └──────────────────┘
                   △
                   │ implements
        ┌──────────┴───────────┐
        │                      │
┌───────────────┐      ┌──────────────────┐
│ FloatToolStat │      │  IntToolStat     │
├───────────────┤      ├──────────────────┤
│ - id: StatId  │      │ - id: StatId     │
│ - default:flt │      │ - default: int   │
│ - min: float  │      │ - min: int       │
│ - max: float  │      │ - max: int       │
├───────────────┤      ├──────────────────┤
│ + clamp()     │      │ + clamp()        │
│ + add()       │      │ + add()          │
│ + multiply()  │      │ + multiply()     │
└───────────────┘      └──────────────────┘

    Common Tool Stats (static registry):
    
    ┌─────────────────────────────┐
    │      ToolStats (static)     │
    ├─────────────────────────────┤
    │ + DURABILITY: IntToolStat   │
    │ + MINING_SPEED: FloatToolSt │
    │ + HARVEST_TIER: IntToolStat │
    │ + ATTACK_DAMAGE: FloatToolS │
    │ + ATTACK_SPEED: FloatToolSt │
    │ + MODIFIER_SLOTS: IntToolSt │
    └─────────────────────────────┘
```

---

## Tool NBT Data Structure

```
        ┌──────────────────────────┐
        │      ToolDataNBT         │
        ├──────────────────────────┤
        │ - materials: MaterialNBT │
        │ - stats: StatsNBT        │
        │ - modifiers: ModifierNBT │
        │ - persistent: PersistNBT │
        │ - volatile: VolatileNBT  │
        ├──────────────────────────┤
        │ + getMaterials()         │
        │ + getStats()             │
        │ + getModifiers()         │
        │ + serialize()            │
        │ + deserialize()          │
        └──────────────────────────┘
                │
                ├───◇ contains
                │
        ┌───────┴────────────────────┬──────────────┐
        │                            │              │
        ▼                            ▼              ▼
┌────────────────┐          ┌────────────┐  ┌──────────────┐
│ MaterialNBT    │          │ StatsNBT   │  │ ModifierNBT  │
├────────────────┤          ├────────────┤  ├──────────────┤
│- materials:    │          │- stats:Map │  │- modifiers:  │
│  List<MatId>   │          │ <StatId,   │  │  List<Entry> │
│                │          │  float>    │  │              │
├────────────────┤          ├────────────┤  ├──────────────┤
│+ get(index)    │          │+ get(stat) │  │+ getLevel()  │
│+ set(index,id) │          │+ set(...)  │  │+ add(...)    │
│+ getList()     │          │+ update()  │  │+ remove(...) │
└────────────────┘          └────────────┘  └──────────────┘
```

**NBT Structure (JSON representation):**
```json
{
  "tic_materials": ["tconstruct:iron", "tconstruct:wood", "tconstruct:flint"],
  "tic_stats": {
    "tconstruct:durability": 204,
    "tconstruct:mining_speed": 6.0,
    "tconstruct:harvest_tier": 2,
    "tconstruct:attack": 2.5
  },
  "tic_modifiers": [
    {"id": "tconstruct:sharpness", "level": 2},
    {"id": "tconstruct:fortune", "level": 1}
  ],
  "tic_persistent": {
    "custom_name": "Iron Breaker"
  },
  "tic_volatile": {
    "damage": 50
  }
}
```

---

## Tool Building Process

```
┌───────────────────────────────────────────────────────────┐
│              Tool Assembly Flow                           │
└───────────┬───────────────────────────────────────────────┘
            │
            ▼
    ┌──────────────────┐
    │ Select Tool Type │
    │ (Definition)     │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ Select Materials │
    │ for Each Part    │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────────────┐
    │   Part Assembly:         │
    │   - Get stats for each   │
    │     material+part combo  │
    │   - Sum base stats       │
    └────────┬─────────────────┘
             │
             ▼
    ┌──────────────────────────┐
    │   Trait Application:     │
    │   - Get traits from      │
    │     materials            │
    │   - Apply as modifiers   │
    └────────┬─────────────────┘
             │
             ▼
    ┌──────────────────────────┐
    │   Stat Calculation:      │
    │   - Base stats from      │
    │     materials            │
    │   - Multipliers from     │
    │     handles              │
    │   - Modifiers applied    │
    └────────┬─────────────────┘
             │
             ▼
    ┌──────────────────────────┐
    │   Create ToolStack:      │
    │   - Write NBT data       │
    │   - Set initial durabil. │
    │   - Return tool item     │
    └──────────────────────────┘
```

---

## Tool Definition System

```
            <<interface>>
         ┌────────────────────┐
         │ IToolDefinition    │
         ├────────────────────┤
         │ + getId()          │
         │ + getItem()        │
         │ + getParts()       │
         │ + getStats()       │
         │ + buildTool()      │
         └────────────────────┘
                 △
                 │ implements
                 │
         ┌────────────────────┐
         │  ToolDefinition    │
         ├────────────────────┤
         │ - id: ResourceLoc  │
         │ - item: Item       │
         │ - parts: List<Part>│
         │ - statTypes: List  │
         │ - modules: List    │
         ├────────────────────┤
         │ + buildTool()      │
         │ + validate()       │
         └────────────────────┘
                 ◇
                 │ uses
                 │
         ┌────────────────────┐
         │  ToolDefinitionPrt │
         ├────────────────────┤
         │ - name: String     │
         │ - statType: ID     │
         │ - item: Item       │
         │ - required: bool   │
         ├────────────────────┤
         │ + getName()        │
         │ + getStatType()    │
         │ + isRequired()     │
         └────────────────────┘

        <<singleton>>
     ┌──────────────────────────┐
     │ ToolDefinitionRegistry   │
     ├──────────────────────────┤
     │ - definitions: Map       │
     ├──────────────────────────┤
     │ + register()             │
     │ + get(id)                │
     │ + getAll()               │
     └──────────────────────────┘
```

---

## Tool Definition JSON

**File:** `data/<namespace>/tinkering/tool_definitions/<tool_id>.json`

```json
{
  "item": "tconstruct:pickaxe",
  "parts": [
    {
      "name": "head",
      "stat_type": "tconstruct:head",
      "item": "tconstruct:pickaxe_head"
    },
    {
      "name": "handle",
      "stat_type": "tconstruct:handle",
      "item": "tconstruct:tool_handle"
    },
    {
      "name": "binding",
      "stat_type": "tconstruct:extra",
      "item": "tconstruct:tool_binding"
    }
  ],
  "stats": [
    "tconstruct:durability",
    "tconstruct:mining_speed",
    "tconstruct:harvest_tier",
    "tconstruct:attack"
  ],
  "modules": [
    {
      "type": "tconstruct:material_stats"
    },
    {
      "type": "tconstruct:material_traits"
    },
    {
      "type": "tconstruct:material_repair"
    }
  ]
}
```

---

## Stat Calculation Algorithm

```
Calculate Tool Stats:

1. Initialize stats map with defaults:
   stats = {stat_id: stat.getDefault() for stat in definition.getStats()}

2. For each part in definition.getParts():
   a. Get material for this part from tool materials
   b. Get material stats for (material, part.getStatType())
   c. Add/multiply stats based on stat type:
      - Head parts: Add directly to stats
      - Handle parts: Multiply existing stats
      - Extra parts: Usually no stats (only traits)
   
3. Apply material traits:
   a. Get traits for each material+stat_type combo
   b. Add traits as modifiers to tool
   
4. Apply modifiers:
   For each modifier in tool.getModifiers():
      a. Get modifier instance
      b. Call modifier.onBuildStats(stats, tool, level)
      c. Modifier can add/multiply/set stats
   
5. Clamp stats to valid ranges:
   For each stat in stats:
      stats[stat_id] = stat.clamp(stats[stat_id])
   
6. Return final stats map
```

**Example Calculation (Pickaxe):**

```
Materials:
- Head: Iron (durability: 204, speed: 6.0, tier: 2, attack: 2.0)
- Handle: Wood (multipliers: 1.0x durability, 1.0x speed, 1.0x attack)
- Binding: Flint (no stats, only traits)

Base Stats from Head:
  durability = 204
  mining_speed = 6.0
  harvest_tier = 2
  attack = 2.0

Apply Handle Multipliers:
  durability *= 1.0 = 204
  mining_speed *= 1.0 = 6.0
  attack *= 1.0 = 2.0

Apply Traits:
  Iron (head): +Magnetic
  Wood (handle): +Maintained
  Flint (binding): +Sharp

Final Stats:
  durability = 204
  mining_speed = 6.0
  harvest_tier = 2
  attack = 2.0
  modifiers = [Magnetic, Maintained, Sharp]
```

---

## Tool Item Class

```
              <<interface>>
           ┌──────────────────┐
           │   IToolItem      │
           ├──────────────────┤
           │ + getDefinition()│
           │ + getDamage()    │
           │ + setDamage()    │
           │ + canPerformAction│
           └──────────────────┘
                   △
                   │ implements
                   │
           ┌──────────────────┐
           │    ToolItem      │
           ├──────────────────┤
           │ - definition: ID │
           ├──────────────────┤
           │ + getDefinition()│
           │ + onBlockBreak() │
           │ + onHit()        │
           │ + getDestroySpeed│
           │ + isCorrectTool()│
           │ + canPerformAct. │
           └──────────────────┘
                   │
                   │ subclasses
        ┌──────────┴──────────┬────────────┐
        │                     │            │
┌───────────────┐   ┌─────────────┐  ┌──────────┐
│  HarvestTool  │   │ WeaponTool  │  │ArmorPiece│
├───────────────┤   ├─────────────┤  ├──────────┤
│- isEffective()│   │- getDamage()│  │-getArmor()│
│- getSpeed()   │   │- canHit()   │  │-getToughn│
└───────────────┘   └─────────────┘  └──────────┘
```

---

## Tool Durability System

```
           ┌──────────────────────┐
           │  ToolDamageUtil      │
           ├──────────────────────┤
           │ + damage(tool, amt)  │
           │ + repair(tool, amt)  │
           │ + isBroken(tool)     │
           │ + canRepair(tool)    │
           └──────────────────────┘
                   │
                   │ uses
                   │
           ┌──────────────────────┐
           │  ToolStack           │
           ├──────────────────────┤
           │ + getDamage()        │
           │ + setDamage()        │
           │ + getMaxDamage()     │
           │ + getRemainingUses() │
           │ + getDurabilityPct() │
           └──────────────────────┘

Durability Rules:
1. Damage increases with use
2. Different actions cost different durability
3. Broken tools (damage >= max) don't work
4. Some modifiers affect durability (Unbreaking, etc.)
5. Repairs restore durability up to max
```

---

## C# Implementation

### Core Classes

```csharp
// Interface
public interface IToolStackView
{
    ItemStack Item { get; }
    ToolDefinition Definition { get; }
    List<MaterialId> Materials { get; }
    Dictionary<ToolStatId, float> Stats { get; }
    List<ModifierEntry> Modifiers { get; }
    int Damage { get; }
}

// Tool Stack
[Serializable]
public class ToolStack : IToolStackView
{
    [SerializeField] private ItemStack item;
    [SerializeField] private ToolDefinitionId definitionId;
    [SerializeField] private List<MaterialId> materials;
    [SerializeField] private Dictionary<ToolStatId, float> stats;
    [SerializeField] private List<ModifierEntry> modifiers;
    [SerializeField] private int damage;
    
    public ItemStack Item => item;
    public ToolDefinition Definition => 
        ToolDefinitionRegistry.Instance.Get(definitionId);
    public List<MaterialId> Materials => materials;
    public Dictionary<ToolStatId, float> Stats => stats;
    public List<ModifierEntry> Modifiers => modifiers;
    public int Damage { get => damage; set => damage = value; }
    
    // Factory method
    public static ToolStack Create(ToolDefinition definition, 
                                   List<MaterialId> materials)
    {
        var tool = new ToolStack();
        tool.definitionId = definition.Id;
        tool.materials = new List<MaterialId>(materials);
        tool.stats = CalculateStats(definition, materials);
        tool.modifiers = ApplyTraits(materials, definition);
        tool.damage = 0;
        return tool;
    }
    
    private static Dictionary<ToolStatId, float> CalculateStats(
        ToolDefinition definition, List<MaterialId> materials)
    {
        // Implementation as per algorithm above
        var stats = new Dictionary<ToolStatId, float>();
        // ... stat calculation logic
        return stats;
    }
}

// Tool Definition as ScriptableObject
[CreateAssetMenu(fileName = "New Tool", menuName = "Tinker/Tool Definition")]
public class ToolDefinition : ScriptableObject
{
    [SerializeField] private string toolId;
    [SerializeField] private GameObject toolItem;
    [SerializeField] private List<ToolPartDefinition> parts;
    [SerializeField] private List<ToolStatId> stats;
    
    public ToolDefinitionId Id => new ToolDefinitionId(toolId);
    public GameObject ToolItem => toolItem;
    public List<ToolPartDefinition> Parts => parts;
    public List<ToolStatId> Stats => stats;
    
    public ToolStack BuildTool(List<MaterialId> materials)
    {
        if (materials.Count != parts.Count)
            throw new ArgumentException("Material count mismatch");
            
        return ToolStack.Create(this, materials);
    }
}

// Registry as ScriptableObject
[CreateAssetMenu(fileName = "ToolRegistry", 
                 menuName = "Tinker/Registries/Tool Registry")]
public class ToolDefinitionRegistry : ScriptableObject
{
    private static ToolDefinitionRegistry instance;
    public static ToolDefinitionRegistry Instance
    {
        get
        {
            if (instance == null)
            {
                instance = Resources.Load<ToolDefinitionRegistry>(
                    "Registries/ToolRegistry");
            }
            return instance;
        }
    }
    
    [SerializeField] private List<ToolDefinition> definitions;
    private Dictionary<ToolDefinitionId, ToolDefinition> definitionMap;
    
    private void OnEnable()
    {
        BuildMap();
    }
    
    private void BuildMap()
    {
        definitionMap = new Dictionary<ToolDefinitionId, ToolDefinition>();
        foreach (var def in definitions)
        {
            definitionMap[def.Id] = def;
        }
    }
    
    public ToolDefinition Get(ToolDefinitionId id)
    {
        return definitionMap.TryGetValue(id, out var def) ? def : null;
    }
}
```

### NBT Data in Unity

```csharp
// Use JSON serialization for "NBT-like" data
[Serializable]
public class ToolData
{
    public List<string> materials;
    public Dictionary<string, float> stats;
    public List<ModifierData> modifiers;
    public Dictionary<string, string> persistentData;
    public int damage;
    
    public string ToJson()
    {
        return JsonUtility.ToJson(this);
    }
    
    public static ToolData FromJson(string json)
    {
        return JsonUtility.FromJson<ToolData>(json);
    }
}

[Serializable]
public class ModifierData
{
    public string id;
    public int level;
}
```

---

## Testing

```csharp
[Test]
public void ToolStack_Create_CalculatesStatsCorrectly()
{
    // Arrange
    var definition = GetTestDefinition(); // Pickaxe
    var materials = new List<MaterialId>
    {
        new MaterialId("test", "iron"),   // head
        new MaterialId("test", "wood"),   // handle
        new MaterialId("test", "flint")   // binding
    };
    
    // Act
    var tool = ToolStack.Create(definition, materials);
    
    // Assert
    Assert.AreEqual(204, tool.Stats[ToolStats.DURABILITY]);
    Assert.AreEqual(6.0f, tool.Stats[ToolStats.MINING_SPEED], 0.01f);
    Assert.AreEqual(2, tool.Stats[ToolStats.HARVEST_TIER]);
}

[Test]
public void ToolStack_Damage_ReducesDurability()
{
    // Arrange
    var tool = CreateTestTool();
    int initialDamage = tool.Damage;
    
    // Act
    ToolDamageUtil.Damage(tool, 10);
    
    // Assert
    Assert.AreEqual(initialDamage + 10, tool.Damage);
}

[Test]
public void ToolDefinition_BuildTool_ThrowsOnMaterialMismatch()
{
    // Arrange
    var definition = GetTestDefinition(); // Requires 3 parts
    var materials = new List<MaterialId> { new MaterialId("test", "iron") }; // Only 1
    
    // Act & Assert
    Assert.Throws<ArgumentException>(() => definition.BuildTool(materials));
}
```

---

## Performance Considerations

### Optimization Strategies

1. **Stat Caching**: Cache calculated stats in NBT, recalculate only when modifiers change
2. **Material Lookup**: Use dictionaries for O(1) lookups
3. **Lazy Loading**: Load tool definitions on demand
4. **Pool Tool Instances**: Reuse ToolStack objects for frequent operations
5. **Batch Updates**: Update multiple tools at once for network sync

### Memory Management

- Tool Stack: ~500-1000 bytes per instance
- Tool Definition: ~1-2 KB (cached, reused)
- 100 active tools in inventory: ~50-100 KB

---

**Related Diagrams:**
- [Material System](./01-Material-System.md) - Materials used in tools
- [Modifier System](./03-Modifier-System.md) - Modifiers applied to tools
- [NBT Data](./06-Data-Loading-System.md) - Data serialization
