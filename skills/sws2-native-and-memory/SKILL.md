---
name: sws2-native-and-memory
description: Implement SwiftlyS2 native interop when no managed API covers a required CS2 operation, including gamedata signatures and offsets, Core.GameData, Core.Memory, virtual function tables, unmanaged function calls, function hooks and detours, KeyValues pointers and raw memory reads or writes. Use when the user mentions signatures, sigscans, offsets, vtables, detours, pointers, memory patching or gamedata files, when porting a native hook, or when a native call breaks after a CS2 update.
---

Check for the operation in the target runtime's managed services, schema extensions, and game hooks. For required interop, verify the native ABI, platform, ownership, and thread context from authoritative source.

Read [gamedata and native calls](references/interop.md) for signatures, custom offsets, KeyValues pointers, and hook lifetimes. Store build-specific data in the plugin's `resources/gamedata` and retain hook IDs for removal on unload. Verify live MCP declarations against the installed package and server version.

Build and validate the operation and hook cleanup on the target server build. Record where each signature and offset came from so it can be checked after a CS2 update.
