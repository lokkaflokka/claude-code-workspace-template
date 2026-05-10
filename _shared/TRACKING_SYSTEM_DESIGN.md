# Tracking System Design

How tracking works in this system: what goes in the task system vs the calendar vs vault files, how items are triaged and routed, and how planning produces a daily picture.

## Core Principle: One Surface Per Job

System sprawl kills trust. The fix is hard separation of concerns:

| Surface | Job | Key property |
|---------|-----|--------------|
| **Task system** (e.g., Apple Reminders) | Push pokes + capture inbox | Always available (mobile, voice) |
| **Calendar** (e.g., Google Calendar) | Visual day shape (time anchors, blocked time) | Persistent, phone-visible, push-enabled |
| **Vault files** | Context, backlog, someday items, meta-work | Read at session start |

**The rule:** Calendar = when. Tasks = what. Vault = how. Don't mix.

**Corollary:** If it doesn't have a date, it's not a reminder — it's vault content. Undated items live in vault backlog files, not in the task system.

## List Architecture

The task system holds three lists:

| List | Purpose | Session-init behavior |
|------|---------|----------------------|
| **Strategic** | Items needing assistant context: deadlines, decisions, complex tasks | Full surface: open items (7-day window) + unprocessed completions |
| **Personal** | Habits, errands, offline tasks | Completion scan only: surface `[x]` items with state implications |
| **Inbox** | Default capture list; voice/mobile capture staging | Process → triage → route → complete |

**Placement decision tree:**
```
1. Quick capture from phone/voice? → Inbox (default, automatic)
2. Can I complete this entirely without an assistant session? → Personal
3. Needs assistant help OR completion changes tracked state? → Strategic
```

**Inbox as default list:** Set the task system's default to "Inbox" so voice capture ("Hey Siri, remind me to X") lands automatically. Session init reads it, triages to Strategic/Personal/INBOX.md, then completes.

**Target counts:**
- Strategic: ~10 items at any time. If past 15, threshold is too low.
- Personal: Unconstrained — habits, errands, life maintenance.
- Inbox: 0 at end of each session. Triage everything.

## Task Hierarchy (Redesigned Homes)

| # | Type | Push surface | Planning surface | Context |
|---|------|-------------|-----------------|---------|
| 1 | Hard deadlines | Reminder (Strategic) | Zone placement in plan | Vault file |
| 2 | Recurring session work | Reminder (Strategic) | Zone placement in plan | Skill file |
| 3 | Project-scoped tasks | **None** | Zone placement OR Left-off | Vault TODO |
| 4 | Evaluative/exploratory | **None** | Spaced resurface in plan | `PROJECT_IDEAS.md` |
| 5 | Habits with details | Reminder (Habits) | Zone placement in plan | Vault log |
| 6 | Personal errands | Reminder (Personal) | Admin zone if time-sensitive | — |
| 7 | Waiting/blocked | Reminder `[waiting-on:]` | Not planned until unblocked | Vault file |
| 8 | Backlog/someday | **None** | Spaced resurface → quarterly review | Vault file |

**Rules:**
1. **No duplication.** One tracking home per item. Reminder + vault ref = OK. Reminder + calendar + left-off = NOT OK.
2. **Strategic admission gate:** External deadline + needs assistant → Strategic. Recurring session trigger → Strategic. Everything else → vault or Personal.
3. **Due dates = real deadlines.** No batch scheduling.
4. **Left-off = pointers** into vault files, not standalone items.

## Three Surfaces, Three Jobs

The framework that ties tracking to action:

| Surface | Job | Always available? |
|---------|-----|-------------------|
| Task system | Push pokes + capture inbox | Yes — mobile, watch, voice |
| Calendar | Visual day shape | Yes — phone-visible, push-enabled |
| Assistant (session) | Intelligence: plan, triage, route, log | On-demand or scheduled |

Two non-negotiable constraints:
1. No guaranteed session at day start — the plan must exist on a phone-visible surface before the laptop opens.
2. Intake happens everywhere (voice, mobile, email, meetings) and must flow in without dedicated triage sessions.

