# Dual PC setup (game PC + streaming PC)

> **Not officially tested.** Stream the Spire is designed and documented for a **single PC** (game, server, and OBS on one machine). This page describes a **likely** dual-PC layout based on how the mod and server work. Your network, firewall, and OBS version may differ - treat this as guidance, not a guaranteed recipe.

## Typical dual-PC layout

| PC | Role |
|----|------|
| **Gaming PC** | Slay the Spire 2 + Stream the Spire mod |
| **Streaming PC** | OBS, capture, rest of your stream |

The overlay is a **Browser Source** that talks to a small **web server**. The **mod** only sends HTTP POSTs to that server. For a two-PC setup, those two pieces must reach each other over your **LAN** (same home network).

---

## Recommended: run the server on the gaming PC

This matches how the mod and **mod icon cache** work today.

```text
Gaming PC                          Streaming PC
──────────                         ──────────────
STS2 + mod  ──POST──►  OverlayServer (0.0.0.0:5055)
       │                      ▲
       │                      │ HTTP + WebSocket
       │                      │
       └── mod cache ──► /mod-icons
                              │
                              └── OBS Browser Source
                                  http://GAMING_PC_IP:5055/overlay
```

**Why this layout:** The server serves **`/mod-icons/`** from the mod’s cache folder on disk (next to `Sts2StreamOverlay.dll`). That cache is written on the **gaming PC** when enchant events fire. Keeping the server there avoids copying cache files across the network.

### Steps (gaming PC hosts server)

1. **Install** Stream the Spire and the mod on the **gaming PC** (release zip - see [Getting started](getting-started.md)).
2. Note the gaming PC’s **LAN IP** (e.g. `192.168.1.10`) - `ipconfig` in PowerShell/CMD.
3. **Before starting the server**, allow it to listen on the network, not only localhost. In the Stream the Spire folder:

   ```powershell
   $env:ASPNETCORE_URLS = 'http://0.0.0.0:5055'
   .\run-server.cmd
   ```

   Default `run-server.cmd` binds to **127.0.0.1** only, which the streaming PC cannot reach. **`0.0.0.0:5055`** listens on all interfaces (including LAN).

4. **Mod URL on gaming PC** - leave default (no env var needed):

   ```text
   STS2_OVERLAY_URL = http://127.0.0.1:5055   (default)
   ```

   The mod and server are on the same machine; localhost POSTs are fine.

5. **Windows Firewall (gaming PC)** - allow **inbound TCP 5055** on **Private** networks only (not Public). Scope the rule to your streaming PC’s IP if you can.

6. **Image catalog** - paths on the config page point at folders on the **gaming PC** (where the server runs). See [Image catalog](image-catalog.md).

7. **Streaming PC - OBS**

   - Browser Source URL: **`http://192.168.1.10:5055/overlay`** (use your gaming PC’s IP).
   - Transparent background, size = your canvas (e.g. 1920×1080).

8. **Config page** - open from either PC in a normal browser:

   ```text
   http://192.168.1.10:5055/config
   ```

   Save layout there; OBS on the streaming PC receives updates over WebSocket to the same server.

9. **Quick test from streaming PC** - before OBS, open the overlay URL in Chrome/Edge on the streaming PC. You should see the page load (and WS status if using `?debug`). If that fails, fix firewall/IP before OBS.

---

## Alternative: server on the streaming PC

```text
Gaming PC                          Streaming PC
──────────                         ──────────────
STS2 + mod  ──POST LAN──►  OverlayServer
                                  ▲
                                  └── OBS → http://127.0.0.1:5055/overlay
```

Possible, but **more caveats**:

| Topic | Detail |
|-------|--------|
| **Mod URL** | Set **`STS2_OVERLAY_URL`** to the streaming PC **before launching STS2** on the gaming PC, e.g. `http://192.168.1.20:5055`. |
| **Firewall** | **Streaming PC** must allow **inbound TCP 5055** from the gaming PC. |
| **Mod icon cache** | Server looks for `mods\Sts2StreamOverlay\cache\icons` on the **streaming PC** by default. Enchant overlays may **miss icons** unless you set **`STS2_MOD_CACHE_ICONS`** on the streaming PC to a network path that points at the gaming PC’s cache (advanced; untested). |
| **Image catalog** | WebP folders must live on the **streaming PC** (where the server runs). |

For most streamers, **server on gaming PC** is the simpler dual-PC option.

---

## Alternative: no LAN overlay - NDI / capture

Run **everything on the gaming PC** (game + `run-server.cmd` + OBS Browser Source on `http://127.0.0.1:5055/overlay`). Send the composed video to the streaming PC via **NDI**, **capture card**, or similar.

- **Pros:** Default localhost setup, no firewall or cross-PC URLs, mod cache and catalog stay local.
- **Cons:** OBS (or a second OBS) runs on the gaming PC; you encode/stream from there or relay video.

Many dual-PC streamers already use this pattern for other overlays.

---

## Environment variables (dual PC summary)

| Variable | Where | Typical dual-PC value |
|----------|--------|------------------------|
| `ASPNETCORE_URLS` | PC running **server** | `http://0.0.0.0:5055` when the other PC must connect |
| `STS2_OVERLAY_URL` | **Gaming PC** before STS2 | Default `http://127.0.0.1:5055` if server is on gaming PC; streaming PC’s `http://IP:5055` if server is on streaming PC |
| `STS2_IMAGE_VERSIONS_ROOT` / catalog paths | PC running **server** | Set on config page and **Save** (local paths on that machine). Env vars are optional fallbacks for empty config. |
| `STS2_MOD_CACHE_ICONS` | PC running **server** | Only if server and mod cache are on different machines (advanced) |

Set env vars **before** starting the server or game.

---

## Troubleshooting (dual PC)

| Symptom | Things to check |
|---------|------------------|
| OBS overlay blank / WS disconnected | From **streaming PC**, open `http://GAMING_IP:5055/overlay?debug` in a browser. If it fails, firewall or wrong IP. |
| Game runs but no toasts | Mod POSTs go to `STS2_OVERLAY_URL`. Ping the server from gaming PC (`curl http://127.0.0.1:5055/api/config` or browser). Server must be running **before** the run. |
| Toasts on gaming PC browser, not OBS | OBS URL must use **gaming PC LAN IP**, not `127.0.0.1` (that means “this PC”, i.e. the streaming PC). |
| Missing enchant art | Server must read mod cache; prefer **server on gaming PC** or configure `STS2_MOD_CACHE_ICONS`. |
| Config saves but OBS unchanged | OBS and config must hit the **same server** (same IP/port). |

More help: [Mod installation → Troubleshooting](mod-installation.md#troubleshooting), [Troubleshooting](troubleshooting.md).

---

## Security note

Binding to `0.0.0.0:5055` exposes the overlay server to your **local network**. Use a **private** LAN, restrict firewall rules, and do not port-forward 5055 to the internet. Stream the Spire has **no authentication** - anyone who can reach the port can hit the API and overlay pages.

---

## Related

- [Getting started](getting-started.md) - single-PC setup (default)
- [Mod installation](mod-installation.md) - mod folder and `STS2_OVERLAY_URL`
