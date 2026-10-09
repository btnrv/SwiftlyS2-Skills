# Custom rendering and Panorama integration

## Extension boundary

The built-in `MenuAPI` renders center HTML. Its `OnRender`, `ProcessPlayerMenu`, and `BuildMenuHtml` are private, and the class is internal and sealed. `IMenuBuilderAPI`/`IMenuManagerAPI` exposes construction and configuration, with no public renderer replacement callback. Option format events customize text inside that renderer.

The manager accepts `IMenuAPI`, so a plugin can implement that interface with a different surface. A Panorama-backed implementation is structurally feasible: manager open/close calls `ShowForPlayer`/`HideForPlayer`, manager navigation calls selection methods, and manager state stores the interface. Test the adapter from opening through selection and closing with a client on the target versions.

## Minimal adapter responsibilities

| Contract | Panorama implementation |
|---|---|
| `ShowForPlayer` | Set dialog variables and visible class for that player; enable capture for a mouse interaction |
| `HideForPlayer` | Hide for that player, disable capture, release owned timers/state |
| Options and current selection | Maintain per-player selection and map authored button IDs to options |
| Move methods | Apply visibility rules, update selection, project selected classes to the HUD |
| Menu metadata | Supply manager, configuration, keybind overrides, parent tuple, options, builder/null, tag |
| Events and disposal | Implement interface events and dispose owned HUD resources/subscriptions |

Use manager open/close methods around the adapter to retain replacement of another active menu, parent reopening, and manager events. A standalone custom HUD can also own its own interaction when manager compatibility is unnecessary.

`ShowForPlayer` and `HideForPlayer` can be called from click continuations as well as the game thread. Marshal HUD mutations onto the game thread or use the documented async operations with owned completion handling. Parent-chain cleanup may call `HideForPlayer` for a parent that is already hidden; repeated hiding should leave the same clean state. Wrap or clamp the unbounded `current + 1` and `current - 1` indices passed to `MoveToOptionIndex`.

## Compatibility to implement explicitly

The manager currently invokes `OptionSelected` only through a concrete `MenuAPI` cast. A custom adapter must supply its own event behavior if consumers use that event. Navigation hover events belong to the renderer implementation.

Freeze/unfreeze and auto-close are implemented in built-in `MenuAPI.ShowForPlayer`/`HideForPlayer`, so an adapter implements those features when its contract requires them. Setting `MenuConfiguration` alone does not provide them.

`MenuOptionBase.Menu` has an internal setter populated by built-in `MenuAPI.AddOption`. Reusing an option in an adapter leaves that ownership unset: `CloseAfterClick` does nothing and `InputMenuOption` cannot install its chat hook. `SubmenuMenuOption` also accepts only a concrete built-in `MenuAPI` target. Implement adapter-owned close, submenu and chat-input actions for those flows. An adapter can be a built-in menu's parent through the `IMenuAPI` parent contract. Claimed input and slider special cases in the manager use built-in option types; inspect them before promising identical behavior.

For mouse clicks, subscribe to `Core.Event.OnCustomHudClicked`. Match the owned layout entity, player's active adapter, and button ID; check the option's per-player visibility/enabled state and in-flight click status before routing through `OnClickAsync`. Use the option's validation path and the same application action as keyboard activation. Close through the manager for a close button. Finish by disabling that player's input capture.

Keep one selection model for mouse and keyboard. Implement only the controls and value types the requested menu needs first. Verify opening, one selection, closing, replacement by another menu, and client asset loading before adding richer interactions. Check whether the required keyboard button states still reach the server while Panorama capture is enabled; source inspection alone does not establish that client interaction.

References: [MenuManagerAPI](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/src/SwiftlyS2.Core/Modules/Menus/MenuManagerAPI.cs), [custom HUD guide](https://swiftlys2.net/docs/development/custom-hud).
