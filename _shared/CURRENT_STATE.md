# Meta-Level Current State

As-of date: YYYY-MM-DD
Patch: YYYY-MM-DD.1

---

## ALERTS (surface on session start)

### Pending Evaluations
| Priority | Technique | Documented | Status |
|----------|-----------|------------|--------|
| — | (none pending) | — | — |

*Add techniques that have been documented but not yet evaluated.*

### Registry Drift
| Priority | Issue | Details |
|----------|-------|---------|
| — | None detected | — |

*Flag when projects exist that aren't in PROJECTS.md, or vice versa.*

### Stale Data
| Item | Last Updated | Days Stale | Status |
|------|--------------|------------|--------|
| CURRENT_STATE.md | YYYY-MM-DD | 0 | Current |

*Track when key files haven't been updated recently.*

---

## Working Memory (expires)

Short-term notes that should be surfaced then cleared. Check expiration dates on session start.

| Note | Expires | Context |
|------|---------|---------|
| — | — | — |

*On session start: surface unexpired items, prompt to clear expired ones.*

---

## Last Session

**Date:** YYYY-MM-DD
**Focus:** [What you worked on]
**Outcome:** [What was accomplished]
**Left off:** [Where to pick up next time]

---

## Active Work Streams

| Stream | Status | Next Action |
|--------|--------|-------------|
| [Project or initiative] | [Status] | [Next step] |

*Track ongoing multi-session work here.*

---

## Technique Application Status

Tracks which techniques have been applied to the meta-level itself.

| Technique | Applicable? | Applied? | Notes |
|-----------|-------------|----------|-------|
| Session Initialization Protocol | Yes | Yes | This file |
| CLAUDE.md as Living Knowledge Base | Yes | Yes | Common Mistakes section |
| Query Routing Table | Yes | Yes | In CLAUDE.md |

*Update as you apply techniques to `_shared/` itself.*

---

## Update Protocol

**After each session at this level:**

1. Update **Last Session** with focus, outcome, and where you left off
2. Update **Active Work Streams** status
3. Clear resolved items from **ALERTS**
4. Add any newly discovered pending items

**Periodically:**

- Run `/consistency-check` to verify registry matches reality
- Review technique freshness (documented but not evaluated)

---

*ChangeLog:*
- YYYY-MM-DD.1: Initial creation from template
