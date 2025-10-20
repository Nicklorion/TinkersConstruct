# Class Diagram: Temperature & Force System

## Overview

The Temperature and Force systems are the two core progression mechanics in Forge & Force. They gate access to materials, tools, and crafting processes based on physical principles.

---

## Temperature System

### Core Classes

```
                  <<abstract>>
               ┌─────────────────┐
               │  HeatSource     │
               ├─────────────────┤
               │ + getTemp()     │
               │ + consume()     │
               │ + isActive()    │
               └─────────────────┘
                       △
                       │ extends
          ┌────────────┼────────────┐
          │            │            │
  ┌───────────┐  ┌──────────┐  ┌────────────┐
  │ FireHeat  │  │ FuelHeat │  │ BlastHeat  │
  ├───────────┤  ├──────────┤  ├────────────┤
  │-baseTemp  │  │-fuelType │  │-forced air │
  │-size      │  │-burnRate │  │-multiplier │
  ├───────────┤  ├──────────┤  ├────────────┤
  │+getTemp() │  │+getTemp()│  │+getTemp()  │
  └───────────┘  └──────────┘  └────────────┘

           ┌──────────────────────┐
           │ TemperatureStructure │
           ├──────────────────────┤
           │ - currentTemp: float │
           │ - maxTemp: float     │
           │ - insulation: float  │
           │ - volume: float      │
           │ - heatSource: Heat   │
           ├──────────────────────┤
           │ + update(deltaTime)  │
           │ + addFuel(fuel)      │
           │ + getTemperature()   │
           │ + canProcess(recipe) │
           └──────────────────────┘
                    │
                    │ uses
                    │
           ┌──────────────────────┐
           │TemperatureController │
           ├──────────────────────┤
           │ - structures: List   │
           ├──────────────────────┤
           │ + update(deltaTime)  │
           │ + register(struct)   │
           │ + unregister(struct) │
           └──────────────────────┘
```

### Temperature Tiers

```csharp
public enum TemperatureTier
{
    Ambient = 0,        // 0-50°C
    FrictionFire = 1,   // 50-400°C
    OpenFlame = 2,      // 400-800°C
    ClayKiln = 3,       // 800-1200°C
    StoneForge = 4,     // 1200-1600°C
    BrickFurnace = 5,   // 1600-2000°C
    AdvancedForge = 6   // 2000-3000°C
}

public static class TemperatureConstants
{
    public static readonly Dictionary<TemperatureTier, float> MinTemperatures = new()
    {
        { TemperatureTier.Ambient, 0f },
        { TemperatureTier.FrictionFire, 50f },
        { TemperatureTier.OpenFlame, 400f },
        { TemperatureTier.ClayKiln, 800f },
        { TemperatureTier.StoneForge, 1200f },
        { TemperatureTier.BrickFurnace, 1600f },
        { TemperatureTier.AdvancedForge, 2000f }
    };
    
    public static readonly Dictionary<TemperatureTier, float> MaxTemperatures = new()
    {
        { TemperatureTier.Ambient, 50f },
        { TemperatureTier.FrictionFire, 400f },
        { TemperatureTier.OpenFlame, 800f },
        { TemperatureTier.ClayKiln, 1200f },
        { TemperatureTier.StoneForge, 1600f },
        { TemperatureTier.BrickFurnace, 2000f },
        { TemperatureTier.AdvancedForge, 3000f }
    };
}
```

### Heat Transfer Algorithm

```
Algorithm: UpdateTemperature(structure, deltaTime)
Input: Temperature structure, Time elapsed
Output: New temperature

1. currentTemp = structure.temperature
2. targetTemp = structure.heatSource.getTemperature()
3. ambientTemp = environment.ambientTemperature (default: 20°C)
4. insulation = structure.getInsulation() // 1.0 = poor, 10.0 = excellent

5. // Calculate heat gain from source
   if structure.hasActiveFuel():
      heatGainRate = structure.heatSource.getHeatRate()
      heatGain = (targetTemp - currentTemp) * heatGainRate * deltaTime
      // Efficiency reduces at higher temperatures
      efficiency = 1.0 - (currentTemp / targetTemp) * 0.3
      heatGain *= efficiency
   else:
      heatGain = 0

6. // Calculate heat loss to environment
   // Newton's law of cooling: dQ/dt = k * (T - T_ambient)
   heatLossRate = 1.0 / insulation
   heatLoss = (currentTemp - ambientTemp) * heatLossRate * deltaTime

7. // Apply heat capacity (larger structures heat/cool slower)
   heatCapacity = structure.volume * structure.material.specificHeat
   tempChange = (heatGain - heatLoss) / heatCapacity

8. // Update temperature with limits
   newTemp = currentTemp + tempChange
   newTemp = clamp(newTemp, ambientTemp, structure.maxTemperature)

9. structure.temperature = newTemp
10. return newTemp
```

