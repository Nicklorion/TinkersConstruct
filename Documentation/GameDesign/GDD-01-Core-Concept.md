# Game Design Document - Core Concept

## Project Title: **Forge & Force** (Working Title)
*A Survival Crafting Game Focused on Temperature and Force Progression*

---

## Executive Summary

**Forge & Force** is a survival crafting game where players progress through technological eras by mastering two core mechanics: **Temperature** (thermal energy) and **Force** (mechanical energy). The game reimagines the Tinkers' Construct mod as a standalone experience, stripping away Minecraft-specific elements to create a focused crafting and progression system.

### Core Pillars

1. **Temperature Mastery** - From friction fires to industrial forges
2. **Force Application** - From stone hammers to precision tempering
3. **Material Customization** - Every tool is unique and upgradeable
4. **Progressive Complexity** - Simple beginnings, deep end-game systems
5. **Meaningful Choices** - Trade-offs between material properties

---

## Game Vision

### Genre
- **Primary**: Survival Crafting
- **Secondary**: Resource Management, Progression-Based Adventure
- **Target Platform**: PC (Windows, Mac, Linux) via Unity
- **Art Style**: Low-poly 3D with realistic material rendering

### Core Loop

```
Gather Resources → Craft/Upgrade Tools → Access New Resources → Unlock New Tech → Repeat
```

### Unique Selling Points (USPs)

1. **Temperature & Force as Progression Gates**: Unlike typical survival games that use arbitrary tech trees, all progression is grounded in physical principles
2. **Deep Tool Customization**: Build tools from components, each with unique material properties
3. **No Generic Recipes**: Every crafted item requires understanding of materials and their interactions
4. **Physics-Based Crafting**: Temperature requirements, force application, and material properties affect outcomes
5. **Non-Linear Progression**: Multiple paths to advancement through different material discoveries

---

## Core Gameplay Mechanics

### 1. Temperature System (Thermal Progression)

Temperature is one of the two primary progression gates in the game.

#### Temperature Tiers

| Tier | Name | Temperature Range | Unlocks | Fuel Sources |
|------|------|------------------|---------|--------------|
| 0 | **Ambient** | 0-50°C | Hand crafting, drying | Natural heat |
| 1 | **Friction Fire** | 50-400°C | Cooking, basic adhesives | Wood, dried plants |
| 2 | **Open Flame** | 400-800°C | Pine resin, wax, primitive casting | Charcoal, coal |
| 3 | **Clay Kiln** | 800-1200°C | Ceramics, copper, bronze | Improved coal, bellows |
| 4 | **Stone Forge** | 1200-1600°C | Iron, steel basics | Advanced coal, oxygen-enriched |
| 5 | **Brick Furnace** | 1600-2000°C | High carbon steel, alloys | Coke, forced air |
| 6 | **Advanced Forge** | 2000-3000°C | Tool steel, special alloys | Specialized fuels |

#### Temperature Mechanics

**Discovery Path:**
1. **Friction Fire** - Rub sticks together (minigame)
2. **Fire Management** - Learn to maintain temperature
3. **Kiln Construction** - Build first temperature-controlled structure
4. **Forge Mastery** - Advanced temperature control systems
5. **Quenching & Tempering** - Rapid temperature change for steel treatment

**Temperature Control:**
- **Fuel Quality**: Different fuels provide different heat levels
- **Oxygen Supply**: Bellows and forced air increase temperature
- **Insulation**: Better materials hold heat longer
- **Structure Design**: Multiblock structures affect efficiency

### 2. Force System (Mechanical Progression)

Force represents the player's ability to shape and work materials.

#### Force Tiers

| Tier | Name | Force Level | Unlocks | Tools |
|------|------|-------------|---------|-------|
| 0 | **Hand Force** | 1-10N | Breaking sticks, soft materials | Bare hands |
| 1 | **Wood Tools** | 10-100N | Wood cutting, soft stone | Wooden clubs, mallets |
| 2 | **Stone Tools** | 100-500N | Hard stone, basic mining | Stone axes, hammers |
| 3 | **Flint Tools** | 500-1000N | Precision cutting, carving | Flint blades, chisels |
| 4 | **Metal Tools** | 1000-5000N | Metal working, deep mining | Bronze/Iron tools |
| 5 | **Hardened Tools** | 5000-10000N | Hard metal working | Tempered steel tools |
| 6 | **Precision Tools** | 10000+N | Fine crafting, specialized work | Master-crafted tools |

