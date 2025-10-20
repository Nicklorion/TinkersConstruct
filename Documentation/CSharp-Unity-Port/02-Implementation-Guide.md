# C# Unity Implementation Guide

## Overview

This guide provides step-by-step instructions for implementing the Forge & Force crafting system in C# using Unity. It covers project setup, core systems implementation, and best practices.

---

## Project Setup

### Unity Version
- **Recommended**: Unity 2021.3 LTS or newer
- **Minimum**: Unity 2020.3 LTS
- **Render Pipeline**: URP (Universal Render Pipeline) recommended

### Project Structure

```
ForgeAndForce/
├── Assets/
│   ├── Scripts/
│   │   ├── Core/
│   │   │   ├── Registry/
│   │   │   ├── Data/
│   │   │   └── Events/
│   │   ├── Materials/
│   │   │   ├── MaterialData.cs
│   │   │   ├── MaterialRegistry.cs
│   │   │   ├── MaterialStats.cs
│   │   │   └── MaterialTraits.cs
│   │   ├── Tools/
│   │   │   ├── ToolStack.cs
│   │   │   ├── ToolDefinition.cs
│   │   │   ├── ToolStats.cs
│   │   │   └── ToolNBT.cs
│   │   ├── Modifiers/
│   │   │   ├── Modifier.cs
│   │   │   ├── ModifierRegistry.cs
│   │   │   └── ModifierEffects/
│   │   ├── Smeltery/
│   │   │   ├── MultiblockStructure.cs
│   │   │   ├── FluidTank.cs
│   │   │   └── MeltingRecipes.cs
│   │   ├── Progression/
│   │   │   ├── TemperatureSystem.cs
│   │   │   └── ForceSystem.cs
│   │   ├── UI/
│   │   │   ├── CraftingUI.cs
│   │   │   ├── InventoryUI.cs
│   │   │   └── SmelteryUI.cs
│   │   └── Utilities/
│   │       ├── JsonLoader.cs
│   │       ├── SaveSystem.cs
│   │       └── Extensions.cs
│   ├── Resources/
│   │   ├── Registries/
│   │   │   ├── MaterialRegistry.asset
│   │   │   ├── ToolRegistry.asset
│   │   │   └── ModifierRegistry.asset
│   │   └── Data/
│   │       ├── Materials/
│   │       ├── Tools/
│   │       └── Recipes/
│   ├── StreamingAssets/
│   │   └── Data/
│   │       ├── materials/
│   │       ├── tools/
│   │       └── recipes/
│   ├── Prefabs/
│   │   ├── Tools/
│   │   ├── Structures/
│   │   └── UI/
│   ├── Models/
│   ├── Textures/
│   └── Sounds/
└── Packages/
    └── manifest.json
```

### Required Packages

Add to `Packages/manifest.json`:
```json
{
  "dependencies": {
    "com.unity.textmeshpro": "3.0.6",
    "com.unity.inputsystem": "1.4.4",
    "com.unity.render-pipelines.universal": "12.1.7",
    "com.unity.cinemachine": "2.8.9"
  }
}
```

---

## Core Systems Implementation

### 1. Material System

#### Step 1: Create Material ID Value Object

```csharp
// Assets/Scripts/Materials/MaterialId.cs

using System;
using UnityEngine;

[Serializable]
public struct MaterialId : IEquatable<MaterialId>
{
    [SerializeField] private string namespace_;
    [SerializeField] private string path;

    public string Namespace => namespace_;
    public string Path => path;

    public MaterialId(string namespace_, string path)
    {
        this.namespace_ = namespace_;
        this.path = path;
    }

    public MaterialId(string fullId)
    {
        var parts = fullId.Split(':');
        if (parts.Length != 2)
            throw new ArgumentException($"Invalid MaterialId format: {fullId}");
        
        this.namespace_ = parts[0];
        this.path = parts[1];
    }

    public override string ToString() => $"{namespace_}:{path}";

    public bool Equals(MaterialId other)
    {
        return namespace_ == other.namespace_ && path == other.path;
    }

    public override bool Equals(object obj)
    {
        return obj is MaterialId other && Equals(other);
    }

    public override int GetHashCode()
    {
        unchecked
        {
            return ((namespace_?.GetHashCode() ?? 0) * 397) ^ (path?.GetHashCode() ?? 0);
        }
    }

    public static bool operator ==(MaterialId left, MaterialId right)
    {
        return left.Equals(right);
    }

    public static bool operator !=(MaterialId left, MaterialId right)
    {
        return !left.Equals(right);
    }
}
```

