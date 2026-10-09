# Layouts and client assets

## Resource layout

Author XML and CSS under `content/csgo_addons/<addon>/panorama/{layout,styles}/custom_game/<namespace>/`. Compile with CS2 Workshop Tools or the game's resource compiler. Output belongs under the matching `game/csgo_addons/<addon>/` tree as `.vxml_c` and `.vcss_c`.

`s2r://` resource paths start at the mounted addon's resource root. A namespace inside `custom_game` helps keep paths unique across addons. The addon directory name itself is not part of that mounted path.

```xml
<root>
    <styles>
        <include src="s2r://panorama/styles/custom_game/sample/menu.vcss_c" />
    </styles>
    <Panel class="sample-screen">
        <Panel id="MenuRoot" class="sample-menu">
            <Label id="Title" text="{s:title}" />
            <Button id="confirm"><Label text="Confirm" /></Button>
            <Button id="close"><Label text="Close" /></Button>
        </Panel>
    </Panel>
</root>
```

```css
.sample-screen {
    width: 100%;
    height: 100%;
}
.sample-menu {
    visibility: collapse;
    flow-children: down;
    horizontal-align: center;
    vertical-align: center;
    background-color: #202020ee;
}
.sample-menu.shown { visibility: visible; }
```

The anonymous outer panel accommodates the loader's root identity. Put updateable IDs inside it. Author the menu hidden because spawning one layout entity makes its authored appearance available to all players; reveal it with a per-player class.

## Validator subset

The custom HUD validator accepts these elements and attributes:

| Category | Accepted |
|---|---|
| `Panel` attributes | `id`, `class`, `hittest` |
| `Label` attributes | `id`, `class`, `hittest`, `text` |
| `Image` attributes | `id`, `class`, `hittest`, `src`, `texturewidth`, `textureheight` |
| `Button` attributes | `id`, `class`; place its caption in a nested `Label` |
| Styles | `<styles>` with `<include>` |

Put text in a Label's `text` attribute. Keep visual state in authored classes and `{s:name}` dialog variables. Plugin code receives clicks by Button ID. JavaScript sections, snippets, inline styles, `onactivate`, `TextEntry`, and typical web attributes fall outside this validator subset. Chat is the available text-input route. Image `src` is static; class-controlled `background-image` supports authored alternatives.

Check the client console when loading fails. `Failed to load layout` points to the resource path or client assets. `did not pass CustomHud validation` with a disallowed attribute points to markup. Fix the reported violation and load again; the validator can report one violation per attempt.

## Styling decisions

Discover the CS2 property index through `panorama_list`, then look up exact names. A property name alone does not establish its syntax. Some entries have empty or placeholder descriptions; use the authoring references below for those values.

| Need | Panorama choice |
|---|---|
| Rows or columns | `flow-children: right` or `down` |
| Hide and remove from layout | `visibility: collapse` |
| Center a child | `horizontal-align: center; vertical-align: center` |
| Fill a known parent width | `width: 100%` or documented `fill-parent-flow(weight)` |
| Colors with alpha | `#rrggbbaa` |
| Shadow | `box-shadow: #00000080 0px 4px 8px 0px` (color first) |
| Button interaction styling | `:hover` / `:active` with input capture enabled |

Use authored class variants for computed states and custom disabled/selected appearance. Web flex/grid, `display`, `calc()`, CSS custom properties, and media queries have no equivalent syntax in this workflow. Confirm animation, clip, background, font, or image syntax through exact property lookups and current examples when those features are needed.

## Delivery and verification

The client needs the compiled resources in a mounted addon. Publishing an addon and arranging its download are separate from server-side entity creation. An additional addon needs MultiAddonManager or equivalent delivery, while the engine handles the map's addon. Follow the deployment's existing addon mechanism and authorization.

Verify with a client that has the addon mounted: layout appears for the intended player, dialog text changes, visibility classes work, and a captured button reaches the server. Restart the game when testing republished assets: layout caching lasts for the client session. Verify before capturing input: a missing client layout can leave a cursor without a usable interface.

References: [Source2Toolkit authoring](https://www.source2toolkit.net/docs/panorama/authoring), [Wend4r custom HUD layouts](https://github.com/Wend4r/s2r-skills/blob/main/custom-hud-layout/SKILL.md). Their helper APIs belong to those frameworks; use [SwiftlyS2 integration](integration.md) for server calls.
