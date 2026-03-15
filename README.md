# wine-proton — hybrid installer  `v1.0.0`

A hybrid Wine/Proton installer that merges any Wine build directly over a Proton base. Most Wine builds work at the moment, like certain versions of upstream Wine from WineHQ(https://www.winehq.org), certain ones you can grab from Kron4ek's Wine build releases page(https://github.com/Kron4ek/Wine-Builds/releases), and one's you can build yourself from places like WineHQ's source page(https://dl.winehq.org/wine/source/), Wine-TKG's source page(https://github.com/Frogging-Family/wine-tkg-git) with their build scripts to use, and Valve Software's source page(https://github.com/ValveSoftware/wine) to name a few good sources.  Any Proton tool works, like the builds you can download from Valve Software off of Steam(usually found in somewhere like ~/.steam/steam/steamapps/common or something similar), Kron4ek's Proton builds(https://github.com/Kron4ek/proton-archive/releases), GE-Proton builds(https://github.com/GloriousEggroll/proton-ge-custom/releases), and the ones you can build yourself from developers like Glorious Eggroll and TKG. Run the installer, use any version of Wine and Proton you'd like, and let it build with whatever combination you choose, producing a single ready-to-use compatibility tool! :3

The result can be registered as a **Steam compatibility tool** to be used in Steam, used **standalone** outside of Steam via the custom wrapper `./proton run program.exe`, or pointed to launchers like Lutris, Heroic, or Bottles.

---

## Why does this exist?

Proton gives you DXVK, VKD3D-Proton, Steam integration, and protonfixes. Certain Wine builds give you bleeding-edge patches, specific game fixes, or experimental features not yet in Proton. This installer merges the two — Wine's binaries and libraries overlay Proton's, while Proton's DXVK, Steam DLLs, and Python launcher are preserved.

---

## Requirements

| Tool | Notes |
|------|-------|
| `bash` 4+ | Ships with every modern distro |
| `rsync` | Used for all tree copies |
| `python3` | Required for Proton's Steam integration (Mode B) |
| `file` | Used for ELF arch detection |
| `find` | Standard GNU findutils |
| `yad` or `zenity` | *Optional* — enables GUI file pickers and progress bar. Falls back to terminal prompts if neither is present. |

Install missing deps on Debian/Ubuntu:
```bash
sudo apt install rsync python3 file yad
```

---

## What you need before running

1. **A Wine build** — a root Wine directory containing the /bin, /lib, /include, and /share folders in your existing Wine build. Then of course `wine`/`wine64`. The standard file directory path for `wine`/`wine64` is `wine-build/bin/`.

2. **A Proton source** — your existing Proton installation's root directory. The standard file directory path for your Proton tool is usually `~/.steam/steam/compatibilitytools.d/Proton/` if the tool is installed to steam. If it's somewhere else, make sure the root directory has all the proper python files(.py), manifest files(.vdf), version file, and the /files folder containing the rest of the tool's binaries, libraries, and necessary files.

3. *(Optional)* **A protonfixes directory** — a [protonfixes](https://github.com/nicowillis/protonfixes) or [umu-protonfixes](https://github.com/Open-Wine-Components/umu-protonfixes) source for per-game fixes. Auto-detected if present alongside your Wine build.

---

## Installation

```bash
git clone https://github.com/blu2442/wine-proton_hybrid
cd wine-proton_hybrid
chmod +x wine-proton_hybrid-v1.0.0.sh
./wine-proton_hybrid-v1.0.0.sh
```

The installer will walk you through source selection, tool naming, and install destination — all inputs can also be passed as CLI flags for headless use (see below).

---

## Usage

### Interactive (default)

```bash
./wine-proton_hybrid-v1.0.0.sh
```

You'll be prompted for:
- Wine build directory
- Proton source directory
- Tool name (default: `wine-proton`)
- protonfixes directory (optional, auto-detected)
- Install destination (Steam / custom)

All file pickers appear **before** the build starts, so the progress bar runs uninterrupted.

### Headless / CLI

All prompts can be bypassed with flags:

```bash
./wine-proton_hybrid-v1.0.0.sh \
  --wine-src      ~/builds/Wine \
  --proton-src    ~/.steam/steam/compatibilitytools.d/Proton \
  --name          wine-proton \
  --install-mode  steam
```

| Flag | Description |
|------|-------------|
| `--wine-src <dir>` | Path to your custom Wine build |
| `--proton-src <dir>` | Path to Proton source |
| `--name <name>` | Tool name (default: `wine-proton`) |
| `--install-mode <mode>` | `steam` · `steam-pick` · `custom` |
| `--install-dir <dir>` | Parent directory for custom installs |
| `--protonfixes-dir <dir>` | protonfixes source (umu-protonfixes, plain checkout, etc.) |
| `--dry-run` | Print commands without executing |
| `--verbose` | Echo every command as it runs |
| `--debug` | Dump Proton lib/wine layout after install |
| `--uninstall` | Remove a previously installed tool |

---

## Install destinations

| Mode | Where it goes |
|------|--------------|
| `steam` | Auto-detects your `compatibilitytools.d` |
| `steam-pick` | You pick the `compatibilitytools.d` manually |
| `custom` | Any directory — use with Lutris, Heroic, Bottles, or standalone |

After a Steam install, fully restart Steam (`killall -9 steam`) for the tool to appear in **Properties → Compatibility**.

---

## Standalone use (no Steam)

```bash
# Basic launch
~/tools/wine-proton/proton run /path/to/game.exe

# Launcher-wrapped game (GTA IV, etc.)
WINE_USE_START=1 ~/tools/wine-proton/proton run /path/to/GTAIV.exe

# Explicit Wine prefix
WINE_PROTON_PREFIX=~/.prefixes/mygame \
  ~/tools/wine-proton/proton run /path/to/game.exe

# Enable DXVK performance overlay
DXVK_HUD=1 ~/tools/wine-proton/proton run /path/to/game.exe

# Full Wine debug log (written to /tmp/proton_<ExeName>.log)
PROTON_LOG=1 ~/tools/wine-proton/proton run /path/to/game.exe

# DX12 game (opt-in — off by default to preserve vkd3d-proton)
WINEDLLOVERRIDES="d3d12=n,b" ~/tools/wine-proton/proton run /path/to/game.exe
```

### Wine prefix management

Prefixes are managed per-game automatically. Priority order:

| Variable | Behaviour |
|----------|-----------|
| `WINE_PROTON_PREFIX` | Use exactly this path — highest priority |
| `WINEPREFIX` (if not `~/.wine`) | Inherited with a warning — consider using `WINE_PROTON_PREFIX` instead |
| *(neither set)* | `~/.wine-proton/<ExeName>` — safe per-game default |

> **Tip:** If you have `WINEPREFIX` set globally in `~/.bashrc`, all games will share one prefix and stomp on each other. Remove it from your shell profile and use `WINE_PROTON_PREFIX` for per-game control.

---

## protonfixes support

The installer accepts any of the following as a protonfixes source:

- A built [umu-protonfixes](https://github.com/Open-Wine-Components/umu-protonfixes) repo (includes bundled `unzip`, `cabextract`, `libmspack`
- Any directory containing a `protonfixes/` subfolder or `__init__.py`

Auto-search paths (checked in order):
```
~/umu-protonfixes
~/protonfixes
~/projects/umu-protonfixes
~/projects/protonfixes
~/git/umu-protonfixes
<wine-src-parent>/umu-protonfixes
<proton-src>/protonfixes
```

---

## Known limitations

### ⚠ Unified binaries

Most Wine builds using the unified WoW64 architecture (single `wine` binary, no real `wine64`) have a known crash with games that have 32-bit processes, a 32bit launcher, use WoW64 etc. Even if the main game executable is 64-bit, it still crashes the game. The symptom is:

```
wine: Unhandled page fault at address 00006FFFFFC0D8A7 (thread 0024), starting debugger...
```

The fault happens at the WoW64 address space boundary during the launcher's process init, before any game code runs. The installer detects this and warns you at build time.

**Fix:** use a Wine build that ships a separate `wine64` binary (split layout). If building Wine yourself, use `--enable-archs=i386,x86_64` instead of the unified `--enable-archs=x86_64` flag. WineHQ, TKG, GE, and Valve Software Wine builds that use a true `wine`/`wine64` split layout(not a `wine64` symlink to the main `wine` binary) are unnaffected.  Sometimes, builds will work when building with a unified `wine` binary, but it's very picky about what it likes and most builds past about version 10.6 with a unified binary will not work due to how WoW64 is handled in recent versions of Wine.

This is all because Steam needs a `wine64` binary to function properly within the Steam Runtime environment. If it does not have it or the `wine` binary does not properly handle WoW64 in a way the `wine64` the script makes likes, then the tool will fail on any 32bit process. The script makes `wine64` if it isn't present by copying `wine` as `wine64` in the /files/bin directory of the tool in order for the tool to properly run in Steam, but if it's an incompatible binary, then the way it handles WoW64 causes the copied `wine64` binary to not be able to handle any processes and opens the crash reporter/debugger.

### EAC / BattlEye

- **EAC / BattlEye games** (Elden Ring, Hunt: Showdown, etc.) require a live Steam session and cannot run standalone through this tool. Use through Steam normally.

### DX12

- **DX12 games** need `WINEDLLOVERRIDES="d3d12=n,b"` if your Wine build doesn't ship its own `d3d12.dll`. Left off by default to avoid breaking games that depend on Proton's vkd3d-proton.

### 32-bit games

- **32-bit-only games** require `WINEARCH=win32` and a 32-bit Wine prefix.

---

## Uninstalling

```bash
# Steam install (auto-detected)
./wine-proton_hybrid-v1.0.0.sh --uninstall --name wine-proton

# Custom install location
./wine-proton_hybrid-v1.0.0.sh --uninstall --name wine-proton \
  --install-dir ~/tools
```

The uninstaller reads the `.wine-proton-install` receipt written into the tool directory to confirm what it's removing, then prompts for confirmation before deleting anything.

---

## How the build works

```
Proton base  (rsync copy into staging)
        │
        ▼
Wine DLLs overlay      (.dll  →  lib/wine/x86_64-windows/)
Wine .so overlay       (.so   →  lib/wine/x86_64-unix/)
Wine binaries          (wine, wineserver, wine-preloader  →  bin/)
Wine libraries         (lib/*.so*, lib64/*.so*)
        │
        ▼
Restore Proton originals:
  • DXVK (d3d9/10/11/dxgi via cp -n — Wine files take precedence)
  • Steam DLLs (lsteamclient, steamclient, vrclient, openvr_api)
  • default_pfx (symlinks materialised into real files)
        │
        ▼
protonfixes module installed
Steam manifests written (compatibilitytool.vdf, toolmanifest.vdf)
Install receipt written (.wine-proton-install)
        │
        ▼
Staged build → final install path
Launch wrapper written (proton)
Permissions fixed
Verified
```

---

## Install receipt

Every successful install writes a `.wine-proton-install` file into the tool directory:

```ini
installer_version="v1.0.0"
install_date="2026-03-08 15:00:00 UTC"
tool_name="wine-proton"
install_path="/home/user/.steam/steam/compatibilitytools.d/wine-proton"
install_mode="steam"
wine_src="/home/user/builds/wine
proton_src="/home/user/.steam/steam/compatibilitytools.d/Proton"
proton_version="Proton"
protonfixes_src="/home/user/protonfixes"
wine_unified_wow64="0"
```

This file is used by `--uninstall` and is useful for auditing what's installed and reproducing a build.

---

## Contributing

Issues and PRs welcome. If you have a game that needs a specific fix or a Wine layout that isn't detected correctly, please open an issue with:
- Your Wine build name and source
- The Proton version
- The layout of your Wine directory (`ls -la <wine-dir>/`)
- Any error or warning output from the installer

---

## Credits

Built on top of [WineHQ](https://www.winehq.org), [ValveSoftware](https://github.com/ValveSoftware), [Wine-TKG](https://github.com/Frogging-Family/wine-tkg-git), [Proton-TKG](https://github.com/Kron4ek/proton-archive), [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom), [umu-protonfixes](https://github.com/Open-Wine-Components/umu-protonfixes), and the broader Wine project.

---

*looni edition — made with love :3*