#### Step 2: Create Material Interface

```csharp
// Assets/Scripts/Materials/IMaterial.cs

public interface IMaterial
{
    MaterialId Id { get; }
    int Tier { get; }
    int SortOrder { get; }
    bool IsCraftable { get; }
    bool IsHidden { get; }
}
```

#### Step 3: Create Material ScriptableObject

```csharp
// Assets/Scripts/Materials/MaterialAsset.cs

using UnityEngine;

[CreateAssetMenu(fileName = "New Material", menuName = "Forge & Force/Material")]
public class MaterialAsset : ScriptableObject, IMaterial
{
    [Header("Identity")]
    [SerializeField] private string materialId;
    [SerializeField] private string displayName;
    [SerializeField, TextArea] private string description;

    [Header("Properties")]
    [SerializeField] private int tier;
    [SerializeField] private int sortOrder;
    [SerializeField] private bool craftable = true;
    [SerializeField] private bool hidden = false;

    [Header("Rendering")]
    [SerializeField] private Color materialColor = Color.white;
    [SerializeField] private Texture2D materialTexture;
    [SerializeField] private int luminosity;

    // IMaterial implementation
    public MaterialId Id => new MaterialId(materialId);
    public int Tier => tier;
    public int SortOrder => sortOrder;
    public bool IsCraftable => craftable;
    public bool IsHidden => hidden;

    // Additional properties
    public string DisplayName => displayName;
    public string Description => description;
    public Color MaterialColor => materialColor;
    public Texture2D MaterialTexture => materialTexture;
    public int Luminosity => luminosity;
}
```

#### Step 4: Create Material Stats

```csharp
// Assets/Scripts/Materials/MaterialStats.cs

using System;
using UnityEngine;

[Serializable]
public class MaterialStatsId : IEquatable<MaterialStatsId>
{
    [SerializeField] private string id;

    public string Id => id;

    public MaterialStatsId(string id)
    {
        this.id = id;
    }

    public override string ToString() => id;
    public bool Equals(MaterialStatsId other) => id == other?.id;
    public override bool Equals(object obj) => obj is MaterialStatsId other && Equals(other);
    public override int GetHashCode() => id?.GetHashCode() ?? 0;
}

public interface IMaterialStats
{
    MaterialStatsId StatsId { get; }
}

[Serializable]
public class HeadMaterialStats : IMaterialStats
{
    [SerializeField] private int durability;
    [SerializeField] private float miningSpeed;
    [SerializeField] private int miningTier;
    [SerializeField] private float attack;

    public MaterialStatsId StatsId => new MaterialStatsId("tconstruct:head");
    public int Durability => durability;
    public float MiningSpeed => miningSpeed;
    public int MiningTier => miningTier;
    public float Attack => attack;

    public HeadMaterialStats(int durability, float miningSpeed, int miningTier, float attack)
    {
        this.durability = durability;
        this.miningSpeed = miningSpeed;
        this.miningTier = miningTier;
        this.attack = attack;
    }
}

[Serializable]
public class HandleMaterialStats : IMaterialStats
{
    [SerializeField] private float durabilityMultiplier;
    [SerializeField] private float miningSpeedMultiplier;
    [SerializeField] private float attackMultiplier;

    public MaterialStatsId StatsId => new MaterialStatsId("tconstruct:handle");
    public float DurabilityMultiplier => durabilityMultiplier;
    public float MiningSpeedMultiplier => miningSpeedMultiplier;
    public float AttackMultiplier => attackMultiplier;

    public HandleMaterialStats(float durability, float miningSpeed, float attack)
    {
        this.durabilityMultiplier = durability;
        this.miningSpeedMultiplier = miningSpeed;
        this.attackMultiplier = attack;
    }
}
```

