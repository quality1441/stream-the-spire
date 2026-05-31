# Image catalog

**Image mode** on the config page shows full card, relic, and potion art on stream. That art comes from a **local folder on your PC** - it is not bundled with Stream the Spire.

If you skip the catalog, use **[text mode](configuration-and-testing.md#display-mode-image-vs-text)** instead. Text toasts work with no extra downloads.

---

## Why a separate download?

Game art is large, updates with every STS2 patch, and is subject to **Mega Crit’s licensing**. Stream the Spire keeps the overlay small by pointing at an optional catalog repo instead of shipping images inside the release zip.

Pre-built catalogs live here:

**[github.com/quality1441/sts2-image-versions](https://github.com/quality1441/sts2-image-versions)**

Download or clone that repo (or sync a release from it), then pick the **version folder** that matches your game patch (e.g. `v0.103.2`).

---

## Folder layout

The server expects WebP files under three subfolders:

```text
{STS2_IMAGE_VERSIONS_ROOT}\{STS2_GAME_VERSION}\
  cards\
  potions\
  relics\
```

Example after downloading the catalog:

```text
C:\dev\sts2-image-versions\
  v0.103.2\
    cards\
    potions\
    relics\
```

- **`STS2_IMAGE_VERSIONS_ROOT`** - parent folder containing one subfolder per game version (`v0.103.2`, `v0.104.0`, …).
- **`STS2_GAME_VERSION`** - name of the subfolder to use (include the `v` prefix if the folder uses it).

Cards may include nested paths (character pools, etc.). Relics and potions follow the same `{kind}\{itemId}.webp` style the server indexes at startup.

---

## Config page (recommended)

1. Open **http://127.0.0.1:5055/config**.
2. Under **Global settings → Image catalog**, set:
   - **Catalog root** → `{STS2_IMAGE_VERSIONS_ROOT}` (e.g. `C:\dev\sts2-image-versions`)
   - **Game version folder** → `{STS2_GAME_VERSION}` (e.g. `v0.103.2`)
3. Set **Display mode** to **Image** (tabs above the layout preview).
4. **Save config**.

The config page shows a **catalog status** line under the version folder (green = loaded at server start; yellow = paths changed - restart server).

---

## Environment variables **(OPTIONAL)**

Set these **before starting the server** if you prefer env over the config page.

| Variable | Default | Purpose |
|----------|---------|---------|
| `STS2_IMAGE_VERSIONS_ROOT` | `C:\dev\sts2-image-versions` | Root folder with version subfolders |
| `STS2_GAME_VERSION` | `v0.103.2` | Version subfolder name under the root |
| `STS2_OVERLAY_VERSION_ROOT` | - | **Advanced:** point directly at a folder that already contains `cards\`, `potions\`, and `relics\` (skips root + version join) |

**Precedence:** If `STS2_OVERLAY_VERSION_ROOT` is set, it wins over catalog root + game version. The legacy alias `STS2_OVERLAY_ASSET_ROOT` is treated the same way.

Config page paths are used when env vars are not set (or for the root/version pair when `STS2_OVERLAY_VERSION_ROOT` is unset).

---

## Restart the server after path changes

The server resolves catalog paths **at startup**. After you change catalog root, game version, or any of the env vars above:

1. **Save** on the config page (if you changed settings there).
2. **Stop** `run-server.cmd` (Ctrl+C).
3. **Start** the server again.

Hard-refresh `/overlay` in OBS if art still looks cached.

---

## Text mode without a catalog

You do not need sts2-image-versions for streaming.

1. On the config page, set **Display mode** to **Text**.
2. Save config.

See **[Configuration and testing](configuration-and-testing.md)** for layout, timing, and sounds in text mode.

---

## Version mismatch symptoms

Using a catalog folder for the **wrong game patch** (or an empty/missing subfolder) often shows up as:

- **Missing art** - blank or text-only fallback toasts in image mode.
- **Wrong frames or borders** - card pool / character frames do not match the item.
- **404 in browser** - a direct `/cards/…` or `/relics/…` URL does not load.

**Fix:** Match **Game version folder** to your STS2 patch, confirm `cards\`, `potions\`, and `relics\` exist under that folder, **restart the server**, and test again.

More fixes: **[Troubleshooting](troubleshooting.md)**.

---

## Related

- [Getting started](getting-started.md) - optional catalog step in first-run checklist
- [Introduction → Display modes](introduction.md#display-modes)
- [sts2-image-versions on GitHub](https://github.com/quality1441/sts2-image-versions)

---

[← Documentation index](README.md)
