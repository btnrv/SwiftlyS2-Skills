# CI and source control

Read repository instructions, branch status and submodule ownership first. Keep an established branch model. For a new standalone GitHub plugin, use a short-lived feature branch from the maintained default branch, review the change through a pull request, and tag a version after the accepted release workflow authorizes it. A monorepo may select server configurations by branch; preserve those boundaries.

For a plugin submodule, make and publish authorized changes inside its own repository before updating the parent pointer. Third-party upstream code needs a writable fork before carrying local patches. Build outputs and credentials stay out of source control.

## Build workflow

Use the repository's existing build tool when it owns packaging. For a standalone template project, CI should perform these steps with the same SDK and package versions used locally:

1. Check out the required source and submodules.
2. Install the SDK declared by the project or `global.json`.
3. Run `dotnet restore` and `dotnet publish PluginName.csproj -c Release --no-restore`.
4. Inspect the template's zip and upload it as the build artifact, with a missing artifact treated as a build failure.

When writing GitHub Actions YAML, resolve current supported versions or commit pins for checkout, SDK setup and artifact upload from their official repositories. Give the build job read access to repository contents. Keep release-write permissions in the explicitly triggered release job.

Use the template zip output after checking its evaluated path. It should contain the plugin-named folder, DLL, resources and required private dependencies. Verify that server-provided SwiftlyS2 assemblies are absent. Package custom assets or exported contract DLLs through explicit project rules when they are required.

Keep the release workflow tied to the repository's chosen version tags or manual release trigger. A successful CI build produces an artifact; publishing that artifact follows the user's release authorization.
