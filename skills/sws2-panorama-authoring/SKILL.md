---
name: sws2-panorama-authoring
description: Author and troubleshoot CS2 custom HUD layouts using Panorama XML and CSS, SwiftlyS2 HUD state APIs, and server-side button clicks.
---

# SwiftlyS2 Panorama authoring

Build the layout and stylesheet as CS2 resources and drive state from the SwiftlyS2 plugin. Start with one visible panel and one working update or click on a client that has the compiled assets.

Use the Panorama MCP `panorama_list` for CS2 to discover supported property names, then `panorama_lookup` for each property used or changed. Confirm syntax from an authoritative example when the lookup lacks a description. Treat the custom HUD validator's XML subset as the markup contract.

- For layout structure, styling, compiling, and client delivery, read [authoring](references/authoring.md).
- For entity creation, per-player state, clicks, and cleanup, read [SwiftlyS2 integration](references/integration.md).
- For sharing navigation and ownership with `Core.MenusAPI`, read [custom menu rendering](../sws2-menus/references/custom-rendering.md).

Confirm the target SwiftlyS2 version and CS2 build. Custom HUD behavior is experimental; use client console output and an in-game check to resolve loading or validation failures.
