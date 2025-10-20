# Mod API - Standardized Interface

## Overview

This directory contains the standardized Mod API interface design for loading Tinkers' Construct-like features as modular content. This API is designed to work both in the original Java/Minecraft context and be portable to Unity/C# implementations.

## Purpose

The Mod API provides:

1. **Standardized Loading**: Consistent interface for loading mod content
2. **Isolation**: Each mod operates in its own context
3. **Discovery**: Automatic mod discovery and registration
4. **Lifecycle Management**: Clear initialization and cleanup phases
5. **Dependency Resolution**: Automatic handling of mod dependencies
6. **Version Compatibility**: Version checking and compatibility validation

## Core API Interfaces

### IMod Interface

The base interface all mods must implement:

```java
/**
 * Base interface for all Tinkers' Construct mods/addons
 */
public interface IMod {
    /**
     * Unique identifier for this mod
     * Format: namespace:modid (e.g., "mymod:custom_content")
     */
    String getModId();
    
    /**
     * Display name shown to users
     */
    String getModName();
    
    /**
     * Semantic version (e.g., "1.0.0")
     */
    String getVersion();
    
    /**
     * List of mod IDs this mod depends on
     */
    default List<String> getDependencies() {
        return Collections.emptyList();
    }
    
    /**
     * List of mods that must load before this mod
     */
    default List<String> getLoadAfter() {
        return Collections.emptyList();
    }
    
    /**
     * Called before any content registration
     * Use for initialization and setup
     */
    void onPreLoad();
    
    /**
     * Called during content registration phase
     * Register materials, tools, modifiers here
     */
    void onLoad();
    
    /**
     * Called after all mods have loaded
     * Use for cross-mod integration
     */
    void onPostLoad();
    
    /**
     * Called when mod is being unloaded
     * Clean up resources here
     */
    default void onUnload() { }
}
```

### IContentRegistry Interface

Central registry for all mod content:

```java
/**
 * Registry for registering mod content
 */
public interface IContentRegistry {
    /**
     * Register a material definition
     */
    void registerMaterial(MaterialDefinition material);
    
    /**
     * Register a tool definition
     */
    void registerTool(ToolDefinition tool);
    
    /**
     * Register a modifier
     */
    void registerModifier(ModifierDefinition modifier);
    
    /**
     * Register a recipe
     */
    void registerRecipe(RecipeDefinition recipe);
    
    /**
     * Register custom stat type
     */
    void registerStatType(StatTypeDefinition statType);
    
    /**
     * Register custom trait
     */
    void registerTrait(TraitDefinition trait);
    
    /**
     * Get the mod context for tracking registrations
     */
    IModContext getModContext(String modId);
}
```

### IModContext Interface

Tracks and manages a mod's registered content:

```java
/**
 * Context for a specific mod's registrations
 */
public interface IModContext {
    /**
     * Get the mod ID
     */
    String getModId();
    
    /**
     * Track a material registration
     */
    void trackMaterial(String materialId);
    
    /**
     * Track a tool registration
     */
    void trackTool(String toolId);
    
    /**
     * Track a modifier registration
     */
    void trackModifier(String modifierId);
    
    /**
     * Get all materials registered by this mod
     */
    List<String> getRegisteredMaterials();
    
    /**
     * Get all tools registered by this mod
     */
    List<String> getRegisteredTools();
    
    /**
     * Get all modifiers registered by this mod
     */
    List<String> getRegisteredModifiers();
    
    /**
     * Unregister all content from this mod
     */
    void unregisterAll();
}
```

### IModLoader Interface

Discovers and loads mods:

```java
/**
 * Mod loader responsible for discovering and loading mods
 */
public interface IModLoader {
    /**
     * Load all mods from the specified directory
     */
    void loadMods(Path modDirectory);
    
    /**
     * Load a specific mod
     */
    void loadMod(Path modFile);
    
    /**
     * Unload a mod by ID
     */
    void unloadMod(String modId);
    
    /**
     * Check if a mod is loaded
     */
    boolean isModLoaded(String modId);
    
    /**
     * Get a loaded mod by ID
     */
    IMod getMod(String modId);
    
    /**
     * Get all loaded mods
     */
    Collection<IMod> getLoadedMods();
    
    /**
     * Resolve mod dependencies and return load order
     */
    List<IMod> resolveLoadOrder();
}
```

## C# Unity Port Implementation

### C# Interface Definitions

