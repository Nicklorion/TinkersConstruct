# Technical Design Document - Overview

## Document Purpose

This Technical Design Document (TDD) provides comprehensive technical specifications for implementing the Forge & Force crafting game as a standalone system. It serves as a bridge between the Game Design Document (game mechanics) and actual implementation.

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐      │
│  │   UI    │  │ Render  │  │  Input  │  │  Audio   │      │
│  │ System  │  │ System  │  │ Handler │  │  System  │      │
│  └─────────┘  └─────────┘  └─────────┘  └──────────┘      │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                    GAME LOGIC LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Material   │  │     Tool     │  │   Modifier   │      │
│  │   System     │  │   System     │  │   System     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Smeltery    │  │   Recipe     │  │  Progression │      │
│  │   System     │  │   System     │  │   System     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                    DATA LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Registry    │  │ Save/Load    │  │   JSON       │      │
│  │  System      │  │  Manager     │  │  Parser      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

#### 1. Presentation Layer
- **UI System**: Menus, HUD, crafting interfaces
- **Render System**: 3D models, materials, effects
- **Input Handler**: Player input processing
- **Audio System**: Sound effects and music

#### 2. Game Logic Layer
- **Material System**: Material definitions, stats, traits
- **Tool System**: Tool creation, stats, durability
- **Modifier System**: Upgrades and enhancements
- **Smeltery System**: Metal processing, alloy creation
- **Recipe System**: Crafting recipes and validation
- **Progression System**: Temperature and Force gates

#### 3. Data Layer
- **Registry System**: Content registration and lookup
- **Save/Load Manager**: Game state persistence
- **JSON Parser**: Data file loading and validation

---

## Core Systems Detail

### 1. Material System

**Purpose**: Manage all materials and their properties

**Components:**
- MaterialRegistry: Singleton registry
- Material: Individual material instances
- MaterialStats: Per-stat-type properties
- MaterialTraits: Special abilities

**Data Flow:**
```
JSON Files → MaterialLoader → Validation → MaterialRegistry → Game Systems
```

**Key Algorithms:**

**Material Stat Lookup:**
```
Algorithm: GetMaterialStats(materialId, statType)
Input: Material ID, Stat Type ID
Output: Material Stats or null

1. registry = MaterialRegistry.Instance
2. material = registry.GetMaterial(materialId)
3. if material == null:
   return null
4. stats = registry.GetStats(materialId, statType)
5. return stats
```

**Performance Requirements:**
- Material lookup: O(1) - use hash map
- Stats lookup: O(1) - use hash map
- Memory: ~100 materials × 500 bytes = ~50 KB

### 2. Tool System

**Purpose**: Create, manage, and track tool instances

**Components:**
- ToolDefinitionRegistry: Tool types
- ToolStack: Individual tool instances
- ToolStats: Dynamic stat calculation
- ToolNBT: Data persistence

**Data Flow:**
```
Definition + Materials → Stat Calculation → Tool Creation → NBT Storage → Usage → Stat Updates
```

**Key Algorithms:**

**Tool Stat Calculation:**
```
Algorithm: CalculateToolStats(definition, materials)
Input: Tool Definition, List of Material IDs
Output: Dictionary of calculated stats

1. Initialize stats = {}
2. For each stat in definition.stats:
   stats[stat] = stat.getDefault()
3. For i, part in enumerate(definition.parts):
   material = materials[i]
   partStats = GetMaterialStats(material, part.statType)
   if part.statType == "head":
      // Head: Add stats directly
      for stat in partStats:
         stats[stat] += partStats[stat]
   elif part.statType == "handle":
      // Handle: Multiply stats
      for stat in partStats:
         stats[stat] *= partStats[stat]
4. Apply traits as modifiers
5. For each modifier in tool.modifiers:
   modifier.modifyStats(stats)
6. Clamp all stats to valid ranges
7. return stats
```

**Performance Requirements:**
- Tool creation: < 5ms
- Stat calculation: < 1ms
- Memory per tool: ~500-1000 bytes

### 3. Modifier System

**Purpose**: Apply upgrades and special abilities to tools

**Components:**
- ModifierRegistry: Available modifiers
- ModifierInstance: Applied modifier
- ModifierHooks: Behavior callbacks
- ModifierEffects: Stat modifications

