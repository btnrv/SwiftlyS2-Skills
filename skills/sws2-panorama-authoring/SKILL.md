---
name: sws2-panorama-authoring
description: Author and troubleshoot CS2 custom HUDs with Panorama XML layouts and CSS styles, SwiftlyS2 per-player HUD state APIs and server-side button clicks through Core.Event.OnCustomHudClicked. Use whenever a SwiftlyS2 plugin needs an on-screen panel, overlay, scoreboard, timer, progress bar or clickable button beyond chat and CenterHTML, or when the user mentions Panorama, a custom HUD, vcss_c or layout files, Panorama CSS properties, or a HUD that does not show or update.
---

# SwiftlyS2 Panorama authoring

Build the layout and stylesheet as CS2 resources and drive state from the SwiftlyS2 plugin. Start with one visible panel and one working update or click on a client that has the compiled assets.

Use the Panorama MCP `panorama_list` for CS2 to discover supported property names, then `panorama_lookup` for each property used or changed. Confirm syntax from an authoritative example when the lookup lacks a description. Treat the custom HUD validator's XML subset as the markup contract.

- For layout structure, styling, compiling, and client delivery, read [authoring](references/authoring.md).
- For entity creation, per-player state, clicks, and cleanup, read [SwiftlyS2 integration](references/integration.md).
- To drive a menu through a Panorama HUD, read the Panorama renderer recipe in [menu renderers](../sws2-menus/references/renderers.md#panorama-hud-renderer). That covers `Core.Menu` on beta. Classic `Core.MenusAPI` adapters are in [legacy custom rendering](../sws2-menus/references/legacy/custom-rendering.md).

Confirm the target SwiftlyS2 version and CS2 build. Custom HUD behavior is experimental; use client console output and an in-game check to resolve loading or validation failures.
