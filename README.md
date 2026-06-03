<img width="1376" height="768" alt="Stream-The-Spire-Wallpaper" src="https://github.com/user-attachments/assets/da7c5b7e-9e20-4181-ab61-6f0eb3fb8d57" />

# Stream the Spire

Live overlay toasts for **Slay the Spire 2** streams.

**Download:** [GitHub Releases](https://github.com/quality1441/sts2-stream-overlay/releases) (`stream-the-spire-win64-vX.Y.Z.zip`)  
**Documentation:** [docs/README.md](docs/README.md): read online, no download required  
**Setup video:** [YouTube walkthrough](docs/setup-video.md)  
**Repository:** [github.com/quality1441/sts2-stream-overlay](https://github.com/quality1441/sts2-stream-overlay)

## What you get

| Piece | URL / path |
|-------|------------|
| Overlay (OBS) | `http://127.0.0.1:5055/overlay` |
| Config | `/config` - layout, animation bands, modes, sounds, ignore lists |
| Help | `/help` - in-app quick guide (server running) |
| Item IDs | `/item-ids` - browse IDs for ignore lists |
| Game mod | `mod/Sts2StreamOverlay/` in the release zip → STS2 `mods/` |

## Quick start

1. Download **`stream-the-spire-win64-vX.Y.Z.zip`** from [Releases](https://github.com/quality1441/sts2-stream-overlay/releases) and extract.
2. Install **[.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0)** if needed.
3. Run **`run-server.cmd`** → server at **http://127.0.0.1:5055**
4. **Copy the mod into Slay the Spire 2**: copy the whole **`mod/Sts2StreamOverlay/`** folder from your extract into the game’s **`mods`** folder:

   ```text
   From:  {your extract}\mod\Sts2StreamOverlay\
   To:    C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2\mods\Sts2StreamOverlay\
   ```

   Restart STS2 if it was already open. Non-default install path: **[docs/mod-installation.md](docs/mod-installation.md)**.

5. OBS **Browser Source** → **`http://127.0.0.1:5055/overlay`** (1920×1080, transparent)
6. Open **`/config`**, place slots, **Save**

Optional full-art mode: [sts2-image-versions](https://github.com/quality1441/sts2-image-versions) - see **[docs/image-catalog.md](docs/image-catalog.md)**.

**Prerequisites:** [.NET 10 ASP.NET Core Runtime](https://dotnet.microsoft.com/download/dotnet/10.0) (Windows x64): see **[docs/getting-started.md](docs/getting-started.md)**.

## Documentation

- **[docs/README.md](docs/README.md)** - full documentation index
- **[docs/getting-started.md](docs/getting-started.md)** - first-run checklist
- With server running: **http://127.0.0.1:5055/help**

Maintainers: **[scripts/README.md](scripts/README.md)** (build, test, `publish-release.cmd`).

## Environment variables (common)

| Variable | Default | Purpose |
|----------|---------|---------|
| `STS2_OVERLAY_URL` | `http://127.0.0.1:5055` | Mod → server URL (set before launching STS2) |
| `STS2_IMAGE_VERSIONS_ROOT` | `C:\dev\sts2-image-versions` | Optional catalog root - see docs |

More variables: **[docs/image-catalog.md](docs/image-catalog.md)**, **[docs/mod-installation.md](docs/mod-installation.md)**.

## License

Slay the Spire 2 and its assets are property of Mega Crit. Unofficial fan tool — see [docs/README.md](docs/README.md) for details.
