# Class Diagram: Material System

## Overview

The Material System manages all materials available for crafting tools. It handles material definitions, stats, traits, rendering information, and provides a centralized registry for material lookup.

---

## Core Class Diagram

```
                                    <<interface>>
                                 ┌──────────────────┐
                                 │   IMaterial      │
                                 ├──────────────────┤
                                 │ + getId()        │
                                 │ + getTier()      │
                                 │ + isCraftable()  │
                                 │ + getSortOrder() │
                                 │ + isHidden()     │
                                 └──────────────────┘
                                          △
                                          │ implements
                                          │
                                 ┌──────────────────┐
                                 │   Material       │
                                 ├──────────────────┤
                                 │ - id: MaterialId │
                                 │ - tier: int      │
                                 │ - craftable: bool│
                                 │ - sortOrder: int │
                                 │ - hidden: bool   │
                                 │ - redirect: ...  │
                                 ├──────────────────┤
                                 │ + Material()     │
                                 │ + getId()        │
                                 │ + getTier()      │
                                 │ + isCraftable()  │
                                 │ + getRedirect()  │
                                 └──────────────────┘
                                          ◇
                                          │ uses
                                          │
                                 ┌──────────────────┐
                                 │   MaterialId     │
                                 ├──────────────────┤
                                 │ - namespace: str │
                                 │ - path: str      │
                                 ├──────────────────┤
                                 │ + toString()     │
                                 │ + equals()       │
                                 │ + hashCode()     │
                                 └──────────────────┘
```

---

## Material Stats System

```
                         <<interface>>
                      ┌─────────────────┐
                      │ IMaterialStats  │
                      ├─────────────────┤
                      │ + getStatsId()  │
                      │ + serialize()   │
                      └─────────────────┘
                              △
                              │ implements
                 ┌────────────┼────────────┐
                 │            │            │
        ┌────────────┐ ┌─────────────┐ ┌──────────────┐
        │   Head     │ │   Handle    │ │    Extra     │
        │MaterialStats│ │MaterialStats│ │MaterialStats │
        ├────────────┤ ├─────────────┤ ├──────────────┤
        │-durability │ │-durability  │ │(no stats)    │
        │-miningSpeed│ │-miningSpeed │ │              │
        │-miningTier │ │-attack      │ │              │
        │-attack     │ │             │ │              │
        ├────────────┤ ├─────────────┤ ├──────────────┤
        │+ getStatsId│ │+ getStatsId │ │+ getStatsId  │
        │+ serialize │ │+ serialize  │ │+ serialize   │
        └────────────┘ └─────────────┘ └──────────────┘

                     <<value object>>
                  ┌──────────────────┐
                  │ MaterialStatsId  │
                  ├──────────────────┤
                  │ - namespace: str │
                  │ - path: str      │
                  ├──────────────────┤
                  │ + toString()     │
                  │ + equals()       │
                  └──────────────────┘
```

**Material Stats Types:**

| Stat Type | Used For | Properties |
|-----------|----------|------------|
| Head | Tool heads (primary working part) | durability, miningSpeed, miningTier, attack |
| Handle | Tool handles (grip and control) | durability multiplier, speed multiplier, attack multiplier |
| Extra | Binding/extra parts | No stats (only traits) |
| Bowstring | Bow strings | draw speed, arrow damage multiplier |
| Armor | Armor pieces | protection, toughness, durability |
| Limb | Bow limbs | draw speed, accuracy, velocity |

---

## Material Traits System

