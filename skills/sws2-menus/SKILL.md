---
name: sws2-menus
description: Build SwiftlyS2 player menus, configure navigation and option behavior, and assess custom renderers or Panorama integration.
---

# SwiftlyS2 menus

Use `Core.MenusAPI` for menu ownership, opening, closing, and navigation. Start with one menu and one working action, then add the options the interaction needs.

Check the plugin's SwiftlyS2 package and server version, and select matching stable or beta API documentation. Confirm signatures through the API MCP or matching source.

- For the built-in menu, keybinds, callbacks, and per-player state, read [built-in menus](references/built-in.md).
- For a different rendering surface or manager integration, read [custom rendering](references/custom-rendering.md).
- For Panorama XML, CSS, client assets, and HUD clicks, use [sws2-panorama-authoring](../sws2-panorama-authoring/SKILL.md).

Keep callbacks that run during rendering cheap. Apply game state changes on the game thread. Close owned menus and remove subscriptions during plugin cleanup.
