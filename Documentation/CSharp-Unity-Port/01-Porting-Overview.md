# Porting Overview - C# Unity Implementation

## Introduction

This document provides a comprehensive overview of porting Tinkers' Construct from Java/Minecraft to C#/Unity. It outlines the scope, strategy, challenges, and roadmap for creating a feature-complete implementation in Unity.

## Why Port to Unity?

### Benefits
1. **Cross-Platform**: Unity supports PC, mobile, consoles
2. **Modern Engine**: Advanced graphics, physics, and audio
3. **Flexible Gameplay**: Not limited by Minecraft mechanics
4. **Better Performance**: Direct control over optimization
5. **Easier Modding**: C# is more accessible than Java
6. **Custom UI**: Full control over user interface

### Challenges
1. **Different Architecture**: Unity vs Minecraft is fundamentally different
2. **No Block System**: Need to create voxel/block system or adapt
3. **Different Physics**: Unity physics vs Minecraft's simpler model
4. **Networking**: Unity netcode vs Forge networking
5. **Asset Pipeline**: Different resource management
6. **Save System**: Need custom serialization

## Scope Definition

### Core Features to Port

#### 1. Materials System ✓
- Material definitions
- Material stats
- Material traits
- Material rendering
- Material registry

#### 2. Tools System ✓
- Tool definitions
- Tool assembly
- Tool parts
- Tool stats calculation
- Tool durability
- Tool rendering

#### 3. Modifiers System ✓
- Modifier definitions
- Modifier application
- Modifier effects
- Modifier levels
- Modifier slots
- Modifier persistence

#### 4. Crafting System ✓
- Part Builder equivalent
- Tinker Station equivalent
- Modifier Worktable equivalent
- Tool repair system
- Part swapping

#### 5. Data System ✓
- JSON data loading
- Datapack system
- Material data
- Tool data
- Recipe data

### Optional Features

#### Smeltery System ?
- Requires voxel/multiblock system
- May need significant adaptation
- Could be simplified or replaced

#### World Generation ?
- Ore spawning
- Structure generation
- Depends on game world type

#### Multiplayer ✓ (Recommended)
- Client-server architecture
- Data synchronization
- Inventory sync

## Architecture Translation

### Java/Forge to C#/Unity Mapping

| Java/Minecraft Concept | Unity Equivalent | Implementation Strategy |
|------------------------|------------------|-------------------------|
| Mod Class | MonoBehaviour Manager | Singleton manager scripts |
| Forge Registry | ScriptableObject Registry | Asset-based registries |
| ItemStack | Item Instance Class | C# class with data |
| NBT Data | Serializable Class | JSON or binary serialization |
| Datapack | Addressables/Resources | Unity asset loading |
| Event Bus | C# Events/UnityEvents | Delegate-based events |
| Packet System | RPC/Network Messages | Unity Netcode |
| Block Entity | MonoBehaviour | Component on GameObject |
| Recipe Manager | Recipe System | ScriptableObject-based |
| Texture Atlas | Sprite Atlas | Unity sprite system |

### Architectural Differences

#### Java (Static/Singleton Heavy)
```java
public class MaterialRegistry {
    private static MaterialRegistry INSTANCE;
    
    public static MaterialRegistry getInstance() {
        return INSTANCE;
    }
}
```

#### C# Unity (ScriptableObject/Singleton Hybrid)
```csharp
// Option 1: ScriptableObject Singleton
[CreateAssetMenu(fileName = "MaterialRegistry", 
                 menuName = "TConstruct/Material Registry")]
public class MaterialRegistry : ScriptableObject {
    private static MaterialRegistry _instance;
    
    public static MaterialRegistry Instance {
        get {
            if (_instance == null) {
                _instance = Resources.Load<MaterialRegistry>(
                    "MaterialRegistry");
            }
            return _instance;
        }
    }
    
    [SerializeField] 
    private List<Material> materials = new List<Material>();
}

// Option 2: MonoBehaviour Manager
public class MaterialManager : MonoBehaviour {
    public static MaterialManager Instance { get; private set; }
    
    void Awake() {
        if (Instance == null) {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        } else {
            Destroy(gameObject);
        }
    }
}
```

## Core Systems Implementation