## Zone-Based Planning Model

External research: 2-3 hour zones outperform 30-minute blocks. Fewer micro-decisions, no time-estimation tax, flexible within zone. Missed zones are data, not visible failures.

| Zone | Typical placement | Contains |
|------|------------------|----------|
| 🧠 Deep Work | Morning (peak cognition) | Project work, session tasks, creative work |
| 💪 Movement | Morning or late afternoon | Workout, walks |
| 📋 Admin | Afternoon | Email, errands, scheduling, household |
| 🔄 Recovery | Post-deep-work or evening | Low-stakes browsing, content consumption |

**Design rules:**
- Double time estimates by default (time-blindness compensation) — user overrides down, not up
- 15m named "transition" buffers between zone types — unlabeled gaps get consumed
- No rigid 30m blocks — zones are stable, tasks within zones are flexible
- Missed zones become data, never guilt artifacts on calendar

## Day Plan Output Format

Two contexts use this format: session start (Today Plan) and session end (Plan Tomorrow). Same zone model, different constraints.

### Today Plan (session start) — max 8 lines

Generated fresh every session from live calendar + reminders. Capacity signal mandatory.

```
📅 Today: <Day, Date> — <capacity signal>

  🧠 Deep Work: <highest-priority project task>
  💪 Movement: <workout if due>
  📋 Admin: <errands, scheduling, quick tasks — batched/compressed>
  ⏸️ Blocked: <waiting-on items>
  ⚠️ <exceptions: stale blockers, inbox items, alerts>
  📆 Ahead:
   <Day>: <location/availability> → <tasks that fit>
   By <date>: <deadline-driven items>
  🔄 Last (<date>): <1-line focus>. Left off: <max 3>
Anything to capture?
```

**Capacity signal** (first line, mandatory): natural-language summary of remaining time. "mostly free", "packed — protect Deep Work", "2h free morning + evening", "wide open". Computed from calendar events. This single signal often gives more planning value than detailed zone placement.

**Hard cap: 8 lines.** When more items exist, compress within zones: "📋 Admin: Dell return + 2 more errands". Exceeding the cap triggers a wall-of-text bounce — making the feature worse than no feature.

**Ahead row** maps tasks to available time windows from the multi-day calendar view. Forward-looking pressure signals are second-highest-value addition after capacity.

**Without calendar (degraded):** Same shape minus capacity signal, anchored events, and Ahead. Zone placement and priority ordering still work from reminders alone.

### Plan Tomorrow (session end) — proposal format

```
📅 Tomorrow: <Day, Date>

  🧠 Deep Work: <task>
  💪 Movement: <if due>
  📋 Admin: <items>
  ⏰ Anchored: <calendar events>
  📆 Ahead: <multi-day deadlines>
  💡 Consider: <1-2 resurface items from backlog>
```

Persisted as a 2-line draft in `CURRENT_STATE.md`. Session start reads it as intent context but always regenerates from live data.

## Planning Lifecycle

| Trigger | What happens | Requires session? |
|---------|-------------|-------------------|
| **`/end plan`** (evening) | Plan Tomorrow: propose zone layout → persist 2-line draft | Yes (explicit request) |
| **`/end` auto-trigger** | Plan Tomorrow activates IF tomorrow has ≥2 calendar events OR ≥2 reminders due | Yes (session close) |
| **`/end` (simple day)** | Plan Tomorrow silently skipped — no question asked | Yes (no overhead) |
| **`/start` (morning)** | **Today Plan:** generate fresh from live calendar + reminders + habits. Use draft as intent context if exists. | Yes |
| **No session** | Reminders push-notify independently; calendar events visible on phone | No |
| **Mobile/voice** | Quick triage: "add X to tomorrow" → Inbox for next session start | Lightweight |

**Optimize for the no-draft case** — Plan Tomorrow has a high silent-skip rate by design, so session-start must produce a complete plan from scratch. The draft is bonus, not dependency.

