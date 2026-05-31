# Overlay layout improvements - implementation prompts

Copy-paste each prompt below into a fresh agent session. Run them **in order** unless noted. Repo: `sts2-stream-overlay`.

**Note:** Prompts 4–7 (stack slot) in this file are superseded by [`reward-playback-prompts.md`](reward-playback-prompts.md) - typed slots + global sequential/parallel playback.

Reference plan: `.cursor/plans/overlay_layout_improvements_a0db9328.plan.md` (or project plan file).

---

## Prompt 0 - Context primer (optional, paste before any task)

```
You are working on the STS2 stream overlay in c:\dev\sts2-stream-overlay.

Key components:
- OverlayServer/Program.cs - ASP.NET server, config API, per-type display queues, slot routing
- OverlayServer/wwwroot/js/overlay-layout.js - Sts2OverlayLayout: slot position math, scale, runtime slot order
- OverlayServer/wwwroot/js/config-app.js - config UI, layout preview drag/resize, scale dock
- OverlayServer/wwwroot/config.html - config page markup
- OverlayServer/wwwroot/overlay.html - live OBS overlay

Conventions:
- Minimize scope; match existing code style
- Do not change persisted JSON field names unless the task says so
- Do not commit unless I ask
- Test by building OverlayServer (dotnet build -c Release) and sanity-checking config + overlay pages
```

---

## Prompt 1 - Rename display mode tabs (Image / Text)

```
Task: Rename the layout preview display mode tabs from "Full art / PNG" / "Text box" to "Image" / "Text".

Scope: UI labels and docs only. Do NOT change persisted config values - they must remain displayMode/cardDisplayMode = "card" | "text".

Files to update:
1. OverlayServer/wwwroot/config.html (~lines 90–94)
   - Button visible text: "Image" and "Text"
   - Keep data-value="card" and data-value="text" unchanged
   - Update aria-label on #displayModeTabs to something like "Preview display mode: Image or Text"

2. README.md
   - Replace user-facing phrases "full art / PNG", "full art", "text box" with "Image" and "Text" where describing the config UI
   - Keep technical notes about PNG/WebP catalog paths where relevant

3. (Optional) scripts/test-overlay-queue.ps1 - align Get-CardDisplayModeLabel strings if present

Acceptance criteria:
- Config page tabs show Image / Text
- Saving config still writes displayMode: "card" or "text"
- Overlay still renders image vs text mode correctly after reload
- No changes to overlay-layout.js logic beyond any label strings if referenced

Do not implement any other features from the layout improvements plan.
```

---

## Prompt 2 - Numeric width control under scale slider

```
Task: Add a numeric width (px) input below the existing scale range slider in the layout preview scale dock.

Context:
- buildPreviewScaleDock() in OverlayServer/wwwroot/js/config-app.js currently has only a range input (10–200%)
- Scale is stored as {card,relic,potion}ImageScale multipliers (0.1–2.0) in config - do NOT add new persisted fields
- Size math already exists in overlay-layout.js: computeApproxImageSize, computeRawImageSize, inverseScaleFromSize, imageSettingsForKind, defaultTextToastBoxSize

UX target:
  Scale          [range slider]
  Width (px)     [number input]
  (existing muted label line: ≈ W × H px · N%)

Behavior:
- Image mode: user edits width in px; height derived from type base aspect ratio (card 400×711, relic/potion 128×128)
- Text mode: width drives text box size using existing defaultTextToastBoxSize × scale logic
- Bidirectional sync: moving slider updates width input; editing width updates slider
- px → scale: use inverseScaleFromSize (respect 48vw cap; existing label already notes cap)
- scale → px: use computeApproxImageSize / computeRawImageSize
- Clamp to equivalent of 0.1–2.0 scale (~40–800px wide for cards)
- Wire in flushPreviewScaleDockToState / syncPreviewScaleDockFromState / updateImageSizeLabels / wirePreviewScaleDock

CSS:
- OverlayServer/wwwroot/css/config-page.css - add a compact row for the number input under .preview-scale-panel
- Match existing config page input styling; respect .is-type-disabled disabling

Acceptance criteria:
- Selecting card/relic/potion tab updates width field for that type's scale
- 400px input ↔ 100% slider for default card
- Changing width preserves aspect ratio in the label
- Save/reload config preserves scale values
- Text mode preview ghost resizes when width changes

Do not implement stack slots or snap-to-peer in this task.
```

