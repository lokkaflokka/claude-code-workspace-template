# CLAUDE.md

```yaml
_context:
  tier: 2  # Project-level
  version: 1.0
  last_updated: 2026-02-07
  inherits_from:
    - ../_shared/TECHNIQUES.md
  techniques_applied:
    - name: "Session Initialization Protocol"
      applied: 2026-02-01
    - name: "CLAUDE.md as Living Knowledge Base"
      applied: 2026-02-03
```

## Overview

This is a knowledge base for tracking a kitchen renovation project. It captures contractor research, quotes, decisions, and timeline. The goal is to have context persist across sessions so I don't re-research the same things.

---

## Session Initialization Protocol

**On every session in this project:**

1. **Read `CURRENT_STATE.md`** — Check for project-level alerts and deadlines
2. **Read `../_shared/CURRENT_STATE.md`** — Check for system-wide working memory
3. **Surface items from both levels:**
   - Project-level: quote expirations, pending decisions, contractor follow-ups
   - System-level: any cross-project items
4. **Show last session context** — What was worked on, where we left off
5. **Ask how to help** — Or suggest relevant actions based on deadlines

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — project context and rules |
| `CURRENT_STATE.md` | Live dashboard with deadlines and pending items |
| `quotes/` | Contractor quotes (PDFs and summaries) |
| `decisions/` | Decision log with rationale |
| `timeline.md` | Project timeline and milestones |

---

## Query Routing

| User Intent | Primary File | Secondary |
|-------------|--------------|-----------|
| "What needs attention?" | `CURRENT_STATE.md` | — |
| "Compare contractor quotes" | `quotes/comparison.md` | `quotes/*.md` |
| "Why did we decide X?" | `decisions/` | — |
| "What's the timeline?" | `timeline.md` | — |
| "Budget status" | `CURRENT_STATE.md` | `decisions/budget.md` |

---

## Project-Specific Rules

- All dates in YYYY-MM-DD format
- Quotes expire after 30 days unless noted otherwise
- Always note the source when adding contractor information
- Dollar amounts should include breakdown (labor vs materials) when available

---

## Common Mistakes

### Quote Handling
- **Don't assume quotes are current** — Always check the quote date before making comparisons. Quotes typically expire after 30 days. (discovered 2026-02-03: compared a stale quote against fresh ones, wasted a session)
- **Don't trust verbal estimates** — If it's not in writing, note it as "verbal, unconfirmed" and get written confirmation before deciding

### Scheduling
- **Don't suggest start dates without checking calendar** — Read CURRENT_STATE.md for any blocked dates before proposing timelines. (discovered 2026-02-05: suggested a start date during a planned trip)

### Recommendations
- **Don't recommend contractors without checking references** — At least 2 references should be contacted before final recommendation. Track reference status in the quote file.

*Add new mistakes above this line as they're discovered.*

---

## Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| — | — | — |

*No external tools yet. May add a budget tracking integration later.*

---

*ChangeLog:*
- 2026-02-07: Added scheduling mistake after calendar conflict
- 2026-02-03: Added quote handling mistakes, applied Living KB technique
- 2026-02-01: Initial creation, applied Session Init