#### Force Mechanics

**Application Methods:**
- **Impact Force**: Hammering, striking (force over short time)
- **Sustained Force**: Pressing, compacting (force over long time)
- **Precise Force**: Carving, cutting (controlled force)
- **Distributed Force**: Rolling, compressing (force across area)

**Tool Durability:**
- Tools degrade based on force applied vs. material hardness
- Harder materials require more force but last longer
- Tool breakage is gradual, with performance degradation

### 3. Material System

Materials are the heart of the crafting system. Each material has unique properties that affect tool performance.

#### Material Properties

Every material has these stats:

**Physical Properties:**
- **Durability**: How many uses before breaking
- **Hardness**: Resistance to deformation (affects force requirements)
- **Toughness**: Resistance to fracture
- **Density**: Weight and momentum properties
- **Flexibility**: Bending without breaking

**Thermal Properties:**
- **Melting Point**: Temperature required to liquify
- **Heat Capacity**: How much heat energy it can store
- **Thermal Conductivity**: How quickly heat spreads
- **Oxidation Temperature**: When it degrades in air

**Working Properties:**
- **Workability**: How easily it can be shaped
- **Edge Retention**: How long cutting edges stay sharp
- **Grain Structure**: Affects fracture patterns
- **Hardenability**: Response to heat treatment

#### Material Categories

**1. Primitive Materials (Tier 0-1)**
- **Wood** (Oak, Birch, Pine, etc.)
  - Easy to work, low durability
  - Tool handles, simple structures
  - Burns as fuel
- **Plant Fibers**
  - Binding, rope, simple cloth
  - Adhesive components when processed
- **Bone**
  - Medium durability, good for handles
  - Can be carved into tools
- **Stone** (Various types)
  - Hard, brittle, moderate durability
  - First real tools and structures

**2. Early Materials (Tier 2-3)**
- **Flint**
  - Very sharp edges, brittle
  - Best early cutting tool
  - Can create sparks for fire
- **Copper**
  - First metal, soft, easy to work
  - Good thermal conductor
  - Forms useful alloys
- **Bronze** (Copper + Tin)
  - First true metal alloy
  - Better than copper in all respects
  - Foundation of metal age
- **Clay/Ceramic**
  - Heat resistant structures
  - Molds for casting
  - Storage vessels

**3. Iron Age Materials (Tier 4)**
- **Iron** (Pure)
  - Strong but rusts easily
  - Can be worked at forge temperatures
  - Base for steel
- **Cast Iron**
  - Brittle but hard
  - Good for molds and structures
  - High carbon content
- **Wrought Iron**
  - Tough, malleable
  - Good for tools
  - Low carbon content

**4. Steel Age Materials (Tier 5-6)**
- **Mild Steel**
  - Balanced properties
  - Versatile tool material
  - Medium carbon (0.3-0.6%)
- **High Carbon Steel**
  - Hard, brittle
  - Excellent edge retention
  - Requires careful heat treatment
- **Tool Steel**
  - Specialized alloys
  - Superior tool performance
  - Complex heat treatment

**5. Special Materials**
- **Obsidian**
  - Extremely sharp but fragile
  - Surgical precision
  - Volcanic glass
- **Damascus/Pattern-Welded Steel**
  - Layered structure
  - Beautiful and functional
  - Requires master crafting
- **Titanium** (End-game)
  - Lightweight, corrosion resistant
  - Very high temperature requirement
  - Difficult to work

#### Material Discovery

Players discover materials through:
1. **Exploration**: Finding ore deposits, trees, clay beds
2. **Experimentation**: Mixing materials to create alloys
3. **Processing**: Smelting ores, treating materials
4. **Trade** (if multiplayer): Exchanging with other players

### 4. Tool System

Tools are not pre-made items but assembled from components.

#### Tool Structure

Every tool consists of:

