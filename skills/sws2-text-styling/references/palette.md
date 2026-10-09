# Chat palette

Tokens in the same row produce the same engine control byte. `Helper.ChatColors` exposes the listed C# names as token strings; the table is a reference for editing configuration and translations.

| Tokens | Byte | `Helper.ChatColors` members |
| --- | --- | --- |
| `[default]`, `[/]`, `[white]` | `0x01` | `Default`, `White` |
| `[darkred]` | `0x02` | `DarkRed` |
| `[lightpurple]` | `0x03` | `LightPurple` |
| `[green]` | `0x04` | `Green` |
| `[olive]` | `0x05` | `Olive` |
| `[lime]` | `0x06` | `Lime` |
| `[red]` | `0x07` | `Red` |
| `[gray]`, `[grey]` | `0x08` | `Grey` |
| `[lightyellow]`, `[yellow]` | `0x09` | `LightYellow`, `Yellow` |
| `[silver]`, `[bluegrey]` | `0x0A` | `Silver`, `BlueGrey` |
| `[lightblue]`, `[blue]` | `0x0B` | `LightBlue`, `Blue` |
| `[darkblue]` | `0x0C` | `DarkBlue` |
| `[purple]`, `[magenta]` | `0x0E` | `Purple`, `Magenta` |
| `[lightred]` | `0x0F` | `LightRed` |
| `[gold]`, `[orange]` | `0x10` | `Gold`, `Orange` |

The native chat parser also resolves `[teamcolor]`: team 3 uses light blue, team 2 yellow, other teams light purple. Direct player sends use the recipient's team; the broadcast parser uses team 0. It is not a sender-team placeholder. There is no corresponding `Helper.ChatColors.TeamColor` member.

Native chat parsing is case-insensitive; use lowercase in resources. The managed `Helper.Colored()` helper performs case-sensitive replacements and does not implement `[teamcolor]`. Send through SwiftlyS2's chat API to use its native processing. Token names do not imply arbitrary RGB chat support, and aliases do not create additional colors.

Use `[newline]` for multiline chat. The native player and broadcast paths split it into separate `TextMsg` messages. `[newline]` and `[teamcolor]` survive the translation loader's managed color pass and are processed during sending. A prefix read from configuration goes through the native parser when sent.

## Editable message

`resources/templates/config.jsonc`:

```json
{
    "Main": {
        "Prefix": "[gold][Plugin][default] "
    }
}
```

`resources/translations/en.jsonc`:

```json
{
    "command.ready": "[white]Ready, [green]{0}[white]."
}
```

Within a game-thread handler, with `prefix` read from the bound configuration:

```csharp
var text = Core.Translation.GetPlayerLocalizer(player);
player.SendChat(prefix + text["command.ready", displayName]);
```

Use a sanitized `displayName` as described in [localization and escaping](localization.md). Translation interpolation uses numbered .NET format placeholders. Keep literal braces escaped according to that formatting syntax. CenterHTML uses its supported HTML subset; Panorama uses XML panels and Panorama CSS. Their styling syntax belongs in the matching resource.

Sources: `managed/src/SwiftlyS2.Shared/Helper.cs`, `src/api/shared/string.cpp`, `src/server/players/{player,manager}.cpp`, and `managed/src/SwiftlyS2.Core/Modules/Translations/Localizer.cs` at [SwiftlyS2 revision 78b4c89](https://github.com/swiftly-solution/swiftlys2/tree/78b4c89a6e21de7b6a4d9485295b6448f58e26d9); MCP `Helper.ChatColors`, `ITranslationService`, `IPlayer`. These sources distinguish the actual parser from illustrative website color swatches.