#### Step 5: Create Material Registry

```csharp
// Assets/Scripts/Materials/MaterialRegistry.cs

using System.Collections.Generic;
using System.Linq;
using UnityEngine;

[CreateAssetMenu(fileName = "MaterialRegistry", menuName = "Forge & Force/Registries/Material Registry")]
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
                if (instance == null)
                {
                    Debug.LogError("MaterialRegistry not found in Resources/Registries/");
                }
            }
            return instance;
        }
    }

    [SerializeField] private List<MaterialAsset> materials = new List<MaterialAsset>();
    [SerializeField] private List<MaterialStatsAsset> stats = new List<MaterialStatsAsset>();

    private Dictionary<MaterialId, MaterialAsset> materialMap;
    private Dictionary<(MaterialId, MaterialStatsId), IMaterialStats> statsMap;

    private void OnEnable()
    {
        BuildMaps();
    }

    private void BuildMaps()
    {
        // Build material map
        materialMap = new Dictionary<MaterialId, MaterialAsset>();
        foreach (var material in materials)
        {
            if (material != null)
            {
                materialMap[material.Id] = material;
            }
        }

        // Build stats map
        statsMap = new Dictionary<(MaterialId, MaterialStatsId), IMaterialStats>();
        foreach (var statsAsset in stats)
        {
            if (statsAsset != null)
            {
                var key = (statsAsset.MaterialId, statsAsset.Stats.StatsId);
                statsMap[key] = statsAsset.Stats;
            }
        }
    }

    public MaterialAsset GetMaterial(MaterialId id)
    {
        return materialMap.TryGetValue(id, out var material) ? material : null;
    }

    public IMaterialStats GetStats(MaterialId materialId, MaterialStatsId statsId)
    {
        return statsMap.TryGetValue((materialId, statsId), out var stats) ? stats : null;
    }

    public IEnumerable<MaterialAsset> GetAllMaterials()
    {
        return materials.Where(m => m != null);
    }

    public IEnumerable<MaterialAsset> GetCraftableMaterials()
    {
        return materials.Where(m => m != null && m.IsCraftable);
    }

    // Editor helpers
#if UNITY_EDITOR
    public void RegisterMaterial(MaterialAsset material)
    {
        if (!materials.Contains(material))
        {
            materials.Add(material);
            BuildMaps();
            UnityEditor.EditorUtility.SetDirty(this);
        }
    }

    public void RegisterStats(MaterialStatsAsset statsAsset)
    {
        if (!stats.Contains(statsAsset))
        {
            stats.Add(statsAsset);
            BuildMaps();
            UnityEditor.EditorUtility.SetDirty(this);
        }
    }
#endif
}

// Helper ScriptableObject to store stats
[CreateAssetMenu(fileName = "MaterialStats", menuName = "Forge & Force/Material Stats")]
public class MaterialStatsAsset : ScriptableObject
{
    [SerializeField] private string materialId;
    [SerializeField] private string statsType; // "head", "handle", etc.
    [SerializeField] private HeadMaterialStats headStats;
    [SerializeField] private HandleMaterialStats handleStats;

    public MaterialId MaterialId => new MaterialId(materialId);
    
    public IMaterialStats Stats
    {
        get
        {
            switch (statsType)
            {
                case "head": return headStats;
                case "handle": return handleStats;
                default: return null;
            }
        }
    }
}
```

### 2. Tool System

#### Step 1: Create Tool Definition

