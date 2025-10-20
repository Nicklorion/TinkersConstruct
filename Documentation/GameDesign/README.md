# Game Design Documentation - Forge & Force

## Overview

This directory contains comprehensive game design documentation for rebuilding Tinkers' Construct as a standalone survival crafting game called **Forge & Force**.

## Purpose

The game design documentation serves to:
1. Define the complete game concept and vision
2. Specify all game mechanics and systems
3. Detail the progression structure
4. Provide content specifications
5. Guide implementation decisions

## Documents

### [GDD-01: Core Concept](./GDD-01-Core-Concept.md)

**The main game design document** covering:

#### Executive Summary
- Game vision: Temperature and Force progression
- Core pillars and unique selling points
- Target platform and audience

#### Core Gameplay Mechanics
1. **Temperature System** - 7 tiers from friction fire to advanced forge
2. **Force System** - 7 tiers from bare hands to master tools
3. **Material System** - 50+ materials with detailed properties
4. **Tool System** - Customizable tools built from parts
5. **Modifier System** - Permanent upgrades and abilities

#### Progression System
- Stage 0: Survival Basics (0-2 hours)
- Stage 1: Fire Discovery (2-5 hours)
- Stage 2: Stone Age (5-10 hours)
- Stage 3: Bronze Age (10-20 hours)
- Stage 4: Iron Age (20-40 hours)
- Stage 5: Steel Age (40-80 hours)
- Stage 6: Master Craftsman (80+ hours)

#### World & Setting
- Biome types for material discovery
- Crafting station progression
- Resource distribution

#### Retention & Engagement
- Short-term goals (session-based)
- Mid-term goals (week-based)
- Long-term goals (month+)
- Replayability features

## Key Design Principles

### 1. Physics-Based Progression

Unlike typical survival games with arbitrary tech trees, progression is grounded in physical principles:

**Temperature**
- Real physics: heat transfer, insulation, fuel efficiency
- Natural discovery: friction → fire → kiln → forge
- Tangible milestones: 400°C, 800°C, 1200°C, etc.

**Force**
- Real mechanics: impact, sustained, precise, distributed
- Tool physics: stress, strain, material failure
- Clear progression: wood → stone → flint → metal → steel

### 2. Material-Centric Design

Every material is unique and meaningful:
- Physical properties (durability, hardness, toughness)
- Thermal properties (melting point, conductivity)
- Working properties (workability, edge retention)
- Trade-offs between properties

### 3. Customization Through Parts

Tools aren't pre-made items:
- Head determines primary function
- Handle affects multipliers
- Binding adds durability
- Each part can use different materials

### 4. Progressive Complexity

Start simple, become complex:
- Early: Basic tool crafting (5 minutes to learn)
- Mid: Temperature and force management (hours to master)
- Late: Alloy creation, heat treatment (deep systems)

### 5. Meaningful Choices

Every decision matters:
- Material selection affects tool performance
- Tool design affects usability
- Modifier choices define playstyle
- Progression path is player-driven

## Game Flow Example

### Typical Player Session (Hour 1-2)

```
1. Start with nothing
   ↓
2. Gather sticks and stones by hand
   ↓
3. Learn friction fire (minigame)
   ↓
4. Cook food, stay warm
   ↓
5. Create first stone tools
   ↓
6. Mine better materials
   ↓
7. Discover clay deposits
   ↓
8. Learn to build kiln (goal: temperature)
   ↓
9. Prepare for metal age
```

### Mid-Game Session (Hour 20-30)

```
1. Have iron tools and basic steel
   ↓
2. Build advanced forge
   ↓
3. Experiment with steel grades
   ↓
4. Learn quenching and tempering
   ↓
5. Create superior tools
   ↓
6. Unlock new mining areas
   ↓
7. Discover rare materials
   ↓
8. Apply advanced modifiers
   ↓
9. Optimize tool set for efficiency
```

### End-Game Session (Hour 80+)

```
1. Master of all systems
   ↓
2. Create pattern-welded steel
   ↓
3. Craft perfect tools
   ↓
4. Experiment with titanium
   ↓
5. Min-max for specific tasks
   ↓
6. Complete material collection
   ↓
7. Create ultimate tool set
   ↓
8. Master craftsman achievement
```

## Design Pillars Deep Dive

### Pillar 1: Temperature Mastery

**Philosophy**: Heat is power, control is mastery

**Progression**:
1. Discovery (friction fire)
2. Understanding (fuel types, oxygen)
3. Control (kiln construction)
4. Mastery (forge operation)
5. Expertise (quenching, tempering)

**Player Experience**:
- Tangible progress (visible temperature)
- Clear goals (reach next tier)
- Real physics (heat transfer)
- Satisfying mastery (perfect heat control)

