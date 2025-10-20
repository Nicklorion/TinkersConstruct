# Materials System - Technical Documentation

## Overview

The Materials System is the foundation of Tinkers' Construct's tool customization. It defines the properties and behaviors of materials that can be used to craft tools, providing stats and traits that influence tool performance.

## System Architecture

```
MaterialRegistry (Singleton)
    │
    ├─── MaterialManager (Definitions)
    ├─── MaterialStatsManager (Stats)
    └─── MaterialTraitsManager (Traits)
         │
         ├─── Material Instances
         │    ├─── MaterialId (Identifier)
         │    ├─── IMaterial (Interface)
         │    └─── Material Data
         │
         ├─── Material Stats
         │    ├─── MaterialStatsId (Stat Type)
         │    ├─── IMaterialStats (Interface)
         │    └─── Concrete Stats (Head, Handle, etc.)
         │
         └─── Material Traits
              ├─── ModifierId (Trait ID)
              ├─── Level (Trait strength)
              └─── Conditions (When applied)
```

## Core Components

### 1. Material Identity

#### MaterialId
Unique identifier for each material.

```java
public class MaterialId extends ResourceLocation {
    public MaterialId(String namespace, String path) {
        super(namespace, path);
    }
    
    public MaterialId(String location) {
        super(location);
    }
}
```

**Example:**
```java
MaterialId wood = new MaterialId("tconstruct", "wood");
MaterialId iron = new MaterialId("tconstruct", "iron");
MaterialId custom = new MaterialId("mymod", "custom_material");
```

#### IMaterial Interface
```java
public interface IMaterial {
    /** Gets the unique ID of this material */
    MaterialId getId();
    
    /** Gets the tier of this material, for tool mining level */
    int getTier();
    
    /** Gets the sort order for display */
    int getSortOrder();
    
    /** Checks if this material can be used for crafting */
    boolean isCraftable();
    
    /** Gets the redirect target if this is a redirect material */
    @Nullable
    MaterialId getRedirect(IMaterialCondition condition);
}
```

### 2. Material Definitions

Loaded from JSON at `data/<namespace>/tinkering/materials/definition/<material>.json`

**Minimal Material JSON:**
```json
{
    "craftable": true,
    "tier": 1,
    "sortOrder": 100
}
```

**Full Material JSON:**
```json
{
    "craftable": true,
    "tier": 3,
    "sortOrder": 200,
    "hidden": false,
    "redirect": {
        "condition": {
            "type": "tconstruct:config",
            "path": "materials.enableDiamond"
        },
        "id": "tconstruct:diamond_alternative"
    }
}
```

**Fields:**
- `craftable` (boolean): Can this material be used in crafting?
- `tier` (integer): Mining level (0=wood, 1=stone, 2=iron, 3=diamond, 4=netherite)
- `sortOrder` (integer): Display order in material lists
- `hidden` (boolean): Hide from material lists (for internal materials)
- `redirect` (object): Conditional redirect to another material

### 3. Material Stats

Each material can have multiple stat types for different tool parts.

#### Common Stat Types

**HeadMaterialStats** - For tool heads (pickaxe head, axe head, etc.)
```java
public class HeadMaterialStats implements IMaterialStats {
    private final int durability;
    private final float miningSpeed;
    private final float miningTier;
    private final float attack;
    
    public static final MaterialStatsId ID = 
        new MaterialStatsId("tconstruct", "head");
}
```

**HandleMaterialStats** - For tool handles
```java
public class HandleMaterialStats implements IMaterialStats {
    private final float durability;  // Multiplier
    private final float miningSpeed; // Multiplier
    private final float attack;      // Multiplier
    
    public static final MaterialStatsId ID = 
        new MaterialStatsId("tconstruct", "handle");
}
```