**1. Tool Parts**
- **Head/Blade**: Primary working component
  - Determines main function (cutting, mining, etc.)
  - Material affects efficiency and durability
- **Handle/Shaft**: Gripping and leverage component
  - Affects tool speed and comfort
  - Material affects durability multiplier
- **Binding/Guard**: Connection component
  - Holds parts together
  - Material affects overall durability
- **Additional Parts** (tool-specific)
  - Cross guards, counterweights, etc.

**2. Tool Types**

**Harvesting Tools:**
- **Axe**: Tree cutting, wood working
  - Parts: Head, Handle, Binding
  - Stats: Cutting speed, durability
- **Pickaxe**: Mining, stone breaking
  - Parts: Head, Handle, Binding
  - Stats: Mining speed, harvest level
- **Shovel**: Digging, moving earth
  - Parts: Head, Handle
  - Stats: Dig speed, capacity

**Crafting Tools:**
- **Hammer**: Forging, shaping metal
  - Parts: Head, Handle
  - Stats: Force application, precision
- **Chisel**: Carving, detailed work
  - Parts: Blade, Handle
  - Stats: Precision, edge retention
- **Saw**: Cutting, precise splitting
  - Parts: Blade, Frame, Handle
  - Stats: Cut quality, speed

**Combat Tools:**
- **Sword**: Fighting, general combat
  - Parts: Blade, Guard, Handle
  - Stats: Damage, speed, reach
- **Spear**: Ranged melee, hunting
  - Parts: Head, Shaft
  - Stats: Reach, throwing damage
- **Bow**: Ranged combat
  - Parts: Limbs, String, Grip
  - Stats: Draw force, accuracy

#### Tool Stats

**Calculated Stats** (from materials):
- **Durability**: Total uses before breaking
- **Mining Speed**: How fast it breaks blocks
- **Harvest Level**: What tier materials it can harvest
- **Attack Damage**: Damage dealt in combat
- **Attack Speed**: How fast attacks can be made
- **Mining Area**: Size of area affected (for AOE tools)

**Dynamic Stats** (change during use):
- **Current Durability**: Remaining uses
- **Sharpness**: Current edge quality (affects efficiency)
- **Temperature**: Current heat (for hot-working tools)

#### Tool Crafting Process

1. **Part Creation**
   - Craft or cast individual parts
   - Material determines properties
   - Quality depends on tools and technique

2. **Assembly**
   - Combine parts at crafting station
   - Order matters for some tools
   - Creates base tool with combined stats

3. **Initial Sharpening** (optional)
   - Use grindstone to sharpen blades
   - Increases initial efficiency
   - Sets baseline for maintenance

4. **Customization** (advanced)
   - Apply modifiers (see section 5)
   - Enchant with special abilities
   - Add decorative elements

### 5. Modifier System

Modifiers are permanent upgrades applied to tools that grant special abilities or enhance properties.

#### Modifier Categories

**1. Enhancement Modifiers**
Improve base tool stats:
- **Sharpness**: Increases cutting efficiency
- **Reinforced**: Adds durability
- **Hardened**: Improves harvest level
- **Lightweight**: Increases mining/attack speed
- **Heavy**: Increases force/damage

**2. Ability Modifiers**
Grant special abilities:
- **Auto-Repair**: Slowly regenerates durability
- **Magnetic**: Pulls items from a distance
- **Smelting**: Auto-smelts harvested blocks
- **Silk Touch**: Harvests blocks intact
- **Fortune**: Increases resource yield
- **Area Mining**: Mine multiple blocks at once

**3. Utility Modifiers**
Add convenience features:
- **Tool Belt**: Quick-swap capability
- **Illumination**: Emits light
- **Waterproof**: Works underwater
- **Heat Resistance**: Protects from fire
- **Frost Touch**: Freezes liquids

**4. Trait Modifiers** (from materials)
Automatic bonuses from material choice:
- **Cheap**: Repairs for less material
- **Maintained**: Efficiency increases when nearly broken
- **Jagged**: Extra damage as tool breaks
- **Splintering**: Causes bleeding damage
- **Dense**: Extra knockback
- **Magnetic**: Natural item attraction

#### Modifier Application