### C# Implementation

```csharp
// Temperature Structure Component
public class TemperatureStructure : MonoBehaviour
{
    [Header("Temperature Properties")]
    [SerializeField] private float currentTemperature = 20f; // Celsius
    [SerializeField] private float maxTemperature = 1000f;
    [SerializeField] private float insulation = 5f; // 1-10 scale
    [SerializeField] private float volume = 1f; // cubic meters
    [SerializeField] private float specificHeat = 1f; // kJ/(kg·°C)

    [Header("Heat Source")]
    [SerializeField] private HeatSource heatSource;
    
    [Header("Status")]
    [SerializeField, ReadOnly] private TemperatureTier currentTier;
    [SerializeField, ReadOnly] private bool isActive;

    public float CurrentTemperature => currentTemperature;
    public float MaxTemperature => maxTemperature;
    public TemperatureTier CurrentTier => currentTier;
    public bool IsActive => isActive;

    private const float AmbientTemperature = 20f;

    private void Update()
    {
        UpdateTemperature(Time.deltaTime);
        UpdateTier();
    }

    private void UpdateTemperature(float deltaTime)
    {
        if (heatSource == null || !heatSource.IsActive)
        {
            // Cool down to ambient
            CoolDown(deltaTime);
            isActive = false;
            return;
        }

        isActive = true;
        float targetTemp = heatSource.GetTemperature();

        // Heat gain from source
        float heatGainRate = heatSource.GetHeatRate();
        float heatGain = (targetTemp - currentTemperature) * heatGainRate * deltaTime;
        
        // Efficiency decreases at higher temperatures
        float efficiency = 1.0f - (currentTemperature / targetTemp) * 0.3f;
        heatGain *= Mathf.Max(0.1f, efficiency);

        // Heat loss to environment
        float heatLossRate = 1.0f / insulation;
        float heatLoss = (currentTemperature - AmbientTemperature) * heatLossRate * deltaTime;

        // Apply heat capacity
        float heatCapacity = volume * specificHeat;
        float tempChange = (heatGain - heatLoss) / heatCapacity;

        // Update temperature
        currentTemperature += tempChange;
        currentTemperature = Mathf.Clamp(currentTemperature, AmbientTemperature, maxTemperature);

        // Consume fuel
        heatSource.Consume(deltaTime);
    }

    private void CoolDown(float deltaTime)
    {
        float heatLossRate = 1.0f / insulation;
        float heatLoss = (currentTemperature - AmbientTemperature) * heatLossRate * deltaTime;
        
        float heatCapacity = volume * specificHeat;
        float tempChange = -heatLoss / heatCapacity;
        
        currentTemperature += tempChange;
        currentTemperature = Mathf.Max(AmbientTemperature, currentTemperature);
    }

    private void UpdateTier()
    {
        foreach (var tier in TemperatureConstants.MinTemperatures.Keys.OrderByDescending(t => t))
        {
            if (currentTemperature >= TemperatureConstants.MinTemperatures[tier])
            {
                currentTier = tier;
                return;
            }
        }
        currentTier = TemperatureTier.Ambient;
    }

    public bool CanProcess(float requiredTemperature)
    {
        return currentTemperature >= requiredTemperature;
    }

    public bool CanProcessTier(TemperatureTier tier)
    {
        return currentTemperature >= TemperatureConstants.MinTemperatures[tier];
    }
}

// Heat Source Base Class
public abstract class HeatSource : MonoBehaviour
{
    [SerializeField] protected float baseTemperature = 100f;
    [SerializeField] protected float heatRate = 1f;
    [SerializeField] protected bool active = false;

    public abstract float GetTemperature();
    public abstract float GetHeatRate();
    public abstract void Consume(float deltaTime);
    public bool IsActive => active;

    public virtual void Activate()
    {
        active = true;
    }

    public virtual void Deactivate()
    {
        active = false;
    }
}

// Fuel-Based Heat Source
public class FuelHeatSource : HeatSource
{
    [SerializeField] private FuelType currentFuel;
    [SerializeField] private float fuelAmount = 0f;
    [SerializeField] private float burnRate = 0.1f; // fuel per second

    public override float GetTemperature()
    {
        if (currentFuel == null || fuelAmount <= 0)
            return baseTemperature;
        
        return currentFuel.Temperature;
    }

    public override float GetHeatRate()
    {
        if (currentFuel == null || fuelAmount <= 0)
            return 0.1f;
        
        return currentFuel.HeatRate * heatRate;
    }

    public override void Consume(float deltaTime)
    {
        if (currentFuel != null && fuelAmount > 0)
        {
            float consumed = burnRate * deltaTime;
            fuelAmount -= consumed;
            
            if (fuelAmount <= 0)
            {
                fuelAmount = 0;
                active = false;
            }
        }
    }

    public void AddFuel(FuelType fuel, float amount)
    {
        currentFuel = fuel;
        fuelAmount += amount;
        active = fuelAmount > 0;
    }
}

[CreateAssetMenu(fileName = "New Fuel", menuName = "Forge & Force/Fuel Type")]
public class FuelType : ScriptableObject
{
    [SerializeField] private string fuelName;
    [SerializeField] private float temperature = 400f;
    [SerializeField] private float heatRate = 1f;
    [SerializeField] private float burnDuration = 60f; // seconds per unit

    public string FuelName => fuelName;
    public float Temperature => temperature;
    public float HeatRate => heatRate;
    public float BurnDuration => burnDuration;
}
```