**Data Flow:**
```
Modifier Request → Validation → Application → Hook Registration → Effect Processing
```

**Key Algorithms:**

**Modifier Application:**
```
Algorithm: ApplyModifier(tool, modifierId, level)
Input: Tool Stack, Modifier ID, Level
Output: Success/Failure + Updated Tool

1. modifier = ModifierRegistry.Get(modifierId)
2. if modifier == null:
   return Failure("Unknown modifier")
3. if not modifier.canApply(tool):
   return Failure("Cannot apply modifier")
4. if tool.modifierSlots < modifier.slotsRequired(level):
   return Failure("Not enough slots")
5. existing = tool.getModifier(modifierId)
6. if existing:
   newLevel = existing.level + level
   if newLevel > modifier.maxLevel:
      return Failure("Max level reached")
   tool.updateModifier(modifierId, newLevel)
7. else:
   tool.addModifier(modifierId, level)
8. tool.modifierSlots -= modifier.slotsRequired(level)
9. tool.recalculateStats()
10. return Success(tool)
```

**Performance Requirements:**
- Modifier application: < 10ms
- Hook execution: < 0.1ms per hook
- Memory per modifier: ~100 bytes

### 4. Temperature & Force System

**Purpose**: Gate progression through physical principles

**Components:**
- TemperatureController: Heat management
- ForceCalculator: Force application
- ProgressionGates: Unlock validation
- HeatSource: Temperature providers

**Temperature Mechanics:**

**Heat Transfer Algorithm:**
```
Algorithm: UpdateTemperature(structure, deltaTime)
Input: Temperature structure, Time elapsed
Output: Updated temperature

1. currentTemp = structure.temperature
2. targetTemp = structure.heatSource.getTemperature()
3. ambientTemp = environment.ambientTemperature
4. insulation = structure.getInsulation()

5. // Heat from source
   if structure.hasActiveFuel():
      heatGain = (targetTemp - currentTemp) * structure.efficiency * deltaTime
   else:
      heatGain = 0

6. // Heat loss to environment
   heatLoss = (currentTemp - ambientTemp) / insulation * deltaTime

7. // Update temperature
   newTemp = currentTemp + heatGain - heatLoss
   newTemp = clamp(newTemp, ambientTemp, maxTemp)

8. structure.temperature = newTemp
9. return newTemp
```

**Force Mechanics:**

**Force Application:**
```
Algorithm: ApplyForce(tool, target, action)
Input: Tool, Target material, Action type
Output: Success/Failure + Material state

1. toolForce = tool.getForceCapacity()
2. materialResistance = target.getHardness()
3. requiredForce = materialResistance * action.forceMultiplier

4. if toolForce < requiredForce:
   return Failure("Insufficient force")

5. // Calculate tool damage
   forceDelta = requiredForce - toolForce
   if forceDelta > 0:
      toolDamage = action.getDamage() * (1 + forceDelta / requiredForce)
   else:
      toolDamage = action.getDamage()

6. tool.damage += toolDamage

7. // Process action
   if action.type == "impact":
      result = ProcessImpact(target, toolForce)
   elif action.type == "sustained":
      result = ProcessSustained(target, toolForce, action.duration)
   elif action.type == "precise":
      result = ProcessPrecise(target, toolForce, tool.precision)

8. return Success(result)
```

**Performance Requirements:**
- Temperature update: < 0.5ms per structure per frame
- Force calculation: < 0.1ms per application
- Heat propagation: Background thread for large structures

### 5. Smeltery/Forge System

**Purpose**: Process materials through heat

**Components:**
- MultiblockStructure: Physical structure
- FluidTank: Molten material storage
- MeltingRecipe: Material → fluid conversion
- CastingRecipe: Fluid → item conversion
- AlloyRecipe: Multiple fluids → new fluid

