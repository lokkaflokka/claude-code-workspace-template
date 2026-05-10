---
description: Session initialization
disable-model-invocation: false
---
# /start

Session initialization. Run at the beginning of every session.

**Invoke:** `/start` (full), `/start quick` (state + alerts only), `/start [vault]` (vault-specific), `/start [activity]` (session framing)

## Token Resolution

Tokens after the command resolve in this order:

1. **Mode tokens:** `quick`, `system`, `today` → set session mode (see Mode Selection)
2. **Vault aliases** (configure in your workspace): short codes that map to full vault directory names (e.g., `fin` → `finance`, `cf` → `content-feed`). Define your own aliases.
3. **Activity tokens:** frame the session by intent (run full init, then bias output)

   | Token | Effect |
   |-------|--------|
   | `triage` | Emphasize inbox, open items, triage table |
   | `plan` | Emphasize Ahead section, scheduling, capacity |
   | `capture` | Fast init, expand capture prompt |
   | `prep` | Pull relevant trip/event file + reminders. **File-first rule:** prep output goes into the trip/event file — never inline chat dump |
   | `digest` | Check staged insights, pipeline status |
   | `explore` | Minimal plan, maximize context for freeform work |

4. **`then` free-text:** `/start then implement X` — runs init, then proceeds to X

**Combinable:** `/start [vault] then [task]`, `/start quick triage` (vault + activity).

## Mode Selection

| Mode | Invocation | Runs | When |
|------|-----------|------|------|
| **Full** (default) | `/start` | Phase A → B → C | First session of day, returning after gap |
| **Quick** | `/start quick` | Phase A (gather quick) → B | Mid-day resume, known context |
| **Vault** | `/start [vault]` | Full + vault `CLAUDE.md` init | Entering a specific domain |
| **Today** | `/start today` | Phase A → Today Inventory | Full view of due/overdue, no compression |
| **System** | `/start system` | Full + Phase D | Strategic/system work session |

**Quick mode guard:** If `CURRENT_STATE.md` Last Session date is not today, prepend "⚡ Quick mode — no full init today" to output.

## Phase A — Gather (parallel reads + cache)

The goal is to load all needed context in one parallel batch, then write it to a stable location for Phase B to consume. Heavy gather logic belongs in a script, not inline in the skill — keeps the skill predictable.

**Reference architecture (adapt to your setup):**

1. **Bash gather script:** runs in parallel and writes JSON to a known cache path (e.g., `~/.cache/personal-vault/start-gather.json`). The script is responsible for: reading state files, listing reminders/tasks, scanning sync inbox, fetching staleness signals from any external sources, etc. Stdout is just `OK <path>`.
2. **Calendar fetch (MCP or API):** today's events + a 4-day window for the Ahead section. 5-second timeout. Calendar is additive, never blocking.
3. **Inbox/email scan (optional):** if you have an email triage system, scan for awareness signals.

**Then (sequential):**

4. **Read** the cache JSON written by step 1 — feeds Phase B.

**Why cache, not stdout:** Large gather output on stdout trips harness persisted-output thresholds and forces a follow-up Read. Routing through a known cache path eliminates the round-trip.

**Failure contracts:**
- Calendar: 5-second hard timeout. On failure, proceed without — calendar is additive.
- Email scan: errors or empty → skip the awareness line silently.
- Gather script: proceed with whatever data is available.

The cache JSON should expose keys for: state file content, changelog, inbox, reminders by list (open + recently completed), no-due-date items, sync packets, staged insights, staleness alerts, upcoming trips, and any heartbeats from autonomous jobs.

## Phase B — Process + Output (1 turn, zero additional tool calls)

Apply processing rules to Phase A data. Internal — not surfaced.