---

## Prompt 3 - Snap to other slots in layout preview

```
Task: Add "snap to other slots" alignment while dragging layout preview ghosts.

Context:
- Drag logic: OverlayServer/wwwroot/js/config-app.js - movePreviewDrag (~1305+), beginPreviewDrag, endPreviewDrag
- Position math: OverlayServer/wwwroot/js/overlay-layout.js - resolveSlotPositionForSlot, getRuntimeSlots
- Existing: #previewSnap8 checkbox snaps to 8×8 grid (or 1px when unchecked)

Add:
1. config.html - checkbox "Snap to slots" next to "Snap 8px" in layout preview toolbar

2. overlay-layout.js - new exported helpers on Sts2OverlayLayout:
   - collectSnapTargets(cfg, canvasW, canvasH, { excludeSlotId })
     Build targets from other slots via resolveSlotPositionForSlot. Include edges: left, right, centerX, top, bottom, centerY. Optionally canvas center horizontal/vertical.
   - snapBoxToTargets(left, top, width, height, targets, threshold)
     Returns { left, top, guides } where guides is optional line descriptors for UI

3. config-app.js - in movePreviewDrag, after clamp to canvas, before or merged with grid snap:
   - If #previewSnapToSlots checked, snap dragged box edges/center to nearest target within 6px canvas space
   - Prefer smallest delta when multiple snaps compete
   - Optional: render transient 1–2px guide lines on .layout-preview-canvas-outer during drag; clear on endPreviewDrag

Important:
- Use resolved left/top/width/height from resolveSlotPositionForSlot (handles text-mode translateX(-50%) anchors)
- Do NOT snap during resize (corner scale handle) in v1
- Snap preference is session UI only - do not persist to config unless trivial

Acceptance criteria:
- Dragging a ghost near another ghost's left edge aligns within 6px
- Center-to-center and edge-to-edge snapping works
- Snap 8px and Snap to slots can both be enabled
- endPreviewDrag still commits via inverseResolveSlotPositionForSlot correctly
- No server changes

Do not implement stack slots in this task.
```

---

## Prompt 4 - Stack slot config model (schema + JS normalization)

```
Task: Add the stack slot type and global StackPlaybackOrder to the overlay config schema and client normalization. Server routing comes in the next prompt - this prompt is model + validation only.

Add to OverlayConfig in OverlayServer/Program.cs:
  public List<string> StackPlaybackOrder { get; set; } = new() { "card", "relic", "potion" };

OverlaySlotConfig.Type may now be "stack" in addition to card/relic/potion.

NormalizeConfig in Program.cs:
- Normalize StackPlaybackOrder to exactly 3 unique valid kinds (card, relic, potion); drop unknowns; pad with defaults if needed
- EnforceSlotLimits: allow type "stack", max 4 stack slots, max 12 total slots overall

Client - overlay-layout.js:
- normalizeSlotEntry: accept type "stack"
- getRuntimeSlots: include stack slots (append after potion slots, ordered by order then id)
- getSlotsForType: return stack slots when type === 'stack'
- validate allowed types include 'stack'

Client - config-app.js:
- validateSlotsConfig: allow type stack, enforce max 4 stack, max 12 total
- defaultSlotLabel: handle "Stack N"
- countSlotsForType('stack')
- addSlot('stack') support (may stub UI button if not added yet)

Acceptance criteria:
- GET/PUT /api/config round-trips stackPlaybackOrder and slots with type "stack"
- Invalid stackPlaybackOrder is corrected on normalize
- Existing configs without stack fields still load with defaults
- dotnet build succeeds

Do NOT yet change TryDequeueToSlots or enqueue routing - that's Prompt 5.
Do NOT yet add config UI - that's Prompt 6.
```