**Multiblock Validation:**
```
Algorithm: ValidateMultiblock(structure, blocks)
Input: Structure definition, Placed blocks
Output: Valid/Invalid + Controller position

1. // Find controller block
   controller = FindBlock(blocks, "controller")
   if controller == null:
      return Invalid("No controller")

2. // Validate structure pattern
   for pattern in structure.patterns:
      if not MatchPattern(blocks, pattern, controller.position):
         return Invalid("Invalid structure")

3. // Calculate capacity
   capacity = 0
   for block in blocks:
      if block.type == "tank":
         capacity += block.capacity

4. // Calculate temperature capability
   maxTemp = min([block.maxTemp for block in blocks])

5. structure.isValid = true
   structure.capacity = capacity
   structure.maxTemperature = maxTemp
   structure.controller = controller

6. return Valid(structure)
```

**Melting Process:**
```
Algorithm: ProcessMelting(smeltery, deltaTime)
Input: Smeltery structure, Time elapsed
Output: Updated fluid contents

1. if not smeltery.isValid or not smeltery.hasHeat():
   return

2. currentTemp = smeltery.temperature

3. // Process items in smeltery
   for item in smeltery.inputItems:
      recipe = FindMeltingRecipe(item)
      if recipe == null:
         continue
      
      if currentTemp >= recipe.temperature:
         // Melt progress
         progress = deltaTime / recipe.time
         item.meltProgress += progress
         
         if item.meltProgress >= 1.0:
            // Item fully melted
            fluid = recipe.outputFluid
            amount = recipe.outputAmount * item.count
            smeltery.addFluid(fluid, amount)
            smeltery.removeItem(item)

4. // Check for alloy formation
   alloy = CheckAlloyFormation(smeltery.fluids)
   if alloy:
      CreateAlloy(smeltery, alloy)

5. return smeltery.fluids
```

**Performance Requirements:**
- Multiblock validation: < 50ms on structure change
- Melting process: < 1ms per frame per active smeltery
- Alloy checking: < 0.5ms per fluid change
- Memory: ~5-10 KB per active smeltery

### 6. Recipe System

**Purpose**: Define and process all crafting operations

**Components:**
- RecipeRegistry: All recipes
- RecipeType: Different recipe categories
- RecipeMatcher: Find matching recipes
- RecipeProcessor: Execute recipes

**Recipe Matching:**
```
Algorithm: FindMatchingRecipe(recipeType, inputs)
Input: Recipe type, Input items/fluids
Output: Matching recipe or null

1. recipes = RecipeRegistry.GetRecipes(recipeType)
2. for recipe in recipes:
   if recipe.matches(inputs):
      return recipe
3. return null

// Recipe matching logic
Algorithm: Recipe.matches(inputs)
1. if inputs.length != this.inputs.length:
   return false
2. for i, requiredInput in enumerate(this.inputs):
   actualInput = inputs[i]
   if not requiredInput.matches(actualInput):
      return false
3. return true
```

**Recipe Types:**
- **Part Building**: Material + Pattern → Tool Part
- **Tool Assembly**: Parts → Tool
- **Melting**: Solid → Fluid (temperature-based)
- **Casting**: Fluid + Mold → Item
- **Alloying**: Multiple Fluids → New Fluid
- **Modifier Application**: Tool + Materials → Modified Tool

**Performance Requirements:**
- Recipe lookup: O(n) worst case, optimizable with indexing
- Recipe execution: < 5ms
- Memory: ~200 bytes per recipe × 500 recipes = ~100 KB

---

## Data Formats

### JSON Structure Standards

**Material Definition:**
```json
{
  "id": "namespace:material_name",
  "craftable": true,
  "tier": 2,
  "sortOrder": 150,
  "hidden": false,
  "redirect": {
    "condition": {"type": "config", "path": "enable_material"},
    "target": "namespace:fallback"
  }
}
```

**Material Stats:**
```json
{
  "durability": 250,
  "mining_speed": 7.0,
  "mining_tier": 2,
  "attack": 2.5,
  "melting_point": 1200,
  "hardness": 5
}
```

**Tool Definition:**
```json
{
  "id": "namespace:tool_name",
  "item": "namespace:tool_item",
  "parts": [
    {"name": "head", "stat_type": "head", "item": "part_item"},
    {"name": "handle", "stat_type": "handle", "item": "handle_item"}
  ],
  "stats": ["durability", "mining_speed", "harvest_tier"],
  "modifier_slots": 3
}
```