```csharp
// Assets/Scripts/Tools/ToolDefinition.cs

using System;
using System.Collections.Generic;
using UnityEngine;

[Serializable]
public struct ToolDefinitionId : IEquatable<ToolDefinitionId>
{
    [SerializeField] private string id;
    
    public string Id => id;
    
    public ToolDefinitionId(string id)
    {
        this.id = id;
    }
    
    public override string ToString() => id;
    public bool Equals(ToolDefinitionId other) => id == other.id;
    public override bool Equals(object obj) => obj is ToolDefinitionId other && Equals(other);
    public override int GetHashCode() => id?.GetHashCode() ?? 0;
}

[Serializable]
public class ToolPartDefinition
{
    [SerializeField] private string name;
    [SerializeField] private string statType;
    [SerializeField] private GameObject partItem;
    [SerializeField] private bool required = true;

    public string Name => name;
    public MaterialStatsId StatType => new MaterialStatsId(statType);
    public GameObject PartItem => partItem;
    public bool Required => required;
}

[CreateAssetMenu(fileName = "New Tool", menuName = "Forge & Force/Tool Definition")]
public class ToolDefinition : ScriptableObject
{
    [Header("Identity")]
    [SerializeField] private string toolId;
    [SerializeField] private string displayName;
    [SerializeField, TextArea] private string description;

    [Header("Configuration")]
    [SerializeField] private GameObject toolPrefab;
    [SerializeField] private List<ToolPartDefinition> parts = new List<ToolPartDefinition>();
    [SerializeField] private List<string> statIds = new List<string>();
    [SerializeField] private int baseModifierSlots = 3;

    public ToolDefinitionId Id => new ToolDefinitionId(toolId);
    public string DisplayName => displayName;
    public string Description => description;
    public GameObject ToolPrefab => toolPrefab;
    public List<ToolPartDefinition> Parts => parts;
    public List<string> StatIds => statIds;
    public int BaseModifierSlots => baseModifierSlots;

    public ToolStack BuildTool(List<MaterialId> materials)
    {
        if (materials.Count != parts.Count)
        {
            throw new ArgumentException(
                $"Material count ({materials.Count}) does not match part count ({parts.Count})");
        }

        return ToolStack.Create(this, materials);
    }

    public bool ValidateMaterials(List<MaterialId> materials)
    {
        if (materials.Count != parts.Count)
            return false;

        var registry = MaterialRegistry.Instance;
        for (int i = 0; i < parts.Count; i++)
        {
            var material = registry.GetMaterial(materials[i]);
            if (material == null)
                return false;

            var stats = registry.GetStats(materials[i], parts[i].StatType);
            if (parts[i].Required && stats == null)
                return false;
        }

        return true;
    }
}
```

#### Step 2: Create Tool Stack

