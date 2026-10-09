---
name: sws2-entity-management
description: Create, query, spawn, track, and remove CS2 entities with SwiftlyS2, including typed spawn values and entity inputs or outputs.
---

Use `Core.EntitySystem` when the map entity system is ready. Verify designer names, schema types, and input/output contracts with SwiftlyS2 MCP for the target runtime.

Create the entity, configure required spawn values, and call `DispatchSpawn` once on the game thread. Track entities across callbacks with `CHandle<T>` and resolve them immediately before use. Clean up owned entities and hooks at their matching lifecycle boundary.

Read [entity lifetime and spawn values](references/entities.md) for the creation route, KeyValues ownership, and schema mutation. Build the plugin and verify creation and cleanup on a live map.