```
                    <<interface>>
                 ┌─────────────────┐
                 │  IMaterialTrait │
                 ├─────────────────┤
                 │ + getModifier() │
                 │ + getLevel()    │
                 └─────────────────┘
                         △
                         │ implements
                         │
                 ┌─────────────────┐
                 │  MaterialTrait  │
                 ├─────────────────┤
                 │ - modifier: ID  │
                 │ - level: int    │
                 │ - condition: .. │
                 ├─────────────────┤
                 │ + getModifier() │
                 │ + getLevel()    │
                 │ + matches()     │
                 └─────────────────┘
                         ◇
                         │ references
                         │
                  ┌──────────────┐
                  │  ModifierId  │
                  ├──────────────┤
                  │ - namespace  │
                  │ - path       │
                  ├──────────────┤
                  │ + toString() │
                  └──────────────┘

        <<manager>>
    ┌──────────────────────┐
    │MaterialTraitsManager │
    ├──────────────────────┤
    │ - traits: Map<...>   │
    ├──────────────────────┤
    │ + load()             │
    │ + getTraits()        │
    │ + getDefaultTraits() │
    │ + getStatTraits()    │
    └──────────────────────┘
```

**Trait Assignment:**
- **Default Traits**: Applied to all uses of the material
- **Per-Stat Traits**: Applied only when material is used for specific part type
- **Conditional Traits**: Applied only when conditions are met

---

## Material Registry System

```
                          <<singleton>>
                    ┌──────────────────────┐
                    │  MaterialRegistry    │
                    ├──────────────────────┤
                    │ - INSTANCE: static   │
                    │ - materials: Map     │
                    │ - stats: Map         │
                    │ - traits: Map        │
                    ├──────────────────────┤
                    │ + getInstance()      │
                    │ + getMaterial()      │
                    │ + getStats()         │
                    │ + getTraits()        │
                    │ + getAllMaterials()  │
                    └──────────────────────┘
                              │
                              │ delegates to
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌────────────────┐  ┌───────────────────┐  ┌──────────────────┐
│MaterialManager │  │MaterialStatsManager│  │MaterialTraits    │
│                │  │                    │  │Manager           │
├────────────────┤  ├───────────────────┤  ├──────────────────┤
│- materials:Map │  │- stats: Map       │  │- traits: Map     │
├────────────────┤  ├───────────────────┤  ├──────────────────┤
│+ load()        │  │+ load()           │  │+ load()          │
│+ get()         │  │+ get()            │  │+ get()           │
│+ getAll()      │  │+ getAll()         │  │+ getDefault()    │
│+ clear()       │  │+ clear()          │  │+ getForStat()    │
└────────────────┘  └───────────────────┘  └──────────────────┘
        │                     │                     │
        │                     │                     │
        └─────────────────────┴─────────────────────┘
                              │
                              │ loads from
                              │
                    ┌──────────────────┐
                    │   JSON Files     │
                    │   (Datapacks)    │
                    ├──────────────────┤
                    │ definition/      │
                    │ stats/           │
                    │ traits/          │
                    │ render_info/     │
                    └──────────────────┘
```

---

## Material Rendering System

```
                  ┌──────────────────────┐
                  │ MaterialRenderInfo   │
                  ├──────────────────────┤
                  │ - color: int         │
                  │ - luminosity: int    │
                  │ - texture: string    │
                  │ - fallback: ...      │
                  ├──────────────────────┤
                  │ + getColor()         │
                  │ + getLuminosity()    │
                  │ + getTexture()       │
                  │ + getFallback()      │
                  └──────────────────────┘
                            △
                            │ extends
                            │
            ┌───────────────┴───────────────┐
            │                               │
    ┌──────────────────┐          ┌────────────────────┐
    │  GradientRender  │          │  TextureRender     │
    │  Info            │          │  Info              │
    ├──────────────────┤          ├────────────────────┤
    │ - colors: List   │          │ - textureMap: Map  │
    │ - transitions:..│          │ - sprites: List    │
    ├──────────────────┤          ├────────────────────┤
    │ + getColor()     │          │ + getTexture()     │
    └──────────────────┘          └────────────────────┘
```

---

## Data Loading Flow

