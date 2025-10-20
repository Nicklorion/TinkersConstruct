# Overall System Architecture

## High-Level Overview

This document provides the complete system architecture for Forge & Force, showing how all systems integrate and communicate.

---

## System Architecture Diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                        PRESENTATION LAYER                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐ │
│  │     UI      │  │   Render    │  │    Input    │  │    Audio     │ │
│  │   System    │  │   System    │  │   Handler   │  │    System    │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘ │
└─────────┼────────────────┼────────────────┼────────────────┼──────────┘
          │                │                │                │
          │                └────────┬───────┘                │
          │                         │                        │
┌─────────▼─────────────────────────▼────────────────────────▼──────────┐
│                          GAME LOGIC LAYER                              │
│ ┌──────────────────────────────────────────────────────────────────┐  │
│ │                     CORE GAME SYSTEMS                            │  │
│ │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │
│ │  │   Material   │  │     Tool     │  │   Modifier   │          │  │
│ │  │   System     │◄─┤   System     │◄─┤   System     │          │  │
│ │  └──────────────┘  └──────────────┘  └──────────────┘          │  │
│ └──────────────────────────────────────────────────────────────────┘  │
│ ┌──────────────────────────────────────────────────────────────────┐  │
│ │                  PROGRESSION SYSTEMS                             │  │
│ │  ┌──────────────┐  ┌──────────────┐                            │  │
│ │  │Temperature   │  │    Force     │                            │  │
│ │  │   System     │  │   System     │                            │  │
│ │  └──────────────┘  └──────────────┘                            │  │
│ └──────────────────────────────────────────────────────────────────┘  │
│ ┌──────────────────────────────────────────────────────────────────┐  │
│ │                   CRAFTING SYSTEMS                               │  │
│ │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │
│ │  │   Recipe     │  │   Smeltery   │  │   Casting    │          │  │
│ │  │   System     │  │   System     │  │   System     │          │  │
│ │  └──────────────┘  └──────────────┘  └──────────────┘          │  │
│ └──────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬────────────────────────────────────────┘
                                │
┌───────────────────────────────▼────────────────────────────────────────┐
│                           DATA LAYER                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                │
│  │   Registry   │  │  Save/Load   │  │     JSON     │                │
│  │   System     │  │   Manager    │  │    Parser    │                │
│  └──────────────┘  └──────────────┘  └──────────────┘                │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Detailed System Relationships

### Core Systems Integration

```
                    MaterialRegistry
                           │
                           │ provides materials
                           ▼
                    ToolDefinition ──────► ToolStack
                           │                   │
                           │ defines           │ instance
                           │                   │
                           │                   ▼
                           │              ToolStats
                           │                   │
                           │                   │ calculated from
                           ▼                   │
                    MaterialStats◄─────────────┘
                           │
                           │ influences
                           ▼
                    MaterialTraits
                           │
                           │ applied as
                           ▼
                    ModifierRegistry
                           │
                           │ provides
                           ▼
                    Modifier Instances
                           │
                           │ modify
                           └───────────────► ToolStats
```

### Progression Flow

```
    Player Actions
          │
          ├─────────────────┬──────────────┐
          │                 │              │
          ▼                 ▼              ▼
   Use Temperature    Use Force      Craft Items
          │                 │              │
          ▼                 ▼              │
TemperatureStructure  ForceCalculator     │
          │                 │              │
          └────────┬────────┘              │
                   │                       │
                   ▼                       │
            ProgressionState◄──────────────┘
                   │
                   │ unlocks
                   │
         ┌─────────┴─────────┐
         │                   │
         ▼                   ▼
    New Recipes        New Materials
```

### Data Flow

```
Game Start
    │
    ▼
┌────────────────┐
│  Load JSON     │
│  Data Files    │
└────────┬───────┘
         │
         ▼
┌────────────────────────────┐
│  Build Registries          │
│  - Materials               │
│  - Tools                   │
│  - Modifiers               │
│  - Recipes                 │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Initialize Game Systems   │
│  - Temperature             │
│  - Force                   │
│  - Progression             │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Ready for Player          │
└────────────────────────────┘

Player Action
    │
    ▼
┌────────────────────────────┐
│  Query Registries          │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Execute Game Logic        │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Update State              │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Trigger Events            │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Update UI                 │
└────────────────────────────┘
```

---

## System Dependencies

### Dependency Graph

