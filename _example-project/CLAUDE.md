# CLAUDE.md

```yaml
_context:
  tier: 2  # Project-level
  version: 1.0
  last_updated: YYYY-MM-DD
  inherits_from:
    - ../_shared/TECHNIQUES.md
  techniques_applied: []
    # Example:
    # - name: "Session Initialization Protocol"
    #   applied: YYYY-MM-DD
```

## Overview

[Describe what this project is and its purpose. 2-3 sentences.]

**Example:**
> This is a personal finance knowledge base for tracking investments, making allocation decisions, and maintaining tax records. It serves as persistent memory for portfolio management across sessions.

---

## Session Initialization Protocol

**On every session in this project:**

1. **Read `CURRENT_STATE.md`** (if it exists) — Check for alerts and context
2. **Surface any time-sensitive items** — Deadlines, pending actions, stale data
3. **Show last session context** — What was worked on, where we left off
4. **Ask how to help** — Or suggest relevant actions

**Example session start:**
```
I've reviewed the project state:

- [ALERT if any]
- [PENDING ACTIONS if any]
- Last session: [what was worked on]

What would you like to focus on?
```

*If this project doesn't need state tracking, you can remove this section and CURRENT_STATE.md.*

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — project context and rules |
| `CURRENT_STATE.md` | Live dashboard with alerts (optional) |

*Add your project's key files here so Claude knows where to look.*

**Example:**
| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project context and rules |
| `CURRENT_STATE.md` | Portfolio alerts and pending actions |
| `decisions/` | Decision log entries |
| `snapshots/` | Point-in-time portfolio data |
| `playbooks/` | Procedures for common tasks |

---

## Query Routing

| User Intent | Primary File | Secondary |
|-------------|--------------|-----------|
| [Intent] | [File] | [File] |

*Map common questions to the files that answer them.*

**Example:**
| User Intent | Primary File | Secondary |
|-------------|--------------|-----------|
| "What needs attention?" | `CURRENT_STATE.md` | — |
| "Past decisions about X" | `decisions/` | — |
| "How do I do X?" | `playbooks/` | — |

---

## Project-Specific Rules

*Add any rules or constraints specific to this project.*

**Example:**
- All dates should be in YYYY-MM-DD format
- Never modify snapshot files in place; create new dated versions
- Dollar amounts should include cents (e.g., $1,234.56)

---

## Common Mistakes

*Capture errors Claude has made in this project so they don't recur.*

### [Category]
- **Don't [mistake]** — [Why and what to do instead]

**Example:**

### Data Handling
- **Don't assume data is current** — Always check the as-of date before making recommendations
- **Don't overwrite historical files** — Create new dated versions instead

### Recommendations
- **Don't suggest actions without checking constraints** — Read CURRENT_STATE.md for any blockers first

*Add new mistakes above this line as they're discovered.*

---

## Dependencies

*List any external tools, packages, or integrations this project uses.*

| Dependency | Version | Purpose |
|------------|---------|---------|
| — | — | — |

*Example:*
| Dependency | Version | Purpose |
|------------|---------|---------|
| mcp-finance-tools | 0.1.0 | Portfolio data fetching |

---

*ChangeLog:*
- YYYY-MM-DD: Initial creation from template
