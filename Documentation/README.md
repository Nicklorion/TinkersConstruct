# Tinkers' Construct - Comprehensive Documentation

This documentation provides an extensive, detailed reference for understanding, modifying, and porting Tinkers' Construct. The documentation is structured to support both developers working with the Java/Minecraft codebase and those porting the system to other platforms (specifically Unity with C#).

## Documentation Structure

### 📐 [Architecture](./Architecture/README.md)
High-level architectural overview of Tinkers' Construct, including:
- System architecture and design patterns
- Module organization and dependencies
- Core initialization flow
- Event system architecture
- Data-driven design principles

### 🔧 [Technical Documentation](./Technical/README.md)
Deep technical documentation of all major systems:
- Materials System - definitions, stats, traits, and rendering
- Tools System - definitions, NBT data, parts, and assembly
- Modifiers System - application, persistence, and effects
- Smeltery System - multiblock structure, recipes, and melting
- Tables System - crafting stations, layouts, and UI
- Data System - JSON loaders, serialization, and synchronization
- Network System - packet structure and client-server sync
- Recipe System - custom recipe types and processing

### 🔌 [API Documentation](./API/README.md)
Complete API reference for mod integration:
- Public interfaces and contracts
- Event system and hooks
- Registration APIs
- Data provider APIs
- Integration guide for addon developers
- Example implementations

### 🎮 [C# Unity Port Guide](./CSharp-Unity-Port/README.md)
Comprehensive porting guide for recreating Tinkers' Construct in Unity:
- System-by-system porting strategy
- C# API design patterns
- Unity-specific implementation considerations
- Modding API architecture for Unity
- Data format specifications
- Performance considerations for Unity

## Purpose

This documentation serves multiple purposes:

1. **Understanding**: Provide complete insight into how Tinkers' Construct works at every level
2. **Modification**: Enable developers to modify and extend the existing codebase
3. **Integration**: Support addon developers creating compatible mods
4. **Porting**: Facilitate accurate recreation in other platforms (C#/Unity)
5. **Maintenance**: Serve as a reference for long-term project maintenance

## How to Use This Documentation

### For Java/Minecraft Developers
1. Start with the [Architecture Overview](./Architecture/README.md) to understand the big picture
2. Dive into [Technical Documentation](./Technical/README.md) for specific systems
3. Reference the [API Documentation](./API/README.md) when integrating or extending

### For C#/Unity Developers
1. Read the [C# Unity Port Guide](./CSharp-Unity-Port/README.md) overview
2. Study the [Architecture](./Architecture/README.md) to understand system design
3. Use [Technical Documentation](./Technical/README.md) as a detailed reference for implementation
4. Follow the modding API patterns in [API Documentation](./API/README.md)

### For Addon Developers
1. Start with the [API Documentation](./API/README.md)
2. Reference [Technical Documentation](./Technical/README.md) for system internals
3. Use the integration guide for best practices

## Key Concepts

Tinkers' Construct is built on several core concepts that are consistent throughout:

- **Data-Driven Design**: Most features are defined through JSON data files
- **Component-Based Architecture**: Systems are modular and loosely coupled
- **Tool Assembly**: Tools are constructed from materials and parts with customizable stats
- **Progressive Modification**: Tools can be enhanced through a modifier system
- **Resource Processing**: The smeltery provides ore doubling and alloy creation

## Version Information

This documentation is based on Tinkers' Construct for Minecraft Forge.
- **Minecraft Version**: Check build.gradle for target version
- **Forge Version**: Check build.gradle for Forge API version
- **Documentation Version**: 1.0.0
- **Last Updated**: 2025-10-20

## Contributing to Documentation

This documentation is intended to be comprehensive and accurate. If you find:
- Missing information
- Inaccurate descriptions
- Unclear explanations
- Areas needing more detail

Please contribute improvements to help other developers.

## License

This documentation is provided alongside Tinkers' Construct and follows the same MIT license as the project code.