---

## Force System

### Core Classes

```
            ┌──────────────────────┐
            │   ForceApplication   │
            ├──────────────────────┤
            │ - type: ForceType    │
            │ - magnitude: float   │
            │ - duration: float    │
            │ - precision: float   │
            ├──────────────────────┤
            │ + apply(target)      │
            │ + validate(tool)     │
            │ + calculateDamage()  │
            └──────────────────────┘
                     │
                     │ uses
                     │
            ┌──────────────────────┐
            │   ForceCalculator    │
            ├──────────────────────┤
            │ + calculateForce()   │
            │ + validateAction()   │
            │ + applyToMaterial()  │
            │ + calculateToolDmg() │
            └──────────────────────┘

    Force Types:
    
┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
│   Impact   │  │ Sustained  │  │  Precise   │  │Distributed │
├────────────┤  ├────────────┤  ├────────────┤  ├────────────┤
│ Quick hit  │  │ Long press │  │ Controlled │  │ Spread out │
│ High force │  │ Med force  │  │ Low force  │  │ Over area  │
│ Hammering  │  │ Pressing   │  │ Carving    │  │ Rolling    │
└────────────┘  └────────────┘  └────────────┘  └────────────┘
```

### Force Tiers

```csharp
public enum ForceTier
{
    HandForce = 0,      // 1-10N
    WoodTools = 1,      // 10-100N
    StoneTools = 2,     // 100-500N
    FlintTools = 3,     // 500-1000N
    MetalTools = 4,     // 1000-5000N
    HardenedTools = 5,  // 5000-10000N
    PrecisionTools = 6  // 10000+N
}

public enum ForceType
{
    Impact,      // Hammering, striking
    Sustained,   // Pressing, holding
    Precise,     // Carving, cutting
    Distributed  // Rolling, spreading
}

public static class ForceConstants
{
    public static readonly Dictionary<ForceTier, float> MinForce = new()
    {
        { ForceTier.HandForce, 1f },
        { ForceTier.WoodTools, 10f },
        { ForceTier.StoneTools, 100f },
        { ForceTier.FlintTools, 500f },
        { ForceTier.MetalTools, 1000f },
        { ForceTier.HardenedTools, 5000f },
        { ForceTier.PrecisionTools, 10000f }
    };
    
    public static readonly Dictionary<ForceTier, float> MaxForce = new()
    {
        { ForceTier.HandForce, 10f },
        { ForceTier.WoodTools, 100f },
        { ForceTier.StoneTools, 500f },
        { ForceTier.FlintTools, 1000f },
        { ForceTier.MetalTools, 5000f },
        { ForceTier.HardenedTools, 10000f },
        { ForceTier.PrecisionTools, float.MaxValue }
    };
}
```

