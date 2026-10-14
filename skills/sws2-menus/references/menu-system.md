# Menu system (`Core.Menu`)

A compact copy of the [Menu System](https://swiftlys2.net/docs/development/menu-system) page, with threading notes checked against the beta source. Beta branch only. Renderers and custom components are covered in [renderers](renderers.md).

Contents: quick start · menu service · builder · components · per-player values · input · progress bars · submenus · dynamic components · actions and key binds · input methods · menus and sessions · threading · server configuration · pitfalls

## Quick start

```csharp
using SwiftlyS2.Shared.Menu;
using SwiftlyS2.Shared.Menu.Components;

var menu = Core.Menu.CreateMenu("myplugin.settings")
    .WithTitle("Settings")
    .Add(new ButtonComponent("Say hello", ctx =>
    {
        ctx.Player.SendChat("Hello!");
        return ValueTask.CompletedTask;
    }))
    .Add(new ToggleComponent("Show damage", defaultValue: true, onChanged: change =>
    {
        change.Player.SendChat($"Show damage: {change.NewValue}");
    }))
    .Build();

menu.Open(player);
```

`CreateMenu(id)` returns an `IMenuBuilder`. Configure it with `With*`, add components to the header, body or footer, and call `Build()` to get an `IMenu`. Open it with `menu.Open(player)` or `menu.OpenForAll()`.

The menu id is also the configuration scope for key binds, renderer and input method. Namespace it with the plugin id, for example `"myplugin.shop"`.

In real plugins, take strings from translations (`Core.Translation.GetPlayerLocalizer(player)`) and keep colours in data files, as described in [sws2-text-styling](../../sws2-text-styling/SKILL.md).

## Menu service

`Core.Menu` is an `IMenuService`. Anything created or registered through it belongs to the calling plugin and is released when that plugin unloads.

| Member | Description |
|---|---|
| `CreateMenu(string id)` | Starts a builder. An empty id throws. |
| `GetSession(IPlayer)` | The player's open `IMenuSession`, from any plugin, or null. |
| `CloseFor(IPlayer)` | Closes the player's menu, only when it belongs to this plugin. |
| `CloseAll()` | Closes every open menu that belongs to this plugin. |
| `Actions` | `IMenuActionRegistry`, the catalogue of bindable actions. |
| `Renderers` | `IMenuRendererRegistry`, renderers and per-renderer component drawing. |

A player has at most one open menu across all plugins. Opening a menu replaces the one they had open, without returning to its parent.

## Builder

Every method returns the builder except `Build()`.

| Method | Description |
|---|---|
| `WithTitle(string)` | Adds a title line to the header. |
| `AddHeader(IMenuComponent)` | Pinned above the body. Never scrolls or takes focus. |
| `Add(IMenuComponent)` | Appends to the scrollable body. |
| `AddFooter(IMenuComponent)` | Pinned below the body. Never scrolls or takes focus. |
| `WithParent(IMenu)` | Menu to return to when this one closes. |
| `WithItemsPerPage(int)` | Maximum body entries shown at once (minimum 1, default 5). |
| `WithAutoClose(TimeSpan?)` | Closes the session after the delay, counted from when it opened, not from the last input. Null (the default) disables it. |
| `WithAction(...)` | Extra key bind for this menu. See actions below. |
| `WithRenderer(string id)` | Defaults to `MenuRendererIds.CenterHtml`. `Build()` throws when the id is not registered. |
| `WithInputMethod(MenuInputMethod)` | `Buttons` or `Chat`. |
| `WithKeybindSource` / `WithRendererSource` / `WithInputMethodSource` | Override sources, for example from the plugin's own config. `menus.jsonc` still wins. |
| `Build()` | Returns the `IMenu`. |

`WithItemsPerPage` is an upper bound. The server-wide `MaxVisibleLines` counts rendered lines, so spacers and multi-line components leave room for fewer neighbours. Only body components with `IsFocusable == true` can hold the selection. Dividers, spacers and progress bars are scrolled past.

## Components

Namespace `SwiftlyS2.Shared.Menu.Components`. Every constructor takes an optional trailing `id`, which is generated when omitted.

| Component | Focusable | Notes |
|---|---|---|
| `TextComponent(text)` | yes | Does nothing when activated. |
| `TitleComponent(title)` | no | What `WithTitle` adds. |
| `ButtonComponent(text, onSelect)` | yes | `onSelect` is `Func<MenuActionContext, ValueTask>`. |
| `ToggleComponent(text, defaultValue, onChanged)` | yes | Per-player `bool`. |
| `SliderComponent(text, min, max, defaultValue, step, onChanged)` | yes | Per-player `float` drawn as a bar. Left and Right step it. Defaults are 0, 100, null and 5. |
| `ChoiceComponent(text, choices, defaultIndex, onItemChanged)` | yes | Cycles forward on each activation. Generic form: `ChoiceComponent<TItem>`. |
| `SelectorComponent<TItem>(text, choices, defaultIndex, onItemChanged)` | yes | Left and Right step through the list and show the neighbouring items. |
| `InputComponent(text, defaultValue, validator, onChanged)` | yes | The player's next chat message becomes the value. |
| `ProgressBarComponent(text, progressProvider, multiLine)` | no | Value from 0 to 1, owned elsewhere. Redraws itself. |
| `SubmenuComponent(text, IMenu or factory)` | yes | Opens another menu. |
| `ChatColoredTextComponent(text, chatColor)` | yes | Text in a `Helper.ChatColors` tag colour. The centre-HTML renderer maps it to the nearest hex colour. |
| `DividerComponent(length = 26, color = "#FFFFFF")` | no | A rule of `─` characters. |
| `SpacerComponent(lines = 1)` | no | Blank vertical space. |

### Common properties (`MenuComponentBase`)

| Property | Description |
|---|---|
| `Visible` / `Enabled` | Applies to every player. |
| `VisibleWhen` / `EnabledWhen` | `Func<IMenuSession, bool>`, evaluated each frame for each player and combined with `Visible` / `Enabled`. |
| `Comment` | Footer hint shown while the component holds the selection. |
| `PlaySound` | Plays the select sound on activation. Default true. |
| `Validating` | `Func<MenuActionContext, ValueTask<bool>>`. Return false to cancel an activation, and send the reason yourself. |
| `WaitingText` / `WaitingColor` | Shown while an async activation runs (default `"Waiting..."`). The same player cannot trigger it twice in the meantime. |
| `Tag` | Free-form object for your own bookkeeping. |

Text components (`TextComponent` and the types derived from it) add `Text`, `TextProvider` (`Func<IMenuSession, string?>` for per-player text), `Style` (`MenuTextStyle`: `Color` as hex, `Bold`, `Size`), `DisabledColor`, `MaxWidth` and `Overflow`. `MenuTextOverflow` has `None`, `TruncateEnd`, `TruncateBothEnds`, `ScrollLeftFade`, `ScrollRightFade`, `ScrollLeftLoop` and `ScrollRightLoop`. Colours are hex strings such as `"#FFD700"`.

The centre-HTML renderer inserts text into its markup without escaping it. Escape player-supplied text, such as names, chat input and database values, before you put it in `Text`, `TextProvider` or a title. See [localization and escaping](../../sws2-text-styling/references/localization.md).

### Buttons

```csharp
var save = new ButtonComponent("Apply and close", async ctx =>
{
    await SaveAsync(ctx.Player);
    await ctx.Player.SendChatAsync("Saved.");
})
{
    CloseOnSelect = true,
    FocusedColor = "#FFD700"
};
```

While `OnSelect` runs, the button draws `WaitingText` and ignores further presses from the same player. `CloseOnSelect` closes the menu once it finishes.

## Per-player values

Toggles, sliders, choices, selectors and inputs keep their values in the player's session, so one instance serves every player. The value is discarded when the menu closes. Persist it in `OnChanged` and load it back with `DefaultValueProvider`.

```csharp
var volume = new SliderComponent("Volume", min: 0, max: 100, defaultValue: 50, step: 10)
{
    TotalBars = 10,
    DefaultValueProvider = session => LoadVolume(session.Player),
    OnChanged = change => SaveVolume(change.Player, change.NewValue)
};

var quality = new ChoiceComponent("Quality", ["Low", "Medium", "High"], defaultIndex: 1,
    onItemChanged: change => Core.Logger.LogInformation("{Old} -> {New}", change.OldValue, change.NewValue));

var team = new SelectorComponent<string>("Team", ["T", "CT", "Spectator"]);

float current = volume.GetValue(session);
volume.SetValue(session, 80f); // true when the value changed
```

`OnChanged` receives a `MenuValueChangedContext<T>` with `Session`, `Component`, `OldValue`, `NewValue` and `Player`. For choices and selectors the stored value is the index: `OnChanged` gets an `int`, `OnItemChanged` gets the item itself, and `GetItem(session)` reads the current item.

## Input

```csharp
var nickname = new InputComponent("Nickname", defaultValue: "",
    validator: value => value.Length >= 3,
    onChanged: change => change.Player.SendChat($"Nickname set to {change.NewValue}"))
{
    MaxLength = 24
};
```

Activating the component captures the player's next chat message, which never appears in chat. Activating it again while it waits cancels the capture. Rejected values show `InvalidText`, and empty input is always rejected.

## Progress bars

```csharp
var bar = new ProgressBarComponent("Round progress", session => RoundProgress(), multiLine: true);
```

The bar polls `ProgressProvider` and redraws whenever the number of filled segments changes. You do not need to call `Invalidate`. Keep the provider cheap and safe to call on the game thread every tick.

## Submenus

```csharp
var advanced = Core.Menu.CreateMenu("myplugin.advanced")
    .WithTitle("Advanced")
    .Add(new TextComponent("Nothing here yet"))
    .Build();

var main = Core.Menu.CreateMenu("myplugin.main")
    .WithTitle("Main")
    .Add(new SubmenuComponent("Advanced", advanced))
    .Add(new SubmenuComponent("Built on demand", session =>
        Core.Menu.CreateMenu("myplugin.lazy").WithTitle("Lazy").Build()))
    .Build();
```

A submenu without its own parent adopts the menu that opened it, so closing it returns the player there. The factory overloads, `Func<IMenuSession, IMenu?>` and `Func<IMenuSession, ValueTask<IMenu?>>`, can return null to cancel the open. When a factory builds a fresh menu on every open, dispose the old ones or reuse a cached menu.

## Dynamic components

Components can be added to or removed from an open menu, and sessions redraw automatically.

```csharp
var online = new TextComponent { TextProvider = _ => $"Players online: {Core.PlayerManager.PlayerCount}" };
menu.Insert(MenuRegion.Body, 0, online);
menu.Add(MenuRegion.Footer, new DividerComponent());
menu.Remove(online);
var body = menu.GetComponents(MenuRegion.Body);
```

## Actions and key binds

Every menu inherits the built-in actions in `MenuActions`:

| Action | Default key | Effect |
|---|---|---|
| `NavigateUp` | Shift | Moves the selection up, wrapping at the top. |
| `NavigateDown` | F | Moves the selection down, wrapping at the bottom. |
| `NavigateLeft` | A | No effect on its own. Sliders and selectors use it while focused. |
| `NavigateRight` | D | No effect on its own. Sliders and selectors use it while focused. |
| `Select` | E | Activates the focused component. |
| `Close` | Tab | Closes the menu, returning to the parent if there is one. |

`MenuKey` is a `[Flags]` enum, so one action can take several keys: `MenuKey.E | MenuKey.Mouse1`. The keys are Mouse1, Mouse2, Space, Ctrl, W, A, S, D, E, Esc, R, Alt, Shift, Weapon1, Weapon2, Grenade1, Grenade2, Tab and F. Testing in game shows whether a player's own binds deliver the key.

```csharp
var menu = Core.Menu.CreateMenu("myplugin.shop")
    .WithTitle("Shop")
    .WithAction("Refresh", MenuKey.R, async ctx =>
    {
        await ReloadStockAsync();
        ctx.Session.Invalidate();
    }, label: "Reload")
    .Add(new ButtonComponent("Buy"))
    .Build();
```

- The footer shows the key as `label: KEY`, using the action name when `label` is null.
- Declaring an action with a built-in name, such as `"Close"`, rebinds it for this menu only.
- The focused component is offered every action first, through `HandleActionAsync`. The menu-level handler only runs when the component does not consume the key.
- `WithAction(MenuActionDescriptor)` gives full control over `Id`, `DefaultKey`, `Label`, `ShowInFooter` and `Order`. `MenuActionId` is `(Scope, Name)`.
- Handlers receive a `MenuActionContext` with `Action`, `Key`, `Session`, `Player` and `Menu`.

The key for each action comes from the highest-priority source that answers:

| Priority | Source |
|---|---|
| `MenuKeybindPriority.Global` (100) | Server-wide `menus.jsonc` |
| `MenuKeybindPriority.Plugin` (50) | `WithKeybindSource` sources (`IMenuKeybindSource.TryGetKey(MenuActionId, out MenuKey)`) |
| `MenuKeybindPriority.CodeDefault` (0) | The action's `DefaultKey` |

Renderers (`IMenuRendererSource.TryGetRenderer(menuId, out id)`) and input methods (`IMenuInputMethodSource.TryGetInputMethod(menuId, out method)`) resolve the same way, through their own `*Priority` constants.

## Input methods

- `Buttons`: keyboard keys bound to actions, as above.
- `Chat`: players type `!1`, `!2` and so on to select the entry with that number on the current page, and `!0` to close. Keys are ignored for that menu, and matching messages are kept out of chat.

When none is set, menus drawn by the chat renderer use `Chat` and all other menus use `Buttons`. Key binds only apply to `Buttons` menus.

## Menus and sessions

`IMenu` is the shared definition. `IMenuSession` is one player's live view of it.

| `IMenu` member | Description |
|---|---|
| `Id`, `Renderer`, `Keymap`, `InputMethod`, `ItemsPerPage`, `AutoCloseDelay` | Values after server overrides are applied. |
| `Parent` | Writable, so lazily built submenus can be given a parent. |
| `Tag` | Free-form object for your own bookkeeping. |
| `Sessions`, `GetSession(player)` | Sessions currently viewing the menu. |
| `Open(player)` / `OpenForAll()` | Opens for one player or for every connected non-bot player. Returns the sessions. |
| `Close(player)` / `CloseAll()` | Closes for one player or for everyone viewing it. |
| `GetComponents(region)`, `Add`, `Insert`, `Remove` | Edit components at runtime. |
| `Opened`, `Closed`, `FocusChanged` | `event Action<IMenuSession>`. |

`IMenu` is `IDisposable`. Disposing it closes it for everyone and clears its components.

| `IMenuSession` member | Description |
|---|---|
| `Player`, `Menu`, `IsOpen` | Who is viewing which menu, and whether it is still open. |
| `FocusedIndex`, `FocusedComponent` | The selection, counted among visible focusable body components. |
| `MoveFocus(delta)`, `SetFocus(index)` | `MoveFocus` wraps. Both return true when the selection changed. |
| `GetState<T>(component)` | Per-player state object for a component. `T` is a class with a parameterless constructor, created on first use. |
| `CaptureChat(Func<string, bool>)` | Routes the player's next chat messages to the callback. Return true to swallow a message. Dispose the handle to stop. It is dropped when the menu closes. |
| `Invalidate()` | Requests a redraw on the next tick. |
| `Close()` | Closes the session and returns to the parent, if there is one. |

```csharp
menu.Opened += session => Core.Logger.LogInformation("{Player} opened {Menu}", session.Player.PlayerID, session.Menu.Id);
menu.Closed += session => SaveSettings(session.Player);
```

Disconnecting players have their sessions removed and `Closed` raised. If a player is invalid at tick time, their session is closed.

## Threading

These notes come from the beta source. Recheck them when the package changes.

| Code | Thread |
|---|---|
| Key presses, `!n` chat selection and `InputComponent` capture | Game thread. `HandleActionAsync`, `Validating`, `OnSelect` and `OnChanged` start there. |
| Code after an `await` that really suspends (database, HTTP, `Task.Delay`) | Not guaranteed to be the game thread. Use the `*Async` player APIs or schedule back with [sws2-thread-management](../../sws2-thread-management/SKILL.md). |
| Composition: component `Render`, `TextProvider`, `VisibleWhen`, `EnabledWhen`, `NeedsRedraw`, `ProgressProvider`, `DefaultValueProvider` | Game thread, on the menu tick. These run every frame or tick, so keep them cheap and non-blocking. |
| `IMenuRenderer.Render` | The dedicated "SwiftlyS2 Menu Render" thread. |
| `IMenuRenderer.Clear` | The thread that closes the session. This is the game thread for key and tick closes, so close menus from the game thread. |

`OpenForAll` and `Open` from a command or event handler are on the game thread. From async code, schedule them onto it.

## Server configuration

Server owners tune menus in `configs/menus/menus.jsonc`. The file is optional and reloads live, and `configs/menus/menus.example.jsonc` is the template. This is separate from the classic system's `Menu` section in `core.jsonc`.

```jsonc
{
    "Menus": {
        // "Renderer": "centerhtml",   // every menu
        // "InputMethod": "buttons",   // every menu
        "core.menu": {
            "NavigateUp": "shift", "NavigateDown": "f",
            "NavigateLeft": "a", "NavigateRight": "d",
            "Select": "e", "Close": "tab"
        }
        // , "myplugin.shop": { "Renderer": "chat", "Select": ["e", "space"], "Close": "esc" }
    },
    "MenuLayout": { "MaxVisibleLines": 5 },
    "MenuSounds": {
        "Scroll": { "Name": "UI.ContractType", "Volume": 0.75 },
        "Select": { "Name": "Vote.Cast.Yes", "Volume": 0.75 },
        "Exit":   { "Name": "Vote.Failed", "Volume": 0.75 },
        "Fail":   { "Name": "Vote.Cast.No", "Volume": 0.75 }
    }
}
```

| Key | Description |
|---|---|
| `Menus.core.menu.<Action>` | Binds that every menu inherits. The value is a key name or an array of key names. |
| `Menus.<menuId>.<Action>` | Overrides one action for one menu. |
| `Menus.<menuId>.Renderer` / `InputMethod` | Forces a renderer or input method for one menu. Placed directly under `Menus`, it applies to every menu, and the per-menu value wins. Both override what plugins request in code. |
| `MenuLayout.MaxVisibleLines` | Cap on body lines drawn at once. Clamped to 1-32, default 5. |
| `MenuSounds.<Scroll\|Select\|Exit\|Fail>` | Sound `Name` and `Volume` (default 0.75). An empty `Name` silences it. |

An unregistered `Renderer` id makes that menu fail to build. When a plugin's menu ids or custom renderer ids are meant to be configurable, document them in its README.

## Pitfalls

- Use a unique, plugin-namespaced id for every menu.
- Keep per-player data in session state or `OnChanged` / `DefaultValueProvider`, not on the component.
- Call `session.Invalidate()` after changing displayed data from outside a component.
- Override `LineCount` on custom multi-line components.
- Keep `NeedsRedraw` and `TextProvider` cheap.
- Escape player-supplied text in menus that may be drawn as centre HTML.