**The /end trigger rule:** Plan Tomorrow is NOT a per-session question. It activates on complexity detection or explicit request. Silent skip otherwise. This respects the anti-ceremony principle.

**Fallback layers (if `/end` doesn't run):**
1. Reminders still push-notify (habits, deadlines) — independent of sessions
2. Calendar has existing events — baseline structure exists
3. Session start generates Today Plan from scratch (primary path, fully capable without `/end`)

## Simplified Triage (3 levels)

Replaces multi-level decision trees. Anti-ceremony: don't force routing decisions at capture time.

```
1. Urgent today?
   Yes → surface in /start exceptions, suggest zone placement
   No ↓

2. Has external deadline?
   Yes + needs assistant → Strategic reminder
   Yes + no assistant → Personal reminder
   No ↓

3. Everything else → Inbox for next /end to place
   (Project work → vault TODO, evaluative → PROJECT_IDEAS.md,
    quick errand → Personal reminder — decided during /end, not intake)
```

## Inbox Triage Workflow

When open items exist in the Inbox list, triage them in a single pass.

### Standard Triage Table

```
| # | Item | Route | Due | Ref | Rationale |
|---|------|-------|-----|-----|-----------|
| 1 | <title> | → Personal | Today | — | Offline task |
| 2 | <title> | → Strategic | <date> | <vault/path> | Needs session context |
| 3 | <title> | → Complete | — | — | Already configured |
```

Routes: `→ Personal`, `→ Strategic`, `→ INBOX.md`, `→ Complete`, `→ Discard`.

**Defaults:** Due date is required for Personal items — default today; user may override but blank is invalid (a Personal item without due date is invisible to date filters).

**Ref column:** Vault entity references (trip file, health event, financial decision) become `[ref:]` tags in the body.

### Question → Answer → Complete fast path

Inbox items phrased as questions ("Do we have X configured?") may be answerable from existing state. When the answer is available:
1. Check existing state (reminders lists, vault files, `CURRENT_STATE.md`)
2. Present the answer inline in the table's Rationale column
3. Route as `→ Complete` — no need to create a new reminder

### Post-triage verification (mandatory)

After all triage moves, list the Personal list and check for items with no due date. If found, surface immediately and set due date before proceeding. Catches both current-session and orphaned prior items.

## Completion Processing Algorithm

Called during session init for both Strategic and Personal lists. Authoritative spec.

### Step 1: Hard cutoff (mechanical — no judgment)

Read the `Completions processed through: [date/session]` line from `CURRENT_STATE.md` Last Session.

```
FOR each [x] item in the list:
  IF item.due_date <= cutoff_date:
    SKIP unconditionally. Do not surface, ask, or cross-reference.
```

This is a date comparison, not a judgment call. It eliminates all previously-processed completions in one pass. Items with no due date proceed to Step 2.

**Step 1 is a pre-filter, not the complete algorithm.** Items passing Step 1 are candidates, NOT confirmed new completions. They MUST go through Steps 2-3 before surfacing.

### Step 2: New-completion detection (mandatory)

For `[x]` items that passed Step 1:
```
Scan: are there any [x] items remaining after Step 1?
  NO  → Done. Skip JSON fetch entirely.
  YES → Read completed items with their completionDate timestamps.
        SKIP items where completionDate <= last_session_date.
```

### Step 3: Filter rules

Run in order. First match → action taken, stop checking.

| Check | Action |
|-------|--------|
| **List move:** open item with same title exists in OTHER list | SKIP silently (move, not completion) |
| **Consolidation artifact:** `CURRENT_STATE.md` describes phase-based reminders that replaced this individual item | SKIP silently |
| **Already reflected:** outcome already in `CURRENT_STATE.md` Active Contexts or Last Session | SKIP silently |
| **Personal: recurring habit/errand** | SKIP silently |
| **Personal: not state-tracked** (doesn't reference active-context topics) | SKIP silently |
| **None of the above** | SURFACE and ask for details |

### Step 4: Update cutoff

At session end, when writing `CURRENT_STATE.md` Last Session, set:
```
Completions processed through: [today's date] session [N]
```

### Why `completionDate`, not `due_date` for Step 2

An item due in March can be completed in February (e.g., moving it to another list). `completionDate` from the task system's API is ground truth for when the action happened.

## CURRENT_STATE.md: Session Companion Only

| Before (Working Memory) | After (Active Contexts) |
|-------------------------|-------------------------|
| 14 items mixing tracking with context | ~5 items with deep context |
| Expiration dates | No expiration dates |
| Manual maintenance every session | Update during session work, no separate ritual |

**What goes in Active Contexts:**
- Items needing the assistant's help to make progress on
- Rich context that would be expensive to reconstruct
- Active sub-projects with decision trees

**What does NOT go in Active Contexts:**
- Simple reminders (→ task system)
- Items that just need a due date nudge
- Anything that doesn't benefit from session context

**Reference line at top of section:**
> Deadline reminders: task system "Strategic" list. Personal tasks + habits: "Personal" list.

## Phase-Based Checklist Convention

When multiple related items span vaults and reminder lists, consolidate into a checklist file with phase-based reminders.

### Structure

- **Entity file:** A checklist in the relevant vault (e.g., `<vault>/<TOPIC>_CHECKLIST.md`) with items grouped into phases.
- **Phase reminders:** One Strategic reminder per phase, body points to the entity file with summary.
- **Individual reminders:** Folded (completed) when the phase reminder replaces them.

### Lifecycle

| Event | Action |
|-------|--------|
| **Item completed** | Check it off in the entity file during the session that completes it. No reminder mutation needed — phase reminder stays open. |
| **Phase fully complete** | Complete the phase reminder. |
| **Item added mid-phase** | Add to the entity file. Update the phase reminder body in the same session. |
| **Item moves between phases** | Move in the entity file. Update both phase reminder bodies. |
| **All phases complete** | Add `Status: Complete` and date to entity file header. File stays as reference. Remove from Active Contexts. |

### Phase assignment verification

When assigning items to phases, verify prerequisites — not just due dates. A reminder due Feb 15 doesn't belong in a "Pre-Day 1" phase if it requires a qualifying event that starts on Day 1.

For each item, document:
- **What:** the action
- **Prerequisite:** what must be true for this to be possible
- **Phase gate:** can this physically happen in this phase given the prerequisite?

Due date ≠ phase. Due dates come from when reminders were set; phases come from structural dependencies.

### Consolidation manifest

When folding individual reminders into a phase-based checklist, log the specific items folded in the entity file changelog:
```
*Created YYYY-MM-DD. Consolidates N items from X vaults + Y reminder lists.*
*Folded reminders: [item 1 title], [item 2 title], ..., [item N title].*
```

This makes future scans unambiguous — completed items matching folded titles are consolidation artifacts, not real task completions.

### Rules

- **Entity file is the source of truth for context** — phase rationale, decisions, item descriptions.
- **Reminder body is the source of truth for completion state** — for items completable offline, the user updates reminders from their phone in real-time. When answering "what's remaining," cross-reference reminder state against entity file. If they conflict, trust reminders for completion and the entity file for context.
- **Check off items in the file, not just in conversation.** A checked-off item in conversation but not in the file is a dropped ball.
- **Session-start refresh:** When a transition checklist is active, refresh entity file checkboxes from reminder state at session start to prevent drift accumulation.

## Body-Tag Conventions

The task system has no custom fields. Body tags are the convention for structured metadata.

### Tag grammar

```
[tag-name: value]
```

Tags are lightweight, grep-able, visible on phone. Standard tags below.

### `[recur:]` — completion-based recurrence

For items that recur relative to when they're completed (not on a fixed calendar schedule). Processed at session start.

**Mutually exclusive with native recurrence** — a reminder must not have both.

```
tag           := "[recur: " value "]"
value         := fixed | incrementing | stop
fixed         := INTEGER " days"              # e.g., [recur: 7 days]
incrementing  := "+" INTEGER " days"          # e.g., [recur: +1 day]
stop          := "stop"                       # e.g., [recur: stop]
```

**Valid examples:**
- `[recur: 7 days]` — fixed 7-day interval
- `[recur: +1 day]` — incrementing, interval grows by 1 each cycle
- `[recur: stop]` — permanently stops recurrence

**Most recurrences should use the task system's native recurrence.** `[recur:]` is for the narrow case where interval is keyed off completion, not a calendar schedule. Use sparingly.

### `[waiting-on:]` + `[waiting-since:]` — blocked items

For items blocked on external input.

```
[waiting-on: <description>]
[waiting-since: YYYY-MM-DD]
```

**Surfacing rules:**
- Items with `[waiting-on:]` surface by normal due-date rules but with `⏸️` prefix instead of 🔴/⏳/•.
- If `[waiting-since:]` >14 days ago, surface with `⚠️ STALE BLOCKER` regardless of due date. Prevents items from parking forever.
- Append: "Check: has [blocker] been resolved?"

**List placement:** Blocked items stay in their original list (Strategic/Personal). The tag is a status overlay, not a routing change.

**Resolution:** When user reports "blocker resolved":
1. Find the reminder with matching `[waiting-on:]` tag
2. Remove `[waiting-on:]` and `[waiting-since:]`
3. Update the reminder body to reflect the unblock
4. Surface as actionable

### `[ref:]` — cross-vault entity references

When a reminder relates to a vault entity, link them explicitly.

```
[ref: vault/path/FILE.md]
```

- Same `[tag: value]` pattern
- Multiple `[ref:]` allowed
- Grep-able: `grep -r "\[ref:" ~/Projects/` finds all linked reminders
- Visible on phone

**Lifecycle:**
- **Creation-time:** add when a reminder is created/triaged that relates to a vault entity. Catch at creation, not after the fact.
- **Stale refs:** session-end checks validate `[ref:]` targets still exist (catches renames).
- **Removal:** remove when the vault entity is archived or the reminder is completed.

### Body structure convention

To reduce accidental tag mutation when editing on phone:

```
User-visible notes go above the delimiter.
Keep instructions, context, personal notes here.
---
[recur: 7 days]
[waiting-on: ...]
[ref: vault/path/FILE.md]
```

User notes above `---`, system tags below. This is a convention, not enforced — parsers scan the entire body — but reduces accidental deletion when editing on phone.

## Active Contexts Bridge

When an active effort spans a vault entity AND the task system, the Active Contexts entry in `CURRENT_STATE.md` must include:
- Vault file path
- Associated reminder title(s) and list
- Key dates/deadlines

**Mechanical threshold:** 3+ open items across both surfaces (vault TODOs + reminders). Single-action reminders don't qualify.

**Sunset rule:** Remove the Active Context entry when all associated reminders are completed.

This makes the connection pass at session start a 1-hop lookup instead of multi-hop inference.

## Spaced-Repetition Resurface

Prevents backlog rot without requiring quarterly reviews. Items in `PROJECT_IDEAS.md` and vault backlogs get resurfaced during Plan Tomorrow.

**Algorithm:**
- 1-2 items per plan — a nudge, not a backlog dump
- Priority-weighted with random floor: 70% highest-priority, 30% random from full backlog
- Snooze penalty: each skip shortens the next interval (item comes back sooner)
- Archive escape valve: after 5 skips, prompt "Archive this? You keep skipping it."
- Relevance decay: items not interacted with for 90+ days auto-move to "Frozen" — not deleted, visually deprioritized; user can thaw

**State persistence:** `_shared/RESURFACE_STATE.md` — lightweight machine-maintained table.

```
| Item | Source | Last Surfaced | Skips | Next Eligible |
|------|--------|--------------|-------|---------------|
```

Updated by Plan Tomorrow only. Not human-edited.

## Habit Evolution

| Aspect | How it works |
|--------|-------------|
| **Poke** | Habits reminder list (push to phone) |
| **Schedule** | Plan Tomorrow places workout in Movement zone |
| **Track** | After completion, next session start asks "what did you do?" → vault log file |
| **Suppress** | Minor habits don't appear in Today Plan — push-notify independently. Surface only if overdue 2+ days |

## Rebalancing: Stability Over Reactivity

| Event | Response | Automation |
|-------|----------|-----------|
| New urgent item arrives mid-session | Surface as exception, suggest zone swap | Manual (user confirms) |
| Meeting added to calendar | Next session start detects conflict, surfaces exception | Semi-auto |
| Habit skipped | Don't carry forward — push-notified already | Auto |
| Task completed early | Note in `/end`, inform next plan | Passive |

**Design choice:** The day plan should be STABLE once set. Real-time rebalancing creates anxiety (constant plan changes = no plan). Only truly urgent items justify mid-day changes.

## Calendar Failure Contract

Calendar reads in session-start have a **hard 5-second timeout**. On failure: single exception line `⚠️ Calendar unreachable — planning without calendar`. Today Plan generates from reminders only — zone placement and priority ordering still work, but without capacity signal, anchored events, or Ahead. Calendar is additive, never blocking.

## Session Protocol Changes

### Session start
1. Read `CURRENT_STATE.md` for deep context
2. Surface Active Contexts (replaces surfacing working memory with expiry dates)
3. No expiry-date checking — task system handles that

### During work
When a new strategic commitment emerges:
1. Assistant suggests reminder text (title + due date + notes)
2. Add to Strategic list via the task system

### Session end
1. Update Active Contexts (~3-5 items) — only items with rich context
2. Update Last Session and Active Work Streams

### Weekly review

A periodic ritual that catches drift:

1. **Completions since last review** — what shipped this week.
2. **What's due this week** — group open Strategic by urgency.
3. **Personal non-recurring scan** — Personal items due this week, excluding recurring habits.
4. **Active Contexts health check** — for each entry: drop / compress / keep / add.
5. **Entity fragmentation scan** (highest leverage) — search for clusters of related items across lists and vaults that lack a parent entity. **Trigger:** 5+ related items across 2+ surfaces without a shared entity file → consolidate via the phase-based checklist convention.
6. **Gap detection** — cross-reference upcoming deadlines against reminders. Items in vault files with deadlines but no reminder. Reminders pointing to changed vault files. Action-reminder pairing gaps.
7. **Update state** — `CURRENT_STATE.md` Active Contexts, Last Session, Left Off, Active Work Streams.

## Tooling Note

This design is task-system-agnostic. Any system supporting lists + due dates + recurrence + body text works. Pick whatever has reliable read/write APIs and good push notifications.

The reference implementation in source uses:
- **Task system:** Apple Reminders (3 lists: Strategic, Personal, Inbox)
- **CLI:** A purpose-built EventKit binary for full CRUD via shell. Important properties for any tool you choose:
  - Single tool for reads AND writes (no asymmetric reliability)
  - List-scoped operations (avoids global-index ambiguity)
  - `--dry-run` on mutations
  - Verify-after-save (detect silent failures)
  - JSON output with `completionDate` field
  - Date-range filtering for efficient session-start scans

## What This Deliberately Does NOT Include

- **No new apps or subscriptions.** Use what's installed and free.
- **No automation buildout.** Manual-first. Automation is a future enhancement, not a prerequisite.
- **No complex project hierarchy.** Three lists. Complexity is the enemy of trust.
- **No separate completion tracking file.** State files are the source of truth for what's been processed.
- **No work list (yet).** Work tracking lives in whatever the employer uses (Trello, Notion, Jira). Add a Work list if a personal/work overlap demands it.

## Cross-References

| Document | Relationship |
|----------|--------------|
| `SIGNAL_CAPTURE_PATTERN.md` | Inbox list is the capture staging surface |
| `RESURFACE_STATE.md` | Spaced-repetition backlog tracking |
| `CURRENT_STATE.md` | Active Contexts pattern + Last Session |
| `.claude/commands/start.md` | Calendar gather + Today Plan generation |
| `.claude/commands/end.md` | Plan Tomorrow optional phase |
| `PROJECT_IDEAS.md` | Receives evaluative items |