```
Level 0 (No Dependencies):
  - MaterialId
  - ToolDefinitionId
  - ModifierId
  - ResourceLocation

Level 1 (Foundation):
  - IMaterial
  - IMaterialStats
  - IToolStat
  - IModifier

Level 2 (Registries):
  - MaterialRegistry
  - ToolDefinitionRegistry
  - ModifierRegistry
  - RecipeRegistry

Level 3 (Core Systems):
  - Material (depends on MaterialRegistry)
  - ToolDefinition (depends on MaterialRegistry, ModifierRegistry)
  - Modifier (depends on ModifierRegistry)

Level 4 (Runtime):
  - ToolStack (depends on ToolDefinition, MaterialRegistry)
  - TemperatureStructure (depends on nothing, standalone)
  - ForceCalculator (depends on nothing, static)

Level 5 (High-Level):
  - ProgressionState (depends on Temperature, Force)
  - CraftingSystem (depends on Recipes, Tools, Materials)
  - SmelterySystem (depends on Temperature, Recipes, Materials)
```

### Critical Path

```
MaterialRegistry → ToolDefinition → ToolStack → Player Usage
                ↓
        ModifierRegistry → Modifiers → Tool Enhancement
                ↓
        RecipeRegistry → Crafting → New Tools/Materials
```

---

## Event System

### Event Flow

```
┌─────────────────────────────────────────────────┐
│              EVENT PUBLISHERS                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   Tool   │  │Material  │  │Modifier  │     │
│  │  System  │  │ System   │  │ System   │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
└───────┼─────────────┼─────────────┼────────────┘
        │             │             │
        │ fires       │ fires       │ fires
        │             │             │
        ▼             ▼             ▼
┌─────────────────────────────────────────────────┐
│              EVENT BUS / MANAGER                 │
│  ┌───────────────────────────────────────────┐  │
│  │  Event Queue                              │  │
│  │  - ToolCreatedEvent                       │  │
│  │  - ToolDamagedEvent                       │  │
│  │  - ToolBrokenEvent                        │  │
│  │  - MaterialUnlockedEvent                  │  │
│  │  - ModifierAppliedEvent                   │  │
│  │  - TemperatureReachedEvent                │  │
│  │  - ForceAchievedEvent                     │  │
│  └───────────────────────────────────────────┘  │
└───────────────────────┬─────────────────────────┘
                        │ dispatches
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│     UI       │ │ Progression  │ │   Audio      │
│   System     │ │   System     │ │   System     │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Event Classes

```csharp
// Base Event
public abstract class GameEvent
{
    public DateTime Timestamp { get; } = DateTime.Now;
    public string EventType => GetType().Name;
}

// Tool Events
public class ToolCreatedEvent : GameEvent
{
    public ToolStack Tool { get; set; }
    public List<MaterialId> Materials { get; set; }
}

public class ToolDamagedEvent : GameEvent
{
    public ToolStack Tool { get; set; }
    public int DamageAmount { get; set; }
    public int RemainingDurability { get; set; }
}

public class ToolBrokenEvent : GameEvent
{
    public ToolStack Tool { get; set; }
}

public class ModifierAppliedEvent : GameEvent
{
    public ToolStack Tool { get; set; }
    public ModifierId Modifier { get; set; }
    public int Level { get; set; }
}

// Progression Events
public class TemperatureReachedEvent : GameEvent
{
    public float Temperature { get; set; }
    public TemperatureTier Tier { get; set; }
}

public class ForceAchievedEvent : GameEvent
{
    public float Force { get; set; }
    public ForceTier Tier { get; set; }
}

public class RecipeUnlockedEvent : GameEvent
{
    public string RecipeId { get; set; }
}

// Event Manager
public class EventManager : MonoBehaviour
{
    private static EventManager instance;
    public static EventManager Instance
    {
        get
        {
            if (instance == null)
            {
                var go = new GameObject("EventManager");
                instance = go.AddComponent<EventManager>();
                DontDestroyOnLoad(go);
            }
            return instance;
        }
    }

    private Dictionary<Type, List<Delegate>> listeners = new Dictionary<Type, List<Delegate>>();

    public void Subscribe<T>(Action<T> listener) where T : GameEvent
    {
        Type eventType = typeof(T);
        if (!listeners.ContainsKey(eventType))
        {
            listeners[eventType] = new List<Delegate>();
        }
        listeners[eventType].Add(listener);
    }