- **Anchor today's date** from your SessionStart hook. Never derive day-of-week mentally.
- **Completion algorithm:** see `TRACKING_SYSTEM_DESIGN.md` → Completion Processing Algorithm. Never surface a completed item as "new" without checking `completionDate` against last session date.
- **Strategic deadlines:** Surface open items due within 7 days. Markers: 🔴 ≤1d, ⏳ 2-3d, • 4-7d.
- **Personal:** Open items due today (excluding recurring habits/errands) + non-trivial completions.
- **No-due-date audit:** If any Personal items have no due date, surface with ⚠️.
- **Changelog landing check:** If process failures logged in previous session → verify resolution captured. Gap → "Previous session gap: [failure] not captured."
- **`INBOX.md`:** Surface only if unrouted items exist.
- **Sync inbox:** Surface count if pending packets exist. Staleness escalation if 3+ valid inbound sessions pending.
- **Staged insights:** If a synthesis output is awaiting routing, surface `⚠️ Digest ready — run /review` (or `/route`).
- **Staleness alerts:** If your monitoring surface flags drift, surface `⚠️ Learning staleness: [summary]`.
- **Heartbeats:** For each entry in `heartbeats[]` where `status` is anything other than `success`, surface `⚠️ Heartbeat: <name> <status> — <details>`. Group multiple entries from the same name into one row. Skip silently if `heartbeats[]` is empty.
- **Stale-failure suppression:** If a heartbeat's `age_hours` > 168 (7 days) AND status is `no-output` or `partial`, treat as stale; if > 336 (14 days) AND non-success, suppress entirely.
- **`/check` staleness:** Parse `Last /check:` from `CURRENT_STATE.md`. If >14 days ago, surface `⚠️ /check stale (last: [date], [N] days)`.
- **Position review check:** If `CURRENT_STATE.md` has `**Position Reviews:**` line with dates ≤ today, surface `⚠️ Position review due: P-001, P-003 — run /revisit`.
- **Alerts:** Surface `persistent` alerts only where action needed. Skip `session-scoped` (remove from `CURRENT_STATE`).
- **Completed items:** When surfaced, immediately reflect in state files. Don't defer to `/end`.

### Today Plan Generation

Primary `/start` output. Always generated fresh from live data.

**Inputs:** Calendar events (4-day window), due/overdue reminders, habits, Plan Tomorrow draft (if exists), left-off items.

**Capacity signal (mandatory, first line):** Compute remaining free time from now forward. Past events = context ("back from X"), not upcoming. Examples: "9:30pm, winding down" / "mostly free after 2pm" / "packed — protect Deep Work" / "wide open". Today's calendar events fold into the capacity header (constraints, not tasks): personal events with full detail (time + title), work blocks as time spans.

**Zone placement:** Tasks by type per `TRACKING_SYSTEM_DESIGN.md`:
- 🧠 **Deep Work** = project/session tasks
- 💪 **Movement** = workout/walks (from Habits)
- 📋 **Admin** = errands, email, scheduling
- ⏸️ **Blocked** = waiting-on items (not zone-placed)

**Ahead section:** Maps tasks to available time windows from the 4-day calendar view. Groups by day (or day range), shows location context, slots tasks where they fit. Deadline-driven items within 7 days included with "By <date>:" prefix. Forward-looking pressure signal.

**Calendar event processing:**
- Personal events with full details (time, title) — today's go in capacity header, future days feed Ahead.
- Work calendar (FreeBusy only): collapse adjacent/overlapping blocks into single spans. All-day "Office"/"Home" = location context, not time blocks.
- Events stand unless user verbally overrides.

**Calendar failure:** Single exception line `⚠️ Calendar unreachable — planning without calendar`. Capacity + Ahead degrade; zones still work.

**Degradation matrix:**

| Calendar | Reminders | Plan Tomorrow | Result |
|----------|-----------|---------------|--------|
| OK | OK | Exists | Full plan, draft-informed |
| OK | OK | Missing | Full plan from scratch (common case) |
| Fail | OK | Exists | Draft plan, no capacity/Ahead |
| Fail | OK | Missing | Zone placement, priority ordering only |
| OK | Fail | Any | Calendar-only: capacity + Ahead, note reminders unavailable |
| Fail | Fail | Any | Last session summary only |

### Special Processing

**Waiting-on items:** `[waiting-on:]` tag → ⏸️ prefix instead of 🔴/⏳/•. Append "Check: has [blocker] been resolved?" If `[waiting-since:]` >14 days ago, surface `⚠️ STALE BLOCKER` regardless of due date. When user reports resolution: find reminder, remove tags, update body.