```csharp
// Assets/Scripts/Tools/ToolStack.cs

using System;
using System.Collections.Generic;
using UnityEngine;

[Serializable]
public class ToolStack
{
    [SerializeField] private string definitionId;
    [SerializeField] private List<string> materialIds = new List<string>();
    [SerializeField] private Dictionary<string, float> stats = new Dictionary<string, float>();
    [SerializeField] private List<ModifierEntry> modifiers = new List<ModifierEntry>();
    [SerializeField] private int damage;
    [SerializeField] private int modifierSlots;

    // Properties
    public ToolDefinitionId DefinitionId => new ToolDefinitionId(definitionId);
    public ToolDefinition Definition => ToolDefinitionRegistry.Instance.Get(DefinitionId);
    public List<MaterialId> Materials => materialIds.ConvertAll(id => new MaterialId(id));
    public Dictionary<string, float> Stats => new Dictionary<string, float>(stats);
    public List<ModifierEntry> Modifiers => new List<ModifierEntry>(modifiers);
    public int Damage { get => damage; set => damage = value; }
    public int ModifierSlots { get => modifierSlots; set => modifierSlots = value; }

    // Calculated properties
    public int MaxDurability => (int)GetStat("durability");
    public float MiningSpeed => GetStat("mining_speed");
    public int HarvestTier => (int)GetStat("harvest_tier");
    public float AttackDamage => GetStat("attack");
    public bool IsBroken => damage >= MaxDurability;
    public float DurabilityPercent => MaxDurability > 0 ? (MaxDurability - damage) / (float)MaxDurability : 0f;

    // Factory method
    public static ToolStack Create(ToolDefinition definition, List<MaterialId> materials)
    {
        var tool = new ToolStack
        {
            definitionId = definition.Id.Id,
            materialIds = materials.ConvertAll(m => m.ToString()),
            modifierSlots = definition.BaseModifierSlots,
            damage = 0
        };

        tool.CalculateStats();
        tool.ApplyTraits();

        return tool;
    }

    private void CalculateStats()
    {
        stats.Clear();
        var definition = Definition;
        var registry = MaterialRegistry.Instance;

        // Initialize with defaults
        foreach (var statId in definition.StatIds)
        {
            stats[statId] = 0f;
        }

        // Apply material stats
        for (int i = 0; i < definition.Parts.Count; i++)
        {
            var part = definition.Parts[i];
            var material = Materials[i];
            var materialStats = registry.GetStats(material, part.StatType);

            if (materialStats == null)
                continue;

            if (materialStats is HeadMaterialStats headStats)
            {
                // Head: Add directly
                AddStat("durability", headStats.Durability);
                AddStat("mining_speed", headStats.MiningSpeed);
                AddStat("harvest_tier", headStats.MiningTier);
                AddStat("attack", headStats.Attack);
            }
            else if (materialStats is HandleMaterialStats handleStats)
            {
                // Handle: Multiply
                MultiplyStat("durability", handleStats.DurabilityMultiplier);
                MultiplyStat("mining_speed", handleStats.MiningSpeedMultiplier);
                MultiplyStat("attack", handleStats.AttackMultiplier);
            }
        }

        // Apply modifier effects
        foreach (var modifier in modifiers)
        {
            var modifierInstance = ModifierRegistry.Instance.Get(modifier.Id);
            modifierInstance?.ModifyStats(this, modifier.Level);
        }
    }

    private void ApplyTraits()
    {
        // Get traits from materials and add as modifiers
        var registry = MaterialRegistry.Instance;
        var definition = Definition;

        for (int i = 0; i < Materials.Count; i++)
        {
            var material = Materials[i];
            var statType = definition.Parts[i].StatType;
            
            var traits = registry.GetTraits(material, statType);
            foreach (var trait in traits)
            {
                AddModifier(trait.Id, trait.Level);
            }
        }
    }

    public float GetStat(string statId)
    {
        return stats.TryGetValue(statId, out var value) ? value : 0f;
    }

    public void SetStat(string statId, float value)
    {
        stats[statId] = value;
    }

    private void AddStat(string statId, float value)
    {
        if (stats.ContainsKey(statId))
            stats[statId] += value;
        else
            stats[statId] = value;
    }

    private void MultiplyStat(string statId, float multiplier)
    {
        if (stats.ContainsKey(statId))
            stats[statId] *= multiplier;
    }

    public void AddModifier(ModifierId id, int level)
    {
        var existing = modifiers.Find(m => m.Id == id);
        if (existing != null)
        {
            existing.Level += level;
        }
        else
        {
            modifiers.Add(new ModifierEntry(id, level));
        }
        
        CalculateStats(); // Recalculate with new modifier
    }

    public void DealDamage(int amount)
    {
        damage = Mathf.Min(damage + amount, MaxDurability);
    }

    public void Repair(int amount)
    {
        damage = Mathf.Max(0, damage - amount);
    }

    // Serialization
    public string ToJson()
    {
        return JsonUtility.ToJson(this);
    }

    public static ToolStack FromJson(string json)
    {
        var tool = JsonUtility.FromJson<ToolStack>(json);
        tool.CalculateStats();
        return tool;
    }
}

[Serializable]
public class ModifierEntry
{
    [SerializeField] private string id;
    [SerializeField] private int level;

    public ModifierId Id => new ModifierId(id);
    public int Level { get => level; set => level = value; }

    public ModifierEntry(ModifierId id, int level)
    {
        this.id = id.ToString();
        this.level = level;
    }
}
```

