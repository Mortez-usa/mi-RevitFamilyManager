<div align="center">

# Revit Family Manager

**A Windows desktop app that turns folders of Revit families into a searchable visual library.**

![Version](https://img.shields.io/badge/version-0.1.0-0F6CBD)
![Status](https://img.shields.io/badge/status-pre--release-orange)
![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-555)
![.NET](https://img.shields.io/badge/.NET-10-512BD4)

[Download](../../releases) · [Features](#features) · [Install](#install) · [Build from source](#build-from-source)

</div>

---

Point Revit Family Manager at any folder of `.rfa` files, on your PC or on a network share. It indexes
every family **without needing Revit**: preview thumbnail, Revit version, category, types and
parameter values. You can browse, search and drag families straight into your Revit projects.
Everything runs offline and stays on your computer.

> [!NOTE]
> **Version 0.1.0 is a pre-release.** The file reading and indexing are tested against real Revit
> families, but the app hasn't been widely tested on Windows yet. Please
> [open an issue](../../issues) if something doesn't look or work right.

## Features

| | |
|---|---|
| **Reads families without Revit** | Preview thumbnail, Revit version, category, OmniClass number, family types and their parameter values, plus type catalogs (`.txt` next to the `.rfa`). |
| **Browse your way** | Grid of thumbnails or a sortable list. Filter by folder, by category, or by subfolder. |
| **Search everything** | Matches family names, file names, categories, type names, parameter names and parameter values. |
| **Details at a glance** | Large preview, file facts, parameters for each type, and the list of types. |
| **Load into Revit** | Drag a card or the preview into an open Revit project to load the family. Double-click to open the family itself in Revit. |
| **Stays up to date** | Watches your folders and rescans only what changed. Revit backup files (`Door.0001.rfa`) are ignored. |
| **Works offline** | The index is stored locally. A network folder that's unreachable keeps showing its last index. |
| **Safe** | Your `.rfa` files are only ever read. They're never modified, moved or copied. |

## Install

1. Download **`RevitFamilyManager-Setup-0.1.0.exe`** from [Releases](../../releases).
2. Run it. The installer isn't code-signed yet, so Windows SmartScreen may warn you. Choose
   **More info → Run anyway**.
3. Open **Revit Family Manager** from the Start menu and click **Add family folder**.

- Installs **for the current user without admin rights** by default
  (`%LOCALAPPDATA%\Programs\Revit Family Manager`). The installer also offers an all-users install.
- No separate .NET install needed.
- **Requirements:** 64-bit Windows 10 or 11.
- **Upgrading:** run the newer installer. Your library is kept.
- **Uninstalling:** use Settings → Apps. You'll be asked whether to also delete the library index.
  Your family files are never touched.

<details>
<summary>Silent install for IT deployment</summary>

```bat
RevitFamilyManager-Setup-0.1.0.exe /VERYSILENT /ALLUSERS
```

Use `/CURRENTUSER` for a per-user install. To uninstall silently, run `unins000.exe /VERYSILENT`
from the install folder.

</details>

## Usage

1. **Add a folder:** click **Add family folder**, pick a folder, and choose whether to include
   subfolders, read type catalogs, watch for changes, and organise by Revit category or by folder structure.
2. **Browse:** pick a folder and a category in the sidebar. Switch between grid and list view in the toolbar.
3. **Inspect:** click a family to see its details. Pick a type to see its parameter values.
4. **Use it:** drag the family into Revit, or use **Open in Revit**, **Show in File Explorer** and
   **Copy path**.

Right-click a folder in the sidebar to **rescan**, **edit** or **remove** it.

### Keyboard shortcuts

| Key | Action |
|---|---|
| `Ctrl` + `F` | Search |
| `Esc` | Clear search |
| `F5` | Rescan |
| `Enter` | Open the focused family in Revit |
| `Space` | Select the focused family |

## How it works

An `.rfa` file is an OLE compound file. Revit Family Manager opens it read-only with
[OpenMcdf](https://github.com/ironfede/openmcdf) and reads three of its internal streams:

| Stream | What it gives |
|---|---|
| `BasicFileInfo` | The Revit version the family was saved in |
| `RevitPreview4.0` | The preview image (a PNG; Revit saves these at 128 × 128) |
| `PartAtom` | An XML summary: category, OmniClass number, types and their parameter values |

> [!IMPORTANT]
> The `PartAtom` summary only contains the type values Revit chooses to export. **Instance
> parameters and formulas are only visible when the family is opened in Revit.** Reading those
> would need a Revit add-in, which isn't part of this version.

### Where your data is kept

Everything is local, in `%LOCALAPPDATA%\RevitFamilyManager\`:

| File | Contents |
|---|---|
| `library.db` | The SQLite index, including thumbnails |
| `settings.json` | View and sort preferences |
| `error.log` | Unexpected errors, if any |

## Build from source

Requires the [.NET 10 SDK](https://dotnet.microsoft.com/download).

Run the tests:

```bash
dotnet test
```

Run the app (Windows only):

```bash
dotnet run --project src/RevitFamilyManager.App
```

Publish a single self-contained `.exe`:

```bash
dotnet publish src/RevitFamilyManager.App -c Release -r win-x64 --self-contained -p:PublishSingleFile=true -p:EnableCompressionInSingleFile=true -o artifacts/publish
```

The core library and its tests run on any OS. The WPF app compiles on any OS
(`EnableWindowsTargeting`) but only runs on Windows.

### Build the installer

Install [Inno Setup 6](https://jrsoftware.org/isdl.php) (6.3 or later), then from the repository root:

```powershell
powershell -ExecutionPolicy Bypass -File installer\build-installer.ps1
```

This publishes the app to `artifacts\app` and compiles `installer\RevitFamilyManager.iss` into
`artifacts\installer\RevitFamilyManager-Setup-<version>.exe`.

### Releasing a new version

1. Change `<Version>` in `src/RevitFamilyManager.App/RevitFamilyManager.App.csproj`
   (for example `<Version>0.2.0</Version>`).
2. Build the installer.
3. Create a GitHub release with a matching tag (for example `v0.2.0`) and attach the installer.

> [!WARNING]
> Never change the `AppId` in `installer/RevitFamilyManager.iss`. Windows uses it to recognise
> upgrades. If it changes, new versions install alongside the old one instead of replacing it.

## Project structure

```
RevitFamilyManager/
├── src/
│   ├── RevitFamilyManager.Core/        .rfa reader, type-catalog parser, SQLite store, folder scanner and watcher (no UI)
│   └── RevitFamilyManager.App/         WPF desktop app (MVVM, CommunityToolkit.Mvvm)
├── tests/
│   └── RevitFamilyManager.Core.Tests/  xUnit tests, including real Revit families in TestData/
└── installer/                          Inno Setup script and build script
```

## Built with

- [.NET 10](https://dotnet.microsoft.com/) and WPF
- [OpenMcdf](https://github.com/ironfede/openmcdf): reads the compound-file format of `.rfa` files
- [Microsoft.Data.Sqlite](https://learn.microsoft.com/dotnet/standard/data/sqlite/): the local library index
- [CommunityToolkit.Mvvm](https://github.com/CommunityToolkit/dotnet): MVVM helpers
- [Inno Setup](https://jrsoftware.org/isinfo.php): the Windows installer

## Roadmap

- [ ] Revit add-in to read instance parameters and formulas, and to load families with one click
- [ ] Favourites and custom tags
- [ ] Export the library to Excel or CSV
- [ ] Code-signed installer

## Credits

The two sample families in `tests/RevitFamilyManager.Core.Tests/TestData/` come from
[DynamoRevit](https://github.com/DynamoDS/DynamoRevit) and are used under the Apache License 2.0.

Revit and Autodesk are trademarks of Autodesk, Inc. This project is not affiliated with or endorsed by Autodesk.

## License

No license has been chosen yet. Until one is added, all rights are reserved by the author.