---

## Prompt 5 - Stack slot server routing and dequeue

```
Task: Wire stack slots into event enqueue eligibility and dequeue/show routing in OverlayServer/Program.cs.

Prerequisite: Prompt 4 (stack slot schema) should be done.

Current behavior:
- displayQueues: separate ConcurrentQueue per card/relic/potion kind
- TryDequeueToSlots: typed slots dequeue only from matching type queue
- SlotCountForType(kind) gates whether events can enqueue
- ShowInSlotAsync(slotId, overlayIndex, slotType, job) - overlay uses item.kind from job for rendering

Implement:

1. HasSlotsForKind(kind) or extend SlotCountForType:
   - Return > 0 if typed slots exist for kind OR any stack slot exists
   - Use this for enqueue eligibility instead of typed-only check

2. TryDequeueToSlots - two passes:
   Pass A (unchanged): typed card/relic/potion slots dequeue from their type queues
   Pass B (new): for each free stack slot (order by order, then id):
     - Walk config.StackPlaybackOrder (normalized)
     - TryDequeue from first non-empty matching type queue
     - Mark slot busy; call ShowInSlotAsync with actual item kind from job.Item (not "stack")

3. RebuildRuntimeSlots: include stack slots in runtime slot list with Type = "stack"

4. Logging: include stack slot id and item kind in toast queued/shown logs

Coexistence rule:
- Typed slots keep priority (Pass A before Pass B)
- Stack slots consume remaining items from shared type queues

Acceptance criteria:
- Config with ONLY one stack slot: card_gained, relic_gained, potion_gained all enqueue and show sequentially in one slot
- With stackPlaybackOrder ["relic","card","potion"], when all three queues have one item, relic shows first
- With both card-1 typed slot and stack-1: first card goes to card-1, next card (if any) can go to stack per Pass B
- dotnet build succeeds

Do not add config UI in this prompt.
```

---

## Prompt 6 - Stack slot placement in overlay-layout.js

```
Task: Implement stack slot positioning so one stack slot uses a single screen position while item size/scale varies by kind.

Prerequisite: Prompt 4 (stack type in normalizeSlotEntry / getRuntimeSlots).

File: OverlayServer/wwwroot/js/overlay-layout.js

Problem:
- Typed slots use imageSettingsForKind(cfg, kind).startX/startY + slot.offsetX/offsetY
- Stack slot must NOT shift when showing relic vs card - same anchor point for all kinds

Implement in resolveSlotPositionForSlot (and inverseResolveSlotPositionForSlot if needed):
- When slot.type === 'stack':
  - Position from slot.left/top if set, else resolveSlotTopLeft(0, 0, slot.offsetX, slot.offsetY) - NOT type startX/Y
  - Size from imageSettingsForKind(cfg, kind) for the kind passed in (preview uses 'card' as default ghost kind)
- When slot.type === 'stack' in text mode: same single anchor; text box size from kind scale

Update positionSlotElementForSlot if it assumes slot.type === kind.

Preview ghost rendering (config-app.js renderLayoutPreviewGhosts):
- Stack ghosts: label "Stack", use card base dimensions for ghost size, meta hint like "card · relic · potion"

Live overlay (overlay.html):
- Verify show() → positionSlotForKind(slotIndex, entry.lastKind) works for stack slot entries - pass item kind, slot config type is stack
- May need positionSlotElementForSlot to resolve stack slot config with runtime kind

Acceptance criteria:
- Dragging stack ghost saves offsetX/offsetY relative to origin (0,0)
- Live overlay: card then relic in same stack slot appear at same top-left; only size changes
- Text mode stack slot behaves consistently

Do not add config HTML buttons in this prompt unless needed for manual testing.
```

---

## Prompt 7 - Stack slot config UI + global playback order + warning

