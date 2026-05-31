# Reward playback order - implementation prompts

Copy-paste each prompt below into a fresh agent session. Run them **in order**.

**Replaces:** stack slot prompts (Prompts 4–7) in [`overlay-layout-improvements-prompts.md`](overlay-layout-improvements-prompts.md). Stack slots were implemented but are being removed in favor of typed slots + global playback timing.

**Reference plan:** `.cursor/plans/reward_playback_order_cbe64586.plan.md`

Repo: `sts2-stream-overlay`

---

## Prompt 0 - Context primer (optional)

```
You are working on the STS2 stream overlay in c:\dev\sts2-stream-overlay.

Problem we are solving:
- User has one card slot, one relic slot, one potion slot (typed slots at different screen positions).
- When 2 cards + 1 relic + 1 potion arrive at once, the overlay currently shows three toasts simultaneously (one per free typed slot).
- User wants an option to play rewards one-after-another in a configurable type order (e.g. relic → card → card → potion).
- Stack slots were tried and rejected - remove them entirely; do NOT extend stack behavior.

Key components:
- OverlayServer/Program.cs - config API, per-type displayQueues, TryDequeueToSlots, ShowInSlotAsync
- OverlayServer/wwwroot/js/overlay-layout.js - Sts2OverlayLayout slot position math
- OverlayServer/wwwroot/js/config-app.js - config UI, layout preview
- OverlayServer/wwwroot/config.html - config page markup
- OverlayServer/wwwroot/overlay.html - live OBS overlay (no stack-specific routing expected)

Conventions:
- Minimize scope; match existing code style
- Default rewardPlaybackMode = "parallel" so existing configs behave unchanged until user opts in
- Do not commit unless I ask
- dotnet build -c Release after server changes; hard-refresh /config after static UI changes
```

---

## Prompt 1 - Config schema: reward playback fields + remove stack from model

```
Task: Add global reward playback config fields and remove stack slot type from the server config model. No dequeue behavior changes yet - schema + normalization + migration only.

Background:
- Today: StackPlaybackOrder + slots with type "stack" exist from the stack slot feature.
- Target: rewardPlaybackMode + rewardTypeOrder; stack type and StackPlaybackOrder removed.

OverlayConfig (Program.cs):
1. ADD:
   public string RewardPlaybackMode { get; set; } = "parallel";
   // Valid: "parallel" | "sequential" (case-insensitive after normalize)

   public List<string> RewardTypeOrder { get; set; } = new() { "card", "relic", "potion" };
   // Used when RewardPlaybackMode == "sequential"

2. REMOVE (or stop writing after migration):
   public List<string> StackPlaybackOrder { get; set; }  // delete property after migration helper runs

OverlaySlotConfig.Type: only "card" | "relic" | "potion" - no "stack".

NormalizeConfig additions:
- NormalizeRewardPlaybackMode(cfg): default "parallel"; accept "sequential"; anything else → "parallel"
- NormalizeRewardTypeOrder(cfg): same logic as today's NormalizeStackPlaybackOrder - exactly 3 unique valid kinds (card/relic/potion), pad with defaults
- MigrateLegacyStackConfig(cfg):
  - If cfg.RewardTypeOrder is empty/null AND legacy StackPlaybackOrder has entries, copy StackPlaybackOrder → RewardTypeOrder once
  - Remove all slots where type equals "stack" (case-insensitive) from cfg.Slots
  - Do not persist StackPlaybackOrder in SaveConfig output (property removed or ignored)

EnforceSlotLimits:
- Remove stack branch (max 4 stack slots)
- Keep card/relic/potion max 4 each, max 12 total

RebuildRuntimeSlots:
- Remove stack slot inclusion block

HasSlotsForKind / enqueue eligibility:
- Revert to typed slots only: SlotCountForType(cfg, kind) > 0
- Remove HasStackSlots helper if only used for stack

Defaults in LoadOrCreateConfig:
- RewardPlaybackMode = "parallel"
- RewardTypeOrder = card, relic, potion
- No stack slots in default slots[]

Acceptance criteria:
- dotnet build succeeds
- GET /api/config returns rewardPlaybackMode and rewardTypeOrder
- Loading old config with stackPlaybackOrder migrates order and drops stack slots on normalize
- PUT with invalid rewardTypeOrder is corrected on normalize
- Existing configs without new fields load with parallel + default order

Do NOT change TryDequeueToSlots yet - Prompt 2.
Do NOT change config UI yet - Prompts 4–5.
```

---

## Prompt 2 - Sequential dequeue in TryDequeueToSlots