**ExtraMaterialStats** - For extra/binding parts
```java
public class ExtraMaterialStats implements IMaterialStats {
    // No stats, just exists for traits
    
    public static final MaterialStatsId ID = 
        new MaterialStatsId("tconstruct", "extra");
}
```

#### Material Stats JSON

Located at: `data/<namespace>/tinkering/materials/stats/<stat_type>/<material>.json`

**Example - Head Stats:**
`data/tconstruct/tinkering/materials/stats/tconstruct/head/iron.json`
```json
{
    "durability": 204,
    "mining_speed": 6.0,
    "mining_tier": 2,
    "attack": 2.0
}
```

**Example - Handle Stats:**
`data/tconstruct/tinkering/materials/stats/tconstruct/handle/wood.json`
```json
{
    "durability": 1.0,
    "miningSpeed": 1.0,
    "attack": 1.0
}
```

### 4. Material Traits

Traits are special abilities granted by materials. They are actually modifiers applied automatically.

#### Material Traits JSON

Located at: `data/<namespace>/tinkering/materials/traits/<material>.json`

**Simple Trait Assignment:**
```json
{
    "default": [
        {
            "name": "tconstruct:cheap",
            "level": 1
        }
    ]
}
```

**Per-Stat-Type Traits:**
```json
{
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

**Conditional Traits:**
```json
{
    "default": [
        {
            "name": "tconstruct:lucky",
            "level": 1
        }
    ],
    "optional": {
        "condition": {
            "type": "tconstruct:stat_type",
            "stats": ["tconstruct:head"]
        },
        "traits": [
            {
                "name": "tconstruct:fortune",
                "level": 1
            }
        ]
    }
}
```

## Material Registry

### MaterialRegistry Class

```java
public final class MaterialRegistry {
    static MaterialRegistry INSTANCE;
    
    private final MaterialManager materialManager;
    private final MaterialStatsManager materialStatsManager;
    private final MaterialTraitsManager materialTraitsManager;
    private final IMaterialRegistry registry;
    
    public static IMaterialRegistry getInstance() {
        return INSTANCE.registry;
    }
    
    public static void init() {
        INSTANCE = new MaterialRegistry();
        // Setup event listeners
    }
}
```

### IMaterialRegistry Interface

Public API for accessing materials:

```java
public interface IMaterialRegistry {
    /** Gets all registered materials */
    Collection<IMaterial> getAllMaterials();
    
    /** Gets a material by ID */
    @Nullable
    IMaterial getMaterial(MaterialId id);
    
    /** Gets material stats for a stat type */
    @Nullable
    IMaterialStats getMaterialStats(MaterialId id, MaterialStatsId statsId);
    
    /** Gets traits for a material and stat type */
    List<ModifierEntry> getTraits(MaterialId id, MaterialStatsId statsId);
}
```

## Data Loading Pipeline

### 1. Server Startup

```
Server Start
    ↓
FMLServerStartingEvent
    ↓
AddReloadListenerEvent
    ↓
Register Material Managers
    ├─ MaterialManager (definitions)
    ├─ MaterialStatsManager (stats)
    └─ MaterialTraitsManager (traits)
    ↓
Load Datapacks
    ↓
Parse JSON Files
    ├─ Validate schema
    ├─ Parse data
    └─ Handle errors
    ↓
Build Registry
    ├─ Create material instances
    ├─ Link stats to materials
    └─ Link traits to materials
    ↓
Fire MaterialsLoadedEvent
```

### 2. Client Synchronization

```
Player Joins Server
    ↓
OnDatapackSyncEvent
    ↓
Create Sync Packets
    ├─ UpdateMaterialsPacket
    ├─ UpdateMaterialStatsPacket
    └─ UpdateMaterialTraitsPacket
    ↓
Send to Client
    ↓
Client Receives Packets
    ↓
Update Client Registry
    ↓
Mark As Loaded
```

### 3. Datapack Reload

```
/reload Command
    ↓
Clear Registries
    ↓
Re-load JSON Data
    ↓