### Force Application Algorithm

```
Algorithm: ApplyForce(tool, target, actionType)
Input: Tool, Target material, Action type
Output: Success/Failure + Material state + Tool damage

1. // Get tool force capacity
   toolForce = tool.GetForceCapacity()
   toolPrecision = tool.GetPrecision()
   
2. // Get target requirements
   materialResistance = target.GetHardness()
   materialToughness = target.GetToughness()
   
3. // Calculate required force for action
   baseForce = materialResistance
   switch (actionType):
      case Impact:
         requiredForce = baseForce * 2.0    // High force needed
         duration = 0.1                      // Quick action
      case Sustained:
         requiredForce = baseForce * 1.0    // Normal force
         duration = 2.0                      // Long action
      case Precise:
         requiredForce = baseForce * 0.5    // Low force
         requiredPrecision = materialResistance * 0.8
         duration = 1.0
      case Distributed:
         requiredForce = baseForce * 1.5    // Higher total
         area = target.GetArea()
         requiredForce /= area               // But spread out
         duration = 1.5

4. // Validate tool can perform action
   if toolForce < requiredForce:
      return Failure("Insufficient force")
   
   if actionType == Precise && toolPrecision < requiredPrecision:
      return Failure("Tool not precise enough")

5. // Calculate tool damage
   forceDelta = requiredForce - toolForce
   if forceDelta > 0:
      // Overstressing the tool
      damageMultiplier = 1.0 + (forceDelta / toolForce)
   else:
      // Tool is sufficient
      damageMultiplier = 1.0
   
   baseDamage = GetActionDamage(actionType)
   toolDamage = baseDamage * damageMultiplier * (materialToughness / 100.0)

6. // Apply durability damage to tool
   tool.Damage(toolDamage)

7. // Process the action on material
   switch (actionType):
      case Impact:
         result = ProcessImpact(target, toolForce, materialToughness)
      case Sustained:
         result = ProcessSustained(target, toolForce, duration)
      case Precise:
         result = ProcessPrecise(target, toolForce, toolPrecision)
      case Distributed:
         result = ProcessDistributed(target, toolForce, area)

8. // Check for critical failure (tool breaks)
   if tool.Damage >= tool.MaxDurability:
      return Failure("Tool broke", result, toolDamage)

9. return Success(result, toolDamage)
```

### C# Implementation