```
Task: Implement reward playback modes in TryDequeueToSlots. Remove all stack slot dequeue logic.

Prerequisite: Prompt 1 (rewardPlaybackMode, rewardTypeOrder on config).

Current TryDequeueToSlots (Program.cs ~321–400):
- Pass A: foreach kind card/relic/potion, fill all free typed slots from matching queue (parallel)
- Pass B: foreach free stack slot, walk StackPlaybackOrder, dequeue one item - DELETE THIS ENTIRELY

New behavior:

Read snapshot under configLock:
  rewardMode = config.RewardPlaybackMode (normalized)
  rewardOrder = config.RewardTypeOrder (normalized list of 3 kinds)

Under runtimeLock:

IF rewardMode == "parallel":
  Keep existing typed-slot loop only (lines ~330–356 pattern):
  - For each kind, free typed slots of that type dequeue from that kind's queue
  - Multiple kinds can assign in the same TryDequeueToSlots call

IF rewardMode == "sequential":
  - If ANY runtimeSlot has IsFree == false → return (no new assignments this pass)
  - Walk rewardOrder in order; for each kind:
      - If that kind's displayQueue is empty → continue
      - Find first free typed slot where slot.Type == kind (order by Order, then SlotId)
      - If no free slot for that kind → continue (skip; try next kind in order)
      - TryDequeue one job from that kind's queue
      - Mark slot busy; add to assignments; STOP (only one assignment per TryDequeueToSlots call)
  - If no assignment made, return

ShowInSlotAsync already sets IsFree = true after delay and calls TryDequeueToSlots() - sequential chain requires no new timer.

Logging:
- Log rewardPlaybackMode on toast queued/shown when useful for debugging
- Remove stack-specific log fields (stackSlots=, slotType=stack)

Acceptance criteria:
- Parallel mode: unchanged from pre-stack behavior for typed slots only
- Sequential mode with order [relic, card, potion] and queues (2 cards, 1 relic, 1 potion):
  1. Relic shows in relic slot
  2. After relic finishes, card 1 in card slot
  3. After card 1 finishes, card 2 in card slot
  4. After card 2 finishes, potion in potion slot
- Only one slot busy at a time in sequential mode
- dotnet build succeeds

Test manually:
  Set config rewardPlaybackMode=sequential via PUT or config UI (once built)
  Use /api/debug/toast or scripts/test-overlay-queue.ps1 to enqueue mixed types

Do not change config UI in this prompt.
```

---

## Prompt 3 - Remove stack slots from overlay-layout.js

```
Task: Remove all stack slot support from shared layout JS. Rename stack playback helpers to reward type order helpers (shared normalization for config UI).

File: OverlayServer/wwwroot/js/overlay-layout.js

REMOVE entirely:
- "stack" from SLOT_TYPES array
- isStackSlot, displayKindForStackSlot, resolveStackSlotTopLeft, stackPlacementOffsets
- Stack branches in resolveSlotPositionForSlot, inverseResolveSlotPositionForSlot, positionSlotElementForSlot
- supportsPreviewResize special case for stack (if any)
- Exports of removed functions

RENAME (keep behavior, new names):
- STACK_PLAYBACK_DEFAULTS → REWARD_TYPE_DEFAULTS (still ['card','relic','potion'])
- normalizeStackPlaybackOrder → normalizeRewardTypeOrder (same dedupe/pad logic)
- reorderStackPlaybackAtIndex → reorderRewardTypeOrderAtIndex (swap-on-duplicate-dropdown behavior)

Keep backward-compat aliases ONLY if config-app.js still references old names temporarily - prefer updating all call sites in this prompt or Prompt 4.

getRuntimeSlots / getSlotsForType / EnforceSlotLimits parity:
- Only card, relic, potion types
- Max 4 per type, 12 total (match server)

Acceptance criteria:
- No references to type === 'stack' in overlay-layout.js
- normalizeRewardTypeOrder exported on Sts2OverlayLayout
- Config page layout preview still works for card/relic/potion slots (drag, resize, snap)
- No runtime errors on /config load

Do not edit config-app.js stack UI removal yet if large - Prompt 4 handles config-app.js cleanup.
```

---

## Prompt 4 - Remove stack UI from config page