### 1. Material System in Unity

#### Material ScriptableObject
```csharp
[CreateAssetMenu(fileName = "New Material", 
                 menuName = "TConstruct/Material")]
public class Material : ScriptableObject {
    [SerializeField] private string materialId;
    [SerializeField] private string displayName;
    [SerializeField] private int tier;
    [SerializeField] private int sortOrder;
    [SerializeField] private bool craftable = true;
    
    [Header("Stats")]
    [SerializeField] private MaterialStats headStats;
    [SerializeField] private MaterialStats handleStats;
    [SerializeField] private MaterialStats extraStats;
    
    [Header("Traits")]
    [SerializeField] private List<ModifierEntry> defaultTraits;
    
    [Header("Rendering")]
    [SerializeField] private Color materialColor = Color.white;
    [SerializeField] private Texture2D materialTexture;
    
    public string Id => materialId;
    public string DisplayName => displayName;
    public int Tier => tier;
    public bool IsCraftable => craftable;
    
    public MaterialStats GetStats(MaterialStatType type) {
        return type switch {
            MaterialStatType.Head => headStats,
            MaterialStatType.Handle => handleStats,
            MaterialStatType.Extra => extraStats,
            _ => null
        };
    }
}

[System.Serializable]
public class MaterialStats {
    public float durability;
    public float miningSpeed;
    public float miningTier;
    public float attack;
    
    // Constructor
    public MaterialStats(float durability, float miningSpeed, 
                        float miningTier, float attack) {
        this.durability = durability;
        this.miningSpeed = miningSpeed;
        this.miningTier = miningTier;
        this.attack = attack;
    }
}

public enum MaterialStatType {
    Head,
    Handle,
    Extra,
    Bowstring,
    Grip,
    Plating
}
```

### 2. Tool System in Unity

#### Tool Data Class
```csharp
[System.Serializable]
public class ToolData {
    public string toolId;
    public List<MaterialSlot> materials;
    public ToolStats stats;
    public List<ModifierEntry> modifiers;
    public int currentDurability;
    
    [System.Serializable]
    public class MaterialSlot {
        public string partName;
        public string materialId;
        public MaterialStatType statType;
    }
    
    public ToolData(string toolId) {
        this.toolId = toolId;
        materials = new List<MaterialSlot>();
        modifiers = new List<ModifierEntry>();
    }
    
    public void CalculateStats(ToolDefinition definition) {
        stats = new ToolStats();
        
        foreach (var slot in materials) {
            var material = MaterialRegistry.Instance.GetMaterial(
                slot.materialId);
            var materialStats = material.GetStats(slot.statType);
            
            // Combine stats based on definition
            stats.durability += materialStats.durability;
            stats.miningSpeed += materialStats.miningSpeed;
            // etc...
        }
        
        currentDurability = (int)stats.durability;
    }
}

[System.Serializable]
public class ToolStats {
    public float durability;
    public float miningSpeed;
    public float attack;
    public int harvestLevel;
    
    // Apply modifiers
    public void ApplyModifier(Modifier modifier, int level) {
        modifier.ModifyStats(this, level);
    }
}
```

#### Tool Item Component
```csharp
public class ToolItem : MonoBehaviour {
    [SerializeField] private ToolData toolData;
    [SerializeField] private MeshFilter meshFilter;
    [SerializeField] private MeshRenderer meshRenderer;
    
    public ToolData Data => toolData;
    
    public void Initialize(ToolData data) {
        toolData = data;
        GenerateMesh();
        ApplyMaterials();
    }
    
    private void GenerateMesh() {
        // Combine meshes from parts
        var toolDef = ToolRegistry.Instance.GetDefinition(
            toolData.toolId);
        
        List<CombineInstance> combines = new List<CombineInstance>();
        
        foreach (var part in toolDef.parts) {
            var partMesh = part.GetMesh();
            CombineInstance ci = new CombineInstance();
            ci.mesh = partMesh;
            ci.transform = Matrix4x4.identity;
            combines.Add(ci);
        }
        
        Mesh combinedMesh = new Mesh();
        combinedMesh.CombineMeshes(combines.ToArray(), true, false);
        meshFilter.mesh = combinedMesh;
    }
    
    private void ApplyMaterials() {
        // Apply material colors using shader
        Material renderMaterial = meshRenderer.material;
        
        foreach (var matSlot in toolData.materials) {
            var material = MaterialRegistry.Instance.GetMaterial(
                matSlot.materialId);
            Color color = material.GetColor();
            
            // Set shader properties per part
            renderMaterial.SetColor($"_Color_{matSlot.partName}", color);
        }
    }
    
    public void UseTool(int damage = 1) {
        toolData.currentDurability -= damage;
        
        if (toolData.currentDurability <= 0) {
            OnToolBroken();
        }
    }
    
    private void OnToolBroken() {
        // Handle tool breaking
        Debug.Log($"Tool {toolData.toolId} has broken!");
        // Could reduce stats, require repair, etc.
    }
}
```