### 3. Save System

```csharp
// Assets/Scripts/Utilities/SaveSystem.cs

using System;
using System.IO;
using UnityEngine;

[Serializable]
public class GameSaveData
{
    public string version = "1.0.0";
    public PlayerData player;
    public WorldData world;
}

[Serializable]
public class PlayerData
{
    public Vector3 position;
    public List<string> inventory; // Serialized ToolStacks
    public ProgressionData progression;
}

[Serializable]
public class ProgressionData
{
    public float maxTemperature;
    public float maxForce;
    public List<string> unlockedRecipes;
}

[Serializable]
public class WorldData
{
    public List<StructureData> structures;
    public List<SmelteryData> smelteries;
}

public static class SaveSystem
{
    private static string SavePath => Path.Combine(Application.persistentDataPath, "saves");
    
    public static void Save(GameSaveData data, string saveName)
    {
        try
        {
            if (!Directory.Exists(SavePath))
            {
                Directory.CreateDirectory(SavePath);
            }

            string filePath = Path.Combine(SavePath, $"{saveName}.json");
            string json = JsonUtility.ToJson(data, true);
            File.WriteAllText(filePath, json);
            
            Debug.Log($"Game saved to {filePath}");
        }
        catch (Exception e)
        {
            Debug.LogError($"Failed to save game: {e.Message}");
        }
    }

    public static GameSaveData Load(string saveName)
    {
        try
        {
            string filePath = Path.Combine(SavePath, $"{saveName}.json");
            
            if (!File.Exists(filePath))
            {
                Debug.LogWarning($"Save file not found: {filePath}");
                return null;
            }

            string json = File.ReadAllText(filePath);
            GameSaveData data = JsonUtility.FromJson<GameSaveData>(json);
            
            Debug.Log($"Game loaded from {filePath}");
            return data;
        }
        catch (Exception e)
        {
            Debug.LogError($"Failed to load game: {e.Message}");
            return null;
        }
    }

    public static bool SaveExists(string saveName)
    {
        string filePath = Path.Combine(SavePath, $"{saveName}.json");
        return File.Exists(filePath);
    }

    public static void DeleteSave(string saveName)
    {
        try
        {
            string filePath = Path.Combine(SavePath, $"{saveName}.json");
            if (File.Exists(filePath))
            {
                File.Delete(filePath);
                Debug.Log($"Save deleted: {filePath}");
            }
        }
        catch (Exception e)
        {
            Debug.LogError($"Failed to delete save: {e.Message}");
        }
    }
}
```

---

## Testing Strategy

### Unit Tests

Create test assembly in `Assets/Tests/`:

