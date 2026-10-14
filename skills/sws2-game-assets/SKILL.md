---
name: sws2-game-assets
description: Compile, package, precache and apply CS2 models, materials, textures, particles, sounds and UI resources used by SwiftlyS2 plugins, and inspect them with Source 2 Viewer and Source2Viewer-CLI (ValveResourceFormat). Use whenever a plugin needs an asset path, a sound event name, Core.Event.OnPrecacheResource, a custom player model, a Workshop addon or VPK, or emits a sound, or when the user mentions VPKs, vmdl, vmat, vpcf, vsndevts or other `_c` files, missing textures or ERROR models, decompiling or extracting CS2 files, Source 2 Viewer, S2V, VRF or Source2Viewer-CLI.
---

Read [resource formats and precache](references/assets.md) for logical paths, compiled dependencies, player models, and manifest lifetime.

Read [sound events](references/sounds.md) when emitting named sounds with recipient filters, source entities or custom parameters.

Read [Source 2 Viewer and Source2Viewer-CLI](references/source2viewer.md) to find stock asset paths and sound event names, list a compiled resource's dependencies, check a VPK before shipping, export files for reference, or read resources from C#. It also covers where to download the tool and which CLI options each release supports.

Verify that resources and dependencies are mounted for the server and receiving clients. Register needed resources through `Core.Event.OnPrecacheResource` before the game session manifest is built, then apply them through the target runtime's managed API. Use compiler tools matching the deployment.

Look up stock asset paths and sound event names in the game's files with Source2Viewer-CLI. Do not guess them. Run `Source2Viewer-CLI --help` before writing a command, because the online guide describes options that release 20.0 does not have.

Build and verify resource loading in a fresh session with a connected client. Validate the visible or audible result and the player-model behavior required by the feature.
