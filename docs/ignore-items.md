# Ignore lists

Use ignore lists when certain items are too noisy, spoil content, or you never want them on stream.

---

## Global ignore IDs

On **`/config`** → **Global settings** → **Ignore item IDs (comma-separated, all events)**:

```text
POTION_SHAPED_ROCK, SOME_CURSE_ID, ANOTHER_ID
```

- Applies to **every event type** unless you turn off matching events separately.
- **`POTION_SHAPED_ROCK`** is ignored by default due to it appearing every combat once you have the Petrified Toad relic (felt was too noisy).
- IDs are **case-sensitive** - match exactly what the mod sends.

**Save config** after editing.

---

## Finding an item ID

### Item ID browser (`/item-ids`)

Open **http://127.0.0.1:5055/item-ids** with the server running.

- Search by **name**, **ID**, or **rarity**.
- Filter by kind (card / relic / potion) or character pool.
- IDs match mod event payloads (`itemId`).
- Upgraded cards use the **same base ID** - not listed separately.

The index comes from your [image catalog](image-catalog.md) at server startup. If the page is empty, check catalog paths and restart the server.

### In-game

If an item appears on stream, note the name and look it up on **`/item-ids`**, or check server console logs for `itemId=` when events enqueue.

---

## Per-type and per-event toggles

Global ignore is not the only filter:

| Control | Effect |
|---------|--------|
| **Cards / Relics / Potions → Off** | Blocks all events for that type |
| **Event tab → Off** (e.g. Cards → **Gained**) | Blocks only that event |
| **Global ignore list** | Blocks matching IDs across enabled events |

Examples:

- **Hide all potion pickups** - Potions tab → **Off**, or disable **Gained** under Potions.
- **Hide one curse** - Add its ID to global ignore; keep `card_gained` enabled.
- **Show relics but not cards** - Cards tab → **Off**; leave Relics **On**.

See [Configuration and testing](configuration-and-testing.md#sounds-animations-and-event-rules).

---

## Workflow example

1. Open **`/item-ids`**, search for `CLUMS` or the card name.
2. Copy the ID (e.g. `CLUMSY`).
3. Paste into **Ignore item IDs** on **`/config`**.
4. **Save config**.
5. Pick up that card in a run - no toast should appear.

---

## Related

- [Configuration and testing](configuration-and-testing.md)
- [Image catalog](image-catalog.md) - catalog required for a full `/item-ids` index
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