```csharp
// Assets/Tests/MaterialSystemTests.cs

using NUnit.Framework;
using UnityEngine;

public class MaterialSystemTests
{
    [Test]
    public void MaterialId_ToString_ReturnsCorrectFormat()
    {
        var id = new MaterialId("tconstruct", "iron");
        Assert.AreEqual("tconstruct:iron", id.ToString());
    }

    [Test]
    public void MaterialId_Equality_WorksCorrectly()
    {
        var id1 = new MaterialId("tconstruct", "iron");
        var id2 = new MaterialId("tconstruct", "iron");
        var id3 = new MaterialId("tconstruct", "gold");

        Assert.AreEqual(id1, id2);
        Assert.AreNotEqual(id1, id3);
    }

    [Test]
    public void MaterialRegistry_GetMaterial_ReturnsCorrectMaterial()
    {
        var registry = MaterialRegistry.Instance;
        var ironId = new MaterialId("tconstruct", "iron");
        var material = registry.GetMaterial(ironId);

        Assert.IsNotNull(material);
        Assert.AreEqual(ironId, material.Id);
    }
}

// Assets/Tests/ToolSystemTests.cs

using NUnit.Framework;
using System.Collections.Generic;

public class ToolSystemTests
{
    [Test]
    public void ToolStack_Create_CalculatesStatsCorrectly()
    {
        // This requires test data setup
        var definition = GetTestToolDefinition();
        var materials = new List<MaterialId>
        {
            new MaterialId("test", "iron"),
            new MaterialId("test", "wood"),
            new MaterialId("test", "flint")
        };

        var tool = ToolStack.Create(definition, materials);

        Assert.Greater(tool.MaxDurability, 0);
        Assert.Greater(tool.MiningSpeed, 0f);
        Assert.AreEqual(0, tool.Damage);
    }

    [Test]
    public void ToolStack_DealDamage_ReducesDurability()
    {
        var tool = GetTestToolStack();
        int initialDamage = tool.Damage;

        tool.DealDamage(10);

        Assert.AreEqual(initialDamage + 10, tool.Damage);
    }

    [Test]
    public void ToolStack_Repair_RestoresDurability()
    {
        var tool = GetTestToolStack();
        tool.DealDamage(50);
        int damagedAmount = tool.Damage;

        tool.Repair(20);

        Assert.AreEqual(damagedAmount - 20, tool.Damage);
    }

    private ToolDefinition GetTestToolDefinition()
    {
        // Load or create test definition
        return Resources.Load<ToolDefinition>("Test/TestPickaxe");
    }

    private ToolStack GetTestToolStack()
    {
        var definition = GetTestToolDefinition();
        var materials = new List<MaterialId>
        {
            new MaterialId("test", "iron"),
            new MaterialId("test", "wood"),
            new MaterialId("test", "flint")
        };
        return ToolStack.Create(definition, materials);
    }
}
```

---

## Performance Optimization

### Tips for Unity

1. **Use Object Pooling**
```csharp
public class ToolPool : MonoBehaviour
{
    [SerializeField] private GameObject toolPrefab;
    [SerializeField] private int poolSize = 50;
    
    private Queue<GameObject> pool = new Queue<GameObject>();
    
    private void Start()
    {
        for (int i = 0; i < poolSize; i++)
        {
            var obj = Instantiate(toolPrefab);
            obj.SetActive(false);
            pool.Enqueue(obj);
        }
    }
    
    public GameObject GetTool()
    {
        if (pool.Count > 0)
        {
            var obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(toolPrefab);
    }
    
    public void ReturnTool(GameObject tool)
    {
        tool.SetActive(false);
        pool.Enqueue(tool);
    }
}
```

2. **Cache Components**
```csharp
public class ToolBehaviour : MonoBehaviour
{
    private Renderer _renderer;
    private Collider _collider;
    
    private Renderer Renderer => _renderer ?? (_renderer = GetComponent<Renderer>());
    private Collider Collider => _collider ?? (_collider = GetComponent<Collider>());
}
```

3. **Use Burst Compiler for Heavy Calculations**
```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;

[BurstCompile]
struct CalculateStatsJob : IJob
{
    public float baseDurability;
    public float multiplier;
    public NativeArray<float> result;
    
    public void Execute()
    {
        result[0] = baseDurability * multiplier;
    }
}
```

---

## Next Steps

1. Implement each system following the patterns shown
2. Create comprehensive test suite
3. Build prototype UI
4. Test performance with profiler
5. Iterate based on feedback

---

**Related Documents:**
- [Class Diagrams](../ClassDiagrams/README.md)
- [Technical Design Document](../TechnicalDesign/TDD-01-Overview.md)
- [Game Design Document](../GameDesign/GDD-01-Core-Concept.md)