```
Task: Remove all stack slot UI and warnings from the config page. Clean up config-app.js stack code paths.

Prerequisites: Prompt 3 (overlay-layout.js no longer has stack type).

config.html:
- REMOVE: "Add stack" button (#btnAddStackSlot) from layout preview toolbar
- REMOVE: Global settings "Stack playback order" section (three dropdowns, lede text) - Prompt 5 replaces with reward playback UI
- REMOVE: #stackCoexistenceNotice element (stack + typed coexistence warning)

config-app.js - REMOVE:
- cachedStackPlaybackOrder, STACK_PLAYBACK_FIELD_IDS, wireStackPlaybackOrderFields
- readStackPlaybackOrderFromForm, applyStackPlaybackOrderToForm, stackPlaybackOrderHint, stackGhostMetaText
- hasStackAndTypedSlots, refreshStackCoexistenceNotice
- addSlot('stack'), btnAddStackSlot listener, stack case in defaultPresetForType / defaultSlotLabel / updateAddSlotButtons
- Stack branch in renderSlotPanels (order/offset fields for stack-only)
- Stack ghost handling in syncPreviewGhostDom / renderLayoutPreviewGhosts (isStack, layout-preview-ghost--stack meta)
- applyGhostStackOrder if only used for stack z-order
- stackPlaybackOrder in readConfigFromForm / saveConfig / loadConfig
- beginPreviewResize / applyStackPreviewResizeScale stack proportional scale logic - revert to per-type-only resize
- layoutPreviewHintText stackNote / hasStack missing-slot exceptions tied to stack

config-app.js - UPDATE:
- validateSlotsConfig: remove stack from byType counts
- Ghost tooltips: remove stack-specific resize text
- Empty slots hint: remove "Add ... / stack" wording

config-page.css - REMOVE:
- .layout-preview-ghost--stack
- .slot-panel-row .slot-type-badge[data-type='stack']
- .slot-coexistence-notice (if only used for stack warning)
- .stack-playback-* rules (Prompt 5 adds reward-playback-* replacements)

Acceptance criteria:
- Config page has no Add stack button, no stack coexistence warning
- Adding card/relic/potion slots still works
- Layout preview ghosts render for typed slots only
- Save/reload still works (reward fields wired in Prompt 5)
- dotnet build not required for this prompt; hard-refresh /config

Do not add reward playback Global settings UI yet - Prompt 5.
```

---

## Prompt 5 - Global settings UI: reward playback mode + type order

```
Task: Add config UI for rewardPlaybackMode and rewardTypeOrder in Global settings. Wire save/load.

Prerequisites: Prompts 1–2 (server fields), Prompt 4 (stack UI removed).

config.html - Global settings (replace old stack playback section):

1. Subsection: "Reward playback"
   - Short lede: explain parallel vs sequential; note sequential needs one slot per type you want to show
   - Select #rewardPlaybackMode:
       - value "parallel" → label "Parallel (types can show together)"
       - value "sequential" → label "Sequential (one reward at a time)"
   - Sub-block #rewardTypeOrderFields (class reward-playback-order, mirror old stack-playback layout):
       - 1st / 2nd / 3rd dropdowns: #rewardTypeOrder0, #rewardTypeOrder1, #rewardTypeOrder2
       - Options: card, relic, potion
   - Hint when parallel selected: "Type order is used when Sequential is selected."

config-app.js:
- readRewardPlaybackModeFromForm() → 'parallel' | 'sequential'
- readRewardTypeOrderFromForm() → Sts2OverlayLayout.normalizeRewardTypeOrder(...)
- applyRewardPlaybackModeToForm(cfg)
- applyRewardTypeOrderToForm(cfg)
- wireRewardPlaybackFields():
  - Mode select change → toggle disabled state on order dropdowns + markConfigDirty
  - Order dropdown change → reorderRewardTypeOrderAtIndex(cachedRewardTypeOrder, index, value) then apply - reuse swap logic from old stack UI
- saveConfig / loadConfig / readConfigFromForm: read/write rewardPlaybackMode, rewardTypeOrder
- cachedRewardTypeOrder variable for dropdown swap base state

config-page.css:
- .reward-playback-lede, .reward-playback-order, .reward-playback-order-row (copy from .stack-playback-* or rename)
- .reward-playback-order.is-disabled { opacity: 0.55; pointer-events: none; } when mode is parallel

layoutPreviewHintText:
- Mention sequential mode under Global settings when relevant (remove any remaining stack references)

Acceptance criteria:
- User can select Parallel or Sequential and set type order
- Order dropdowns swap correctly (no silent revert on duplicate pick)
- Save → reload preserves rewardPlaybackMode and rewardTypeOrder
- Order controls visually disabled (but values preserved) in parallel mode
- Sequential mode + order affects live dequeue (Prompt 2) after save

Do not update README yet - Prompt 6.
```

---

## Prompt 6 - README, config table, and test script notes

