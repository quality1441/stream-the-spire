# OBS setup

Stream the Spire shows toasts in OBS with a **Browser Source** pointed at the overlay page - not the config page. The overlay is transparent; only the card, relic, and potion toasts are visible on stream.

**Prerequisites:** Overlay server running (`run-server.cmd`) and layout saved on the [config page](configuration-and-testing.md). See [Getting started](getting-started.md) if you have not installed yet.

**Setup video:** [Watch on YouTube](setup-video.md): includes OBS browser source setup.

---

## Add a Browser Source

1. In OBS, add a source → **Browser**.
2. Name it something clear (e.g. **Stream the Spire overlay**).
3. Set the properties below before clicking OK.

---

## URL

Use the **overlay** page only:

```text
http://127.0.0.1:5055/overlay
```

| Use this | Not this |
|----------|----------|
| `/overlay` | `/config` - config UI is for editing settings in a normal browser, not for OBS |
| `/overlay` | `/overlay.html` - `/overlay` redirects correctly; either works, prefer `/overlay` |

**Dual PC:** If the server runs on your gaming PC, use that machine’s LAN IP instead of `127.0.0.1` - see [Dual PC setup](dual-pc-setup.md).

**Debug connection on stream (temporary):** append **`?debug`** to show WebSocket status on screen:

```text
http://127.0.0.1:5055/overlay?debug
```

You can also use **`?status`** (same effect). Remove `?debug` once everything works so viewers do not see the status box.

While debugging, the overlay may show a sample toast URL (`/api/debug/toast`) - that is for manual testing in a browser, not required for normal streaming.

---

## Width and height

Set **Width** and **Height** to your **stream canvas** size - the same resolution you stream at.

Example for 1080p:

| Setting | Value |
|---------|-------|
| Width | `1920` |
| Custom CSS | *(leave empty unless you know you need it)* |

Match this size to the **layout preview canvas** on the config page and **save config**. Slot positions are stored in that coordinate system and scaled to the browser source size.

Using the wrong size (e.g. 800×600 when you stream 1920×1080) will misplace toasts relative to your layout.

---

## Transparent background

The overlay page is designed for a **transparent** background so only toasts appear over your game capture.

OBS Browser Source settings vary by version:

| OBS version | What to check |
|-------------|----------------|
| **OBS 28+** (CEF) | **Control level** - default is usually fine. Transparency is handled by the page CSS (`background: transparent`). |
| **Older OBS** | Look for **“Shutdown source when not visible”** / custom CSS notes below. Some builds expose **“Use custom frame rate”** separately from transparency. |

**Custom CSS (if the background is not transparent):**

```css
body { background-color: rgba(0, 0, 0, 0); }
```

Paste into the Browser Source **Custom CSS** field only if you see an opaque box behind the toasts. The overlay already sets transparent backgrounds in its own styles; custom CSS is a fallback for picky CEF builds.

Do **not** set a solid background color in custom CSS unless you want a visible panel on stream.

---

## Refresh when the overlay looks stale

OBS embeds Chromium (CEF). It can **cache** the overlay page after you update Stream the Spire or change server files.

If toasts stop updating, layout looks wrong after a config change, or you upgraded the release:

1. Right-click the Browser Source → **Refresh** (or **Interact** → refresh, depending on OBS version).
2. Or toggle the source off and on.
3. As a last resort, remove and re-add the source with the same URL.

You usually do **not** need to refresh when changing settings on `/config` - the live overlay receives updates over WebSocket. Refresh helps when the **page itself** is outdated in OBS’s cache.

---

## Shutdown source when not visible (optional)

In Browser Source properties, **Shutdown source when not visible** can reduce CPU/GPU use when you switch to a scene that hides the overlay.

- **Enable** if you have many browser sources or the overlay scene is off most of the time.
- **Disable** if you notice delayed toasts or a slow reconnect when returning to the game scene (the source must reload when shown again).

Either choice is fine for most STS2 streams where the overlay stays on the main game scene.

---

## Common mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| **Config URL in OBS** (`/config`) | Full settings UI on stream, not toasts | Use **`/overlay`** |
| **Wrong host/port** | Blank source, WS disconnected in `?debug` | Server running? URL matches `STS2_OVERLAY_URL` / [Dual PC](dual-pc-setup.md)? |
| **Opaque background** | Black or gray rectangle over game | Transparent page + optional custom CSS above |
| **Zero or tiny width/height** | Nothing visible | Set to stream resolution (e.g. 1920×1080) |
| **Canvas mismatch** | Toasts in wrong corners vs preview | Set config preview canvas to OBS size, save, refresh source |
| **Server not running** | No toasts; debug shows disconnected | Start `run-server.cmd` before playing |
| **Forgot mod** | Server OK, no events in-game | [Mod installation](mod-installation.md) |

---

## Quick checklist

1. `run-server.cmd` running.
2. Browser Source → **`http://127.0.0.1:5055/overlay`**
3. Size **1920×1080** (or your canvas).
4. Transparent background (no solid panel).
5. Config preview canvas matches OBS size → **Save config**.
6. Test in a run (or `?debug` + browser test toast while setting up).

---

## Related

- [Getting started](getting-started.md) - install and first-run
- [Dual PC setup](dual-pc-setup.md) - LAN IP instead of localhost
- [Configuration and testing](configuration-and-testing.md) - slot layout and display mode
- [Troubleshooting](troubleshooting.md)
- [← Documentation index](README.md)
