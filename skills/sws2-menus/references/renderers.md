# Renderers and custom components

A compact copy of the renderer, component renderer and custom component sections of the [Menu System](https://swiftlys2.net/docs/development/menu-system) page, with notes checked against the beta source. Building menus is covered in [menu system](menu-system.md).

Contents: choosing an approach · built-in renderers · implementing `IMenuRenderer` · render context and frame · node types · component renderers · custom components · recipes (HTML, Panorama)

## Choosing an approach

Components produce a renderer-neutral tree of `MenuNode`s. Renderers turn that tree into output and never look at concrete component types, so every component works on every renderer. Use the smallest change that does the job:

| Goal | Approach |
|---|---|
| Chat menu instead of centre screen | `WithRenderer(MenuRendererIds.Chat)`. Input defaults to `!n` chat selection. |
| Change colour, size, bold or overflow of a line | `Style`, `FocusedColor`, `DisabledColor` and `Overflow` on the component |
| Change how one component type looks on one renderer | `IComponentRenderer`, including for the built-in `centerhtml` and `chat` |
| A new kind of row (behaviour or layout) | Custom component derived from `MenuComponentBase` |
| Renderer-specific markup from a component | Return a `MenuRawNode` for that renderer id, with generic nodes otherwise |
| A different output surface (Panorama HUD, alternative HTML, console) | Custom `IMenuRenderer` |

## Built-in renderers

| Id (`MenuRendererIds`) | Output |
|---|---|
| `CenterHtml` (`"centerhtml"`) | Centre-screen HTML through a core-managed persistent render, which is not public API. This is the default. The selection marker is `➤ `, and numbered entries show `n. `. Text nodes become `<font class='fontSize-*' color='…'>`, and `MenuTextSize` maps to the `fontSize-xs` through `fontSize-xl` classes. Text is not escaped. |
| `Chat` (`"chat"`) | Sends three blank lines and then one chat line per row on each redraw. `Clear` does nothing, because the lines scroll away. |
| `"myplugin.…"` | A renderer registered by a plugin |

A server owner can force any registered renderer on any menu with `"Renderer"` in `menus.jsonc`, so a plugin menu should still make sense when drawn by another renderer.

## Implementing `IMenuRenderer`

`IMenuRenderer` has `Id`, `Render(IMenuRenderContext)` and `Clear(IPlayer)`.

```csharp
using SwiftlyS2.Shared.Menu;
using SwiftlyS2.Shared.Players;

public sealed class PlainChatRenderer : IMenuRenderer
{
    public string Id => "myplugin.plain";

    public void Render(IMenuRenderContext context)
    {
        var frame = context.Frame;
        foreach (var node in frame.Header.Concat(frame.Body).Concat(frame.Footer))
        {
            foreach (var line in Flatten(node, context))
            {
                _ = context.Player.SendChatAsync(line);
            }
        }
    }

    public void Clear(IPlayer player) { } // nothing persistent on screen

    private IEnumerable<string> Flatten(MenuNode node, IMenuRenderContext context)
    {
        switch (node)
        {
            case MenuTextNode text: yield return text.Text; break;
            case MenuLineNode line: yield return string.Concat(line.Children.SelectMany(c => Flatten(c, context))); break;
            case MenuStackNode stack: foreach (var l in stack.Children.SelectMany(c => Flatten(c, context))) yield return l; break;
            case MenuBlankNode blank: for (var i = 0; i < blank.Lines; i++) yield return " "; break;
            case MenuSelectionNode sel: yield return sel.Number > 0 ? $"{sel.Number}. " : sel.Focused ? "> " : "  "; break;
            case MenuRawNode raw when raw.RendererId == Id: yield return raw.Payload; break;
            case MenuRawNode: break; // aimed at another renderer
            default: context.ReportUnsupported(node); break;
        }
    }
}
```

Register it in `Load` and select it by id:

```csharp
_renderer = Core.Menu.Renderers.Register(new PlainChatRenderer()); // IDisposable
var menu = Core.Menu.CreateMenu("myplugin.info").WithRenderer("myplugin.plain").WithTitle("Info").Build();
```

- Registering an id that already exists replaces the old renderer. Disposing the handle removes it. Registrations are released when the plugin unloads.
- Use a plugin-namespaced id so it cannot clash with another plugin or with `MenuRendererIds`.
- Register before building any menu that uses the renderer, because `Build()` throws for an unknown id.

### Threading

- `Render` runs on the dedicated menu render thread, one job at a time. Use only data from the context and calls that are safe off the game thread. Do not read live player, controller or pawn state there. The docs' own example calls `SendChat`, but `IPlayer.SendChat` and `SendCenterHTML` are marked `[ThreadUnsafe]`, so use `SendChatAsync`, `SendCenterHTMLAsync` and the `*Async` HUD setters.
- `Clear` runs when a session closes. The docs say it runs on the game thread, but it actually runs on whichever thread closed the session. Close from the game thread, and make `Clear` safe to call twice.
- When one menu replaces another that uses the same renderer, `Clear` is not called for the old one. The next `Render` has to overwrite the whole previous frame.
- `Render` must not throw. Exceptions are logged and that frame is lost.

## Render context and frame

| `IMenuRenderContext` | Description |
|---|---|
| `Frame` | The composed `MenuFrame` |
| `Menu`, `Session`, `Player` | The menu, the session and the player being drawn for |
| `ReportUnsupported(node)` | Logs a node the renderer cannot draw. Deduplicated per renderer and node type, so it is safe every frame. |

| `MenuFrame` | Behaviour |
|---|---|
| `Header` | Pinned above the body. Never scrolls or takes focus. Holds the title by default. |
| `Body` | Already reduced to the visible page. Focusable entries carry a `MenuSelectionNode` slot. |
| `Footer` | Pinned below. Holds action key hints and the focused component's `Comment`. |
| `PageOffset`, `TotalItems` | Index of the first body entry, and the number of focusable entries across all pages, for page indicators. |

Draw header, body and footer from top to bottom, and keep the header and footer on every page. Draw nothing for an empty region rather than leaving a gap. Focus, paging and visibility are already resolved, so hidden components are absent. Only the body is paged, which makes header and footer lines extra to `MaxVisibleLines`. Frames arrive only when a session is invalidated or a component reports `NeedsRedraw`, so a renderer needs no refresh loop of its own.

## Node types

| Node | Meaning |
|---|---|
| `MenuTextNode` | A run of text with a `MenuTextStyle` (`Color` as hex, `Bold`, and `Size` from `ExtraSmall` to `ExtraLarge`) |
| `MenuLineNode` | Children laid out horizontally on one line |
| `MenuStackNode` | Children stacked vertically, one per line |
| `MenuBlankNode` | `Lines` of empty space |
| `MenuSelectionNode` | Selection marker slot. `Focused` is true when it holds the selection. `Number` is the `!n` value under chat input, or 0 when entries are not numbered. The renderer chooses the indicator. |
| `MenuRawNode` | `Payload` aimed at one `RendererId`. Ignore it when it is addressed to another renderer. |

Rules for renderers:

- Skip node types you do not know and call `ReportUnsupported`. Never throw, because new node types can be added.
- Map sizes and colours onto what the output supports, or ignore them.
- Pair the renderer with an input method. Every renderer except `chat` defaults to `Buttons`. If the output shows `!1`, `!2` and so on, use `WithInputMethod(MenuInputMethod.Chat)` or a `WithInputMethodSource`.
- Remove anything kept on screen in `Clear`.

## Component renderers

Use an `IComponentRenderer` to give one component type richer output on one renderer:

```csharp
public sealed class SliderPlainRenderer : IComponentRenderer
{
    public string RendererId => "myplugin.plain";              // may be MenuRendererIds.Chat or CenterHtml
    public Type ComponentType => typeof(SliderComponent);       // derived types match too

    public bool TryRender(IMenuComponent component, IMenuComponentRenderContext context, out MenuNode? node)
    {
        var slider = (SliderComponent)component;
        node = new MenuTextNode($"{slider.Text}: {slider.GetValue(context.Session)}");
        return true;                                            // false falls back to the component's Render
    }
}

_sliderRenderer = Core.Menu.Renderers.RegisterComponentRenderer(new SliderPlainRenderer());
```

`IMenuComponentRenderContext` provides `Component`, `Menu`, `Session`, `Player`, `IsFocused`, `IsEnabled` and `RendererId`. With no matching component renderer, the component's own output is used, so an unrecognised component is never an error. Component renderers run during composition on the game thread and are released on unload.

## Custom components

Derive from `MenuComponentBase` and override `Render`. Keep per-player data in session state, so one instance can be shared across menus and players.

```csharp
public sealed class CounterComponent : MenuComponentBase
{
    public override bool IsFocusable => true;

    public override MenuNode Render(IMenuComponentRenderContext context)
    {
        var count = context.Session.GetState<CounterState>(this).Count;
        var color = context.IsFocused ? "#FFD700" : "#FFFFFF";
        return new MenuTextNode($"Clicks: {count}", MenuTextStyle.Default.WithColor(color));
    }

    public override ValueTask<bool> HandleActionAsync(MenuActionContext context)
    {
        if (!Matches(context, MenuActions.Select))
            return ValueTask.FromResult(false);                 // let the menu navigate or close

        return ActivateAsync(context, ctx =>
        {
            ctx.Session.GetState<CounterState>(this).Count++;
            ctx.Session.Invalidate();
            return ValueTask.CompletedTask;
        });
    }

    private sealed class CounterState { public int Count { get; set; } }
}
```

- Returning true from `HandleActionAsync` consumes the action. Only the focused component is offered actions, and taking `NavigateLeft` or `NavigateRight` is how sliders and selectors work.
- `ActivateAsync` runs the work through the shared pipeline. It refuses when the component is disabled or busy, runs `Validating`, and shows `WaitingText` while the work runs.
- Override `LineCount` when `Render` returns more than one line. A `MenuStackNode` costs the sum of its children, a `MenuBlankNode` costs its `Lines`, and everything else costs 1. Without the override the page overflows.
- Override `GetHint(session)` for a footer hint while focused. Override `NeedsRedraw(session, now)` for animated or live content. It is asked every tick for each visible component, so keep it cheap.
- Use `MenuLineNode` for several styled parts on one line. To target one renderer, check `context.RendererId` and return a `MenuRawNode`, and return generic nodes for any other id.

## Recipes

### Custom HTML look

Try these first: component styles, an `IComponentRenderer` for `MenuRendererIds.CenterHtml`, or a `MenuRawNode` addressed to `"centerhtml"`. The built-in renderer writes raw payloads straight into its markup. All three keep the core's persistent centre-menu render.

Write a full HTML renderer only when the whole layout must change. It would have to send through `SendCenterHTMLAsync(html, duration)`, which is a timed message. Frames are only sent on change, so check in game how the duration behaves on an idle menu, and use a `NeedsRedraw` component or a long duration to keep it visible. In `Clear`, send an empty message. Follow [CenterHTML](../../sws2-text-styling/references/center-html.md) for tags and limits, and escape player text.

### Panorama HUD renderer

Panorama layouts, assets and clicks are covered in [sws2-panorama-authoring](../../sws2-panorama-authoring/SKILL.md). The renderer adapts frames to them:

1. Author a layout with fixed rows: a header label, a body sized to `MaxVisibleLines`, and a footer. Give the rows panel ids and dialog variables.
2. Create and own the `CCSCustomHudLayout` entity on the game thread when the map is ready. The renderer only receives frames.
3. In `Render`, flatten each region to row strings and selection flags. Push them with `SetDialogVariableStringForPlayerAsync` and `SetHasClassForPlayerAsync`, then show the root class. Hide unused rows. Do not touch the entity's lifetime from the render thread.
4. In `Clear`, remove the shown class and call `SetInputCaptureEnabledForPlayerAsync(playerId, false)`, or the synchronous versions when on the game thread.
5. Keyboard navigation keeps working through the menu's `Buttons` keymap. For mouse clicks, handle `Core.Event.OnCustomHudClicked`. Match the owned layout, call `Core.Menu.GetSession(player)` and check that `session.Menu.Renderer.Id` is yours. Map the button id to a body row, then call `session.SetFocus(index)` and dispatch `MenuActions.Select` to `session.FocusedComponent.HandleActionAsync(new MenuActionContext { Action = MenuActions.Select, Key = MenuKey.None, Session = session })`. This skips the menu's select sound and menu-level handler. Add them yourself if needed.
6. Check in game: opening, one selection by key and one by click, closing, replacement by another plugin's menu, map change, and whether key states still reach the server while input capture is on.