**Requirements:**
- **Modifier Slots**: Each tool has limited slots
- **Prerequisite Modifiers**: Some require others first
- **Material Compatibility**: Not all modifiers work with all materials
- **Technology Level**: Must have unlocked the technology

**Application Process:**
1. Place tool in Modifier Workbench
2. Select desired modifier
3. Provide required materials/components
4. Apply (consumes 1 modifier slot)

**Modifier Levels:**
- Most modifiers can be applied multiple times
- Each application increases effect
- Higher levels may require more slots
- Maximum level varies by modifier

---

## Progression System

### Progression Philosophy

Progression in **Forge & Force** is gated by understanding and mastering two physical concepts: Temperature and Force. Players cannot simply "grind" to the next level - they must demonstrate understanding of the principles.

### Progression Stages

#### Stage 0: Survival Basics (0-2 hours)
**Goal**: Survive first day/night cycle

**Available:**
- Hand gathering (sticks, stones, plant fibers)
- Simple crafting (cordage, stone tools)
- Basic shelter construction
- Food gathering

**Temperature**: Ambient only
**Force**: Hand strength only

**Key Unlock**: Discovery of friction fire

#### Stage 1: Fire Discovery (2-5 hours)
**Goal**: Create and maintain fire, craft first advanced tools

**Unlocked by**: Creating friction fire
**Available:**
- Cooking food
- Primitive adhesives (pine resin, hide glue)
- Hardened wood tools
- Simple wooden structures
- Basic leather working

**Temperature**: Up to 400°C (open fire)
**Force**: Wooden tools (up to 100N)

**Key Unlock**: Stone tool crafting, clay discovery

#### Stage 2: Stone Age (5-10 hours)
**Goal**: Master stone working, build first permanent structures

**Unlocked by**: Creating refined stone tools
**Available:**
- Flint tools (sharp blades, precise cutting)
- Stone structures (simple houses)
- Clay pottery (storage)
- Advanced gathering (mining, logging)
- Wax rendering (candles, adhesives)

**Temperature**: Up to 800°C (hot fire + basic kiln)
**Force**: Stone and flint tools (up to 1000N)

**Key Unlock**: Clay kiln construction, copper ore identification

#### Stage 3: Bronze Age (10-20 hours)
**Goal**: Smelt first metals, create alloys, build forge

**Unlocked by**: Building operational clay kiln
**Available:**
- Copper smelting
- Bronze alloy creation (copper + tin)
- Metal casting (basic shapes)
- Improved kilns
- Ceramic molds
- Metal tools (first real tools)

**Temperature**: Up to 1200°C (advanced kiln + bellows)
**Force**: Bronze tools (up to 2000N)

**Key Unlock**: Iron ore smelting, steel basics

#### Stage 4: Iron Age (20-40 hours)
**Goal**: Master iron working, begin steel production

**Unlocked by**: Successfully smelting iron ore
**Available:**
- Iron smelting
- Wrought iron production
- Bloomery operation
- Stone forge construction
- Basic steel (low carbon)
- Metal tool refinement
- Quenching and tempering basics

**Temperature**: Up to 1600°C (stone forge)
**Force**: Iron tools (up to 5000N)

**Key Unlock**: High-carbon steel, advanced heat treatment

#### Stage 5: Steel Age (40-80 hours)
**Goal**: Perfect steel making, master heat treatment

**Unlocked by**: Creating high-quality steel
**Available:**
- Consistent steel production
- Multiple steel grades
- Advanced heat treatment (tempering, annealing)
- Hardening techniques
- Precision tools
- Complex alloys
- Advanced forge structures

**Temperature**: Up to 2000°C (brick furnace)
**Force**: Steel tools (up to 10000N)

**Key Unlock**: Pattern welding, specialized tool steel

#### Stage 6: Master Craftsman (80+ hours)
**Goal**: Create ultimate tools, discover rare materials

**Unlocked by**: Mastering multiple crafting disciplines
**Available:**
- Pattern-welded (Damascus) steel
- Tool steel alloys
- Specialized heat treatment
- Ultimate tools
- Advanced modifiers
- Titanium working (if discovered)
- Complete mastery of all systems

**Temperature**: Up to 3000°C (advanced furnaces)
**Force**: Master tools (unlimited practical force)