```csharp
namespace TinkersConstruct.API
{
    /// <summary>
    /// Base interface for all Tinkers' Construct mods/addons
    /// </summary>
    public interface IMod
    {
        /// <summary>
        /// Unique identifier for this mod
        /// Format: namespace:modid (e.g., "mymod:custom_content")
        /// </summary>
        string ModId { get; }
        
        /// <summary>
        /// Display name shown to users
        /// </summary>
        string ModName { get; }
        
        /// <summary>
        /// Semantic version (e.g., "1.0.0")
        /// </summary>
        string Version { get; }
        
        /// <summary>
        /// List of mod IDs this mod depends on
        /// </summary>
        List<string> Dependencies { get; }
        
        /// <summary>
        /// List of mods that must load before this mod
        /// </summary>
        List<string> LoadAfter { get; }
        
        /// <summary>
        /// Called before any content registration
        /// </summary>
        void OnPreLoad();
        
        /// <summary>
        /// Called during content registration phase
        /// </summary>
        void OnLoad(IContentRegistry registry);
        
        /// <summary>
        /// Called after all mods have loaded
        /// </summary>
        void OnPostLoad();
        
        /// <summary>
        /// Called when mod is being unloaded
        /// </summary>
        void OnUnload();
    }
    
    /// <summary>
    /// Registry for registering mod content
    /// </summary>
    public interface IContentRegistry
    {
        void RegisterMaterial(MaterialDefinition material);
        void RegisterTool(ToolDefinition tool);
        void RegisterModifier(ModifierDefinition modifier);
        void RegisterRecipe(RecipeDefinition recipe);
        void RegisterStatType(StatTypeDefinition statType);
        void RegisterTrait(TraitDefinition trait);
        
        IModContext GetModContext(string modId);
    }
    
    /// <summary>
    /// Context for a specific mod's registrations
    /// </summary>
    public interface IModContext
    {
        string ModId { get; }
        
        void TrackMaterial(string materialId);
        void TrackTool(string toolId);
        void TrackModifier(string modifierId);
        
        List<string> RegisteredMaterials { get; }
        List<string> RegisteredTools { get; }
        List<string> RegisteredModifiers { get; }
        
        void UnregisterAll();
    }
    
    /// <summary>
    /// Mod loader responsible for discovering and loading mods
    /// </summary>
    public interface IModLoader
    {
        void LoadMods(string modDirectory);
        void LoadMod(string modFile);
        void UnloadMod(string modId);
        
        bool IsModLoaded(string modId);
        IMod GetMod(string modId);
        IReadOnlyCollection<IMod> LoadedMods { get; }
        
        List<IMod> ResolveLoadOrder();
    }
}
```

## Example Mod Implementation

### Java Example

```java
package com.example.custommod;

public class CustomMod implements IMod {
    private static final String MOD_ID = "custommod:content";
    private static final String MOD_NAME = "Custom Materials Mod";
    private static final String VERSION = "1.0.0";
    
    @Override
    public String getModId() {
        return MOD_ID;
    }
    
    @Override
    public String getModName() {
        return MOD_NAME;
    }
    
    @Override
    public String getVersion() {
        return VERSION;
    }
    
    @Override
    public List<String> getDependencies() {
        return Arrays.asList("tconstruct:core");
    }
    
    @Override
    public void onPreLoad() {
        // Initialize configuration
        System.out.println("Pre-loading " + MOD_NAME);
    }
    
    @Override
    public void onLoad() {
        // Register content through ContentRegistry
        IContentRegistry registry = ContentRegistry.getInstance();
        
        // Register custom material
        MaterialDefinition obsidian = new MaterialDefinition(
            "custommod:obsidian",
            "Obsidian",
            3, // tier
            true // craftable
        );
        obsidian.setHeadStats(new HeadStats(500, 7.0f, 3, 3.0f));
        obsidian.setHandleStats(new HandleStats(0.8f, 1.2f, 1.1f));
        registry.registerMaterial(obsidian);
        
        // Register custom modifier
        ModifierDefinition explosive = new ExplosiveModifier();
        registry.registerModifier(explosive);
    }
    
    @Override
    public void onPostLoad() {
        // Cross-mod integration
        if (ModLoader.isModLoaded("anothermod:content")) {
            // Integrate with another mod
        }
    }
    
    @Override
    public void onUnload() {
        // Cleanup
        System.out.println("Unloading " + MOD_NAME);
    }
}
```

### C# Unity Example

```csharp
using TinkersConstruct.API;
using System.Collections.Generic;

namespace CustomMod
{
    public class CustomMod : IMod
    {
        public string ModId => "custommod:content";
        public string ModName => "Custom Materials Mod";
        public string Version => "1.0.0";
        
        public List<string> Dependencies => new List<string> 
        { 
            "tconstruct:core" 
        };
        
        public List<string> LoadAfter => new List<string>();
        
        public void OnPreLoad()
        {
            // Initialize configuration
            Debug.Log($"Pre-loading {ModName}");
        }
        
        public void OnLoad(IContentRegistry registry)
        {
            // Register content
            
            // Register custom material
            var obsidian = new MaterialDefinition
            {
                Id = "custommod:obsidian",
                DisplayName = "Obsidian",
                Tier = 3,
                Craftable = true,
                HeadStats = new HeadStats 
                {
                    Durability = 500,
                    MiningSpeed = 7.0f,
                    MiningTier = 3,
                    Attack = 3.0f
                },
                HandleStats = new HandleStats
                {
                    DurabilityMultiplier = 0.8f,
                    MiningSpeedMultiplier = 1.2f,
                    AttackMultiplier = 1.1f
                }
            };
            registry.RegisterMaterial(obsidian);
            
            // Register custom modifier
            var explosive = new ExplosiveModifier();
            registry.RegisterModifier(explosive);
        }
        
        public void OnPostLoad()
        {
            // Cross-mod integration
            var loader = ModLoader.Instance;
            if (loader.IsModLoaded("anothermod:content"))
            {
                // Integrate with another mod
            }
        }
        
        public void OnUnload()
        {
            // Cleanup
            Debug.Log($"Unloading {ModName}");
        }
    }
}
```