    public void Unsubscribe<T>(Action<T> listener) where T : GameEvent
    {
        Type eventType = typeof(T);
        if (listeners.ContainsKey(eventType))
        {
            listeners[eventType].Remove(listener);
        }
    }

    public void Publish<T>(T gameEvent) where T : GameEvent
    {
        Type eventType = typeof(T);
        if (listeners.ContainsKey(eventType))
        {
            foreach (var listener in listeners[eventType].ToList())
            {
                ((Action<T>)listener)?.Invoke(gameEvent);
            }
        }
    }
}

// Usage Example
public class ToolSystem : MonoBehaviour
{
    public void CreateTool(ToolDefinition definition, List<MaterialId> materials)
    {
        var tool = ToolStack.Create(definition, materials);
        
        // Publish event
        EventManager.Instance.Publish(new ToolCreatedEvent
        {
            Tool = tool,
            Materials = materials
        });
    }
}

public class UIManager : MonoBehaviour
{
    private void Start()
    {
        // Subscribe to events
        EventManager.Instance.Subscribe<ToolCreatedEvent>(OnToolCreated);
        EventManager.Instance.Subscribe<ToolBrokenEvent>(OnToolBroken);
    }

    private void OnDestroy()
    {
        // Unsubscribe
        EventManager.Instance.Unsubscribe<ToolCreatedEvent>(OnToolCreated);
        EventManager.Instance.Unsubscribe<ToolBrokenEvent>(OnToolBroken);
    }

    private void OnToolCreated(ToolCreatedEvent evt)
    {
        Debug.Log($"New tool created: {evt.Tool.Definition.DisplayName}");
        // Update UI
    }

    private void OnToolBroken(ToolBrokenEvent evt)
    {
        Debug.Log($"Tool broke: {evt.Tool.Definition.DisplayName}");
        // Show notification
    }
}
```

---

## Initialization Sequence

### Game Startup

```
1. Unity Scene Load
   ↓
2. Create Core Managers
   - EventManager
   - SaveSystem
   - ProgressionManager
   ↓
3. Load Registries
   - MaterialRegistry.LoadFromResources()
   - ToolDefinitionRegistry.LoadFromResources()
   - ModifierRegistry.LoadFromResources()
   - RecipeRegistry.LoadFromResources()
   ↓
4. Load StreamingAssets Data
   - Custom materials from JSON
   - Custom tools from JSON
   - Custom recipes from JSON
   ↓
5. Merge Data
   - Combine built-in with custom content
   - Resolve dependencies
   - Validate data integrity
   ↓
6. Initialize Game Systems
   - TemperatureSystem.Initialize()
   - ForceSystem.Initialize()
   - CraftingSystem.Initialize()
   ↓
7. Load Save File (if exists)
   - Load player data
   - Load world state
   - Load progression
   ↓
8. Initialize UI
   - Main menu or game HUD
   - Bind event listeners
   ↓
9. Ready for Player Input
```

### C# Implementation

```csharp
public class GameInitializer : MonoBehaviour
{
    [Header("Registries")]
    [SerializeField] private MaterialRegistry materialRegistry;
    [SerializeField] private ToolDefinitionRegistry toolRegistry;
    [SerializeField] private ModifierRegistry modifierRegistry;
    [SerializeField] private RecipeRegistry recipeRegistry;

    [Header("Systems")]
    [SerializeField] private GameObject temperatureSystemPrefab;
    [SerializeField] private GameObject craftingSystemPrefab;

    private void Awake()
    {
        StartCoroutine(InitializeGame());
    }

    private IEnumerator InitializeGame()
    {
        // Step 1: Create managers
        yield return CreateManagers();

        // Step 2: Load registries
        yield return LoadRegistries();

        // Step 3: Load custom data
        yield return LoadCustomData();

        // Step 4: Initialize systems
        yield return InitializeSystems();

        // Step 5: Load save or start new game
        yield return LoadGameState();

        // Step 6: Initialize UI
        yield return InitializeUI();

        // Step 7: Ready
        OnGameReady();
    }

    private IEnumerator CreateManagers()
    {
        // Ensure EventManager exists
        _ = EventManager.Instance;
        
        Debug.Log("Managers created");
        yield return null;
    }

    private IEnumerator LoadRegistries()
    {
        // Registries load automatically from Resources
        // Wait a frame for them to initialize
        yield return null;

        Debug.Log($"Loaded {MaterialRegistry.Instance.GetAllMaterials().Count()} materials");
        Debug.Log($"Loaded {ToolDefinitionRegistry.Instance.GetAll().Count()} tool definitions");
        Debug.Log($"Loaded {ModifierRegistry.Instance.GetAll().Count()} modifiers");
    }

