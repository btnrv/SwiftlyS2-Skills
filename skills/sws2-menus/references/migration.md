# Migrating `Core.MenusAPI` to `Core.Menu`

The classic API still compiles on beta, with `[Obsolete]` warnings. Port one menu at a time and test each one in game. Both systems can run in the same plugin, but a player can have only one menu open in each system, and the two do not know about each other. Close the classic menu before opening the new one for the same player.

## Concepts

| Classic | `Core.Menu` |
|---|---|
| `Core.MenusAPI.CreateBuilder()` | `Core.Menu.CreateMenu("myplugin.id")`, which needs a namespaced id |
| `.Design.SetMenuTitle(t)` | `.WithTitle(t)`, or `AddHeader(new TitleComponent { TextProvider = … })` for per-player titles |
| `.AddOption(o)` | `.Add(component)`, plus `AddHeader` / `AddFooter` for pinned rows |
| `.Build()` then `OpenMenuForPlayer(player, menu)` | `.Build()` then `menu.Open(player)` |
| `OpenMenu(menu)` | `menu.OpenForAll()` |
| `CloseMenuForPlayer` / `CloseActiveMenu` | `menu.Close(player)` / `Core.Menu.CloseFor(player)` (own plugin only) |
| `CloseMenu(menu)` / `CloseAllMenus()` | `menu.CloseAll()` / `Core.Menu.CloseAll()` (own plugin only) |
| `GetCurrentMenu(player)` | `Core.Menu.GetSession(player)?.Menu` |
| `BindToParent(parent)` | `WithParent(parent)`, or let a `SubmenuComponent` adopt it |
| `SetAutoCloseDelay(seconds)` | `WithAutoClose(TimeSpan.FromSeconds(s))` |
| `SetMaxVisibleItems(n)` (1-5) | `WithItemsPerPage(n)`, capped by `MenuLayout.MaxVisibleLines` (1-32) |
| `SetSelectButton` and the other `Set*Button(KeyBind)` overrides | `WithAction("Select", MenuKey.E)` and so on, or a `WithKeybindSource` |
| `AddExtraButton(key, label, (p, m) => …)` | `WithAction(name, key, ctx => …, label)` |
| `KeyBind` | `MenuKey` (same key set, `[Flags]`) |
| `core.jsonc` `Menu` section (`InputMode`, `Buttons`, `Sound`, `ItemsPerPage`) | `configs/menus/menus.jsonc` (`Menus`, `MenuLayout`, `MenuSounds`) |
| `InputMode: "wasd"` | Rebind actions in `menus.jsonc`. Chat selection is `MenuInputMethod.Chat`. |
| `SetPlayerFrozen` / `FreezePlayer` | No built-in equivalent. Freeze in `Opened` and unfreeze in `Closed` on the game thread if the feature is required. |
| `EnableSound` / `DisableSound` | `PlaySound` on each component, or empty sound names in `MenuSounds` |
| `DisableExit` | No direct equivalent. Read the beta key resolver before promising a menu that cannot be closed. `menus.jsonc` binds take priority over code. |
| `MenuManagerEventArgs` `MenuOpened` / `MenuClosed` | `menu.Opened` / `menu.Closed` (`Action<IMenuSession>`) |
| Custom `IMenuAPI` adapter | `IMenuRenderer` registered through `Core.Menu.Renderers` |

## Options to components

| Classic option | Component |
|---|---|
| `TextMenuOption` | `TextComponent` |
| `ButtonMenuOption` + `Click` | `ButtonComponent(text, onSelect)` (`CloseOnSelect` replaces close-after-click) |
| `ToggleMenuOption` | `ToggleComponent` |
| `SliderMenuOption(text, min, max, default, step, totalBars)` | `SliderComponent(text, min, max, default, step) { TotalBars = … }` |
| `ChoiceMenuOption` | `ChoiceComponent` / `ChoiceComponent<T>` |
| `SelectorMenuOption<T>` | `SelectorComponent<T>` |
| `InputMenuOption` | `InputComponent` |
| `ProgressBarMenuOption` | `ProgressBarComponent` |
| `SubmenuMenuOption` | `SubmenuComponent` |

| Classic event or member | `Core.Menu` |
|---|---|
| `ValueChanged` (`Player`, `OldValue`, `NewValue`) | `onChanged` / `OnChanged` (`MenuValueChangedContext<T>`) |
| `Validating` with `Cancel` | `Validating = ctx => ValueTask.FromResult(allowed)`. Send the reason yourself. |
| `BeforeFormat` / `AfterFormat` | `TextProvider`, `Style`, an `IComponentRenderer`, or a custom component |
| `OptionHovering` / `OptionHovered` | `menu.FocusChanged`, or `NeedsRedraw` for per-frame content |
| `OptionSelected` | The component's own callback |
| `SetVisible(player, …)` / `SetEnabled(player, …)` | `VisibleWhen` / `EnabledWhen` predicates over your own per-player data |
| `MoveToOptionIndex(player, i)` | `menu.GetSession(player)?.SetFocus(i)` |
| `Comment` / `SetDefaultComment` | `Comment` on each component |
| `MenuOptionScrollStyle` / `MenuOptionTextStyle` | `Overflow` (`MenuTextOverflow`) on text components |

## Port checklist

1. Switch to the beta package and runtime, and confirm `Core.Menu` with `apidocs_lookup` on `beta`.
2. Choose an id for each menu (`"<plugin>.<menu>"`) and record the ids for server owners who use `menus.jsonc`.
3. Move per-player option state (`SetVisible`, `SetEnabled`, values) into predicates, `DefaultValueProvider` and `OnChanged` persistence.
4. Replace HTML in `AfterFormat` with `Style` or a component renderer, so the menu still works when drawn as chat.
5. Remove the classic `MenuOpened` / `MenuClosed` subscriptions. Registrations made through `Core.Menu` are released on unload.
6. Test opening, navigation, each component, submenu return, closing, and a second plugin's menu replacing yours.