## Mod Discovery

### Java Discovery

```java
public class ModDiscovery {
    public static List<IMod> discoverMods(Path modDirectory) {
        List<IMod> mods = new ArrayList<>();
        
        try {
            Files.walk(modDirectory, 1)
                .filter(p -> p.toString().endsWith(".jar"))
                .forEach(jarFile -> {
                    try {
                        URLClassLoader loader = new URLClassLoader(
                            new URL[] { jarFile.toUri().toURL() });
                        
                        ServiceLoader<IMod> serviceLoader = 
                            ServiceLoader.load(IMod.class, loader);
                        
                        for (IMod mod : serviceLoader) {
                            mods.add(mod);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        return mods;
    }
}
```

### C# Unity Discovery

```csharp
public class ModDiscovery
{
    public static List<IMod> DiscoverMods(string modDirectory)
    {
        var mods = new List<IMod>();
        
        if (!Directory.Exists(modDirectory))
            return mods;
        
        var dllFiles = Directory.GetFiles(modDirectory, "*.dll");
        
        foreach (var dllFile in dllFiles)
        {
            try
            {
                var assembly = Assembly.LoadFrom(dllFile);
                var modTypes = assembly.GetTypes()
                    .Where(t => typeof(IMod).IsAssignableFrom(t) 
                           && !t.IsInterface && !t.IsAbstract);
                
                foreach (var modType in modTypes)
                {
                    var mod = (IMod)Activator.CreateInstance(modType);
                    mods.Add(mod);
                    Debug.Log($"Discovered mod: {mod.ModName} v{mod.Version}");
                }
            }
            catch (Exception e)
            {
                Debug.LogError($"Failed to load mod from {dllFile}: {e}");
            }
        }
        
        return mods;
    }
}
```

## Dependency Resolution

### Topological Sort for Load Order

```java
public class DependencyResolver {
    public static List<IMod> resolveLoadOrder(Collection<IMod> mods) {
        Map<String, IMod> modMap = new HashMap<>();
        Map<String, Set<String>> dependencies = new HashMap<>();
        
        // Build dependency graph
        for (IMod mod : mods) {
            modMap.put(mod.getModId(), mod);
            dependencies.put(mod.getModId(), new HashSet<>());
            
            dependencies.get(mod.getModId())
                .addAll(mod.getDependencies());
            dependencies.get(mod.getModId())
                .addAll(mod.getLoadAfter());
        }
        
        // Topological sort
        List<IMod> loadOrder = new ArrayList<>();
        Set<String> visited = new HashSet<>();
        Set<String> visiting = new HashSet<>();
        
        for (String modId : modMap.keySet()) {
            if (!visited.contains(modId)) {
                topologicalSort(modId, modMap, dependencies, 
                              visited, visiting, loadOrder);
            }
        }
        
        return loadOrder;
    }
    
    private static void topologicalSort(
            String modId,
            Map<String, IMod> modMap,
            Map<String, Set<String>> dependencies,
            Set<String> visited,
            Set<String> visiting,
            List<IMod> loadOrder) {
        
        if (visiting.contains(modId)) {
            throw new IllegalStateException(
                "Circular dependency detected: " + modId);
        }
        
        if (visited.contains(modId)) {
            return;
        }
        
        visiting.add(modId);
        
        for (String dependency : dependencies.get(modId)) {
            if (modMap.containsKey(dependency)) {
                topologicalSort(dependency, modMap, dependencies,
                              visited, visiting, loadOrder);
            }
        }
        
        visiting.remove(modId);
        visited.add(modId);
        loadOrder.add(modMap.get(modId));
    }
}
```

## Mod Metadata

### mod.json Format

```json
{
    "mod_id": "custommod:content",
    "mod_name": "Custom Materials Mod",
    "version": "1.0.0",
    "author": "YourName",
    "description": "Adds custom materials and tools",
    "dependencies": [
        "tconstruct:core@1.0.0"
    ],
    "load_after": [],
    "main_class": "com.example.custommod.CustomMod",
    "assets": {
        "materials": "data/materials/",
        "tools": "data/tools/",
        "textures": "assets/textures/"
    }
}
```

## Best Practices

1. **Version All Content**: Use semantic versioning
2. **Declare Dependencies**: Explicitly list all dependencies
3. **Clean Up**: Always unregister content on unload
4. **Error Handling**: Gracefully handle missing dependencies
5. **Documentation**: Document your mod's API
6. **Testing**: Test with various mod combinations
7. **Namespacing**: Use unique mod IDs to avoid conflicts

## Navigation

- [Back to API Documentation ↑](../README.md)