```
┌────────────────────────────────────────────────────────┐
│                    Server Start                        │
└────────────────┬───────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────┐
│           MaterialManager.load()                       │
│   - Scans data/*/tinkering/materials/definition/      │
│   - Parses JSON files                                  │
│   - Creates Material instances                         │
└────────────────┬───────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────┐
│      MaterialStatsManager.load()                       │
│   - Scans data/*/tinkering/materials/stats/           │
│   - Parses JSON for each stat type                    │
│   - Links stats to materials                          │
└────────────────┬───────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────┐
│      MaterialTraitsManager.load()                      │
│   - Scans data/*/tinkering/materials/traits/          │
│   - Parses trait assignments                          │
│   - Links traits to materials                         │
└────────────────┬───────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────┐
│           MaterialRenderInfoManager.load()             │
│   - Scans data/*/tinkering/materials/render_info/     │
│   - Parses rendering data                             │
│   - Links render info to materials                    │
└────────────────┬───────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────┐
│           Build Complete Material Registry             │
│   - All materials loaded                               │
│   - All stats linked                                   │
│   - All traits linked                                  │
│   - Ready for tool creation                            │
└────────────────────────────────────────────────────────┘
```

---

## Key Methods and Interactions

### MaterialRegistry

```java
// Singleton access
public static IMaterialRegistry getInstance()

// Material lookup
public IMaterial getMaterial(MaterialId id)
public Collection<IMaterial> getAllMaterials()
public Collection<IMaterial> getCraftableMaterials()

// Stats lookup
public IMaterialStats getStats(MaterialId material, MaterialStatsId statsId)
public Collection<IMaterialStats> getAllStats(MaterialId material)

// Traits lookup
public List<ModifierEntry> getTraits(MaterialId material, MaterialStatsId statsId)
public List<ModifierEntry> getDefaultTraits(MaterialId material)
```

### Material

```java
// Getters
public MaterialId getId()
public int getTier()
public int getSortOrder()
public boolean isCraftable()
public boolean isHidden()

// Redirects (for conditional materials)
public MaterialId getRedirect(IMaterialCondition condition)
public boolean hasRedirect()
```

### MaterialStats (example: HeadMaterialStats)

```java
// Stats
public int getDurability()
public float getMiningSpeed()
public int getMiningTier()
public float getAttack()

// Serialization
public MaterialStatsId getStatsId()
public JsonObject serialize()
public static HeadMaterialStats deserialize(JsonObject json)
```

---

## JSON Data Structures

### Material Definition
**File:** `data/<namespace>/tinkering/materials/definition/<material_id>.json`

```json
{
  "craftable": true,
  "tier": 2,
  "sortOrder": 150,
  "hidden": false,
  "redirect": {
    "condition": {
      "type": "tconstruct:config",
      "path": "materials.enableBronze"
    },
    "id": "tconstruct:copper"
  }
}
```

### Material Stats (Head Example)
**File:** `data/<namespace>/tinkering/materials/stats/tconstruct/head/<material_id>.json`

```json
{
  "durability": 250,
  "mining_speed": 7.0,
  "mining_tier": 2,
  "attack": 2.5
}
```

### Material Traits
**File:** `data/<namespace>/tinkering/materials/traits/<material_id>.json`

```json
{
  "default": [
    {
      "name": "tconstruct:cheap",
      "level": 1
    }
  ],
  "per_stat": {
    "tconstruct:head": [
      {
        "name": "tconstruct:jagged",
        "level": 2
      }
    ],
    "tconstruct:handle": [
      {
        "name": "tconstruct:maintained",
        "level": 1
      }
    ]
  }
}
```

### Material Render Info
**File:** `data/<namespace>/tinkering/materials/render_info/<material_id>.json`

```json
{
  "color": "A97A4D",
  "luminosity": 0,
  "texture": "tconstruct:block/bronze_block"
}
```

---

## C# Implementation Notes

### Key Differences in C#

