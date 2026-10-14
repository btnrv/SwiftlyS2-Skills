---
name: sws2-entity-management
description: Create, find, spawn, modify, track and remove CS2 entities in SwiftlyS2 plugins with Core.EntitySystem, including designer names, CreateEntityByDesignerName, CEntityKeyValues spawn values, DispatchSpawn, CHandle references, schema field changes, Despawn, and entity inputs and outputs. Use whenever a plugin spawns props, triggers, lights, beams, particles, weapons or other entities, iterates entities by class, keeps entity references across ticks or callbacks, fires an entity input, or cleans entities up on round or map end.
---

Use `Core.EntitySystem` when the map entity system is ready. Verify designer names, schema types, and input/output contracts with SwiftlyS2 MCP for the target runtime.

Create the entity, configure required spawn values, and call `DispatchSpawn` once on the game thread. Track entities across callbacks with `CHandle<T>` and resolve them immediately before use. Clean up owned entities and hooks at their matching lifecycle boundary.

Read [entity lifetime and spawn values](references/entities.md) for the creation route, KeyValues ownership, and schema mutation. Build the plugin and verify creation and cleanup on a live map.