```csharp
public class ForceApplication
{
    public ForceType Type { get; set; }
    public float Magnitude { get; set; }
    public float Duration { get; set; }
    public float Precision { get; set; }

    public ForceApplication(ForceType type, float magnitude, float duration = 1f, float precision = 0f)
    {
        Type = type;
        Magnitude = magnitude;
        Duration = duration;
        Precision = precision;
    }
}

public class ForceResult
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public float ToolDamage { get; set; }
    public object ProcessedResult { get; set; }

    public ForceResult(bool success, string message = "", float toolDamage = 0f, object result = null)
    {
        Success = success;
        Message = message;
        ToolDamage = toolDamage;
        ProcessedResult = result;
    }
}

public static class ForceCalculator
{
    public static ForceResult ApplyForce(ToolStack tool, MaterialAsset target, ForceType actionType)
    {
        // Get tool capabilities
        float toolForce = tool.GetStat("force_capacity");
        float toolPrecision = tool.GetStat("precision");
        
        // Get target properties
        float materialHardness = target.Hardness;
        float materialToughness = target.Toughness;
        
        // Calculate required force
        float requiredForce = CalculateRequiredForce(materialHardness, actionType);
        float requiredPrecision = actionType == ForceType.Precise ? materialHardness * 0.8f : 0f;
        
        // Validate tool can perform action
        if (toolForce < requiredForce)
        {
            return new ForceResult(false, "Insufficient force to work this material");
        }
        
        if (actionType == ForceType.Precise && toolPrecision < requiredPrecision)
        {
            return new ForceResult(false, "Tool lacks required precision");
        }
        
        // Calculate tool damage
        float toolDamage = CalculateToolDamage(toolForce, requiredForce, materialToughness, actionType);
        
        // Apply damage to tool
        tool.DealDamage((int)toolDamage);
        
        // Process action
        object result = ProcessAction(target, toolForce, actionType, toolPrecision);
        
        // Check if tool broke
        if (tool.IsBroken)
        {
            return new ForceResult(false, "Tool broke during use!", toolDamage, result);
        }
        
        return new ForceResult(true, "Success", toolDamage, result);
    }
    
    private static float CalculateRequiredForce(float hardness, ForceType type)
    {
        float baseForce = hardness;
        
        switch (type)
        {
            case ForceType.Impact:
                return baseForce * 2.0f;  // High force needed
                
            case ForceType.Sustained:
                return baseForce * 1.0f;  // Normal force
                
            case ForceType.Precise:
                return baseForce * 0.5f;  // Lower force, but needs precision
                
            case ForceType.Distributed:
                return baseForce * 1.5f;  // Total force is higher but spread
                
            default:
                return baseForce;
        }
    }
    
    private static float CalculateToolDamage(float toolForce, float requiredForce, float toughness, ForceType type)
    {
        // Base damage per action type
        float baseDamage = type switch
        {
            ForceType.Impact => 2f,
            ForceType.Sustained => 1f,
            ForceType.Precise => 0.5f,
            ForceType.Distributed => 1.5f,
            _ => 1f
        };
        
        // Calculate stress multiplier
        float stressRatio = requiredForce / toolForce;
        float stressMultiplier = 1.0f;
        
        if (stressRatio > 0.9f)  // Tool is close to its limit
        {
            stressMultiplier = 1.0f + (stressRatio - 0.9f) * 5f;  // Increases damage significantly
        }
        
        // Material toughness affects tool wear
        float toughnessMultiplier = toughness / 100f;
        
        return baseDamage * stressMultiplier * toughnessMultiplier;
    }
    
    private static object ProcessAction(MaterialAsset target, float force, ForceType type, float precision)
    {
        // Implementation depends on what you're doing with the material
        // Could be mining, crafting, shaping, etc.
        
        switch (type)
        {
            case ForceType.Impact:
                // Breaking/mining action
                return ProcessImpact(target, force);
                
            case ForceType.Sustained:
                // Pressing/compacting action
                return ProcessSustained(target, force);
                
            case ForceType.Precise:
                // Carving/precise shaping
                return ProcessPrecise(target, force, precision);
                
            case ForceType.Distributed:
                // Rolling/spreading
                return ProcessDistributed(target, force);
                
            default:
                return null;
        }
    }
    
    private static object ProcessImpact(MaterialAsset target, float force)
    {
        // Example: Mining/breaking
        float breakThreshold = target.Hardness * 2f;
        if (force >= breakThreshold)
        {
            return new { broke = true, fragments = Mathf.FloorToInt(force / breakThreshold) };
        }
        return new { broke = false };
    }
    
    private static object ProcessSustained(MaterialAsset target, float force)
    {
        // Example: Compacting/pressing
        float compressionRatio = force / target.Hardness;
        return new { compressed = true, ratio = compressionRatio };
    }
    
    private static object ProcessPrecise(MaterialAsset target, float force, float precision)
    {
        // Example: Carving/shaping
        float quality = precision / (target.Hardness * 0.8f);
        quality = Mathf.Clamp01(quality);
        return new { carved = true, quality = quality };
    }
    
    private static object ProcessDistributed(MaterialAsset target, float force)
    {
        // Example: Rolling/flattening
        float spreadArea = force / target.Hardness;
        return new { flattened = true, area = spreadArea };
    }
}

// Extension method for ToolStack
public static class ToolForceExtensions
{
    public static ForceTier GetForceTier(this ToolStack tool)
    {
        float force = tool.GetStat("force_capacity");
        
        foreach (var tier in ForceConstants.MinForce.Keys.OrderByDescending(t => t))
        {
            if (force >= ForceConstants.MinForce[tier])
            {
                return tier;
            }
        }
        
        return ForceTier.HandForce;
    }
    
    public static bool CanApplyForce(this ToolStack tool, float requiredForce)
    {
        return tool.GetStat("force_capacity") >= requiredForce;
    }
}
```