```csharp
// Interface naming convention
public interface IMaterial { }

// Properties instead of getters
public class Material : IMaterial
{
    public MaterialId Id { get; private set; }
    public int Tier { get; private set; }
    public bool IsCraftable { get; private set; }
    
    // Constructor
    public Material(MaterialId id, int tier, bool craftable)
    {
        Id = id;
        Tier = tier;
        IsCraftable = craftable;
    }
}

// Singleton in C# (Unity-friendly)
public class MaterialRegistry : ScriptableObject
{
    private static MaterialRegistry instance;
    public static MaterialRegistry Instance
    {
        get
        {
            if (instance == null)
            {
                instance = Resources.Load<MaterialRegistry>("Registries/MaterialRegistry");
            }
            return instance;
        }
    }
    
    [SerializeField]
    private List<Material> materials = new List<Material>();
    
    public IMaterial GetMaterial(MaterialId id)
    {
        return materials.Find(m => m.Id.Equals(id));
    }
}

// Dictionary usage
private Dictionary<MaterialId, IMaterial> materialMap = 
    new Dictionary<MaterialId, IMaterial>();

// JSON deserialization in Unity
[Serializable]
public class MaterialData
{
    public bool craftable;
    public int tier;
    public int sortOrder;
    public bool hidden;
}

// Load from JSON
Material LoadMaterial(string json)
{
    MaterialData data = JsonUtility.FromJson<MaterialData>(json);
    // Create Material from data
}
```

### Unity ScriptableObject Pattern

```csharp
[CreateAssetMenu(fileName = "New Material", menuName = "Tinker/Material")]
public class MaterialAsset : ScriptableObject, IMaterial
{
    [SerializeField] private string materialId;
    [SerializeField] private int tier;
    [SerializeField] private bool craftable;
    [SerializeField] private int sortOrder;
    
    public MaterialId Id => new MaterialId(materialId);
    public int Tier => tier;
    public bool IsCraftable => craftable;
    public int SortOrder => sortOrder;
}
```

---

## Testing Considerations

### Unit Tests

```csharp
[Test]
public void MaterialRegistry_GetMaterial_ReturnsMaterial()
{
    // Arrange
    var registry = MaterialRegistry.Instance;
    var ironId = new MaterialId("tconstruct", "iron");
    
    // Act
    var material = registry.GetMaterial(ironId);
    
    // Assert
    Assert.IsNotNull(material);
    Assert.AreEqual(ironId, material.Id);
}

[Test]
public void Material_GetStats_ReturnsCorrectStats()
{
    // Arrange
    var registry = MaterialRegistry.Instance;
    var ironId = new MaterialId("tconstruct", "iron");
    var headStatsId = new MaterialStatsId("tconstruct", "head");
    
    // Act
    var stats = registry.GetStats(ironId, headStatsId);
    
    // Assert
    Assert.IsNotNull(stats);
    Assert.IsInstanceOf<HeadMaterialStats>(stats);
}
```

### Integration Tests

```csharp
[Test]
public void MaterialSystem_LoadFromJSON_PopulatesRegistry()
{
    // Arrange
    string json = File.ReadAllText("test_materials.json");
    var loader = new MaterialLoader();
    
    // Act
    loader.Load(json);
    
    // Assert
    var registry = MaterialRegistry.Instance;
    Assert.Greater(registry.GetAllMaterials().Count, 0);
}
```

---

## Performance Considerations

### Caching
- Material instances cached after load
- Stats cached per material+stat type combination
- Trait lookups cached
- Render info cached

### Memory
- Each material: ~100-500 bytes
- Total for 100 materials: ~10-50 KB
- Minimal overhead

### Optimization Tips
1. Use dictionary/hashmap for O(1) lookup
2. Cache stat calculations
3. Lazy-load render info
4. Pre-compute trait lists

---

**Related Diagrams:**
- [Tool System](./02-Tool-System.md) - Uses materials for tool creation
- [Modifier System](./03-Modifier-System.md) - Traits are modifiers
- [Data Loading System](./06-Data-Loading-System.md) - How materials are loaded