**Learning activation:** Read a `**Learning Activation:**` cache block from `CURRENT_STATE.md` (populated by `/end`). Match today's work context against entries. Surface 0-2 items only when an entity has NEW information since last session. Output: 💡 row, e.g., `💡 P-001 (decision support) connects to today's work`. Counts toward 8-line cap (drops first when full). Empty/missing cache → skip silently.

**Awareness signal (gated):** If you have an email triage scan or similar inbound-signal source, gate the surface to keep it quiet ≥90% of days:

- Emit 📬 if ANY: rate ≥3/day, OR new sender, OR dormant sender returning
- Format: `📬 N awareness since <date> · <breakdown> · NEW: <sender> · dormant: <sender>`
- Single line. Drop empty segments. Counts toward 8-line cap. **First row to drop** at cap.
- Skip silently if the source is unavailable or returns empty.

### Connection Pass

After processing each category independently, scan for overlaps before generating output.

1. **`[ref:]` resolution:** If a surfaced reminder body contains `[ref: path]`, check whether the referenced file's key facts are available from Phase A data (Active Contexts). If not AND reminder is **overdue or due today**, read the target file. Future-dated items note the connection without fetching. Cap: 3 selective reads per init.

2. **Active Context matching:** If a surfaced reminder's subject matches an Active Contexts entry, group them. **Spec-path attachment:** when the matched entity has entries in your active-specs cache, append paths as `[spec: <path>]` so the spec is visible when you later ask — prevents producing plans from one-liners when a spec exists.

3. **Name/date clustering:** If multiple surfaced items reference the same person or overlapping date range (within 7 days), present as one connected group.

4. **Offline completion evidence:** For overdue/today items representing offline-completable correspondence/external actions, if email scanning is available: scan for evidence (sender name, subject terms). Present as "likely completed" with evidence rather than open task. Cap: 2 selective scans per init.

**Spec-before-action gate:** Before producing any plan, breakdown, sizing, or "what's involved" answer about a topic, check active-specs for an entity matching the topic. If found, **read the spec file(s) before answering** — even if the answer feels obvious. Inferring task shape from `CURRENT_STATE` one-liners when a spec exists is the failure mode this prevents.

### Optional: Specialized Processing

If your system maintains specialized state (trip files with attendees, plant-care reminders with fertilization tags, recurring social check-ins, etc.), add focused processors here. Each should:

- Use Phase A data, not extra tool calls
- Surface a single row in the alerts band when action is warranted
- Count toward the 8-line cap
- Skip silently when no action is needed

The reference implementation includes pre-trip check-in surfacing (intersect upcoming-trip attendees with open check-in reminders) and plant-care fertilizer prefix toggling. These are domain-specific and only worth implementing when you have the underlying data structures.

## Phase C — Interactive (only if inbox/no-due-date/email items exist)

### Inbox Triage

"Anything to capture since last session?" — triage to Strategic (needs assistant), Personal (offline), dismiss, or **process inline** (system improvement / technique / skill signal — evaluate during triage).

**`INBOX.md` is not a triage destination** — it's a `/capture` staging area only. Reminders Inbox items route to action lists, never park in `INBOX.md`.

Standard Triage Table per `TRACKING_SYSTEM_DESIGN.md` → Standard Triage Table.

**Due date is required for Personal items — no exceptions.** Default: today; user may override but blank is invalid.

**Post-triage verification (mandatory):** After all triage moves, list the Personal list and check for items with no due date. If found, surface immediately and set due date before proceeding. Catches both current-session and orphaned prior items.

### Capture Prompt
"Anything to capture since last session?" — process per triage rules.

## Output Format — Today Plan

Zone-based priority queue informed by calendar availability. **Hard cap: 8 lines** (excluding capture prompt). Compress when over: "📋 Admin: Dell return + 2 more errands". Never exceed.

### Structure

```
📅 Today: <Day, Date> <current time> — <capacity signal>

  🧠 Deep Work: <highest-priority project/session task>
  💪 Movement: <workout if due>
  📋 Admin: <errands, scheduling, quick tasks>
  ⏸️ Blocked: <waiting-on items>
  ⚠️ Alert: <exceptions>
  💡 Learning: <0-2 relevant items with NEW info>
  📬 Awareness: <gated>
  📆 Ahead:
   <Day>: <location/availability> → <tasks that fit>
   By <date>: <deadline-driven items>
  🔄 Last (<date>): <1-line focus>. Left off: <max 3 items>
