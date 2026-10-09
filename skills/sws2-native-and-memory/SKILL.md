---
name: sws2-native-and-memory
description: Implement SwiftlyS2 native calls, hooks, signatures, custom offsets, and memory access when required CS2 operations need engine interop.
---

Check for the operation in the target runtime's managed services, schema extensions, and game hooks. For required interop, verify the native ABI, platform, ownership, and thread context from authoritative source.

Read [gamedata and native calls](references/interop.md) for signatures, custom offsets, KeyValues pointers, and hook lifetimes. Store build-specific data in the plugin's `resources/gamedata` and retain hook IDs for removal on unload. Verify live MCP declarations against the installed package and server version.

Build and validate the operation and hook cleanup on the target server build. Record where each signature and offset came from so it can be checked after a CS2 update.