### Pillar 2: Force Application

**Philosophy**: Right tool for right job, skill matters

**Progression**:
1. Bare hands (weak, limited)
2. Wood tools (basic tasks)
3. Stone tools (real work begins)
4. Metal tools (power unlocked)
5. Master tools (precision and power)

**Player Experience**:
- Immediate feedback (success/failure)
- Skill expression (technique matters)
- Tool variety (different approaches)
- Progression clarity (quantified force)

### Pillar 3: Material Customization

**Philosophy**: Every tool tells a story

**Progression**:
1. Limited materials (wood, stone)
2. First metals (copper, bronze)
3. Iron age (strength unlocked)
4. Steel age (quality matters)
5. Master materials (perfect choices)

**Player Experience**:
- Expression (unique tools)
- Optimization (best combinations)
- Experimentation (try new materials)
- Collection (material discovery)

### Pillar 4: Progressive Complexity

**Philosophy**: Easy to learn, lifetime to master

**Progression**:
1. Simple crafting (immediate)
2. Part assembly (minutes)
3. Material selection (hours)
4. Heat treatment (days)
5. Perfect crafting (weeks/months)

**Player Experience**:
- Never overwhelming (gradual)
- Always learning (depth)
- Constant discovery (new mechanics)
- Long-term goals (mastery)

### Pillar 5: Meaningful Choices

**Philosophy**: No bad choices, only trade-offs

**Examples**:
- Sharp but fragile vs. dull but durable
- Fast but weak vs. slow but strong
- Lightweight but limited vs. heavy but powerful
- Specialized vs. general purpose

**Player Experience**:
- Strategic thinking required
- Personalization of playstyle
- No "correct" answer
- Replayability through choices

## Content Specifications

### Materials by Tier

**Tier 0-1 (Primitive)**:
- Wood (5 types)
- Stone (3 types)
- Plant fibers (2 types)
- Bone

**Tier 2-3 (Early Metal)**:
- Flint
- Copper
- Tin
- Bronze (alloy)
- Clay/Ceramic

**Tier 4 (Iron Age)**:
- Iron (pure)
- Cast Iron
- Wrought Iron

**Tier 5-6 (Steel Age)**:
- Mild Steel
- High Carbon Steel
- Tool Steel
- Damascus Steel
- Special alloys

**Special**:
- Obsidian
- Titanium (end-game)

### Tool Types

**Harvesting**:
- Axe (tree cutting)
- Pickaxe (mining)
- Shovel (digging)
- Sickle (crops)

**Crafting**:
- Hammer (forging)
- Chisel (carving)
- Saw (precision cutting)
- File (smoothing)

**Combat** (optional):
- Sword (general)
- Spear (reach)
- Bow (ranged)

### Modifier Categories

**Enhancement** (20+ modifiers):
- Sharpness, Reinforced, Hardened, Lightweight, etc.

**Ability** (15+ modifiers):
- Auto-Repair, Magnetic, Smelting, Silk Touch, Fortune, etc.

**Utility** (10+ modifiers):
- Tool Belt, Illumination, Waterproof, etc.

**Trait** (from materials):
- Material-specific bonuses

## Balance Considerations

### Time Investment vs. Power

| Stage | Hours | Power Level | Content Access |
|-------|-------|-------------|----------------|
| 0 | 0-2 | 1x | 10% |
| 1 | 2-5 | 2x | 25% |
| 2 | 5-10 | 3x | 40% |
| 3 | 10-20 | 5x | 60% |
| 4 | 20-40 | 8x | 80% |
| 5 | 40-80 | 12x | 95% |
| 6 | 80+ | 15x | 100% |

### Difficulty Curve

```
Difficulty
    │     ╱
    │    ╱
    │   ╱──────  (plateau at endgame)
    │  ╱
    │ ╱
    │╱
    └────────────────────> Time
```

- Initial spike (learning)
- Steady climb (progression)
- Plateau (mastery)
- Optional challenges (post-game)

## Future Expansion

### Potential Additions

**Content**:
- More materials (gems, exotic metals)
- More tool types (specialized tools)
- More modifiers (unique abilities)

**Systems**:
- Multiplayer trading
- Competitive modes
- Challenge runs
- Seasonal content

**Quality of Life**:
- Tool presets
- Material favorites
- Quick crafting
- Visual guides

## Related Documentation

- [Technical Design Document](../TechnicalDesign/TDD-01-Overview.md)
- [Class Diagrams](../ClassDiagrams/README.md)
- [C# Implementation Guide](../CSharp-Unity-Port/02-Implementation-Guide.md)
- [Rebuild Guide](../REBUILD-FROM-SCRATCH.md)
