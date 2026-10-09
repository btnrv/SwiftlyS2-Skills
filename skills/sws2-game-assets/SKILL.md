---
name: sws2-game-assets
description: Compile, package, precache, and apply CS2 models, materials, textures, particles, sounds, and UI resources used by SwiftlyS2 plugins.
---

Read [resource formats and precache](references/assets.md) for logical paths, compiled dependencies, player models, and manifest lifetime.

Verify that resources and dependencies are mounted for the server and receiving clients. Register needed resources through `Core.Event.OnPrecacheResource` before the game session manifest is built, then apply them through the target runtime's managed API. Use compiler tools matching the deployment.

Build and verify resource loading in a fresh session with a connected client. Validate the visible or audible result and the player-model behavior required by the feature.
