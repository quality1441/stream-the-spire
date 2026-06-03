# Configuration and testing

Stream the Spire settings live on the **config page**. The **overlay** page is for OBS only. Use **Test** and **Export setup** to verify layout without launching STS2.

---

## Web pages

| URL | Use |
|-----|-----|
| **`/config`** | Layout, animation bands, display mode, sounds, ignore lists, catalog paths. Edit and **Save** here |
| **`/overlay`** | OBS Browser Source only. See [OBS setup](obs-setup.md) |
| **`/item-ids`** | Search item IDs for ignore lists. See [Ignore lists](ignore-items.md) |
| **`/help`** | In-app quick guide (same server) |

Default base: **http://127.0.0.1:5055**

---

## Terminology

These terms appear throughout the config page and this guide.

| Term | Meaning |
|------|---------|
| **Toast** | The animated notification on your stream when something happens in a run, usually picking up a **card**, **relic**, or **potion**. It enters with your chosen animation, stays for the configured **display duration**, then exits. **Image mode** shows art; **text mode** shows a compact name and description. |
| **Slot** | A fixed place on screen where a toast can appear. You drag **slot ghosts** on the layout preview to match your stream layout. Each slot is typed (card, relic, or potion). |
| **Parallel** | **At the same time.** In a given scope, multiple toasts may be on screen together. Example: two card slots with **slot playback** set to parallel → two cards can show at once when both are queued. |
| **Sequential** | **One after another.** In a given scope, the next toast waits until the current one has finished (slot frees up). Example: **slot playback** sequential with two card slots → only one card toast visible at a time. |
| **Band** | A group of slots that share playback rules. **Band 1**, **Band 2**, … play in list order when **band playback** is sequential. |
| **Queue** | If rewards arrive faster than slots can show them, extras wait in line (per item type: card, relic, potion). When a slot frees up, the next queued item plays. |
| **Sound group** | A burst of game events that arrive close together (within about 250 ms). Used by **First event and last event end** sound mode to decide when enter and exit sounds play. |
| **Sound mode** | Global setting: **On**, **Off**, or **First event and last event end** (default). Controls enter/exit sounds for each toast. See [Sounds](#sounds-animations-and-event-rules). |

**Parallel** and **sequential** apply at **two levels** on the config page:

1. **Band playback**: between bands (Band 1 vs Band 2).
2. **Slot playback**: inside each band (among that band’s slots).

The same words mean the same thing at both levels; only the **scope** changes. Details: [Animation bands](#animation-bands).

---

## Config page overview

The config page is one long form. Work top to bottom, then **Save config** in the bottom action bar.

| Section | What you set |
|---------|----------------|
| **Layout preview** | Canvas size, display mode (Image / Text), slot positions on stream, scale, snap, **Test** / **Reset slots** |
| **Animation bands** | Which slots play together, in what order, and optional per-band enter/exit animation |
| **Events** | Enable/disable whole types (Cards / Relics / Potions) and individual event rules |
| **Global settings** | **Timing and animation** (delay, duration, enter/exit animation), **Sound** (mode, enter/exit sound pickers), **Event defaults** (ignore IDs), catalog paths |
| **Bottom bar** | **Export setup…**, **Import setup…**, **Save config**, **Reload** |

Unsaved edits exist only in the browser until you **Save config**. **Reload** discards unsaved changes and reads `data/config.json` again.

---

## Layout preview

On **`/config`**, the layout preview is a miniature of your stream canvas.

### Canvas and toolbar

- **Canvas**: Preset (1920×1080, 2560×1440) or **Custom** W×H. Match your OBS Browser Source size and **Save config**.
- **Reset slots**: Moves all slot ghosts back to default corners (unsaved until Save).
- **Test**: Applies current form values to the running server **in memory** and fires one debug toast **per configured slot**. Does not write disk. See [Save vs Test](#save-vs-test).

### Slot ghosts

- **Drag** ghosts to position card, relic, and potion toasts. Positions are pixel offsets on the preview canvas.
- **Resize** a ghost’s corner handle (or use the scale dock): scale applies to the whole **type** (all card ghosts share card scale).
- **Snap 8px**: Grid snap while dragging.
- **Snap to slots**: Align edges/centers to other slots (6px threshold).
- Click a ghost (or a band chip) to select that slot for **Add selected slot** in a band.

If preview and OBS disagree, canvas size or browser source dimensions are usually mismatched.

### Display mode: Image vs Text

Tabs **above** the layout preview set **display mode** for all item types:

| Mode | Behavior |
|------|----------|
| **Image** | Full art from your [image catalog](image-catalog.md) (cards use layered preview when available) |
| **Text** | Compact name / description toasts: no catalog required |

Saved as `displayMode` / `cardDisplayMode`: `card` (image) or `text`.

---

## Slots

Each **slot** is one on-screen position for a toast. Slots are typed: **card**, **relic**, or **potion**.

| Limit | Value |
|-------|--------|
| Per type | 4 |
| Total | 12 |
| Minimum per type | 0 (you can omit a type you never show) |

### Adding and removing slots

In the **Animation bands** toolbar:

- **Add card** / **Add relic** / **Add potion**: Creates a new slot and assigns it to a band (usually the first band or the band you are editing).
- Remove a slot with **×** on its chip in a band drop zone.

Every slot must belong to **exactly one band**. On save, the server normalizes band membership (orphan slots are appended to the first band).

### Slot data saved in config

Each slot in `config.json` includes `id`, `type`, `label`, `order`, `preset`, `offsetX`, `offsetY`, and text-mode placement fields when used. Legacy `slot1` / `slot2` are kept in sync with the first card and relic slots for older tools.

---

## Animation bands

Animation bands control **when** toasts play and optional per-band enter/exit animation. If **parallel** and **sequential** are new to you, read [Terminology](#terminology) first.

Under the layout preview, **Animation bands** group slots and set timing at two levels:

### Two levels of timing

| Level | Control (UI) | Behavior |
|-------|----------------|----------|
| **Across bands** | **Band playback** (top of Animation bands) | **Sequential**: Band 1 must fully finish before Band 2 starts. **Parallel**: every incomplete band can assign toasts at once. |
| **Within a band** | **Slot playback** on each band card | **Parallel**: each free slot in the band can take the next queued item of its type. **Sequential**: only one slot in the band shows at a time; the next item waits until the current toast finishes. |

A band is **finished** when every slot in it is idle **and** there are no queued items left for those slots’ types.

### Band editor UI

Below the layout preview canvas:

1. **Band playback**: Sequential or parallel across bands.
2. **Add card / relic / potion / band**: Slot and band actions (right side of the band toolbar).
3. **Band cards**: One card per band, in play order (Band 1, Band 2, …).

Each band card includes:

| Control | Purpose |
|---------|---------|
| **Slot playback** | Sequential or parallel within this band |
| **Enter / Exit animation** | Override global animation for toasts assigned from this band (empty = use **Global settings**) |
| **Slot chips** | Drag to reorder within the band, drag between bands, or remove with **×** |
| **Add selected slot** | Assigns the currently selected ghost/chip to this band (accessibility / touch fallback) |
| **Remove band** | Deletes the band (at least one band must remain) |

Select a slot by clicking its chip or its preview ghost before using **Add selected slot**.

### Example setups

**Default-like (everything at once)**: One band, all slots, **Band playback** parallel, **Slot playback** parallel. Same feel as old “parallel reward playback.”

**Old sequential type order (relic → card → potion)**: Three bands in order, each with that type’s slots, **Band playback** sequential, **Slot playback** parallel within each band. Migrates automatically from legacy configs.

**Combat clarity**: Band 1: both card slots, **Slot playback** sequential (one card at a time). Band 2: relic + potion slots, **Slot playback** parallel (relic and potion together after cards finish). **Band playback** sequential.

**Same beat, different motion**: Band 1: card slot with enter `slide_in_left`. Band 2: relic slot with enter `fade_in`. **Band playback** parallel so both can show when rewards arrive together, but each uses its band animation.

### Mixed-type bands

A single band may contain card, relic, and potion slots. **Slot playback** parallel shows every free slot that has a queued item. **Slot playback** sequential shows one slot at a time following **chip order** in the band (not round-robin across two card slots: the first free slot in order gets the next item of its type).

### Migration from older configs

Configs that used **Sequential reward playback** with 1st / 2nd / 3rd type dropdowns are migrated on load to **one band per type** in that order. **Parallel** reward playback becomes **one band** with all slots. Legacy `rewardPlaybackMode` / `rewardTypeOrder` fields are no longer shown in the UI but may still appear in old JSON until rewritten.

### Known limitations (v1)

- A band with **zero slots** does nothing.
- Queues are still typed by item kind from the mod (card / relic / potion); bands choose **which slots** and **when**, not which events fire.
- No per-band sound overrides or stagger delay yet.
- **Test** uses current form bands but does not write `config.json`: **Save** when done.

---

## Export and import setup

Use **Export setup…** and **Import setup…** in the **bottom action bar** (not in the site footer). These share **layout-only** settings as a JSON file: useful for backups, second PCs, or stream templates.

### Included in `sts2-overlay-setup.json`

| Field | Contents |
|-------|----------|
| `slots` | All slot positions and labels |
| `animationBands` | Band playback mode, bands, slot IDs, per-band animations |
| `displayMode` | Image vs text |
| `previewCanvasPreset`, `previewCanvasWidth`, `previewCanvasHeight` | Layout preview canvas |
| `previewSnapGrid`, `previewSnapToSlots` | Snap toggles |
| Per-type image/text **scales** and image **start offsets** | Card, relic, potion sizing |

File header: `"version": 1`, `"kind": "sts2-overlay-setup"`, `"setups": null` (reserved for future named presets).

### Not included

Events, global delay/duration, **sound mode** and global sounds, global animations, ignore lists, catalog paths, game version, and item-type enable toggles stay on the config page you import **into**. After import, review **Global settings** and **Events**, then **Save config**.

### Typical workflow

1. Arrange layout and bands on machine A → **Export setup…**
2. On machine B → **Import setup…** → confirm dialog → form updates (unsaved)
3. Adjust catalog paths if needed → **Save config**
4. Hard-refresh `/config` to confirm bands and slots persisted

Invalid files (wrong `kind` or `version`) show an error in the status line.

---

## Sounds, animations, and event rules

### Global settings

Grouped on the config page under **Timing and animation**, **Sound**, and **Event defaults**.

**Timing and animation**

- **Global delay** (0 to 10 s) and **display duration** before exit animation
- **Enter animation** and **Exit animation** defaults for all events (bands can override animation at dequeue time)

**Sound**

Under **Global settings → Sound** on the config page:

- **Sound mode** (`soundMode` in `config.json`): when enter and exit sounds play
- **Enter sound** and **Exit sound**: global defaults (built-in whoosh/pop/beep or uploaded files)
- The overlay **never plays two sounds at once**; if several are scheduled together, they play one after another in order

| Sound mode | Enter | Exit |
|------------|-------|------|
| **On (every toast)** | Every toast | Every toast |
| **Off** | None | None |
| **First event and last event end** (default) | First toast in a **sound group** only | Last toast assigned from that group when nothing from the group is still waiting in queue |

#### How **First event and last event end** works

1. **Sound group**: Events that arrive within about **250 ms** of each other belong to one group (typical combat reward burst).
2. **Coalesce**: The server waits about **100 ms** after the last enqueue before assigning slots, so parallel reward picks usually assign in one pass.
3. **Enter**: Only the **first** toast shown from the group plays the enter sound.
4. **Exit**: Only the **last** toast **assigned** from the group plays the exit sound, and only when no items from that group are still waiting in queue. If slots assign one at a time (sequential slot playback), exit is deferred until the final item is assigned.
5. **Single pickup**: One card alone is a group of one → one enter and one exit on that toast.

Server logs include `enterSound=` and `exitSound=` on each `Toast shown` line. Middle items in a reward burst should show `enterSound=none` and `exitSound=none` when this mode is active.

#### Examples

| Situation | Sound mode | What you hear |
|-----------|------------|---------------|
| Parallel reward: 2 cards + 1 relic at once | First event and last event end | One enter (first assigned toast), one exit (last assigned toast when queues empty) |
| Single card pickup between fights | First event and last event end | One enter, one exit |
| Every item should whoosh/pop | On (every toast) | Enter and exit on each toast (queued, not overlapping) |
| Silent stream | Off | Nothing |

Custom sounds can be uploaded from the config page (stored under `data/sounds/`).

**Event defaults**

- **Ignore item IDs** (comma-separated, all events)
- **Catalog root** and **game version**. See [Image catalog](image-catalog.md). Restart server after path changes.

Per-event enter/exit animation and sound overrides are **not** in the UI; use globals or edit `config.json`.

See [Ignore lists](ignore-items.md) for global and per-event filtering. Default global ignore includes `POTION_SHAPED_ROCK`.

### Events section

- **Cards / Relics / Potions** tabs: Turn a whole type off with **On**.
- **Event tabs** (Gained, Removed, …): Disable specific events or override delay/duration for that event only.

---

## Save vs Test

| Action | What it does |
|--------|----------------|
| **Save config** | Writes settings to **`data/config.json`** next to the server (release zip: beside `OverlayServer.exe`). Overlay picks up changes over WebSocket. |
| **Export setup…** | Downloads layout JSON only: does not change server config until you import elsewhere and Save. |
| **Import setup…** | Loads layout JSON into the form: unsaved until **Save config**. |
| **Reload** | Discards unsaved form changes; reloads from disk. |
| **Test** (layout toolbar) | Sends **current form values** (including **sound mode**) to the server, updates **in-memory** config, and fires one debug toast **per configured slot** (simulated reward screen). **Does not write** `config.json`. |

**Important:** After **Test**, the running server remembers those values until you **Save**, **Reload** from disk, or restart the server. Always **Save** when your layout is final.

**Test** requires OBS (or a browser tab) open on **`/overlay`**. Status will note if no overlay is connected on WebSocket.

---

## Config file location

| Install | Path |
|---------|------|
| **Release zip** | `{extract folder}\data\config.json` |

Restart the server after changing **catalog paths** in config. Most other changes apply live after **Save**.

### `animationBands` in config.json

After save, config includes:

```json
"animationBands": {
  "bandPlaybackMode": "sequential",
  "bands": [
    {
      "id": "band-…",
      "order": 0,
      "label": "Band 1",
      "slotPlaybackMode": "parallel",
      "enterAnim": "",
      "exitAnim": "",
      "slotIds": ["card-1", "relic-1"]
    }
  ]
}
```

Empty `enterAnim` / `exitAnim` fall back to global enter/exit animation at playback time.

### `soundMode` in config.json

```json
"soundMode": "firstAndLast",
"globalEnterSound": "whoosh",
"globalExitSound": "pop"
```

Allowed values: `on`, `off`, `firstAndLast`. Default is `firstAndLast`. Legacy fields (`soundMasterEnabled`, `enterSoundPlaybackMode`, `exitSoundPlaybackMode`) are migrated on load if present.

---

## Related

- [Introduction: layout and playback](introduction.md#layout)
- [OBS setup](obs-setup.md)
- [Ignore lists](ignore-items.md)
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