```
Task: Update documentation and test helpers for reward playback. Deprecate stack slot docs.

README.md:
- REMOVE section "Event-reward area (stack slot)"
- ADD section "Sequential reward playback" (or similar):
  - Problem: multiple reward types at once with one slot per type shows everything simultaneously
  - Solution: Global settings → Reward playback → Sequential + type order
  - Example: 2 cards + 1 relic + 1 potion with order relic → card → potion plays four toasts one at a time in the correct slots
  - Note: Parallel is default (legacy behavior)
- Update "Notable options on the config page" table:
  - Remove stack playback order / stack slot references
  - Add rewardPlaybackMode, rewardTypeOrder

docs/overlay-layout-improvements-prompts.md (optional small edit):
- Add note at top: Prompts 4–7 (stack) superseded by docs/reward-playback-prompts.md

scripts/test-overlay-queue.ps1 (optional enhancement):
- Add switch -SequentialRewards that sets rewardPlaybackMode=sequential on config before test, restores after (like -TwoCardSlots pattern)
- Or document manual PUT snippet in README

Acceptance criteria:
- README accurately describes parallel vs sequential
- No stack slot workflow in user-facing docs
- Test instructions let a developer verify sequential chain in ~2 minutes

Do not commit unless I ask.
```

---

## Prompt 7 - Integration pass + stack removal verification

```
Task: End-to-end verification that stack slots are fully removed and reward playback works.

Checklist:

1. Grep repo for dead stack references (fix or document intentional migration-only reads):
   - stackPlaybackOrder, StackPlaybackOrder, type === 'stack', btnAddStackSlot, layout-preview-ghost--stack

2. Config round-trip:
   - parallel + default order → save → reload
   - sequential + custom order (e.g. relic, card, potion) → save → reload

3. Migration:
   - Load config JSON with stack slots + stackPlaybackOrder → normalize → stacks gone, rewardTypeOrder populated

4. Live overlay:
   - One card/relic/potion slot each, parallel: 3 debug toasts → 3 visible together (when all slots free)
   - Same setup, sequential: 3 toasts → one at a time in configured order

5. Layout preview (from earlier prompts - regression):
   - Image/Text tabs, scale dock, snap 8px + snap to slots persist on save
   - Typed slot drag/resize still works

6. Build:
   cd OverlayServer && dotnet build -c Release

Commands:
  .\scripts\run-server.cmd
  .\scripts\test-overlay-queue.ps1
  # Manual: PUT /api/config with rewardPlaybackMode sequential, enqueue via debug API

Fix any gaps found. Update README config table if anything was missed in Prompt 6.

Do not commit unless I ask.
Report: summary of verified behavior, migration notes, and any known limitations (e.g. sequential skips a kind if no typed slot exists for that kind).
```

---

## Prompt dependency graph

```
Prompt 1 (schema + migration)
    └──► Prompt 2 (sequential dequeue)

Prompt 3 (overlay-layout.js) ──┐
Prompt 4 (remove stack UI)     ├──► Prompt 5 (reward playback UI)
                               │
Prompt 1 + 2 ──────────────────┘

Prompt 5 ──► Prompt 6 (README)
Prompt 6 ──► Prompt 7 (integration)
```

Prompts 1 and 3 can run in parallel after Prompt 0.
Prompt 2 needs Prompt 1.
Prompt 4 needs Prompt 3 (or run 3 then 4 sequentially).
Prompt 5 needs Prompts 1, 2, and 4.
Prompt 7 should run last.

---

## Single mega-prompt (one long session)

```
Implement reward playback order and remove stack slots entirely in sts2-stream-overlay.

Requirements:
1. Remove stack slot type from server, overlay-layout.js, config UI, CSS, README
2. Add rewardPlaybackMode ("parallel" | "sequential", default parallel) and rewardTypeOrder (3 unique kinds)
3. Migrate legacy stackPlaybackOrder → rewardTypeOrder; drop stack entries from slots[] on normalize
4. Parallel dequeue: existing typed-slot behavior (multiple types at once)
5. Sequential dequeue: only one toast globally at a time; walk rewardTypeOrder; dequeue one item from first non-empty kind with a free typed slot; chain via ShowInSlotAsync → TryDequeueToSlots
6. Config UI: Global settings - mode select + three order dropdowns with swap-on-change; disable order when parallel
7. Remove Add stack, stack coexistence warning, all stack ghost/panel code
8. Update README; run dotnet build; verify sequential chain with 2 cards + 1 relic + 1 potion

Follow docs/reward-playback-prompts.md Prompts 1–7 for acceptance criteria.
Do not commit unless I ask.
```

---

## Notes for the human

- **Stack work was not wasted:** placement math, playback order dropdowns, swap-on-duplicate UX, and config normalization patterns carry over directly.
- **Default stays parallel** so nobody’s live overlay changes until they opt into sequential.
- **Phased modes** (“cards at end”, “after relic”) are out of scope for v1; sequential + type order solves the stated combat-reward scenario.