```
Task: Add config page UI for stack slots and global stack playback order.

Prerequisites: Prompts 4, 5, 6.

config.html:
1. "Add stack" button in layout preview toolbar next to Add card/relic/potion
2. Global settings section - "Stack playback order":
   - When a stack slot has multiple item types waiting, show in this order
   - UI: three dropdowns or drag-reorder list (1st, 2nd, 3rd) with values card/relic/potion, no duplicates
   - Bind to stackPlaybackOrder in config JSON

config-app.js:
1. wireAddStackSlot button → addSlot('stack')
2. Stack slot panel in renderSlotPanels - same fields as typed slots (label, order, offsets); type read-only "stack"
3. readConfigFromForm / applyConfigToForm - read/write stackPlaybackOrder
4. Non-blocking warning (#slotValidationNotice or new element) when both stack slots AND any typed slot exist:
   "Stack slots share queues with typed slots. For a single event-reward area, use a stack slot instead of separate card/relic/potion slots."
5. layoutPreviewHint text update explaining stack slots briefly

config-page.css:
- Style playback order controls and stack ghost if needed (.layout-preview-ghost--stack)

README.md:
- Short section: event-reward workflow - add one stack slot, set playback order, remove overlapping typed slots

Acceptance criteria:
- User can add/remove stack slots from config UI
- Playback order saves and affects server dequeue (test with test-overlay scripts or debug toast API)
- Warning appears when stack + typed slots coexist
- Save/reload preserves all stack settings

This completes the stack slot feature.
```

---

## Prompt 8 - Final integration pass + testing

```
Task: Integration verification and fixes for overlay layout improvements (Image/Text rename, numeric width, snap-to-slots, stack slots).

Run through this checklist and fix any gaps:

1. Image/Text tabs - labels correct, config values unchanged
2. Numeric width - slider ↔ px sync for card/relic/potion, image and text modes
3. Snap to slots - aligns to peer ghosts; guides clear after drag; works with Snap 8px
4. Stack slots:
   - Add stack only → sequential card/relic/potion in one position
   - stackPlaybackOrder changes dequeue priority
   - Stack + typed slot coexistence warning shows
   - overlay.html positions correctly per item kind

Commands:
  cd OverlayServer && dotnet build -c Release
  .\scripts\run-server.cmd (if available)
  .\scripts\test-overlay-queue.cmd (if available)

Fix linter/build errors. Update README table of config options if any new fields were added (stackPlaybackOrder).

Do not commit unless I ask.
Report a concise summary of what was verified and any remaining limitations.
```

---

## Prompt dependency graph

```
Prompt 1 (rename) ──────────────────────────────┐
Prompt 2 (numeric width) ─────────────────────┤
Prompt 3 (snap) ──────────────────────────────┼──► Prompt 8 (integration)
Prompt 4 (stack model) ──► Prompt 5 (routing) │
         └──────────────► Prompt 6 (placement)│
                            Prompt 7 (UI) ────┘
```

Prompts 1–3 are independent and can run in parallel.
Prompts 4 → 5 and 4 → 6 can run in parallel after Prompt 4; Prompt 7 needs 5+6.
Prompt 8 should run last.

---

## Single mega-prompt (all features at once)

Use only if you want one long session instead of incremental prompts:

```
Implement the full overlay layout improvements plan for sts2-stream-overlay:

1. Rename display mode tabs to Image/Text (UI only; keep displayMode values card/text)
2. Add numeric width (px) input under scale slider, synced to ImageScale multipliers
3. Add Snap to slots in layout preview (6px threshold, optional guide lines)
4. Add stack slot type with global stackPlaybackOrder, server dequeue, shared placement, config UI, README

Follow the plan at docs/overlay-layout-improvements-prompts.md and .cursor/plans/overlay_layout_improvements_a0db9328.plan.md.

Implementation order: rename → numeric width → snap → stack (model → server → layout → UI).
Match existing code conventions. Minimize scope. Do not commit unless I ask.
Run dotnet build -c Release when done and summarize test results.
```
