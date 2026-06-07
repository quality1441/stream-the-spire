# Config & overlay manual tests

Use after config UI changes. Server: `scripts\run-server.cmd` (http://127.0.0.1:5055). Work in `OverlayServer/wwwroot/` (source); hard-refresh `/config` after static changes; restart the server after `Program.cs` / schema changes.

## Config page layout (smoke)

| Area | What to check |
|------|----------------|
| **Header** | Title left; **Help** and **Donate** buttons right (`/config` only). |
| **Global settings** | **Timing and animation** (delay, duration, enter/exit anim); **Sound** (mode, enter/exit sound pickers); **Event defaults** (ignore IDs); catalog root + GitHub link; game version; catalog status after **Save**. |
| **Layout preview** | **Display mode** tabs (Image / Text) above canvas; canvas preset / custom W/H; Snap 8px / Snap to slots; **Reset slots** and **Test** on top toolbar; scale dock beside preview. |
| **Animation bands** | Band playback dropdown; **Add card/relic/potion/band**; band cards with slot playback, enter/exit anim, draggable chips, **Add selected slot**, **Remove band**; validation notice when misconfigured. |
| **Item types** | Cards / Relics / Potions browser tabs with **On**; fused panel below with **Event rules**. |
| **Event rules** | Per-event browser tabs (Gained, Removed, …) each with **On**; delay + show overrides. |
| **Bottom action bar** | Status line; **Export setup…**, **Import setup…**, **Save config**, **Reload**. |
| **Validation** | Add slots past limits (4/type, 12 total): notice appears and save is blocked until fixed. |

## Config round-trip (`slots[]`, `itemTypes`, `animationBands`, globals)

Automated:

```text
scripts\verify-config-roundtrip.cmd
```

Manual:

1. Set global ignore ID, change global enter animation, tweak one event’s delay override.
2. Create two bands with different slot playback and enter animations; drag chips between bands.
3. **Save config**: status should show slot count.
4. Hard-refresh `/config`: values match.
5. Inspect `data/config.json`: `slots`, `animationBands`, `itemTypes`, `displayMode`, `soundMode`, `previewCanvasPreset` / width / height, globals, per-type scales.

## Image catalog (live reload)

1. Note current **Game version folder** and catalog status line on `/config`.
2. Change to another valid version folder that exists on disk; **Save config**.
3. Catalog status should update without restarting the server (`/api/catalog-health` `gameVersion` matches saved config).
4. Optional: `scripts\test-overlay-enchanted.cmd` in **image mode** shows layered enchant art.

## Sound modes

Requires server restart after `Program.cs` changes; **Save config** after UI changes. Open **`/overlay`** (or OBS) to hear sounds.

| Setup | Expected |
|-------|----------|
| **Sound mode** = First event and last event end, **Test** with 3+ slots | One enter sound total, one exit sound total; server logs show `enterSound=none` / `exitSound=none` on middle toasts |
| **Sound mode** = On (every toast), **Test** | Enter and exit on every toast (queued, not overlapping) |
| **Sound mode** = Off | No sounds; logs show `enterSound=none` and `exitSound=none` on all toasts |
| Change sound mode, **Test** without Save | Test uses current form value (including unsaved sound mode) |
| Change sound mode, Save, hard-refresh | Dropdown matches `soundMode` in `config.json` |

Automated round-trip:

```text
scripts\verify-config-roundtrip.cmd
```

(confirms `soundMode` persists through PUT/GET)

## Animation bands

Automated scenarios:

```text
scripts\test-overlay-queue.cmd -BandTest -FastDisplay
scripts\test-overlay-queue.cmd -BandTest -BandScenario SequentialWithinBand -NoBrowser
```

Manual:

| Setup | Expected on overlay / server logs |
|-------|-----------------------------------|
| One band, 2 card slots, slot playback **parallel**, fire 2 cards quickly | Both show together; two `Band assign` lines |
| One band, 2 card slots, slot playback **sequential**, fire 2 cards quickly | First card in slot 1; second in slot 2 while first is still up; second slides to slot 1 when first exits |
| Band 1 = card, Band 2 = relic, band playback **sequential**, 1 card + 1 relic | Card first; relic after card band done |
| Per-band enter anim differs from global | Toast uses band enter anim when assigned from that band |

## Export / import setup

1. Arrange slots + bands → **Export setup…** → `sts2-overlay-setup.json` downloads.
2. Change layout on form → **Import setup…** → confirm → form restores export (unsaved).
3. **Save config** → hard-refresh → layout and bands match export.
4. Import invalid file (wrong `kind`) → status error, form unchanged.

Import must **not** wipe global delay, ignore lists, or event rules until you Save over them (merge strategy: layout fields only).

## Global event defaults

1. Add an item ID to **Ignore item IDs (all events)** → save.
2. Trigger that ID for `card_gained` and `relic_gained` (debug or in-game): both should be ignored.
3. Change **Enter animation** globally → save → confirm multiple event types use the new animation on stream.

## Per-event rules

1. Cards → **Gained** tab → uncheck **On** → save → `card_gained` should not enqueue.
2. **Removed** tab → enable delay/show overrides with non-default values → save → only that event uses overrides.

## Preview vs OBS overlay

- Set layout preview canvas to your OBS Browser Source resolution (e.g. **1920 × 1080**) and **save**: reload should restore the same canvas size.
- **Display mode** (tabs above preview): applies to card, relic, and potion (image vs text toast).
- Drag ghosts to place slots; corner resize updates **type** scale for all ghosts of that type.
- **Gap:** preview canvas size is not auto-read from OBS: set to match your Browser Source and save.

## FIFO queue (same type, multiple slots)

Requires **2–4 slots of one type** in one band (or parallel slot playback) and that type **On**.

```text
scripts\test-overlay-queue.cmd -TwoCardSlots
```

With 2 card slots and 5 events, expect events to cycle through available card slots as each frees up.

```text
scripts\test-overlay-all-types.cmd -TwoSlotsPerType -FastDisplay
```

## Disabled type / no slots / disabled event

| Case | Expected |
|------|----------|
| Type **Off** on Cards/Relics/Potions tab | `Item type disabled`: events not enqueued |
| 0 slots for type | `No slots configured for type` |
| Event tab **On** unchecked | `Event disabled` |
| Global ignore list contains `itemId` | `Item ignored` |

## Slot limits

| Limit | Value |
|-------|--------|
| Per type | 4 |
| Total | 12 |
| Minimum per type | 0 |

## Known UX gaps

- Preview canvas size is persisted in config but not auto-detected from OBS.
- Per-event animation/sound overrides removed from UI: use globals, band enter/exit, or edit `config.json`.
- Sequential slot playback within a band uses first free slot in chip order for the next item of that type. Multiple slots of the same type can stack (second slot while the first is still showing); when the front slot clears, the overlay slides the next item into the first ghost position.