**Recipe:**
```json
{
  "type": "namespace:recipe_type",
  "inputs": [
    {"item": "namespace:item", "count": 1},
    {"tag": "forge:ingots/iron", "count": 2}
  ],
  "output": {"item": "namespace:result", "count": 1},
  "conditions": [
    {"type": "temperature", "min": 1200, "max": 1600}
  ]
}
```

### Save File Format

**Player Save Data:**
```json
{
  "version": "1.0.0",
  "player": {
    "position": [100, 64, 200],
    "inventory": [...],
    "progression": {
      "max_temperature": 1200,
      "max_force": 5000,
      "unlocked_recipes": [...]
    }
  },
  "world": {
    "structures": [...],
    "active_smelteries": [...]
  }
}
```

---

## Performance Targets

### Frame Budget (60 FPS = 16.67ms)

| System | Budget | Notes |
|--------|--------|-------|
| Rendering | 8ms | 3D, effects, UI |
| Game Logic | 5ms | All systems combined |
| Physics | 2ms | Tool/block interactions |
| Audio | 0.5ms | Sound processing |
| Other | 1ms | Input, networking, etc. |

### Memory Budget

| Category | Limit | Notes |
|----------|-------|-------|
| Materials | 100 KB | ~100 materials |
| Tools | 1 MB | ~1000 tools (player + world) |
| Recipes | 100 KB | ~500 recipes |
| Textures | 200 MB | Models and UI |
| Audio | 50 MB | Sound effects and music |
| Total | ~500 MB | Base game |

### Load Times

- Initial load: < 10 seconds
- Material registry: < 1 second
- World load: < 5 seconds
- Recipe index: < 1 second

---

## Threading Model

### Main Thread
- Game logic updates
- Rendering
- Input processing
- UI updates

### Background Threads
- Asset loading
- Save file I/O
- Large structure validation
- Recipe indexing

### Thread Safety
- Use locks for shared data access
- Immutable data structures where possible
- Message passing for cross-thread communication

---

## Error Handling

### Error Categories

**1. Data Errors**
- Missing material definitions
- Invalid JSON syntax
- Circular dependencies
- Missing textures/models

**Strategy**: Log error, use fallback, continue operation

**2. Logic Errors**
- Invalid recipe matches
- Stat calculation overflow
- Tool creation failures

**Strategy**: Return error result, display to user, allow retry

**3. Critical Errors**
- Save file corruption
- Memory exhaustion
- System resource failures

**Strategy**: Display error dialog, attempt graceful shutdown, preserve data

### Logging Levels

- **ERROR**: Critical failures that break functionality
- **WARN**: Issues that don't break but should be fixed
- **INFO**: Important state changes
- **DEBUG**: Detailed execution information
- **TRACE**: Extremely detailed (development only)

---

## Testing Strategy

### Unit Tests
- Test individual classes and methods
- Mock dependencies
- Coverage target: 80%+

### Integration Tests
- Test system interactions
- Use real data files
- Validate data flow

### Performance Tests
- Measure frame times
- Profile memory usage
- Stress test with many tools/structures

### Playtest Scenarios
- Complete progression path
- Edge cases (min/max values)
- Error recovery
- Save/load cycles

---

## Deployment

### Build Process
```
1. Compile C# code
2. Bundle assets
3. Generate data file index
4. Create platform-specific builds
5. Run automated tests
6. Package for distribution
```

### Platforms
- Windows: Standalone executable
- MacOS: .app bundle
- Linux: AppImage or native binary

### Distribution
- Steam (primary)
- Itch.io (secondary)
- Direct download

---

## Conclusion

This Technical Design Document provides the foundation for implementing the Forge & Force crafting game. Each system is designed to be:

- **Modular**: Systems can be developed and tested independently
- **Performant**: Meets frame rate and memory targets
- **Extensible**: Easy to add new materials, tools, modifiers
- **Maintainable**: Clear structure and documentation
- **Testable**: Comprehensive testing strategy

---

**Related Documents:**
- [Game Design Document](../GameDesign/GDD-01-Core-Concept.md)
- [Class Diagrams](../ClassDiagrams/README.md)
- [C# Implementation Guide](../CSharp-Unity-Port/01-Porting-Overview.md)