Rebuild Registries
    ↓
Sync to All Clients
```

## Material Rendering

### MaterialRenderInfo

Defines how a material looks:

```java
public class MaterialRenderInfo {
    private final int color;
    private final int luminosity;
    private final String texture;
    
    public int getVertexColor(int tintIndex) {
        return color;
    }
}
```

### Material Colors JSON

Located at: `data/<namespace>/tinkering/materials/render_info/<material>.json`

```json
{
    "color": "FF0000",
    "luminosity": 0,
    "texture": "tconstruct:block/seared_brick"
}
```

### Dynamic Textures

Materials can use:
1. **Solid Color**: Tint base texture
2. **Texture Reference**: Use specific texture
3. **Gradient**: Multiple colors for different parts
4. **Fallback**: Default if not specified

## Material Variants

Some materials have variants (e.g., different wood types):

```java
public class MaterialVariantId {
    private final MaterialId material;
    private final String variant;
    
    public MaterialVariantId(MaterialId material, String variant) {
        this.material = material;
        this.variant = variant;
    }
}
```

**Example:**
- `tconstruct:wood` (base material)
  - Variant: `oak`
  - Variant: `birch`
  - Variant: `spruce`

## Stat Calculation

### Base Stats from Material

When building a tool:
1. Get material for each part
2. Get stats for material + stat type
3. Combine stats based on tool definition
4. Apply material traits

### Example Calculation

**Pickaxe with:**
- Head: Iron
- Handle: Wood
- Binding: Flint

```
Durability = head.durability * handle.durability_multiplier
           = 204 * 1.0
           = 204

Mining Speed = head.mining_speed * handle.mining_speed_multiplier
             = 6.0 * 1.0
             = 6.0

Attack = head.attack * handle.attack_multiplier
       = 2.0 * 1.0
       = 2.0
```

## Material Predicates

Materials can be tested with predicates:

### Common Predicates

**MaterialPredicate:**
```java
public interface MaterialPredicate extends IJsonPredicate<MaterialId> {
    boolean matches(MaterialId material);
}
```

**Examples:**
- `material`: Matches specific material
- `tier`: Matches materials of a tier
- `tag`: Matches materials in a tag
- `stat_type`: Matches materials with a stat type

## Error Handling

### Missing Material
If a material is referenced but doesn't exist:
1. Log error
2. Use fallback material (often wood)
3. Continue operation

### Invalid Stats
If stats are invalid:
1. Log warning
2. Use default values
3. Material may not be craftable

### Circular Redirects
If material redirects form a loop:
1. Detect cycle
2. Break at detection point
3. Use original material

## Performance Considerations

### Caching
- Material instances cached after load
- Stats cached per material+type combination
- Render info cached
- Trait lists cached

### Lazy Loading
- Materials loaded only on server start
- Client receives only used materials
- Render info loaded on-demand

### Memory
- Material data is small (<1KB per material)
- Registry overhead minimal
- Total for ~100 materials: <100KB

## Events

### MaterialsLoadedEvent
Fired when materials finish loading:

```java
@SubscribeEvent
public void onMaterialsLoaded(MaterialsLoadedEvent event) {
    // All materials are now available
    IMaterialRegistry registry = event.getRegistry();
    // Can now safely query materials
}
```

## Extension Points

### Adding Materials (Datapack)
1. Create material definition JSON
2. Add stat JSONs for each stat type
3. Add trait JSON
4. Add render info JSON
5. Material automatically available

### Adding Materials (Code)
1. Register material in event
2. Provide stats programmatically
3. Assign traits programmatically
4. Less flexible than datapack

### Custom Stat Types
1. Create IMaterialStats implementation
2. Register MaterialStatsId
3. Create JSON loader
4. Use in tool definitions

## Navigation

- [Next: Tools System →](./02-Tools-System.md)
- [Back to Technical Documentation ↑](./README.md)
