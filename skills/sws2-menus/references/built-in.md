# Built-in menus

## Build and open

Options are in `SwiftlyS2.Core.Menus.OptionsBase`; menu contracts and `KeyBind` are in `SwiftlyS2.Shared.Menus`.

```csharp
var text = Core.Translation.GetPlayerLocalizer(player);
var action = new ButtonMenuOption(text["menu.settings.open"]);
action.Click += async (sender, args) =>
{
    await args.Player.SendChatAsync(prefix + text["menu.settings.selected"]);
};

var menu = Core.MenusAPI.CreateBuilder()
    .Design.SetMenuTitle(text["menu.settings.title"])
    .AddOption(action)
    .Build();

Core.MenusAPI.OpenMenuForPlayer(player, menu);
// Close this menu when the interaction ends:
// Core.MenusAPI.CloseMenuForPlayer(player, menu);
```

This instance uses the opening player's localizer; `prefix` comes from plugin configuration. Build per-player instances when their text or state differs. Store these translation keys and styling as described in [chat texts](../../sws2-text-styling/SKILL.md).

Each `.Design.Set*` call returns the builder; repeat `.Design` for another design call. Use `OpenMenuForPlayer`, `CloseMenuForPlayer`, or `CloseActiveMenu` for manager state and events. `ShowForPlayer` and `HideForPlayer` only affect the rendering implementation.

Choose the option matching the value: `TextMenuOption`, `ButtonMenuOption`, `ToggleMenuOption`, `SliderMenuOption`, `ChoiceMenuOption`, `InputMenuOption`, `ProgressBarMenuOption`, `SubmenuMenuOption`, or `SelectorMenuOption<T>`. Value options expose `ValueChanged` with `Player`, `OldValue`, and `NewValue`. Input uses chat. Confirm constructor parameters for the target version.

## Navigation and server settings

The `Menu` object in SwiftlyS2 `configs/core.jsonc` controls `InputMode`, `Buttons.Use`, `Buttons.Scroll`, `Buttons.ScrollBack`, `Buttons.Exit`, `NavigationPrefix`, `ItemsPerPage`, and `Sound` settings. Current source defaults are `button`, E to use, Shift forward, F backward, Tab exit, and five items per page. Read the actual server configuration before choosing per-menu controls.

`AvailableInputModes` contains `button` and `wasd`; `NavigationPrefix` defaults to `➤`. Under `Menu.Sound`, each action has `Name` and `Volume`: `Exit` uses `Vote.Failed`, `Scroll` uses `UI.ContractType`, and `Use` uses `Vote.Cast.Yes`, each at `0.75` volume.

| Input mode | Behavior |
|---|---|
| `button` | Uses core defaults or per-menu `MenuKeybindOverrides` |
| `wasd` | W previous, S next, A back/exit, D select; standard per-menu overrides have no effect |

Builder overrides are `SetSelectButton`, `SetMoveForwardButton`, `SetMoveBackwardButton`, and `SetExitButton`. `KeyBind` is a flags enum, so `KeyBind.E | KeyBind.Mouse1` accepts either key. `AddExtraButton(key, label, action)` adds an action and footer label. Keep bindings distinct: standard input uses an ordered branch chain, and extra actions are processed afterward.

Recognized config button names: `mouse1`, `mouse2`, `space`, `ctrl`, `w`, `a`, `s`, `d`, `e`, `esc`, `r`, `alt`, `shift`, `weapon1`, `weapon2`, `grenade1`, `grenade2`, `tab`, `f`. These represent framework input flags; verify the player's actual game binds when testing.

`MaxVisibleItems` accepts 1 through 5; unset/default `-1` uses core `ItemsPerPage`. The setter logs out-of-range assignments and stores `-1`. Current rendering can add a row for each hidden title/footer with `AutoIncreaseVisibleItems`, up to seven; check source when exact row count matters. `MenuOptionScrollStyle` provides `LinearScroll`, `CenterFixed`, and `WaitingCenter`.

## Callbacks and player state

- `Click` is asynchronous and returns a `ValueTask`. Current manager dispatch runs it through `Task.Run`; use the scheduler or documented async API for thread-unsafe game operations.
- `Validating` can set `Cancel`; its arguments expose `Player` and `Option`. Send a reason separately when needed.
- `BeforeFormat` and `AfterFormat` customize option text through `CustomText`. They participate in center HTML formatting.
- `OptionHovering` runs each render frame; `OptionHovered` runs on a selection change; `OptionSelected` runs on activation.
- Global `Visible` and `Enabled` combine with per-player `SetVisible` and `SetEnabled` overrides. Keep player choices in per-player values.
- Subscribe to manager `MenuOpened`/`MenuClosed` when needed and unsubscribe during unload. Submenus may be prebuilt or created lazily.

In the built-in keyboard path, `OptionHovered`, `OptionSelected` and extra-button actions execute synchronously in the game-thread key event. `OptionHovering`, `BeforeFormat` and `AfterFormat` execute on the background render task; `Validating` and `Click` execute on the click task. Explicit calls from plugin code inherit their caller's context. Schedule engine access accordingly and keep render callbacks cheap. `OptionSelected` precedes validation, so put the accepted action in the validated click path.

The manager closes its menus on disconnect and map unload. Plugin cleanup should close its own menus and dispose owned menu instances; reserve `CloseAllMenus` for intentionally global operations.

Reference: [Menu guide](https://swiftlys2.net/docs/development/menus).
