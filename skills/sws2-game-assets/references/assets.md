# Resource formats and precache

Common definition/output pairs are `.vmdl`/`.vmdl_c` (model), `.vmat`/`.vmat_c` (material), `.vtex`/`.vtex_c` (texture), `.vpcf`/`.vpcf_c` (particles), `.vsnd`/`.vsnd_c` (sound), and `.vsndevts`/`.vsndevts_c` (sound events). Raw meshes, images, and audio feed definitions. Build changed dependencies before the resources using them.

Use logical model paths such as `characters/example/player.vmdl` with `AddItem` and `SetModel`; mounted game assets are compiled. Precaching references available resources. Package custom content and dependencies through the deployment's supported server mount and client-delivery route.

Register a named handler in `Load` and remove it in `Unload`:

```csharp
using SwiftlyS2.Shared.Events;

private void PrecachePlayerModel(IOnPrecacheResourceEvent resource)
{
    resource.AddItem("characters/example/player.vmdl");
}
// Load: Core.Event.OnPrecacheResource += PrecachePlayerModel;
// Unload: Core.Event.OnPrecacheResource -= PrecachePlayerModel;
```

The event runs during game session manifest construction and borrows a native manifest pointer. Add entries inside the callback. Loading a plugin after that phase does not replay the completed manifest; validate new entries in the next session/map lifecycle that rebuilds it.

Apply the precached model to a current valid pawn on the game thread with the `SetModel` extension. Reapply after spawn when the feature persists through pawn replacement. Use a model authored for the game's player skeleton, animation, attachments, and required hitbox behavior, and verify those behaviors live.

Panorama uses `.xml`/`.vxml_c`, `.css`/`.vcss_c`, and `.svg`/`.vsvg_c`. Layout consumers may accept logical `.vxml` while stylesheet includes use `.vcss_c`. Consult the [resource compiler reference](https://github.com/Wend4r/s2r-skills/blob/main/resource-compiler/references/resources.md) for precise inputs and naming. Installed tools determine supported types.

Reference: [Precache event example](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/SwiftlyS2.PluginTemplate/templates/examples/Events.example.cs).
