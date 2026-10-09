# SwiftlyS2 custom HUD integration

Use `SwiftlyS2.Shared.SchemaDefinitions.CCSCustomHudLayout` and confirm APIs against the installed SwiftlyS2 release's stable or beta documentation.

## Spawn and show

```csharp
var hud = Core.EntitySystem.CreateEntity<CCSCustomHudLayout>();
hud.StrLayout = "panorama/layout/custom_game/sample/menu.xml";
hud.StrLayoutUpdated();
hud.DispatchSpawn();

hud.SetDialogVariableStringForPlayer(player.PlayerID, "Title", "title", "Settings");
hud.SetHasClassForPlayer(player.PlayerID, "MenuRoot", "shown",
    EHudPanelClassStatus_t.k_eHudPanelClassStatus_HasClass);
hud.SetInputCaptureEnabledForPlayer(player.PlayerID, true);
```

SwiftlyS2's example uses the source `.xml` name; Wend4r's entity `layout` keyvalue example uses the logical `.vxml` name. Both describe references to compiled `.vxml_c` assets. Confirm the accepted path with the target build and client console. Source2Toolkit's helper taking a short layout name is its own API.

Per-player dialog strings use `(playerId, panelId, variableName, value)`. A Label with `id="Title" text="{s:title}"` uses panel `Title` and variable `title`. The global variant omits `playerId`. Per-player getters return `null` for an unset override; removing the override restores the global displayed value.

Class state uses `EHudPanelClassStatus_t`: `k_eHudPanelClassStatus_HasClass`, `k_eHudPanelClassStatus_DoesNotHaveClass`, or `k_eHudPanelClassStatus_Undefined`. Pass the enum to SwiftlyS2 methods, matching the target signature. The boolean examples in other frameworks do not define this API.

Synchronous mutation methods are thread-unsafe. From background/async work use documented counterparts such as `SetDialogVariableStringForPlayerAsync`, `SetHasClassForPlayerAsync`, and `SetInputCaptureEnabledForPlayerAsync`, or schedule onto the game thread. Getters remain synchronous.

## Receive clicks

Subscribe to `Core.Event.OnCustomHudClicked` during load and unsubscribe during unload. `IOnCustomHudClickedEvent` exposes `PlayerId`, `ButtonId`, and `CustomHudLayout`.

Identify the owned layout by entity identity before resolving the Button ID to an action. Confirm the player's active interaction and current eligibility server-side. Use existing action/validation logic for both mouse and keyboard routes. Click notifications are shared across layouts.

Input capture enables the cursor and Button clicks. Show only the intended player's layout before capturing. On close, remove that player's `shown` class with `k_eHudPanelClassStatus_DoesNotHaveClass` and call `SetInputCaptureEnabledForPlayer(playerId, false)`. Release capture when replacing the interaction as well.

Layout construction resets input capture. Enable it after spawning the layout and reapply it for an active interaction after a layout rebuild, including a `StrLayout` change.

## Lifecycle and manager integration

Track the HUD entity and subscriptions as plugin-owned resources. Create map-scoped entities when the map is ready, recreate them for a new map, and remove owned entities through the entity API during cleanup while still valid. Release player interaction state on disconnect and disable capture on close/unload.

For menu manager coexistence, implement the `IMenuAPI` adapter described in [custom rendering](../../sws2-menus/references/custom-rendering.md). The manager can own open/close and selection; the adapter supplies Panorama state updates and bridges clicks. Scope compatibility to the controls implemented and verify the whole route in game.

Reference: [SwiftlyS2 custom HUD guide](https://swiftlys2.net/docs/development/custom-hud).
