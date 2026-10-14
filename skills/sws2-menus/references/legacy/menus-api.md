# Classic menus API (`Core.MenusAPI`), deprecated

A compact copy of [Menus (Deprecated)](https://swiftlys2.net/docs/development/menus). Use it to maintain existing code or for plugins that stay on stable. New menus should use [`Core.Menu`](../menu-system.md), and porting is covered in [migration](../migration.md). Source-level behaviour (threading, `core.jsonc`, render callbacks) is in [built-in notes](built-in.md), and the `IMenuAPI` adapter is in [custom rendering](custom-rendering.md).

On beta, every type here (`IMenuManagerAPI`, `IMenuBuilderAPI`, `IMenuDesignAPI`, `IMenuAPI`, `IMenuOption`, `KeyBind`, and the config, event-argument and enum types) and `ISwiftlyCore.MenusAPI` carry `[Obsolete]`. Nothing else changed from stable.

## Workflow

1. `Core.MenusAPI.CreateBuilder()`.
2. Configure behaviour (sound, freeze, auto-close, key overrides) and appearance (`.Design`).
3. `AddOption(...)` and `Build()`.
4. Open and close through `Core.MenusAPI`.

| `IMenuManagerAPI` | Description |
|---|---|
| `CreateBuilder()` | New `IMenuBuilderAPI` |
| `CreateMenu(config, keybindOverrides, parent, scrollStyle, textStyle)` | Builds from `MenuConfiguration` / `MenuKeybindOverrides` objects |
| `OpenMenuForPlayer(player, menu[, onClosed])` | Opens for one player and closes their current menu |
| `OpenMenu(menu[, onClosed])` | Opens for every player |
| `CloseMenuForPlayer(player, menu)` | Closes that menu for the player if it is open |
| `CloseActiveMenu(player)` | Closes whatever the player has open |
| `CloseMenu(menu)` / `CloseAllMenus()` | Closes one menu for everyone, or every menu |
| `GetCurrentMenu(player)` | The active menu or null |
| `Core`, `Configuration` | The owning core instance, and the resolved `core.jsonc` `Menu` settings (`NavigationPrefix`, `InputMode`, `ButtonsUse/Scroll/ScrollBack/Exit`, `Sound*Name/Volume`, `ItemsPerPage`) |

The `onClosed` callback is `(IPlayer, IMenuAPI)`. `IMenuAPI.ShowForPlayer` and `HideForPlayer` only change the display, so use the manager methods for state and events.

## Builder and design

```csharp
var menu = Core.MenusAPI.CreateBuilder()
    .EnableSound()
    .SetPlayerFrozen(false)
    .SetAutoCloseDelay(0f)
    .SetSelectButton(KeyBind.E | KeyBind.Mouse1)
    .SetMoveForwardButton(KeyBind.W)
    .SetMoveBackwardButton(KeyBind.S)
    .SetExitButton(KeyBind.Esc)
    .AddExtraButton(KeyBind.R, "Reset", (p, m) => p.SendChat("Reset action executed."))
    .Design.SetMenuTitle("Gameplay Settings")
    .Design.SetMenuTitleItemCountVisible(true)
    .Design.SetMaxVisibleItems(5)
    .Design.SetGlobalScrollStyle(MenuOptionScrollStyle.WaitingCenter)
    .Build();
Core.MenusAPI.OpenMenuForPlayer(player, menu);
```

Every `.Design` call returns the builder, so repeat `.Design.` for each design call.

| `IMenuBuilderAPI` | Description |
|---|---|
| `BindToParent(IMenuAPI)` | Closing returns to the parent |
| `AddOption(IMenuOption)` | Adds an option |
| `EnableSound` / `DisableSound`, `EnableExit` / `DisableExit` | Interaction sounds and the exit button |
| `SetPlayerFrozen(bool)` | Freezes movement while open |
| `SetAutoCloseDelay(float seconds)` | 0 disables it |
| `SetSelectButton` / `SetMoveForwardButton` / `SetMoveBackwardButton` / `SetExitButton(KeyBind)` | Override the `Menu.Buttons` defaults. Ignored in `wasd` input mode. |
| `AddExtraButton(KeyBind, label, Action<IPlayer, IMenuAPI>)` | Footer button |
| `Build()` | Returns `IMenuAPI` |

| `IMenuDesignAPI` | Description |
|---|---|
| `SetMenuTitle(string?)`, `SetMenuTitleVisible`, `SetMenuTitleItemCountVisible` | Title |
| `SetMenuFooterVisible`, `SetCommentVisible`, `SetDefaultComment(string)` | Footer and comments |
| `SetMaxVisibleItems(int)` | 1-5, or -1 to use `ItemsPerPage`. Out-of-range values log an error and reset to -1. |
| `EnableAutoAdjustVisibleItems` / `DisableAutoAdjustVisibleItems` | Hidden title or footer rows add visible items |
| `SetGlobalScrollStyle(MenuOptionScrollStyle)` | `LinearScroll`, `CenterFixed`, `WaitingCenter` |
| `SetNavigationMarkerColor` / `SetMenuFooterColor` / `SetVisualGuideLineColor` / `SetDisabledColor` | Hex string (`#RGB`, `#RGBA`, `#RRGGBB`, `#RRGGBBAA`, null resets), `Natives.Color` or `System.Drawing.Color` |

`KeyBind` is a `[Flags] uint` enum: Mouse1, Mouse2, Space, Ctrl, W, A, S, D, E, Esc, R, Alt, Shift, Weapon1, Weapon2, Grenade1, Grenade2, Tab, F.

## Options

Namespace `SwiftlyS2.Core.Menus.OptionsBase`: `TextMenuOption`, `ButtonMenuOption`, `ToggleMenuOption`, `SliderMenuOption`, `ChoiceMenuOption`, `InputMenuOption`, `ProgressBarMenuOption`, `SubmenuMenuOption`, `SelectorMenuOption<T>`.

```csharp
var slider = new SliderMenuOption(text: "Round Time", min: 60f, max: 300f, defaultValue: 120f, step: 30f, totalBars: 8);
slider.ValueChanged += (sender, args) => args.Player.SendChat($"Round time: {args.NewValue:0}s");
```

Value options raise `ValueChanged` with `MenuOptionValueChangedEventArgs<T>` (`Player`, `OldValue`, `NewValue`). Submenus take a built menu or a `Func<IMenuAPI>` that builds it on click.

## Events

| Event | Arguments |
|---|---|
| `Core.MenusAPI.MenuOpened` / `MenuClosed` | `MenuManagerEventArgs` (`Player`, `Menu`). Unsubscribe in `Unload`. |
| `IMenuOption.Validating` | `Player`, `Option`, settable `Cancel` and `CancelReason`. Runs before `Click`. |
| `IMenuOption.Click` | `Player` and a read-only `CloseMenu`. Returns `ValueTask`. |
| `IMenuOption.BeforeFormat` / `AfterFormat` | `Player`, `Option`, settable `CustomText` |
| `IMenuOption.VisibilityChanged` / `EnabledChanged` / `TextChanged` | `MenuOptionEventArgs`. `Player` is null because these are global changes. |
| `IMenuAPI.OptionHovering` | Every render frame while hovering, so keep it cheap |
| `IMenuAPI.OptionHovered` / `OptionSelected` | When the hovered option changes, and when an option is activated |

## Runtime updates

- `menu.AddOption`, `menu.RemoveOption` (returns false if the option is missing), `MoveToOption` / `MoveToOptionIndex(player, i)`, and `GetCurrentOption` / `GetCurrentOptionIndex`.
- `IMenuAPI` exposes `MenuManager`, `Configuration`, `KeybindOverrides`, `OptionScrollStyle`, `Builder`, `Tag`, `Parent` (`ParentMenu`, `TriggerOption`) and `Options`. It is `IDisposable`.
- `IMenuOption.Visible` and `Enabled` are global. `SetVisible(player, …)` and `SetEnabled(player, …)` sit under them, so a globally hidden option stays hidden for everyone.
- Other `IMenuOption` members: `Text`, `Comment`, `Tag`, `LineCount`, `MaxWidth`, `TextSize` (`ExtraSmall` to `ExtraLarge`), `TextStyle` (`TruncateEnd`, `TruncateBothEnds`, `Scroll*`), `PlaySound`, `CloseAfterClick` and `Menu`. For custom options: `GetDisplayText(player, displayLine)`, `IsClickTaskCompleted`, `OnValidatingAsync` and `OnClickAsync`.

## `CreateMenu` without the builder

```csharp
var menu = Core.MenusAPI.CreateMenu(
    new MenuConfiguration { Title = "Raw Menu", MaxVisibleItems = 5 },
    new MenuKeybindOverrides { Select = KeyBind.E, Move = KeyBind.W, MoveBack = KeyBind.S, Exit = KeyBind.Esc },
    parent: null, optionScrollStyle: MenuOptionScrollStyle.CenterFixed, optionTextStyle: MenuOptionTextStyle.TruncateEnd);
```

`MenuConfiguration` fields and defaults: `Title` ("Menu"); `HideTitle`, `HideTitleItemCount`, `HideFooter` and `HideComment` (false); `PlaySound` (true); `MaxVisibleItems` (-1); `AutoIncreaseVisibleItems` (true); `FreezePlayer` (false); `DisableExit` (false); `AutoCloseAfter` (0); `NavigationMarkerColor`, `FooterColor`, `VisualGuideLineColor` and `DisabledColor` (null); `DefaultComment` (""); `ExtraButtons` (`MenuExtraButton`: `KeyBind`, `Label`, `Action`). `MenuKeybindOverrides` has `Select`, `Move`, `MoveBack` and `Exit`, all `KeyBind?`, where null keeps the core default.

## Pitfalls

- Open and close through the manager, not only with `ShowForPlayer` / `HideForPlayer`.
- Keep `OptionHovering` light.
- Unsubscribe `MenuOpened` / `MenuClosed` in `Unload()`.