    private IEnumerator LoadCustomData()
    {
        // Load JSON files from StreamingAssets
        string dataPath = Path.Combine(Application.streamingAssetsPath, "Data");
        
        if (Directory.Exists(dataPath))
        {
            var jsonLoader = new JSONDataLoader();
            yield return jsonLoader.LoadCustomContent(dataPath);
        }
        
        Debug.Log("Custom data loaded");
    }

    private IEnumerator InitializeSystems()
    {
        // Instantiate system objects
        if (temperatureSystemPrefab != null)
        {
            Instantiate(temperatureSystemPrefab);
        }

        if (craftingSystemPrefab != null)
        {
            Instantiate(craftingSystemPrefab);
        }

        Debug.Log("Systems initialized");
        yield return null;
    }

    private IEnumerator LoadGameState()
    {
        // Check for save file
        if (SaveSystem.SaveExists("autosave"))
        {
            var saveData = SaveSystem.Load("autosave");
            if (saveData != null)
            {
                // Apply save data to game state
                yield return ApplySaveData(saveData);
                Debug.Log("Save loaded");
            }
        }
        else
        {
            // Start new game
            Debug.Log("Starting new game");
        }

        yield return null;
    }

    private IEnumerator ApplySaveData(GameSaveData saveData)
    {
        // Restore player state
        // Restore world state
        // Restore progression
        yield return null;
    }

    private IEnumerator InitializeUI()
    {
        // Initialize UI systems
        // Bind event handlers
        yield return null;
        Debug.Log("UI initialized");
    }

    private void OnGameReady()
    {
        Debug.Log("Game ready!");
        EventManager.Instance.Publish(new GameReadyEvent());
    }
}

public class GameReadyEvent : GameEvent { }
```

---

## Performance Optimization Strategy

### Frame Budget Allocation

```
60 FPS = 16.67ms per frame

Breakdown:
┌────────────────────────────────────┐
│ Rendering: 8ms (48%)               │
├────────────────────────────────────┤
│ Game Logic: 5ms (30%)              │
│  - Registry lookups: 0.5ms         │
│  - Stat calculations: 1ms          │
│  - Temperature updates: 1ms        │
│  - Event processing: 0.5ms         │
│  - Other systems: 2ms              │
├────────────────────────────────────┤
│ Physics: 2ms (12%)                 │
├────────────────────────────────────┤
│ Audio: 0.5ms (3%)                  │
├────────────────────────────────────┤
│ Other: 1ms (7%)                    │
└────────────────────────────────────┘
```

### Optimization Techniques

**1. Object Pooling**
```csharp
// Pool frequently created objects
public class ToolPool : MonoBehaviour
{
    private Queue<GameObject> pool = new Queue<GameObject>();
    
    public GameObject Get()
    {
        if (pool.Count > 0)
        {
            var obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(prefab);
    }
    
    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        pool.Enqueue(obj);
    }
}
```

**2. Caching**
```csharp
// Cache component references
private Renderer _renderer;
private Renderer Renderer => _renderer ?? (_renderer = GetComponent<Renderer>());

// Cache registry lookups
private Dictionary<MaterialId, MaterialAsset> materialCache = new();
```

**3. Lazy Loading**
```csharp
// Load data only when needed
private MaterialAsset GetMaterial(MaterialId id)
{
    if (!materialCache.ContainsKey(id))
    {
        materialCache[id] = MaterialRegistry.Instance.GetMaterial(id);
    }
    return materialCache[id];
}
```

**4. Batch Processing**
```csharp
// Update temperature structures in batches
public class TemperatureManager : MonoBehaviour
{
    private List<TemperatureStructure> structures = new();
    private int currentBatch = 0;
    private const int BatchSize = 10;
    
    private void Update()
    {
        // Update only a batch per frame
        int start = currentBatch * BatchSize;
        int end = Mathf.Min(start + BatchSize, structures.Count);
        
        for (int i = start; i < end; i++)
        {
            structures[i].UpdateTemperature(Time.deltaTime);
        }
        
        currentBatch = (currentBatch + 1) % Mathf.CeilToInt(structures.Count / (float)BatchSize);
    }
}
```

**5. Unity Jobs System**
```csharp
using Unity.Jobs;
using Unity.Collections;
using Unity.Burst;

