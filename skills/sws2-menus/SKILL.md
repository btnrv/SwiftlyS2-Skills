---
name: sws2-menus
description: Build SwiftlyS2 player menus with the component-based Core.Menu system (beta), including centre-HTML and chat menus, custom renderers, component renderers, custom components, key binds and menus.jsonc; also covers the deprecated Core.MenusAPI for existing code and migration. Use whenever a SwiftlyS2 plugin needs a menu, settings screen, shop, vote, picker, submenu or any interactive per-player selection, even if the user does not say "menu".
---

# SwiftlyS2 menus

SwiftlyS2 has two menu systems:

| System | Entry point | Status |
|---|---|---|
| Menu system | `Core.Menu` (`IMenuService`), namespaces `SwiftlyS2.Shared.Menu` and `SwiftlyS2.Shared.Menu.Components` | Current. Beta branch only. |
| Classic menus | `Core.MenusAPI` (`IMenuManagerAPI`), namespaces `SwiftlyS2.Shared.Menus` and `SwiftlyS2.Core.Menus.OptionsBase` | `[Obsolete]` on beta and scheduled for removal |

Build new menus with `Core.Menu`. It separates what a menu contains (components) from how it is drawn (renderers), so the same menu works as centre-screen HTML, as chat lines, or through a renderer the plugin registers. Server owners can also change key binds and renderers in `configs/menus/menus.jsonc` without a rebuild.

## Choose the API

1. Read the project's SwiftlyS2 package version and the server's runtime version. Look up `IMenuService` with `apidocs_lookup` on the `beta` branch, then compare against the package the project actually compiles against.
2. If the project is on beta, use `Core.Menu`. When the project already uses `Core.MenusAPI`, say that it is deprecated and offer to migrate. Port only the menus the task touches unless the user asks for the whole plugin.
3. If the project is on stable, `Core.Menu` does not exist there yet. Tell the user the new system needs the beta package and runtime, and recommend it for new menu work. Use the classic API only if they stay on stable, and write it so the later port is mechanical: one builder method per menu, with callbacks kept apart from layout.
4. When the user asks for the classic API by name, use it, and mention the deprecation once.

Confirm every signature through the SwiftlyS2 MCP on the branch you chose ([sws2-api-mcp](../sws2-api-mcp/SKILL.md)). The beta API is still moving. If the docs and the package disagree, the package wins.

## Where to read next

- Building menus with `Core.Menu`: components, per-player values, submenus, actions and key binds, input methods, sessions, `menus.jsonc` and threading. Read [menu system](references/menu-system.md).
- Changing how menus look: the built-in `centerhtml` and `chat` renderers, a custom `IMenuRenderer`, `IComponentRenderer`, custom components, or a Panorama HUD renderer. Read [renderers and custom components](references/renderers.md).
- Porting `Core.MenusAPI` code to `Core.Menu`. Read [migration](references/migration.md).
- Maintaining classic `Core.MenusAPI` code. Read [legacy/menus-api.md](references/legacy/menus-api.md) for the API reference, [legacy/built-in.md](references/legacy/built-in.md) for `core.jsonc` and callback threading, and [legacy/custom-rendering.md](references/legacy/custom-rendering.md) for `IMenuAPI` adapters.
- Panorama XML, CSS, assets and HUD clicks. Use [sws2-panorama-authoring](../sws2-panorama-authoring/SKILL.md).
- Menu text, colours and translations. Use [sws2-text-styling](../sws2-text-styling/SKILL.md).

## Ground rules for `Core.Menu`

- Start with one menu, one working component and an open call, then add components.
- Give every menu a unique id namespaced with the plugin, such as `"myplugin.shop"`. The id is the configuration scope for key binds, renderer and input method in `menus.jsonc`, so a shared id lets one menu's binds leak into another.
- One component instance serves every player. Keep per-player data in the session (value components, `GetState<T>`, `TextProvider`, `VisibleWhen`), never in fields on the component. Persist values in `OnChanged` and load them with `DefaultValueProvider`, because session values are lost when the menu closes.
- Take menu text from translations and pass the player's localized strings, or use `TextProvider` for per-player text in a shared menu.
- After changing something a player sees from outside a component, call `session.Invalidate()`. Frames are only sent on invalidation. `Add`, `Insert` and `Remove` invalidate for you.
- Threading: key presses and chat input reach components on the game thread. Code after an `await` that really suspends is not guaranteed to be back on it. `IMenuRenderer.Render` always runs on the menu render thread. The details are in each reference.
- Registrations made through `Core.Menu` (renderers, component renderers, actions) and the plugin's open menus are released when the plugin unloads. Dispose a menu or registration earlier only if the plugin is done with it before then.

Docs: [Menu System](https://swiftlys2.net/docs/development/menu-system), [Menus (Deprecated)](https://swiftlys2.net/docs/development/menus).
