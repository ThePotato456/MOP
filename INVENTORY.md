# MOP Codebase Inventory

Snapshot taken 2026-11-03, at upstream `master` (`af049c2`, MOP **3.10.4**), before any fork work.

## Overview

**MOP (Modern Optimization Plugin)** is a performance mod for *My Summer Car* (MSC). It raises FPS by
disabling objects, vehicles and items that are far away, by applying LODs, and by working around game
bugs this toggling exposes. Other mods stay compatible through downloadable **rule files**.

| | |
|---|---|
| Original author | Konrad Figura ([Athlon007](https://github.com/Athlon007/MOP)), Dec 2019 to Dec 2022 |
| Origin | Fork of [KruFPS](https://github.com/Krutonium/KruFPS) by Krutonium |
| Upstream status | Unmaintained. Its README invites forks |
| License | **GPLv3** (`LICENSE.md`). Keep existing copyright notices and state your changes (§5a) |
| Last release | 3.10.4 (2022-12-10). Last commit 2023-08-11 (README only) |

## Git history

- **1,144 commits**, all by the original author: `Athlon007` (769), `Athlon` (371) and `Konrad Figura` (13, GitHub web UI).
- **216 tags**, from `Beta-1.1` to `3.10.4`. Pre-releases use the `-beta_N` / `-dev.N` suffixes.
- Branches: `master`, `development`, `legacy`, `3.4`. Only `master` is checked out locally; the rest are `origin/*`.
- `.git` is **171 MB**, mostly art source files and screenshots (see Repo hygiene).
- Fork plan: keep the history and original authorship. Rename `origin` to `upstream`, then point `origin` at your fork.

## Repository layout

```
MOP.sln                     VS 2019 solution (Debug / Release / ProRelease)
MOP/                        The mod (C# class library -> MOP.dll)
  MOP.csproj                Old-style (non-SDK) project, net35, C# 7.3
  Properties/               AssemblyInfo (v3.10.4.0), Resources.resx
  Resources/                Embedded: icon, changelog.txt, donates.txt,
                            sources.json (rule servers), itemcounter_to_transform.csv,
                            mop.bundle (Unity asset bundle)
  src/                      126 .cs files, ~16.7k lines
AssetBundle/                Unity 5.0.0f4 project for the loading-screen canvas (MOP_Canvas.prefab)
Rulefiles/
  Database/                 40 .mopconfig rule files + servercontent.mop
  Website/                  Static site for the rule-file server (index.html, css, js, rulefiles.zip)
Scripts/build.ps1           Windows-only release script (builds both editions, zips, SHA256)
images/                     Banners, icons, benchmark screenshots, art sources (.xcf/.kra/.gvdesign)
.github/                    FUNDING.yml, issue templates, stale + issue-check workflows
CHANGELOG.md                Full release history (~85 KB)
```

## Source map (`MOP/src`)

| Path | Role |
|---|---|
| `MOP.cs` (664) | MSCLoader `Mod` entry point: settings UI, `MenuOnLoad`, `PostLoad`, `SecondPassOnLoad`, version/edition info |
| `Hypervisor.cs` (1,559) | **Core loop.** Owns the managers and runs the distance-based enable/disable passes. Largest file and main refactor target |
| `GameFixes.cs` (584) | Patches for vanilla game bugs that object toggling triggers |
| `Sector.cs` | Sectors: regions (e.g. buildings) where outside objects get disabled |
| `zModLoad.cs` | Load sequencing helper |
| `ModConsole.cs` | Logging wrapper |
| `Common/` | Settings (`MopSettings`), data, paths, bug reporter, exception manager, load screen, compatibility checks, console commands, save-slot support, enums (`PerformanceMode`: Performance / Balanced / Quality / Safe) |
| `Managers/` | `ItemsManager`, `PlaceManager`, `SectorManager`, `VehicleManager`, `WorldObjectManager` |
| `Vehicles/` | `Vehicle` base + one case per vehicle (Satsuma, Hayosiko, Gifu, Kekmet, Ferndale, Ruscko, Jonnez, Flatbed, Combine, Boat). `Managers/` holds hinge, hand-throttle and bus-roll fixes plus ~11 Satsuma-specific fixers |
| `Satsuma.cs` (1,170) | Player car handling. Second-largest file |
| `Items/` | `ItemBehaviour` (898), `ItemFreezer`, special-case items (suitcase money, junk, grill, bumper, cash register, floor jack…) |
| `Places/` | `Place` base + Farm, Inspection, RepairShop, Teimo (shop), Yard |
| `FSM/` | PlayMaker FSM manipulation: `FsmManager`, `FsmExtensions`, and ~16 custom FSM actions that replace game logic |
| `LOD/LodObject.cs` | Generated LODs for distant objects |
| `WorldObjects/` | Generic, renderer, multi-renderer and simple object toggles, plus skidmarks |
| `Rules/` | Rule-file system: `RulesManager`, `Configuration/Loader.cs` (813, parser + downloader), `ServerSources`, rule types |
| `Helpers/` | Save manager (776), save-bug detection, draw distance, item anti-clip, pause menu, death handling |
| `DebugTools/DebugMonitor.cs` | Debug overlay |

### Rule-file keywords (parsed in `Rules/Configuration/Loader.cs`)
`ignore`, `ignore_full`, `ignore_mod_vehicles`, `toggle`, `item`, `vehicle`, `vehicle_physics`, `renderer`,
`sector`, `change_parent`, `no_lod`, `no_lods`, `min_ver`, `satsuma_ignore_renderer`, `skip_fury_collider_fix`,
`toggle_all_vehicles_physics_only`, `dont_destroy_empty_bottles`, `dont_destroy_empty_beer_bottles`

### Console commands (`mop …`)
`help`, `version`, `rules`, `reload [all]`, `new|open|delete [ModID]`, `open-config`, `cat [file]`,
`generate-list`, `load-rules`, `force-crash [critical]`, `resolution …`, `force-load-restart`,
`stop`, `start`, `quality [0-5]`, `debug_monitor` (misspelled `debug_montior` in the help text), `presave`

## Build & dependencies

- **Target:** .NET Framework **3.5**. This is required because MSC runs on Unity 5's old Mono runtime. Don't move to modern .NET; newer *C# language* versions are fine.
- **References** (none are in the repo; they come from the game install). All are hardcoded to
  `C:\Program Files (x86)\Steam\steamapps\common\My Summer Car\mysummercar_Data\Managed\`:
  `Assembly-CSharp`, `PlayMaker`, `MSCLoader`, `ES2`, `Ionic.Zip`, `Newtonsoft.Json`, `UnityEngine`, `UnityEngine.UI`
- **Editions:** `Release` = MSCLoader, `ProRelease` = Mod Loader Pro (`#if PRO` in 27 places, different DLL path).
  The `Pro` config exists in the csproj but not in the `.sln`.
- **MSCLoader settings API:** about 100 `Settings.Add*` calls in `MOP.cs` (AddButton ×27, AddHeader ×16,
  AddText ×15, AddCheckBox ×9, AddToggle ×9, AddSlider ×8…), written against MSCLoader 1.2.8.
- **Build script:** `Scripts/build.ps1` is Windows-only and uses a hardcoded VS 2022 MSBuild path.
- **Post-build event:** copy-to-Mods-folder lines, all commented out.
- **Local toolchain:** this machine has no `dotnet`, `mono` or `msbuild` installed.

## External services & hardcoded URLs

| What | Where | Status / action |
|---|---|---|
| Rule-file servers `kfigura.nl/mop/rulefiles`, `athlon.kkmr.pl/mop/rulefiles` | `Resources/sources.json` | Original author's domains, probably offline (not checked). **Needs your own host**, e.g. GitHub Pages from `Rulefiles/Website` |
| Wiki/FAQ `kfigura.nl/mop/wiki`, `athlon.kkmr.pl/...` | `src/` + README | Same. Move docs into the repo or a GitHub wiki |
| `UpdateLink`, issue URL `github.com/Athlon007/MOP` | `MOP.cs`, bug reporter | Point at the fork |
| Ko-fi / PayPal | `MOP.cs`, README, `FUNDING.yml`, `donates.txt` | Original author's. Replace or remove |
| `Author => "Athlon"` | `MOP.cs:37` | Update, e.g. "Athlon, ThePotato456" |
| Nexus Mods page | `MOP.cs` | Original author's listing |

## Known problems / tech debt

1. **Non-portable build:** old csproj format, absolute Windows paths, no CI build.
2. **Loader drift:** targets MSCLoader 1.2.8. Newer MSCLoader versions changed APIs (verify against the current release). Mod Loader Pro may be discontinued, in which case `PRO` can be removed.
3. **Dead infrastructure:** rule servers and wiki links (see above).
4. **Broken workflow:** `.github/workflows/template-issues.yml` sets shell variables in separate steps, which don't share variables, so the comparison always fails and the job exits 1 on every new issue.
5. **Aggressive stale bot:** issues close after 14 + 2 days.
6. **Large classes:** `Hypervisor.cs`, `Satsuma.cs`, `ItemBehaviour.cs`, `Loader.cs`, `SaveManager.cs`.
7. **No tests.**
8. **Unity 5.0.0f4** AssetBundle project. Rebuilding the bundle needs that exact old editor.

## Repo hygiene

- `mop.bundle` is committed twice (`AssetBundle/AssetBundles/` and `MOP/Resources/`).
- Art source files (`.xcf`, `.kra`, `.gvdesign`), `images/3.0/Thumbs.db` and many benchmark screenshots
  account for most of the 171 MB history. Consider Git LFS or a separate assets repo for new art.
- `Rulefiles/Website/rulefiles.zip` is a generated artifact committed to the repo.

## Licensing obligations (GPLv3)

- Keep the existing `Copyright © Konrad Figura` notices (121 source files + `AssemblyInfo.cs`); add your own beside them.
- State clearly that this is a modified version (README + changelog).
- Keep credits for Athlon007, Krutonium/KruFPS (RedJohn260, EPS) and the listed contributors/testers.
- Distributed builds must stay GPLv3 and come with the source.

## Upgrade roadmap (proposed)

1. **Fork setup.** Fork on GitHub, re-point remotes, update README/branding/credits, bump to a new version line.
2. **Modern build.** Convert to an SDK-style csproj (`net35` via `Microsoft.NETFramework.ReferenceAssemblies`),
   with a configurable `MscPath` in a git-ignored `Directory.Build.props.user`, a newer `LangVersion`, and a cross-platform build script.
3. **CI.** GitHub Actions for compile checks (needs stub/reference assemblies, since the game DLLs can't be committed),
   a release workflow, and removal of the broken issue-check workflow.
4. **Loader update.** Port to the current MSCLoader API and decide whether to drop the `PRO` edition.
5. **Rule files.** Self-host on GitHub Pages and audit the 40 rule files against current versions of those mods.
6. **Improvements.** Break up `Hypervisor`, profile the main loop, triage open upstream issues, and add tests for the rule parser.