[BurstCompile]
struct CalculateStatsJob : IJobParallelFor
{
    [ReadOnly] public NativeArray<float> baseDurability;
    [ReadOnly] public NativeArray<float> multipliers;
    public NativeArray<float> results;
    
    public void Execute(int index)
    {
        results[index] = baseDurability[index] * multipliers[index];
    }
}

// Usage
public void CalculateToolStats(List<ToolStack> tools)
{
    var job = new CalculateStatsJob
    {
        baseDurability = new NativeArray<float>(tools.Count, Allocator.TempJob),
        multipliers = new NativeArray<float>(tools.Count, Allocator.TempJob),
        results = new NativeArray<float>(tools.Count, Allocator.TempJob)
    };
    
    // Fill input arrays
    for (int i = 0; i < tools.Count; i++)
    {
        job.baseDurability[i] = tools[i].GetBaseDurability();
        job.multipliers[i] = tools[i].GetMultiplier();
    }
    
    // Schedule job
    var handle = job.Schedule(tools.Count, 64);
    handle.Complete();
    
    // Read results
    for (int i = 0; i < tools.Count; i++)
    {
        tools[i].SetDurability(job.results[i]);
    }
    
    // Dispose
    job.baseDurability.Dispose();
    job.multipliers.Dispose();
    job.results.Dispose();
}
```

---

## Error Handling

### Error Categories and Strategies

```csharp
public enum ErrorSeverity
{
    Info,       // Informational, no action needed
    Warning,    // Potential issue, continue with fallback
    Error,      // Functional issue, try recovery
    Critical    // Fatal error, cannot continue
}

public class ErrorHandler
{
    public static void Handle(Exception ex, ErrorSeverity severity, string context)
    {
        // Log error
        string message = $"[{severity}] {context}: {ex.Message}";
        
        switch (severity)
        {
            case ErrorSeverity.Info:
                Debug.Log(message);
                break;
                
            case ErrorSeverity.Warning:
                Debug.LogWarning(message);
                break;
                
            case ErrorSeverity.Error:
                Debug.LogError(message);
                TryRecovery(ex, context);
                break;
                
            case ErrorSeverity.Critical:
                Debug.LogError(message);
                HandleCriticalError(ex, context);
                break;
        }
    }
    
    private static void TryRecovery(Exception ex, string context)
    {
        // Attempt to recover from error
        if (context.Contains("Material"))
        {
            // Use default material
            Debug.Log("Using default material as fallback");
        }
        else if (context.Contains("Tool"))
        {
            // Clear and rebuild tool
            Debug.Log("Attempting to rebuild tool");
        }
    }
    
    private static void HandleCriticalError(Exception ex, string context)
    {
        // Save emergency backup
        try
        {
            SaveSystem.Save(GameManager.Instance.GetSaveData(), "emergency_backup");
        }
        catch { }
        
        // Show error dialog to player
        UIManager.Instance.ShowErrorDialog(
            "Critical Error",
            $"A critical error occurred: {ex.Message}\nThe game will now exit. Your progress has been saved."
        );
        
        // Exit gracefully
        Application.Quit();
    }
}
```

---

## Testing Strategy

### Test Pyramid

```
                    ┌─────────┐
                    │   E2E   │ (10%)
                    │  Tests  │
                ┌───┴─────────┴───┐
                │  Integration    │ (30%)
                │     Tests       │
            ┌───┴─────────────────┴───┐
            │      Unit Tests         │ (60%)
            │                         │
            └─────────────────────────┘
```

**Unit Tests (60%)**
- Test individual classes and methods
- Fast execution (< 100ms each)
- No dependencies

**Integration Tests (30%)**
- Test system interactions
- Medium execution (< 1s each)
- Use real registries

**E2E Tests (10%)**
- Test complete workflows
- Slow execution (seconds to minutes)
- Full game simulation

---

## Conclusion

This architecture provides:
- **Modularity**: Systems can be developed independently
- **Scalability**: Easy to add new materials, tools, modifiers
- **Maintainability**: Clear separation of concerns
- **Performance**: Optimized for 60 FPS target
- **Testability**: Comprehensive testing strategy
- **Extensibility**: Plugin/mod support built-in

---

**Related Documents:**
- [All Class Diagrams](./README.md)
- [Technical Design Document](../TechnicalDesign/TDD-01-Overview.md)
- [C# Implementation Guide](../CSharp-Unity-Port/02-Implementation-Guide.md)
