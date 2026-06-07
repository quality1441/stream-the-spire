# Troubleshooting

Symptom → checks → fix for **Stream the Spire**. Work through the [first-time checklist](#first-time-setup-checklist) if you are new.

---

## Symptom matrix

| Symptom | Likely cause | Fix |
|---------|----------------|-----|
| **No toasts in OBS** | Server stopped, wrong OBS URL, mod not posting, filters | [No toasts in OBS](#no-toasts-in-obs) |
| **Toasts in browser, not OBS** | Wrong URL, hidden source, zero size | [Browser works, OBS does not](#toasts-in-browser-but-not-obs) |
| **Text only / missing art** | Text mode, no catalog, wrong catalog version | [Missing art](#text-only--missing-art) |
| **Wrong or missing card frames** | Catalog version mismatch | [Image catalog](image-catalog.md#version-mismatch-symptoms) |
| **`/item-ids` empty** | Catalog paths wrong or not loaded | [Empty item IDs](#item-ids-empty) |
| **Config changes not applying** | Forgot Save, OBS cache, unsaved catalog fields | [Config not applying](#config-changes-not-applying) |
| **Test works, game does not** | Mod missing, wrong `STS2_OVERLAY_URL`, not in a run | [Test works, game does not](#test-works-but-game-does-not) |
| **Missing enchant layers** | Text mode, dual-PC cache, wrong catalog | [Missing enchant art](#missing-enchant-art-image-mode) |
| **Animations stutter in OBS** | Browser source perf, shutdown-when-hidden | [Stuttery animations](#animations-stutter-in-obs) |
| **Port 5055 in use** | Second server instance | [Port in use](#port-5055-already-in-use) |

---

## No toasts in OBS

1. **`run-server.cmd` running?** Open **http://127.0.0.1:5055/config** in a browser.
2. **OBS URL** - Must be **`http://127.0.0.1:5055/overlay`**, not `/config`. [OBS setup](obs-setup.md).
3. **WebSocket** - Add **`?debug`** to overlay URL; status should show connected (green). If red, server down or wrong host/port ([Dual PC](dual-pc-setup.md) uses LAN IP).
4. **Mod** - `mods\Sts2StreamOverlay\` present? Restart STS2 after copying. [Mod installation](mod-installation.md).
5. **In a run?** - Events fire during a run, not on the main menu.
6. **Config filters** - Type/event toggles off, or ID on ignore list. [Ignore lists](ignore-items.md).
7. **Slots** - At least one slot for that item type; check layout preview.

**Where to look:** Server console (`Toast queued` / `Toast shown`), overlay **`?debug`**, game log (`Sts2StreamOverlay: events endpoint …`).

---

## Toasts in browser but not OBS

- OBS source URL must match server ( **`127.0.0.1` vs LAN IP** - see [Dual PC setup](dual-pc-setup.md)).
- Browser Source **width/height** not zero; transparent background not a solid black panel.
- Source visible in current scene; not covered by other sources.
- **Refresh** the Browser Source (CEF cache) - [OBS setup](obs-setup.md#refresh-when-the-overlay-looks-stale).

---

## Text only / missing art

- **Display mode** on `/config` - **Text** skips catalog art. Switch to **Image** and **Save**.
- **Catalog** - Paths set on config page? Folders `cards\`, `relics\`, `potions\` exist? **Save config** after path changes (server reloads catalog live). Check the green catalog status line under **Game version folder**. [Image catalog](image-catalog.md).
- **Version folder** - Must match your STS2 patch (e.g. `v0.103.2`).

---

## `/item-ids` empty

- Catalog **root** and **game version** correct on config page; **Save config** so the index reloads.
- Catalog download includes item metadata; without a catalog, use in-game names and logs to find IDs, or run in **text mode** without `/item-ids`.

---

## Config changes not applying

| Change | Action |
|--------|--------|
| Layout, ignore, modes, sounds | Click **Save config** on `/config` |
| Catalog root / game version | Click **Save config** (catalog reloads immediately) |
| OBS still old layout | **Refresh** Browser Source; confirm canvas size matches preview |
| After **Test** only | **Test** does not write disk - **Save** to persist |

Overlay receives saved config over WebSocket; no OBS restart needed for most settings.

---

## Test works, game does not

| Check | Fix |
|-------|-----|
| **Test** uses debug API | Game needs **mod** posting to same server |
| **`STS2_OVERLAY_URL`** | Default `http://127.0.0.1:5055`; set before launching STS2 if server uses another URL |
| **Server down** | Start before playing |
| **Not in run** | Start or continue a run |
| **Mod folder** | `Sts2StreamOverlay.dll` under `mods\Sts2StreamOverlay\` |

---

## Animations stutter in OBS

- Lower other browser source load; disable unnecessary sources.
- Try **Shutdown source when not visible** off on the overlay source - [OBS setup](obs-setup.md).
- Match Browser Source FPS to canvas if OBS exposes the option.
- Simpler enter/exit animations on config page.

---

## Missing enchant art (image mode)

- **Display mode** must be **Image** on `/config` (enchant layers are not shown in text mode).
- **Catalog** must match your game patch; enchant wedge/tab WebPs live under the catalog `cards\` folder when exported.
- **Mod cache** supplies some enchant icons. On dual-PC setups, run the server on the **gaming PC** or set **`STS2_MOD_CACHE_ICONS`** to the gaming PC cache path. See [Dual PC setup](dual-pc-setup.md) and [Image catalog → Enchanted cards](image-catalog.md#enchanted-cards-image-mode).
- Test without the game: `scripts\test-overlay-enchanted.cmd` (server running, image mode).

---

## Port 5055 already in use

Another **OverlayServer** (or app using 5055) is running.

- Close the other server window or end the process.
- Dev builds: `Get-Process OverlayServer | Stop-Process` in PowerShell.
- Or change port via `STS2_OVERLAY_URL` / `ASPNETCORE_URLS` (advanced; update OBS and mod URLs to match).

---

## First-time setup checklist

1. Extract release zip; run **`run-server.cmd`**.
2. **`http://127.0.0.1:5055/config`** loads.
3. Copy **`mod/Sts2StreamOverlay/`** → STS2 **`mods\`**; restart game if it was open.
4. OBS Browser Source → **`/overlay`**, 1920×1080, transparent.
5. Config: canvas size = OBS size, place slots, **Save**.
6. **`?debug`** on overlay - WebSocket connected.
7. Start a run; gain an item - toast appears.

Full walkthrough: [Getting started](getting-started.md).

---

## When to restart vs refresh

| Situation | Restart server | Hard-refresh browser / OBS source |
|-----------|----------------|-----------------------------------|
| Changed catalog paths | No ( **Save config** reloads catalog) | Refresh OBS if art looks cached |
| Saved layout / ignore / modes | No | Refresh if stale |
| Upgraded Stream the Spire files | **Yes** | Refresh OBS |
| WebSocket disconnected | Check server first | Refresh overlay source |

---

## Related

- [Getting started](getting-started.md)
- [Mod installation](mod-installation.md)
- [OBS setup](obs-setup.md)
- [Configuration and testing](configuration-and-testing.md)
- [← Documentation index](README.md)
