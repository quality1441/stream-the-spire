# Config & overlay manual tests

Use after config UI changes. Server: `scripts\run-server.cmd` (http://127.0.0.1:5055). Work in `OverlayServer/wwwroot/` (source); hard-refresh `/config` after static changes; restart the server after `Program.cs` / schema changes.

## Config page layout (smoke)

| Area | What to check |
|------|----------------|
| **Global settings** | Delay (0–10s), display duration, sound master, catalog root + GitHub link, game version, **Event defaults** (global ignore IDs, enter/exit anim + sound). No “overlay enabled” checkbox (overlay is always on when saved). |
| **Layout preview** | **Display mode** tabs (full art vs text) fused above canvas; canvas preset / custom W/H (four digits visible; **saved in config**); Snap 8px; Reset slot positions; **Add card/relic/potion** on the toolbar right. Scale dock beside preview (slider only). |
| **Slots** | Grid of slot tiles; add buttons only in preview toolbar. |
| **Item types** | Cards / Relics / Potions browser tabs with **On**; fused panel below with **Event rules**. |
| **Event rules** | Per-event browser tabs (Gained, Removed, …) each with **On**; one row for Delay + Show overrides (checkbox + seconds). No per-event ignore/anim/sound (those are global). |
| **Item IDs** | “Browse all item IDs” uses accent link styling (not default blue). |
| **Save / reload** | Status shows slot count; hard-refresh preserves slots, toggles, globals, and event rules. |
| **Validation** | Add slots past limits (4/type, 12 total) - notice appears and save is blocked until fixed. |

## Config round-trip (`slots[]`, `itemTypes`, globals)

Automated:

```text
scripts\verify-config-roundtrip.cmd
```

Manual:

1. Set global ignore ID, change global enter animation, tweak one event’s delay override.
2. **Save config** - status should show slot count.
3. Hard-refresh `/config` - values match.
4. Inspect active `data/config.json` (e.g. under `OverlayServer\bin\Release\net10.0\data\`): `slots`, `itemTypes`, `displayMode`, `previewCanvasPreset` / `previewCanvasWidth` / `previewCanvasHeight`, `globalIgnoreItemIds`, `globalEnterAnim` / `globalExitAnim` / `globalEnterSound` / `globalExitSound`, per-type `*ImageScale`, `*ImageMaxHeightVh` (always `100`), legacy `slot1`/`slot2` synced from first card/relic slots.

## Global event defaults

1. Add an item ID to **Ignore item IDs (all events)** → save.
2. Trigger that ID for `card_gained` and `relic_gained` (debug or in-game) - both should be ignored.
3. Change **Enter animation** globally → save → confirm multiple event types use the new animation on stream.

## Per-event rules

1. Cards → **Gained** tab → uncheck **On** → save → `card_gained` should not enqueue (`Event disabled` in server logs if applicable).
2. **Removed** tab → enable delay/show overrides with non-default values → save → only that event uses overrides; others use global delay/duration.

## Preview vs OBS overlay

- Set layout preview canvas to your OBS Browser Source resolution (e.g. **1920 × 1080**) and **save** - reload should restore the same canvas size.
- **Display mode** (tabs above preview): applies to card, relic, and potion (image vs text toast).
- Drag ghosts to place slots; corner resize updates **type** scale for all ghosts of that type.
- Stream size is scale + ~48% canvas width cap (no viewport height cap in UI).
- Card **text** mode: center presets use CSS transform on stream; preview uses `overlay-layout.js` math.
- **Gap:** preview canvas size is not auto-read from OBS - set to match your Browser Source and save.

## FIFO queue (same type, multiple slots)

Requires **2–4 slots of one type** and that type **On**.

```text
scripts\test-overlay-queue.cmd -TwoCardSlots
```

Watch server console: `Toast queued` with rising `queueDepth`, then `Toast shown` with `slotId=card-1`, `card-2`, … as slots free up.

With 2 card slots and 5 events, expect events to cycle through available card slots (assignment uses each slot’s `order` in config; set at slot creation).

```text
scripts\test-overlay-all-types.cmd -TwoSlotsPerType -FastDisplay
```

Optional: `-Interleaved` to mix types; each type’s queue is independent.

## Disabled type / no slots / disabled event

| Case | Expected |
|------|----------|
| Type **Off** on Cards/Relics/Potions tab | `Item type disabled` - events not enqueued |
| 0 slots for type | `No slots configured for type` |
| Event tab **On** unchecked | `Event disabled` |
| Global ignore list contains `itemId` | `Item ignored` |

## Slot limits

| Limit | Value |
|-------|--------|
| Per type | 4 |
| Total | 12 |
| Minimum per type | 0 (type may be unused) |

Config UI blocks add past limits; save validates; server `EnforceSlotLimits` trims on normalize if needed.

## Known UX gaps (post polish)

- Preview canvas size is persisted in config but not auto-detected from OBS.
- Per-event animation/sound overrides removed from UI - use global fields or edit `config.json`.
- `enabled` on `OverlayConfig` may still appear in JSON but is normalized to on; harmless if present.
- Native `<select>` dropdowns in OBS CEF may need future custom dropdown if option list contrast is poor.
