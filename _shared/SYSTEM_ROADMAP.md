# System Roadmap

Platform-level evolution roadmap for your orchestration system.

**Scope:** This file tracks platform capabilities and infrastructure. Individual packages maintain their own ROADMAPs for implementation details. Domain ideas live in `PROJECT_IDEAS.md` as backlog.

This is a template — fill in your own phases and items.

---

## Completed Phases (Summary)

Collapse historical phases to a one-line summary once they're done. Move details to `CHANGELOG.md`. Keep this section short — it's a marker that the work happened, not a record of what shipped.

### Phase Name (Date) — ✅
One-line description of what shipped. Details in `CHANGELOG.md`.

---

## Product Pass Arsenal

Track external tools/subscriptions powering the system. Useful for renewal decisions and tool-rationalization passes.

| Tool | Role | Status |
|------|------|--------|
| Tool A | Voice/dictation, automation, etc. | Active (expires YYYY-MM-DD) |
| Tool B | Research acceleration | Active (expires YYYY-MM-DD) |

---

## Current Phase: <Phase Name>

**Goal:** One-sentence statement of what this phase is for.

**Phase entered:** YYYY-MM-DD. **Target completion:** YYYY-MM-DD. **Transition criteria:** What conditions move us to the next phase?

| Item | Status | Remaining | Done When |
|------|--------|-----------|-----------|
| Item 1 | ✅ Done / ⏳ Pending / 🚧 In progress | What's left | Concrete completion criterion |
| Item 2 | ... | ... | ... |

**Phase assessment** (date): One-paragraph snapshot of where the phase stands. Update periodically.

---

## Future Phases (Exploration)

Speculative phases — capabilities or directions to explore but not yet committed to.

### Phase: <Name> (target: month/quarter)

**Context:** Why this phase exists. What signal made it worth tracking.

**Goal:** What you're trying to accomplish.

| Item | Status | Notes |
|------|--------|-------|
| Item 1 | ⏳ Pending | Brief notes on what this entails |
| Item 2 | ⏳ Pending | ... |

**Non-goal:** What you're explicitly NOT trying to do in this phase. Helps prevent scope creep.

---

## Tier-Based Sequencing (optional)

When multiple improvements are identified at once, sequence them by dependency and leverage. Tier 1 = ready to ship, Tier 2 = next, Tier 3 = exploration.

**Tier 1 — Active:**

| Item | Status | Evidence |
|------|--------|----------|
| ... | ✅ Done / 🚧 | What proves it's done |

**Tier 2 — Next:**

| Item | Target | Notes |
|------|--------|-------|

**Tier 3 — Exploration:**

| Item | Target | Effort | Notes |
|------|--------|--------|-------|

---

## Backlog (Opportunistic)

Items built when the itch emerges or dependencies resolve. Not actively scheduled.

| Item | Trigger | Notes |
|------|---------|-------|
| Item 1 | What unblocks it | Brief description |
| Item 2 | ... | ... |

---

## Cross-References

| Document | Relationship |
|----------|--------------|
| `JOBS_ASSESSMENT.md` | Strategic basis for current priorities (if maintained) |
| `CURRENT_STATE.md` → Active Work Streams | Status snapshot |
| `PROJECT_IDEAS.md` | Domain idea backlog |
| `ARCHITECTURE.md` | System overview |

---

## Changelog

Track significant roadmap revisions inline. Most-recent first.

- YYYY-MM-DD: **Phase transition** — what moved from active to complete, what new phase opened.
- YYYY-MM-DD: **Lean pass** — collapsed completed phases to summaries, removed done items from backlog, etc.
- YYYY-MM-DD: **Initial creation** — foundation phase defined.
