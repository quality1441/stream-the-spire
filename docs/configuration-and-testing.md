# Configuration and testing

Stream the Spire settings live on the **config page**. The **overlay** page is for OBS only. Use **Test** and the optional test scripts to verify layout without launching STS2.

---

## Web pages

| URL | Use |
|-----|-----|
| **`/config`** | Layout, display mode, sounds, ignore lists, catalog paths - edit and **Save** here |
| **`/overlay`** | OBS Browser Source only - see [OBS setup](obs-setup.md) |
| **`/item-ids`** | Search item IDs for ignore lists - see [Ignore lists](ignore-items.md) |
| **`/help`** | In-app quick help (same server) |

Default base: **http://127.0.0.1:5055**

---

## Layout preview

On **`/config`**, the layout preview is a miniature of your stream canvas.

- **Canvas size** - Set to match your OBS Browser Source (e.g. **1920 × 1080**) and **Save config**.
- **Drag slot ghosts** - Position card, relic, and potion toasts. Positions save as pixel offsets on that canvas.
- **Add slots** - Toolbar: add up to **4 slots per type**, **12 total**.
- **Scale** - Drag a slot’s corner handle or use the scale dock; scale applies per **type** (all card ghosts share card scale).
- **Snap** - Optional 8px grid and alignment to other slots.

If preview and OBS disagree, canvas size or browser source dimensions are usually mismatched.

---

## Display mode: Image vs Text

Tabs **above** the layout preview set **display mode** for all item types:

| Mode | Behavior |
|------|----------|
| **Image** | Full art from your [image catalog](image-catalog.md) (cards use layered preview when available) |
| **Text** | Compact name / description toasts - no catalog required |

Saved as `displayMode` / `cardDisplayMode`: `card` (image) or `text`.

---

## Reward playback

Under the layout preview:

| Mode | Behavior |
|------|----------|
| **Sequential** (default) | Types play in order (1st → 2nd → 3rd dropdowns). Each type waits for the previous to finish; same-type slots fill together. |
| **Parallel** | Up to one toast per free slot per type; multiple types can show at once. |

Set **1st / 2nd / 3rd** to `card`, `relic`, or `potion` when using sequential mode.

---

## Sounds, animations, and event rules

- **Global settings** - Delay, display duration, sound master, enter/exit animation and sound for all events.
- **Cards / Relics / Potions tabs** - Turn a whole type off, or use **event tabs** (Gained, Removed, …) to disable specific events or override delay/duration.
- **Ignore item IDs** - Comma-separated global list - see [Ignore lists](ignore-items.md). Default includes `POTION_SHAPED_ROCK`.

Custom sounds can be uploaded from the config page (stored under `data/sounds/`).

---

## Save vs Test

| Action | What it does |
|--------|----------------|
| **Save config** | Writes settings to **`data/config.json`** next to the server (release zip: beside `OverlayServer.exe`). Overlay picks up changes over WebSocket. |
| **Test** (layout toolbar) | Sends **current form values** to the server, updates **in-memory** config, and fires one debug toast **per configured slot** (simulated reward screen). **Does not write** `config.json`. |

**Important:** After **Test**, the running server remembers those values until you **Save**, **Reload** from disk, or restart the server. Always **Save** when your layout is final.

**Test** requires OBS (or a browser tab) open on **`/overlay`** - status will note `No overlay connected` if nothing is listening on WebSocket.

---

## Config file location

| Install | Path |
|---------|------|
| **Release zip** | `{extract folder}\data\config.json` |

Restart the server after changing **catalog paths** in config. Most other changes apply live after **Save**.

---

## Related

- [OBS setup](obs-setup.md)
- [Ignore lists](ignore-items.md)
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