### 3. Modifier System in Unity

#### Modifier ScriptableObject
```csharp
[CreateAssetMenu(fileName = "New Modifier", 
                 menuName = "TConstruct/Modifier")]
public abstract class Modifier : ScriptableObject {
    [SerializeField] private string modifierId;
    [SerializeField] private string displayName;
    [SerializeField] private int maxLevel = 5;
    [SerializeField] private int slotsRequired = 1;
    
    public string Id => modifierId;
    public string DisplayName => displayName;
    public int MaxLevel => maxLevel;
    
    // Override in subclasses
    public abstract void ModifyStats(ToolStats stats, int level);
    
    public virtual void OnBlockMined(ToolItem tool, int level, 
                                    GameObject block) { }
    
    public virtual void OnEntityHit(ToolItem tool, int level, 
                                   GameObject entity) { }
    
    public virtual float ModifyDamage(float baseDamage, int level) {
        return baseDamage;
    }
}

// Example concrete modifier
[CreateAssetMenu(fileName = "Sharpness", 
                 menuName = "TConstruct/Modifiers/Sharpness")]
public class SharpnessModifier : Modifier {
    [SerializeField] private float damagePerLevel = 0.5f;
    
    public override void ModifyStats(ToolStats stats, int level) {
        stats.attack += damagePerLevel * level;
    }
    
    public override float ModifyDamage(float baseDamage, int level) {
        return baseDamage + (damagePerLevel * level);
    }
}

[System.Serializable]
public class ModifierEntry {
    public string modifierId;
    public int level;
    
    public ModifierEntry(string id, int level) {
        this.modifierId = id;
        this.level = level;
    }
}
```

### 4. Data Loading System

#### JSON Data Loader
```csharp
public class DataLoader : MonoBehaviour {
    public static DataLoader Instance { get; private set; }
    
    void Awake() {
        if (Instance == null) {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
    }
    
    public async Task LoadAllData() {
        await LoadMaterials();
        await LoadTools();
        await LoadModifiers();
        await LoadRecipes();
    }
    
    private async Task LoadMaterials() {
        // Load from StreamingAssets or Addressables
        string dataPath = Path.Combine(
            Application.streamingAssetsPath, 
            "data/materials");
        
        if (Directory.Exists(dataPath)) {
            var files = Directory.GetFiles(dataPath, "*.json");
            
            foreach (var file in files) {
                string json = await File.ReadAllTextAsync(file);
                MaterialData data = JsonUtility.FromJson<MaterialData>(json);
                
                // Create ScriptableObject or register data
                Material material = CreateMaterialFromData(data);
                MaterialRegistry.Instance.Register(material);
            }
        }
    }
    
    private Material CreateMaterialFromData(MaterialData data) {
        Material material = ScriptableObject.CreateInstance<Material>();
        // Set properties from data
        // Use reflection or manual assignment
        return material;
    }
}

[System.Serializable]
public class MaterialData {
    public string id;
    public string displayName;
    public int tier;
    public bool craftable;
    public HeadStatsData headStats;
    public HandleStatsData handleStats;
    public TraitData[] traits;
}
```

## Modding API Design

### Mod Interface
```csharp
public interface ITConstructMod {
    string ModId { get; }
    string ModName { get; }
    string Version { get; }
    
    void OnPreLoad();
    void OnLoad();
    void OnPostLoad();
    
    void RegisterMaterials(MaterialRegistry registry);
    void RegisterTools(ToolRegistry registry);
    void RegisterModifiers(ModifierRegistry registry);
}
```