**End Goal**: Create the "perfect" tool for every situation

---

## World & Setting

### Environment Types

While not detailed here (depends on final implementation), the world should contain:

**Biomes for Material Discovery:**
- **Forest**: Wood types, plant fibers, wildlife
- **Mountains**: Stone, ore deposits, minerals
- **Plains**: Clay, common resources, farming
- **Desert**: Sand (glass), rare plants, specific ores
- **Volcanic**: Obsidian, high-temperature environment
- **Tundra**: Ice, specific wood types, unique materials

**Resource Distribution:**
- Common materials: Everywhere
- Uncommon materials: Specific biomes
- Rare materials: Rare spawns or biome-specific
- Legendary materials: Require extensive exploration or achievements

### Crafting Stations

#### Basic Stations (Always Available)
- **Crafting Log**: Simple hand crafting
- **Campfire**: Cooking, basic heating
- **Stone Platform**: Surface for material processing

#### Intermediate Stations (Unlocked Through Progression)
- **Part Builder**: Create tool components
- **Tool Assembly Station**: Combine parts into tools
- **Kiln**: High-temperature processing
- **Forge**: Metal smelting and working
- **Anvil**: Metal shaping with force
- **Grindstone**: Sharpening and maintenance

#### Advanced Stations
- **Modifier Workbench**: Apply tool modifiers
- **Blast Furnace**: High-efficiency smelting
- **Pattern Table**: Create casting patterns
- **Casting Station**: Pour molten metal into molds
- **Tempering Station**: Controlled heat treatment

#### Master Stations
- **Precision Forge**: Ultimate control over temperature and force
- **Alloy Smelter**: Complex multi-metal alloys
- **Research Table**: Experiment with new combinations

---

## Multiplayer Considerations

### Co-op Mode (2-4 players)
- Shared progression
- Distributed specialization (one player focuses on temperature, another on force)
- Shared crafting stations
- Collaborative builds

### Competitive Mode (Optional)
- Race to achieve certain progression milestones
- PvP with crafted tools
- Resource competition

---

## Retention & Engagement

### Short-Term Goals (Session-based)
- Gather resources for next tier
- Upgrade specific tool
- Build new crafting station
- Experiment with new material combination

### Mid-Term Goals (Week-based)
- Reach next progression stage
- Master a specific crafting discipline
- Complete material collection
- Build ultimate version of tool type

### Long-Term Goals (Month+)
- Complete all progression stages
- Create perfect tool set
- Discover all materials
- Master all crafting techniques
- Achieve 100% completion

### Replayability
- Different material paths
- Varied biome spawns
- Challenge modes (speedrun, limited resources)
- Randomized material properties (optional hard mode)

---

## Technical Requirements (High-Level)

### Minimum Specs
- OS: Windows 10, MacOS 10.14, Ubuntu 18.04
- CPU: Dual-core 2.5GHz
- RAM: 4GB
- GPU: Integrated graphics (Intel HD 4000 or equivalent)
- Storage: 2GB

### Recommended Specs
- OS: Latest Windows/Mac/Linux
- CPU: Quad-core 3.0GHz+
- RAM: 8GB
- GPU: Dedicated GPU (GTX 1050 or equivalent)
- Storage: 5GB SSD

---

## Conclusion

**Forge & Force** takes the deep crafting and progression systems of Tinkers' Construct and reimagines them as the core of a standalone survival game. By focusing on Temperature and Force as the two pillars of progression, the game creates a unique experience where mastery comes from understanding physical principles rather than arbitrary progression gates.

The game offers:
- **Depth**: Hundreds of material combinations and tool configurations
- **Progression**: Clear stages from primitive to master craftsman
- **Freedom**: Multiple paths to advancement
- **Mastery**: High skill ceiling for dedicated players
- **Accessibility**: Simple to start, complex to master

This design document serves as the foundation for development, providing clear vision and concrete systems that can be implemented and tested.

---

**Next Documents:**
- [GDD-02: Detailed Mechanics](./GDD-02-Detailed-Mechanics.md)
- [Class Diagrams](../ClassDiagrams/)
- [Technical Design Document](../TechnicalDesign/)
