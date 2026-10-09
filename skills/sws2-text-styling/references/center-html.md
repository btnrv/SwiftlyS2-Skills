# CenterHTML presentation

## Select the surface

| Surface | Presentation | API |
|---|---|---|
| Chat | Engine color bytes, bracket tokens, `[newline]` | `IPlayer.SendChat`; `Core.PlayerManager.SendChat` for broadcasts |
| Plain center text | Plain message | `SendCenter` |
| CenterHTML | Rich text in the existing center-message surface | `SendCenterHTML(message, duration)` |
| Custom HUD | Compiled Panorama XML/CSS, per-player classes and variables | `CCSCustomHudLayout` |

Built-in menus use center HTML rendering and take precedence over a pending center message in the checked implementation. That message's duration continues while the menu is shown. Coordinate ownership of this area.

## Markup and properties

The SwiftlyS2 styling guide documents `div`, `span`, `p`, `a`, `img`, `br`, `hr`, `h1` through `h6`, `strong`, `em`, `b`, `i`, `u`, and `pre`. Use simple text layouts; verify richer tags on the target client. This list belongs to CenterHTML, while custom HUD XML has its own validator.

Use the guide's direct `color` attribute and built-in `class` names, such as `<span color="red">`. Browser `style="color:red"` is outside that documented syntax. Color may be a name or hex value. Validate additional inline properties against the target rich-text renderer before relying on them; a property in a CSS sheet does not establish a rich-text attribute.

SwiftlyS2's own menu renderer also uses `<font color='#FFFFFF' class='fontSize-m'>` and `<br>`. Keep tags balanced and put presentation in translations or configuration.

## Fonts and sizes

Current game `csgostyles.css` defines these classes:

| Class | Pixel size |
|---|---|
| `fontSize-xs` | 8 |
| `fontSize-s` | 12 |
| `fontSize-sm` | 16 |
| `fontSize-m` | 18 |
| `fontSize-ml` | 20 |
| `fontSize-l` | 24 |
| `fontSize-xl` | 32 |
| `fontSize-xxl` | 40 |
| `fontSize-xxxl` | 64 |

The stylesheet's default Label font is `notosans` with `Arial Unicode MS` fallback. `stratum-font` selects `Stratum2`; `mono-spaced-font` and `mono-spaced-font-bold` select its regular and bold monodigit faces. Weight classes are defined with this capitalization: `fontWeight-Bold`, `fontWeight-Medium`, `fontWeight-Normal`, `fontWeight-Light`.

These are stylesheet definitions, not proof that every class affects every rich-text element. Use classes available to the center surface, and check glyph coverage, size, and contrast in game. Custom font resources and dedicated CSS belong to a custom HUD workflow.

The guide lists `fontStyle-m`, `fontWeight-bold`, and `CriticalText`. The audited game stylesheet uses `fontWeight-Bold` and does not define the other two there; inspect the loaded game's sheets and target renderer before relying on those guide names. The built-in menu confirms `fontSize-s`, `fontSize-sm`, and `fontSize-m` use on `<font>`.

## Localized example and duration

In `resources/translations/en.jsonc`:

```json
{
    "center.ready": "<font class='fontSize-m' color='#FFFFFF'>Ready: {0}</font><br><font class='fontSize-sm'>{1}</font>"
}
```

In a game-thread handler:

```csharp
var text = Core.Translation.GetPlayerLocalizer(player);
player.SendCenterHTML(text["center.ready", readyCount, safeDescription], 5000);
```

Encode `safeDescription` for HTML before interpolation. Duration is an integer in milliseconds, default `5000`; five seconds is `5000`, not `5`. Current native implementation clears this player's custom center message when sent an empty string. Synchronous player and broadcast sends are thread-unsafe; use `SendCenterHTMLAsync` or schedule onto the game thread from background work.

## Evidence

Audited 2026-10-09: [SwiftlyS2 styling guide](https://swiftlys2.net/docs/guides/chat-and-html-styling) provides the documented markup and direct-property syntax. Its short numeric duration examples do not establish seconds; implementation and API defaults determine the unit.

MCP beta `apidocs_lookup` for `IPlayer` and `IPlayerManagerService` confirms `SendChat`, `SendCenter`, `SendCenterHTML(string, int duration = 5000)` and their async variants.

[SwiftlyS2 78b4c89](https://github.com/swiftly-solution/swiftlys2/tree/78b4c89a6e21de7b6a4d9485295b6448f58e26d9): `managed/src/SwiftlyS2.Core/Modules/Menus/MenuAPI.cs` supplies `<font>` examples; `src/server/players/player.cpp` and `src/api/shared/string.cpp` establish duration, clearing, and milliseconds. [CS2 stylesheet snapshot 10f3693](https://github.com/SteamDatabase/GameTracking-CS2/blob/10f3693c381475016b128c549928d14eb3adfcf2/game/csgo/pak01_dir/panorama/styles/csgostyles.css) supplies typography values and exact class spelling. Recheck against the installed versions.
