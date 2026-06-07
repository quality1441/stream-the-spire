# Getting started

Download the **Windows 64-bit release zip**, extract it, run the server, copy the mod folder, and add OBS. No Git or build tools required.

**Prefer video?** [Watch the full setup guide on YouTube](setup-video.md).

## What you need

- **Windows** PC
- **[.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0)** (Windows x64): required to run `OverlayServer.exe`; install once if `run-server.cmd` asks for a runtime
- **Slay the Spire 2** (Steam)
- **OBS Studio** (or any app with a transparent browser source)
- Optional: local [image catalog](image-catalog.md) for full-art toasts

---

## Install from the release zip

### 1. Download the release

On GitHub, open **[Releases](https://github.com/quality1441/sts2-stream-overlay/releases)** and download the latest Windows package:

```text
stream-the-spire-win64-vX.Y.Z.zip
```

Replace `vX.Y.Z` with the version tag on the release you picked.

### 2. Extract the zip

Extract to any folder you like. Example:

```text
C:\StreamTheSpire\
```

Avoid paths that need Administrator rights for everyday use (e.g. don’t extract only under `Program Files` unless you intend to run as admin).

### 3. What’s inside the zip

| Path | Purpose |
|------|---------|
| `run-server.cmd` | Starts the overlay server (double-click or run from a terminal) |
| `OverlayServer.exe` | Server executable (requires [.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0) on your PC) |
| `wwwroot/` | Overlay, config, and item-ID web pages |
| `data/config.json` | Saved settings (created or updated when you use the config page) |
| `mod/Sts2StreamOverlay/` | Pre-built game mod (`Sts2StreamOverlay.dll`, manifest, etc.) |
| `GETTING_STARTED.txt` | Short copy of these steps inside the zip |

Keep the folder intact - `run-server.cmd` expects `OverlayServer.exe`, `wwwroot/`, and `data/` next to it.

### 4. Run the overlay server

Install the **[.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0)** (Windows x64) if you have not already.

Double-click **`run-server.cmd`** (or run it from PowerShell/CMD inside the extract folder).

When it starts successfully:

- Server: **http://127.0.0.1:5055**
- Overlay (OBS): **http://127.0.0.1:5055/overlay**
- Config: **http://127.0.0.1:5055/config**

Leave this window open while you play and stream. Close it or press Ctrl+C to stop the server.

If the port is already in use, another copy of the server may still be running - close it and try again.

If Windows says a **.NET runtime** is missing, install **ASP.NET Core Runtime 10.0** from the link above and run `run-server.cmd` again.

### 5. Install the game mod

Copy the entire **`mod/Sts2StreamOverlay/`** folder into your STS2 **mods** directory so you have:

```text
{Slay the Spire 2}\mods\Sts2StreamOverlay\
  Sts2StreamOverlay.dll
  Sts2StreamOverlay.json
  …
```

**Default Steam path:**

```text
C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2\mods\Sts2StreamOverlay\
```

If your game is installed elsewhere, copy into that install’s `mods\Sts2StreamOverlay\` folder instead.

**Non-default install:** set `STS2_GAME_ROOT` to your game folder before copying (PowerShell example):

```powershell
$game = 'D:\Games\Slay the Spire 2'
Copy-Item -Recurse -Force 'C:\StreamTheSpire\mod\Sts2StreamOverlay' (Join-Path $game 'mods\Sts2StreamOverlay')
```

See [Mod installation](mod-installation.md) for `STS2_OVERLAY_URL` if the server is not on `http://127.0.0.1:5055`.

STS2 loads mods from that folder automatically - no separate enable step. Restart the game if it was already running when you copied the files.

### 6. Add the overlay to OBS

1. In OBS, add a **Browser Source**.
2. **URL:** `http://127.0.0.1:5055/overlay`
3. Set **Width** and **Height** to your stream canvas (e.g. **1920 × 1080**).
4. Enable a **transparent** background (default for the overlay page).

You should see a small WebSocket status line when debugging; add `?debug` to the URL if you need connection hints. Hide that line in production by using the URL without debug once things work.

### 7. Customize on the config page

Open **http://127.0.0.1:5055/config** in a normal browser tab (not necessarily in OBS).

From there you can:

- Drag **slot** positions on the layout preview (match preview canvas size to your OBS browser source).
- Choose **image** or **text** display mode.
- Set animations, **sound mode** (on / off / first and last event), ignore lists, and **animation bands** (when multiple toasts play).

Click **Save config** - the OBS overlay picks up changes over WebSocket without restarting OBS.

Browse item IDs for ignore lists at **http://127.0.0.1:5055/item-ids**.

### 8. Optional: image catalog (full-art mode)

Image mode uses WebP files from a local catalog (not included in the release zip). Download **[sts2-image-versions](https://github.com/quality1441/sts2-image-versions)**, then set **Catalog root** and **Game version** on the config page and **Save config**. The server reloads art immediately; no restart needed.

Details: [Image catalog](image-catalog.md) - or [Introduction → Optional image catalog](introduction.md#optional-image-catalog).

**Text mode** works without a catalog.

---

## First-run checklist

Work through this list in order on your first stream:

1. Download **`stream-the-spire-win64-vX.Y.Z.zip`** from [GitHub Releases](https://github.com/quality1441/sts2-stream-overlay/releases).
2. Extract to a permanent folder (e.g. **`C:\StreamTheSpire\`**).
3. Install **[.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0)** if you do not already have it.
4. Run **`run-server.cmd`** and confirm **http://127.0.0.1:5055/config** opens in your browser.
5. Copy **`mod/Sts2StreamOverlay/`** into **`{Slay the Spire 2}\mods\Sts2StreamOverlay\`** (use `STS2_GAME_ROOT` if Steam is not in the default path). Restart STS2 if it was already open.
6. Add an OBS **Browser Source** → **`http://127.0.0.1:5055/overlay`**, transparent, sized to your canvas.
7. On **`/config`**, set layout preview canvas to your OBS size, place at least one slot per type you want, and **Save config**.
8. Start a run, pick up a card/relic/potion, and confirm a toast appears in OBS.
9. *(Optional)* Set up an [image catalog](image-catalog.md) for full-art toasts.

---

## Next steps

- [Introduction](introduction.md) - what the overlay can and can’t do
- [OBS setup](obs-setup.md) - Browser Source details
- [Documentation index](README.md)
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