Anything to capture?
```

### Row Rules

| Row | When | Notes |
|-----|------|-------|
| 📅 Header + capacity | Always | Capacity = remaining time from now forward. Past events are context. Today's calendar events fold in. |
| 🧠 Deep Work | Task exists today | Session work, project tasks, creative work |
| 💪 Movement | Workout/walk due today | From Habits |
| 📋 Admin | Errands/scheduling due/overdue | Batch — compress multiple |
| ⏸️ Blocked | Waiting-on items within 7 days | Not zone-placed |
| ⚠️ Alert | Stale blockers, inbox, sync packets, staged insights, no-due-date items, persistent alerts, position reviews, **operational heartbeats (non-success status)** | One line per exception type; multiple entries collapse into one row |
| 💡 Learning | Cache has items with NEW info | Second to drop at cap. Cache only — zero tool calls |
| 📬 Awareness | Gate: rate ≥3/day OR new sender OR dormant wake | **First to drop at cap.** Single line |
| 📆 Ahead | Tasks within next 3 days OR deadlines within 7 days | Maps tasks to windows from calendar |
| 🔄 Last session | Always | 1-line focus + max 3 left-off items |
| Capture prompt | Always | "Anything to capture?" |

**Rows with nothing to show are omitted.** No "all clear" lines.

### Anti-patterns
- Output exceeds 8-line cap (wall of text — bounce)
- Category-grouped flat list instead of zone-placed priority queue
- Calendar events as their own row instead of capacity context
- Listing future dates without mapping tasks to windows
- 📬 Awareness firing when gate is quiet — silence is correct
- "All clean" or no-news lines
- Algorithm trace visible in output
- Zone-placing blocked/waiting items (they go in ⏸️ Blocked)

### Completion + Alert Processing

- **Completed items:** Immediately reflect in state files. Algorithm trace internal — output states completion + significance only.
- **ALERTS:** Skip `session-scoped` (remove from `CURRENT_STATE`). Surface `persistent` as ⚠️ lines.

## Output Format — Today Inventory (`/start today`)

Full view of due/overdue. No 8-line cap. Same Phase A gather, same processing — different output shape.

### Structure

```
📅 Today: <Day, Date> <current time> — <capacity signal>

**Strategic** (due/overdue)
  🔴/⏳/• <item> — <due date>

**Personal** (due/overdue)
  <item> — <due date>

**Habits** (due/overdue)
  <item> — <due date>

**Inbox**
  <item> — <due date>

⚠️ <alerts>
🔄 Last (<date>): <focus>. Left off: <max 3>
Anything to capture?
```

### Rules
- Each list section appears only if it has items due/overdue.
- Strategic items use deadline markers (🔴/⏳/•). Personal/Habits plain.
- Waiting-on items show ⏸️ prefix inline (not a separate Blocked zone here).
- Capacity header still mandatory.
- Ahead section omitted — this mode is zoomed into today.
- No line cap — show every item.

Phase C (triage + capture) same as Full mode.

## Session Routing

After init:
- **Project-specific?** If request clearly belongs to one vault, ask which.
- **Meta-level patterns:** Technique evaluation → `_shared/CLAUDE.md`. Cross-project task → identify involved projects. New project setup → `_shared/PROJECTS.md`.

## Phase D — System Audit (only `/start system`)

Runs after Phase B (and C if applicable). Surfaces high-leverage system work.

**Phase D Gather (parallel reads):** `_shared/SYSTEM_ROADMAP.md`, `_shared/JOBS_ASSESSMENT.md` (if maintained), `_shared/ARCHITECTURE.md`, `_shared/SKILL_INDEX.md`, `_shared/PROJECT_IDEAS.md`.

**Process (zero tool calls):**

**Staleness check** — flag files past threshold:

| File | Source | Stale |
|------|--------|-------|
| `JOBS_ASSESSMENT.md` | As-of date | 30 days |
| `ARCHITECTURE.md` | Last meaningful update | 14 days |
| `SYSTEM_ROADMAP.md` | As-of or last changelog entry | 14 days |
| `SKILL_INDEX.md` | Last updated | 14 days |
| `PROJECT_IDEAS.md` | Last updated footer | 30 days |

**Roadmap phase:** What phase? Next milestone? Items blocked — what unblocks them? Phase completion criteria met → recommend transition.

**Tier check:** Active tier? Done vs. remaining? Tier transitions ready?

**Skill health:** Scan recent changelog "Skills invoked" lines. Skills unused 10+ sessions → retirement/redesign candidate. Synthesis thresholds hit (10-session mark)? Manual processes that could be skills?

**Strategic system reminders:** Filter Strategic for system/platform items (not vault-specific personal). Surface with effort estimates.

**Output format:**
```
## System Work Options

| # | Item | Type | Effort | Why Now |
|---|------|------|--------|---------|
| 1 | ... | Stale file / Roadmap / Tier N / Skill | 1-2 sessions | ... |
```

Follow with: "Which items? Or pick a direction."

## Vault-Aware Behavior

When run from a vault directory or with `/start [vault]`:
- Read vault `CLAUDE.md` for vault-specific init protocol
- Follow vault's session initialization steps
- Protocol is vault-agnostic — reads whatever local context exists