### Mod Loader
```csharp
public class ModLoader : MonoBehaviour {
    private List<ITConstructMod> loadedMods = new List<ITConstructMod>();
    
    public void LoadModsFromDirectory(string modDirectory) {
        var dllFiles = Directory.GetFiles(modDirectory, "*.dll");
        
        foreach (var dll in dllFiles) {
            try {
                Assembly assembly = Assembly.LoadFrom(dll);
                var modTypes = assembly.GetTypes()
                    .Where(t => typeof(ITConstructMod).IsAssignableFrom(t) 
                           && !t.IsInterface)
                    .ToList();
                
                foreach (var modType in modTypes) {
                    ITConstructMod mod = (ITConstructMod)
                        Activator.CreateInstance(modType);
                    
                    mod.OnPreLoad();
                    loadedMods.Add(mod);
                    
                    Debug.Log($"Loaded mod: {mod.ModName} v{mod.Version}");
                }
            }
            catch (Exception e) {
                Debug.LogError($"Failed to load mod from {dll}: {e}");
            }
        }
        
        // Call OnLoad for all mods
        foreach (var mod in loadedMods) {
            mod.OnLoad();
            mod.RegisterMaterials(MaterialRegistry.Instance);
            mod.RegisterTools(ToolRegistry.Instance);
            mod.RegisterModifiers(ModifierRegistry.Instance);
        }
        
        // Call OnPostLoad
        foreach (var mod in loadedMods) {
            mod.OnPostLoad();
        }
    }
}
```

## Development Roadmap

### Phase 1: Foundation (2-4 weeks)
- [ ] Set up Unity project
- [ ] Create core data structures
- [ ] Implement material system
- [ ] Implement tool data system
- [ ] Basic JSON loading
- [ ] Material registry

### Phase 2: Tool System (3-6 weeks)
- [ ] Tool definition system
- [ ] Tool assembly logic
- [ ] Tool stats calculation
- [ ] Tool rendering
- [ ] Basic tool usage
- [ ] Durability system

### Phase 3: Modifiers (2-4 weeks)
- [ ] Modifier system
- [ ] Modifier application
- [ ] Modifier hooks/events
- [ ] Example modifiers
- [ ] Modifier UI

### Phase 4: UI System (3-5 weeks)
- [ ] Crafting station UI
- [ ] Part builder UI
- [ ] Modifier table UI
- [ ] Tool tooltip display
- [ ] Inventory integration

### Phase 5: Data & Assets (2-3 weeks)
- [ ] JSON schemas
- [ ] Data validation
- [ ] Asset pipeline
- [ ] Material textures
- [ ] Tool models

### Phase 6: Modding API (2-4 weeks)
- [ ] Mod loader
- [ ] API documentation
- [ ] Example mods
- [ ] Mod tools

### Phase 7: Polish (2-4 weeks)
- [ ] Performance optimization
- [ ] Bug fixes
- [ ] Testing
- [ ] Documentation

### Phase 8: Multiplayer (3-6 weeks) [Optional]
- [ ] Networking setup
- [ ] Data synchronization
- [ ] Client-server architecture
- [ ] Testing

**Total Estimated Time: 3-6 months** (depending on scope and team size)

## Success Criteria

### Must Have
- ✓ Materials load from JSON
- ✓ Tools can be assembled from parts
- ✓ Tool stats calculate correctly
- ✓ Modifiers can be applied
- ✓ Tools can be used and break
- ✓ UI for crafting and modification
- ✓ Save/load system works
- ✓ Mods can be loaded

### Nice to Have
- Multiplayer support
- Advanced rendering effects
- Smeltery equivalent
- World generation
- Sound effects
- Particle effects
- Achievements

### Performance Targets
- Load 100+ materials in < 1 second
- Assemble tool in < 100ms
- Apply modifier in < 50ms
- Render 100+ tools at 60 FPS
- Support 1000+ item inventory

## Navigation

- [Next: Unity Architecture →](./02-Unity-Architecture.md)
- [Back to C# Unity Port Guide ↑](./README.md)
