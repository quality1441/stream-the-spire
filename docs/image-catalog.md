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

Cards may include nested paths (character pools, etc.). Relics and potions follow the same `{kind}\{itemId}.webp` style the server indexes when the catalog loads.

---

## Config page (recommended)

1. Open **http://127.0.0.1:5055/config**.
2. Under **Global settings → Image catalog**, set:
   - **Catalog root** (e.g. `C:\dev\sts2-image-versions`)
   - **Game version folder** (e.g. `v0.103.3`)
3. Set **Display mode** to **Image** (tabs above the layout preview).
4. **Save config**.

The server reloads the catalog when you save. You do **not** need to restart `run-server.cmd` after changing catalog root or game version.

The **catalog status** line under the version folder updates after save (green = folders found and WebP files indexed; red = missing folder or empty catalog). Hard-refresh `/overlay` in OBS if art still looks cached in the browser.

---

## Environment variables (optional fallbacks)

The running overlay server reads **Catalog root** and **Game version folder** from saved config (`data/config.json`). Environment variables are optional fallbacks when those config fields are empty, and are still used by maintainer scripts (for example `build-item-index.cmd`).

| Variable | Default | Purpose |
|----------|---------|---------|
| `STS2_IMAGE_VERSIONS_ROOT` | `C:\dev\sts2-image-versions` | Fallback catalog root when config does not set one |
| `STS2_GAME_VERSION` | `v0.103.2` | Fallback version subfolder when config does not set one |
| `STS2_OVERLAY_VERSION_ROOT` | - | **Advanced:** point directly at a folder that already contains `cards\`, `potions\`, and `relics\` (skips root + version join). Used only when config catalog fields are empty. Legacy alias: `STS2_OVERLAY_ASSET_ROOT`. |

**Precedence:** Saved config wins over `STS2_IMAGE_VERSIONS_ROOT` and `STS2_GAME_VERSION`. Direct version root env vars apply only when config catalog fields are empty.

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

**Fix:** Match **Game version folder** to your STS2 patch, confirm `cards\`, `potions\`, and `relics\` exist under that folder, **Save config** on the config page, and test again.

More fixes: **[Troubleshooting](troubleshooting.md)**.

---

## Enchanted cards (image mode)

When a **`card_enchanted`** event fires in **image mode**, the overlay can show:

- The base card art from your catalog
- Enchant wedge, icon, and amount layers (when present under `cards\` or from the mod icon cache)
- Extra enchant description text on the card panel (from live game data, with server-side fallbacks in `data/enchantments.json`)

**Text mode** shows a compact name toast only (no layered enchant art).

If enchant icons are missing on a dual-PC setup, the server may not see the mod cache on the gaming PC. See [Dual PC setup](dual-pc-setup.md) and set **`STS2_MOD_CACHE_ICONS`** when the server and game run on different machines.

---

## Related

- [Getting started](getting-started.md) - optional catalog step in first-run checklist
- [Introduction → Display modes](introduction.md#display-modes)
- [sts2-image-versions on GitHub](https://github.com/quality1441/sts2-image-versions)

---

[← Documentation index](README.md)
