# /end

Session-end persistence. Run before closing any session.

## Why Phases Matter

Session-end work involves reading current state, deciding what changed, and writing updates. Without phase discipline, reads and writes interleave — causing stale reads, missed updates, and forgotten persistence. The Gather-Process-Emit pattern prevents this: batch all reads first, reason about what needs updating with zero tool calls, then batch all writes.

## Phase A — Gather (1 turn, all reads in parallel)

Read everything needed to determine what changed this session. **Make ALL reads in a single parallel batch:**

1. Read `_shared/CURRENT_STATE.md`
2. Read project-level `CURRENT_STATE.md` (if work touched a specific project)
3. Read `_shared/TECHNIQUES.md` (for pattern/technique check)
4. If package/code commits were made: check version tags and build status

**If any read fails, proceed with available data.**

## Phase B — Process (1 turn, zero tool calls)

Using Phase A data + session context, determine all updates needed. Do NOT make any tool calls during this phase — just reason.

Produce an internal checklist:

- **CURRENT_STATE.md updates:** Last Session (date, focus, done, left off), Active Work Streams changes, ALERTS resolved/added, Working Memory changes
- **Project state updates:** Any project-level CURRENT_STATE.md changes
- **Roadmap updates:** Any phase/item status changes (skip if none)
- **Common Mistakes:** Any new mistakes discovered this session (with session number + date)
- **Technique check:** Was a technique applied? Did a novel pattern emerge worth capturing?
- **Vault-first check:** Every reminder or tracked item referencing a vault file — does that file contain the content?
- **Action-reminder pairing:** Every "Left off" or "Pending" item — does it have a paired reminder?
- **Sync capture:** Anything generalizable that should cross to another machine?
- **Decay:** Completed roadmap phases to collapse? Resolved alerts to remove? Done backlog items to clear?
- **Package sync** (if applicable): Version bump, tag, build verification, reference updates

Present the update plan to the user for confirmation before writing.

## Phase C — Emit (1 turn, all writes)

Execute all updates identified in Phase B. **Batch writes where possible.**

1. Write `_shared/CURRENT_STATE.md` updates
2. Write project-level state updates
3. Write roadmap updates (if any)
4. Write new Common Mistakes entries (if any)
5. Write technique entries (if any)
6. Write sync content (if any)
7. Package operations (if any): version bump, tag, build

## Rules

- Phase A and Phase C are execution phases — tool calls allowed, minimal reasoning.
- Phase B is reasoning only — zero tool calls. All data was loaded in Phase A.
- The state file update (Phase C step 1) is never skippable.
- If time pressure is real, Phase B can flag items as "skip — no changes" but must explicitly decide, not silently omit.
- Common Mistakes entries include session number and date: `[S12, 2026-03-10: description of what happened]`
