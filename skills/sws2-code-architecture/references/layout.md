# Layout and packaging

```text
PluginName/
  PluginName.csproj
  src/
    PluginName.cs
    Commands/PlayerCommands.cs
    Configuration/PluginConfig.cs
    Feature/FeatureService.cs
  resources/
    templates/config.jsonc
    translations/en.jsonc
    gamedata/
    exports/PluginName.Contract.dll
  examples/
  build/
```

This tree shows possible responsibilities, not required empty folders. Use the project's namespace convention; a folder boundary does not require a new namespace or interface. For a new plugin, use one root namespace until a distinct contract or module benefits from its own.

The current template targets `net10.0`, enables nullable references and implicit usings, and excludes `examples/**/*.cs` from compilation. Preserve its package runtime exclusions: the server supplies SwiftlyS2 and the listed framework dependencies. Pin `SwiftlyS2.CS2` to the deployed runtime's compatible version for a reproducible build.

`resources/gamedata`, `resources/templates` and `resources/translations` are copied by the template. Add explicit packaging rules for other required resources, such as contract exports. A contracts project inside the plugin directory also needs exclusion from the plugin's default recursive compile glob.

`dotnet publish -c Release` runs the template's `CreateZip` target. The documented distribution paths are `build/publish/<PluginId>/` and `build/<AssemblyName>.zip`. Inspect the generated project's evaluated `PublishDir`, `OutputPath` and zip target when the template version or command-line output overrides differ. The zip must contain `<PluginId>/<PluginId>.dll` plus its resources and required private dependencies. Deploy that folder under `addons/swiftlys2/plugins/`.

Keep contract source in a separate `PluginName.Contract` project containing public interfaces and value types. Deploy the resulting DLL at `plugins/<Provider>/resources/exports/PluginName.Contract.dll`. SwiftlyS2 scans these export folders before loading plugins and shares their types across plugin load contexts. Consumers reference the same contract assembly identity. Replacing an exported contract DLL requires a server restart with this loader: exports load during framework initialization into a non-collectible context, and plugin reload does not reload them.

Reference: [Plugin template](https://github.com/swiftly-solution/swiftlys2/tree/master/managed/SwiftlyS2.PluginTemplate/templates). Use the generated project's packaging rules for the chosen template version.
