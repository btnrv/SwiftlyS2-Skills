# Source 2 Viewer and Source2Viewer-CLI

Source 2 Viewer (S2V) opens, previews, decompiles and exports compiled Source 2 resources (`*_c` files) and VPK archives. It comes from the [ValveResourceFormat](https://github.com/ValveResourceFormat/ValveResourceFormat) (VRF) project. For SwiftlyS2 work, use it to:

- find the logical path of a stock CS2 model, particle, material or sound to precache or apply
- read sound event names and parameters before calling `SoundEvent`
- list what a compiled custom model or material references, so every dependency gets shipped
- read stock Panorama XML, CSS and JS as a reference for custom HUDs
- check what a VPK actually contains before you ship it

S2V reads and decompiles. It does not compile. Build `.vmdl`, `.vmat`, `.vpcf`, `.vsndevts` and similar sources with the CS2 Workshop Tools resource compiler, then use S2V to inspect the output.

## Contents

- [Get it](#get-it)
- [Version differences](#version-differences)
- [CLI basics](#cli-basics)
- [Options in release 20.0](#options-in-release-200)
- [Recipes for SwiftlyS2 work](#recipes-for-swiftlys2-work)
- [Pitfalls](#pitfalls)
- [GUI](#gui)
- [Missing dependencies and `--game`](#missing-dependencies-and---game)
- [The ValveResourceFormat library](#the-valveresourceformat-library)
- [Compiled resource file layout](#compiled-resource-file-layout)
- [License and credit](#license-and-credit)
- [Upstream docs](#upstream-docs)

## Get it

Everything is on the [GitHub releases page](https://github.com/ValveResourceFormat/ValveResourceFormat/releases), which [s2v.app](https://s2v.app) also links to. Release 20.0 (August 2026) was the latest when this was written.

| Download | What it is |
| --- | --- |
| `Source2Viewer.exe` | GUI. Windows only; on Linux and macOS, run it with Wine. |
| `cli-windows-x64.zip`, `cli-windows-arm64.zip` | CLI for Windows |
| `cli-linux-x64.zip`, `cli-linux-arm64.zip`, `cli-linux-arm.zip` | CLI for Linux |
| `cli-macos-x64.zip`, `cli-macos-arm64.zip` | CLI for macOS |
| `ValveResourceFormat.nupkg`, `ValveResourceFormat.Renderer.nupkg` | .NET libraries, also on NuGet |

Both the GUI and the CLI are portable and need no installer. The CLI is published self-contained, so it does not need a .NET runtime.

Linux CLI install (needs `curl` and `unzip`). The `latest/download` URL always fetches the newest release:

```sh
mkdir -p ~/tools/s2v && cd ~/tools/s2v
curl -LO https://github.com/ValveResourceFormat/ValveResourceFormat/releases/latest/download/cli-linux-x64.zip
unzip cli-linux-x64.zip
chmod +x Source2Viewer-CLI
./Source2Viewer-CLI --version
```

The archive contains `Source2Viewer-CLI` plus `libSkiaSharp.so` and `libspirv-cross.so`. Keep all three in the same folder. On Windows the binary is `Source2Viewer-CLI.exe`.

Dev builds come from every commit to `master`. The GUI dev build is on [s2v.app/dev](https://s2v.app/dev/) (`https://update.s2v.app/v1/dev/Source2Viewer.exe`). Dev CLI builds are the `cli-<os>-<arch>` artifacts of the [build workflow](https://github.com/ValveResourceFormat/ValveResourceFormat/actions/workflows/build.yml?query=branch%3Amaster), and downloading them needs a GitHub login. Dev builds can be unstable.

## Version differences

The [online CLI guide](https://s2v.app/ValveResourceFormat/guides/command-line.html) tracks `master`, which has moved ahead of release 20.0. These options and behaviors exist only in builds after 20.0:

- `--vpk_create`, `--vpk_create_chunk_size`, `--kv_flatten`, `--quiet`, `--skip_softbody_reconstruction`, `--shader_list_combos`, `--shader_combo`
- `steam:<appid>/...` input paths and `vpk:` links
- `-o -` to print decompiled text to stdout (20.0 writes a file named `-` instead)
- `--gltf_export_format` implying `-d`, and `--recursive_vpk` working without `--recursive`
- exit code `2` when a file fails

Before you write a command, run `Source2Viewer-CLI --help` and use only the options it lists. Upstream does not promise a stable CLI, so scripts that call it can break on upgrade.

## CLI basics

```text
Source2Viewer-CLI -i <file|folder|pak01_dir.vpk> [filters] [action]
```

- With only `-i`, the CLI prints a summary: file counts by type for a VPK, or the resource type and blocks for a single file.
- For a split package, pass the `_dir.vpk` file (`pak01_dir.vpk`), not the numbered `pak01_000.vpk` chunks.
- CS2's main package is `<CS2 install>/game/csgo/pak01_dir.vpk`. Maps are separate VPKs in `game/csgo/maps/`.
- Paths inside a VPK are lowercase and use `/`.
- The CLI writes progress and errors to the console. Errors are also appended to `exceptions.txt` in the current folder.

## Options in release 20.0

| Option | Use |
| --- | --- |
| `-i`, `--input` | File, folder or VPK to process. Required. |
| `-o`, `--output` | Where to write. For a VPK or folder input, a folder. |
| `-d`, `--vpk_decompile` | Decompile to source formats instead of copying the compiled files |
| `-l`, `--vpk_list` | List files in a VPK with CRC and size |
| `--vpk_dir` | List files with more archive metadata |
| `-e`, `--vpk_extensions` | Extension filter, comma-separated: `"vmdl_c,vsndevts_c"` |
| `-f`, `--vpk_filepath` | Path filter, comma-separated: `"models/chicken/,soundevents/"` |
| `--recursive` | Scan subfolders of a folder input |
| `--recursive_vpk` | With `--recursive`, also look inside VPKs in the folder |
| `-a`, `--all` | Print every block of a resource |
| `-b`, `--block` | Print one block, such as `DATA` or `RERL` |
| `--game` | Path to a `gameinfo.gi` to load dependencies from |
| `--texture_decode_flags` | `none`, `auto` (default) or `ForceLDR` |
| `--gltf_export_format` | `gltf` or `glb`. Needs `-d` in 20.0. |
| `--gltf_export_materials`, `--gltf_export_animations`, `--gltf_animation_list`, `--gltf_mesh_list`, `--gltf_textures_adapt`, `--gltf_export_extras`, `--gltf_compose_additive` | glTF export details |
| `--vpk_verify` | Check VPK checksums and signatures |
| `--vpk_cache` | Keep a `.manifest.txt` next to the VPK and write only changed files on later exports |
| `--threads <n>` | Process files in parallel |
| `--tools_asset_info_short` | Print only file paths for `tools_asset_info` files |

The `--stats*` options are for VRF developers.

## Recipes for SwiftlyS2 work

The examples use Linux paths with `CS2=<Steam library>/steamapps/common/Counter-Strike Global Offensive`. On Windows, use `Source2Viewer-CLI.exe` and Windows paths.

Find the path to pass to `AddItem` or `SetModel`. List the compiled files, then drop the `_c` suffix:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/pak01_dir.vpk" -l -e vmdl_c -f "characters/models/"
# characters/models/.../foo.vmdl_c  ->  "characters/models/.../foo.vmdl"
```

Read sound event names. Each top-level key in a decompiled `.vsndevts` file is an event name that `SoundEvent` accepts, and its fields (`volume`, `pitch`, `vsnd_files`, and so on) show what the event plays:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/pak01_dir.vpk" -e vsndevts_c -d -o ./cs2_soundevents
grep -rn "Weapon_AK47" ./cs2_soundevents
```

List what a compiled custom resource depends on. The `RERL` block lists every external reference: materials, meshes, textures and child models. Each one must be in the server's mounted content and delivered to clients:

```sh
./Source2Viewer-CLI -i ./build/models/example/player.vmdl_c -b RERL
```

Export stock Panorama files to use as a reference:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/pak01_dir.vpk" -e "vxml_c,vcss_c,vjs_c" -f "panorama/" -d -o ./cs2_panorama
```

Check an addon VPK before shipping. List it, confirm every `RERL` dependency is inside it or in the base game, and verify it:

```sh
./Source2Viewer-CLI -i ./my_addon_dir.vpk -l
./Source2Viewer-CLI -i ./my_addon_dir.vpk --vpk_verify
```

Copy compiled files out of a VPK without converting them. Leave out `-d`:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/pak01_dir.vpk" -f "models/chicken/" -o ./raw
```

Decompile one file from a VPK. When the filter matches exactly one file, `-o` can name the output file:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/maps/de_dust2.vpk" -f "maps/de_dust2/entities/default_ents.vents_c" -d -o dust2_ents.txt
```

Preview a model in Blender:

```sh
./Source2Viewer-CLI -i "$CS2/game/csgo/pak01_dir.vpk" -f "models/chicken/chicken.vmdl_c" \
  -d -o ./chicken --gltf_export_format glb --gltf_export_materials --gltf_export_animations
```

## Pitfalls

These were checked against release 20.0 on Linux.

- `-f` matches the start of the path and, in 20.0, is case-sensitive. `models/chicken/` matches; `chicken` matches nothing. Wildcard filters (`*chicken*`) need a build newer than 20.0. Use `-l` and `grep` to find the exact path first.
- Folder and VPK inputs with `-o <folder>` keep the internal folder structure, which is the most predictable mode. For a loose single file, 20.0 can ignore a folder `-o` and write next to the input file, or replace the file name you give with its own. Either name the output file with the correct extension (`-o crate.vmdl`) or put the file in a folder and pass that folder with `--recursive`.
- A model can write several files: the model plus meshes, animations, a `_physics` file, and `.dmx` or glTF parts. A sound writes `.wav` or `.mp3` (whichever format it was stored in) plus a `.txt` with its metadata. A texture writes `.png`, or `.exr` for HDR.
- `Failed to load "<path>"` means a dependency was not found. In 20.0 the exit code can still be `0` when a file fails, so check the output for errors as well as the exit code.
- `--vpk_verify` reads version 2 VPKs, the version CS2 uses. On older VPKs it fails with `Only version 2 is supported`.
- VPKs nested in VPKs, such as workshop map VPKs, are treated as plain files. Extract them with `-e vpk -o <folder>`, then run the CLI on the extracted VPK.
- A decompiled file can differ from the original source. Use it for reading. Rebuild real assets from their sources with Workshop Tools.

## GUI

On Windows, Source 2 Viewer's **Explorer** tab finds installed Steam games and their VPKs automatically. Double-click a file to preview it. Right-click a file to **Export as is** (compiled) or **Decompile & export** (models to glTF or `.vmdl`, textures to PNG or EXR, sounds to WAV or MP3, materials to `.vmat`). <kbd>Ctrl</kbd>+<kbd>F</kbd> searches by file name, full path, regex or file contents. **Copy URL** copies a `vpk:` link, which builds after 20.0 accept as `-i` input. **File → Create VPK from folder** builds a basic VPK.

## Missing dependencies and `--game`

A model needs its materials and a material needs its textures. S2V finds these through the game's search paths, the same way the game does. Files opened from inside an installed game folder find their dependencies on their own. For files outside a game folder, such as your build output or a downloaded VPK, pass the game's `gameinfo.gi`:

```sh
./Source2Viewer-CLI -i ./chicken.vmdl_c -d -o chicken.glb --gltf_export_format glb --gltf_export_materials \
  --game "$CS2/game/csgo/gameinfo.gi"
```

In the GUI, add `gameinfo.gi` under **Settings → Game content search paths**. Keep paths for only one game in that list, or files may load from the wrong game.

## The ValveResourceFormat library

To read resources from C#, reference the [`ValveResourceFormat`](https://www.nuget.org/packages/ValveResourceFormat/) NuGet package. Version 20.0 targets `net10.0`, the same as SwiftlyS2 plugins. It pulls in [`ValvePak`](https://www.nuget.org/packages/ValvePak/) for VPK access, SkiaSharp and other native libraries. Use it in build tools and offline scripts. Do not load it into a running server plugin unless you have checked its native dependencies on the server platform, and keep file parsing off the game thread.

This example, from the upstream guide, reads a texture from a VPK and saves it as PNG:

```csharp
using System.Diagnostics;
using System.IO;
using SteamDatabase.ValvePak;
using ValveResourceFormat;
using ValveResourceFormat.IO;
using ValveResourceFormat.ResourceTypes;

using var package = new Package();
package.Read("pak01_dir.vpk");

var packageEntry = package.FindEntry("textures/debug.vtex_c");
package.ReadEntry(packageEntry, out var rawFile);

using var ms = new MemoryStream(rawFile);
using var resource = new Resource
{
    FileName = packageEntry.GetFullPath()
};
resource.Read(ms);

Debug.Assert(resource.ResourceType == ResourceType.Texture);

var texture = (Texture)resource.DataBlock;
using var bitmap = texture.GenerateBitmap();
var png = TextureExtract.ToPngImage(bitmap);

File.WriteAllBytes("image.png", png);
```

For other types, cast `resource.DataBlock` to the matching class in `ValveResourceFormat.ResourceTypes`, such as `Model`, `Material` or `Sound`. See the [API reference](https://s2v.app/ValveResourceFormat/api/ValveResourceFormat.html).

## Compiled resource file layout

All compiled `*_c` resources share one container format. Values are little-endian.

Header:

| Offset | Size | Field | Notes |
| --- | --- | --- | --- |
| `0x00` | 4 | `FileSize` (uint32) | May not match the size on disk for resources with streamed data, such as sounds and textures |
| `0x04` | 2 | `HeaderVersion` (uint16) | Always 12 for current resources |
| `0x06` | 2 | `Version` (uint16) | Version of this resource type |
| `0x08` | 4 | `BlockOffset` (uint32) | Counted from offset `0x08`. Usually 8, so the block table starts at `0x10`. |
| `0x0C` | 4 | `BlockCount` (uint32) | Number of block table entries |

Each block table entry is 12 bytes:

| Offset | Size | Field | Notes |
| --- | --- | --- | --- |
| `0x00` | 4 | `BlockType` | FourCC, for example `DATA` = `0x41544144` |
| `0x04` | 4 | `Offset` | Relative to the position of this field, not to the file start |
| `0x08` | 4 | `Size` | Can be 0 |

Block data is usually aligned to 16 bytes. The common blocks are `DATA` (the resource content), `RERL` (external references) and `REDI` or `RED2` (edit info: the source inputs and compiler that built the file). The type comes from the file extension, or from the compiler identifier in `REDI`/`RED2`. To look at the blocks of a file, run `Source2Viewer-CLI -i <file> --all`.

## License and credit

VRF is MIT licensed, except its `Tests/Files` folder. If you redistribute S2V binaries or VRF code, include its copyright notice. The project also asks anyone who used it to export assets or to learn a format to credit it with a link to [s2v.app](https://s2v.app). Leave it to the user whether and where to add that credit.

## Upstream docs

- [Getting started](https://s2v.app/ValveResourceFormat/guides/getting-started.html)
- [Command-line utility](https://s2v.app/ValveResourceFormat/guides/command-line.html), which describes `master`
- [Reading resources](https://s2v.app/ValveResourceFormat/guides/read-resource.html)
- [Game search paths](https://s2v.app/ValveResourceFormat/guides/game-search-paths.html)
- [Format support](https://s2v.app/ValveResourceFormat/guides/format-support.html): what can be viewed, decompiled and exported, and what data is lost on export
- [Discord](https://discord.gg/s9QQ7Wg7r4) and [issues](https://github.com/ValveResourceFormat/ValveResourceFormat/issues)