---

## Progression Integration

### Unlock System

```csharp
[Serializable]
public class ProgressionState
{
    [SerializeField] private float maxAchievedTemperature = 20f;
    [SerializeField] private float maxAchievedForce = 1f;
    [SerializeField] private List<string> unlockedRecipes = new List<string>();
    [SerializeField] private List<string> unlockedMaterials = new List<string>();

    public float MaxTemperature => maxAchievedTemperature;
    public float MaxForce => maxAchievedForce;
    
    public void UpdateTemperature(float temperature)
    {
        if (temperature > maxAchievedTemperature)
        {
            maxAchievedTemperature = temperature;
            CheckTemperatureUnlocks();
        }
    }
    
    public void UpdateForce(float force)
    {
        if (force > maxAchievedForce)
        {
            maxAchievedForce = force;
            CheckForceUnlocks();
        }
    }
    
    private void CheckTemperatureUnlocks()
    {
        // Unlock recipes based on temperature
        foreach (var tier in TemperatureConstants.MinTemperatures.Keys)
        {
            if (maxAchievedTemperature >= TemperatureConstants.MinTemperatures[tier])
            {
                UnlockTemperatureTier(tier);
            }
        }
    }
    
    private void CheckForceUnlocks()
    {
        // Unlock tools based on force
        foreach (var tier in ForceConstants.MinForce.Keys)
        {
            if (maxAchievedForce >= ForceConstants.MinForce[tier])
            {
                UnlockForceTier(tier);
            }
        }
    }
    
    private void UnlockTemperatureTier(TemperatureTier tier)
    {
        // Implementation: Unlock recipes, materials, etc.
        Debug.Log($"Unlocked temperature tier: {tier}");
    }
    
    private void UnlockForceTier(ForceTier tier)
    {
        // Implementation: Unlock tool types, materials, etc.
        Debug.Log($"Unlocked force tier: {tier}");
    }
    
    public bool CanAccessRecipe(string recipeId, float requiredTemp, float requiredForce)
    {
        return maxAchievedTemperature >= requiredTemp && 
               maxAchievedForce >= requiredForce;
    }
}
```

---

## Testing

```csharp
[Test]
public void TemperatureStructure_HeatsUp_WhenFuelAdded()
{
    var structure = CreateTestStructure();
    var fuel = CreateTestFuel(temperature: 800f);
    
    structure.GetComponent<FuelHeatSource>().AddFuel(fuel, 10f);
    
    // Simulate heating for 10 seconds
    for (int i = 0; i < 100; i++)
    {
        structure.Update(0.1f);
    }
    
    Assert.Greater(structure.CurrentTemperature, 100f);
}

[Test]
public void ForceCalculator_Validates_InsufficientForce()
{
    var tool = CreateWeakTool(force: 100f);
    var material = CreateHardMaterial(hardness: 500f);
    
    var result = ForceCalculator.ApplyForce(tool, material, ForceType.Impact);
    
    Assert.IsFalse(result.Success);
    Assert.That(result.Message, Does.Contain("Insufficient"));
}

[Test]
public void Tool_BreaksWhen_OverUsed()
{
    var tool = CreateTestTool(maxDurability: 100);
    var material = CreateTestMaterial(toughness: 10f);
    
    // Use tool until it breaks
    for (int i = 0; i < 200; i++)
    {
        ForceCalculator.ApplyForce(tool, material, ForceType.Impact);
        if (tool.IsBroken) break;
    }
    
    Assert.IsTrue(tool.IsBroken);
}
```

---

## Performance Considerations

**Temperature System:**
- Update active structures only (10-100 per frame)
- Cache heat transfer calculations
- Use background thread for large structures
- Target: < 0.5ms per structure per frame

**Force System:**
- Stateless calculations (no caching needed)
- Single-pass validation
- Minimal allocations
- Target: < 0.1ms per application

---

**Related Diagrams:**
- [Material System](./01-Material-System.md) - Materials have temperature/force requirements
- [Tool System](./02-Tool-System.md) - Tools provide force capacity
- [Recipe System](./05-Recipe-System.md) - Recipes require temperature
