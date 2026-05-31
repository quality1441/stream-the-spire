# Mod installation

The **Stream the Spire** game mod watches your run and sends card, relic, and potion events to the overlay server on your PC. It hooks STS2 when a run starts and **POSTs** JSON events to `{server}/api/events/raw` - for example when you pick up a relic, gain a card, or use a potion.

The mod does not draw anything on screen. OBS shows toasts from the separate **overlay server** (`run-server.cmd`).

---

## Where mods live

STS2 loads mods from a folder under your game install:

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

If Steam installed the game elsewhere, use that install’s `mods\` folder instead.

**Custom install path** - set **`STS2_GAME_ROOT`** to your game folder (the directory that contains `mods\`), then copy or deploy into:

```text
%STS2_GAME_ROOT%\mods\Sts2StreamOverlay\
```

Example (manual copy):

```powershell
$game = 'D:\Games\Slay the Spire 2'
Copy-Item -Recurse -Force 'C:\StreamTheSpire\mod\Sts2StreamOverlay' (Join-Path $game 'mods\Sts2StreamOverlay')
```

---

## Install from the release zip (recommended)

1. Download and extract **`stream-the-spire-win64-vX.Y.Z.zip`** - see [Getting started](getting-started.md).
2. Copy the entire **`mod/Sts2StreamOverlay/`** folder from the zip into **`{Slay the Spire 2}\mods\Sts2StreamOverlay\`** (overwrite if updating).
3. If STS2 was already running, **restart the game** so it picks up new mod files.

STS2 loads mods from that folder automatically - there is no separate enable step in the game.

---

## Server must be running while you play

The mod only **sends** events; it does not host the overlay.

1. Start **`run-server.cmd`** from your Stream the Spire folder **before** launching STS2 (or before starting a run).
2. Leave the server window open for the whole stream session.
3. Default URL: **http://127.0.0.1:5055**

If the server is stopped, the mod still runs but events are dropped (you will not see toasts in OBS).

---

## Overlay server URL (`STS2_OVERLAY_URL`)

When the mod loads, it reads **`STS2_OVERLAY_URL`**. If unset, it defaults to **`http://127.0.0.1:5055`**.

| Variable | Default | Purpose |
|----------|---------|---------|
| `STS2_OVERLAY_URL` | `http://127.0.0.1:5055` | Base URL of the overlay server (no trailing path) |

The mod trims trailing slashes and POSTs to:

```text
{STS2_OVERLAY_URL}/api/events/raw
```

Set the variable **before launching STS2** only if your server uses a different host or port (unusual for normal setups):

```powershell
$env:STS2_OVERLAY_URL = 'http://127.0.0.1:5055'
# then start Slay the Spire 2
```

On startup the mod logs the resolved endpoint - look for **`Sts2StreamOverlay: events endpoint …`** in the game log if you need to confirm the URL.

---

## Firewall and network

Stream the Spire is **localhost-only** by default:

- The mod POSTs to **127.0.0.1** (or whatever you set in `STS2_OVERLAY_URL`).
- The overlay server listens on your PC; OBS Browser Source loads **http://127.0.0.1:5055/overlay**.

You do **not** need inbound firewall rules for streaming on a single PC. No port forwarding or cloud exposure is required.

---

## Troubleshooting

### Mod installed but no toasts on stream

| Check | What to do |
|-------|------------|
| **Server not running** | Start `run-server.cmd`. Open **http://127.0.0.1:5055/config** in a browser to confirm it responds. |
| **Wrong URL** | Mod and server must use the same base URL. Default is `http://127.0.0.1:5055`. Set `STS2_OVERLAY_URL` before launching STS2 if you changed the port. |
| **Not in a run** | The mod hooks **run** events. Main menu / out of run → no card/relic/potion toasts. Start or continue a run and pick up an item. |
| **OBS source** | Browser Source URL must be **`http://127.0.0.1:5055/overlay`** (same host/port as the server). |
| **Config filters** | On `/config`, check ignore lists, disabled item types, or disabled event tabs - they can suppress toasts. |
| **Mod files missing** | Confirm `Sts2StreamOverlay.dll` and `Sts2StreamOverlay.json` exist under `mods\Sts2StreamOverlay\`. Restart STS2 after copying. |

### Updating the mod

Replace the contents of **`mods\Sts2StreamOverlay\`** with the newer **`mod/Sts2StreamOverlay/`** from a new release zip. Restart STS2.

---

## Related

- [Getting started](getting-started.md) - full first-run checklist including OBS and config
- [Dual PC setup](dual-pc-setup.md) - game PC + streaming PC *(untested)*
- [Image catalog](image-catalog.md) - optional full-art mode (server-side; mod works without it)
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
