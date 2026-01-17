# CLAUDE.md

This is the workspace root — a meta/organizational layer that spans all projects.

## Session Initialization Protocol

**When a session starts at this level, immediately:**

1. **Read `_shared/CURRENT_STATE.md`** — This is the meta-level dashboard

2. **Surface ALERTS** — Proactively tell the user about:
   - Pending technique evaluations (documented but not yet evaluated)
   - Registry drift (projects exist that aren't in PROJECTS.md, or vice versa)
   - Stale state (CURRENT_STATE hasn't been updated recently)
   - Working memory items (unexpired notes; prompt to clear expired ones)
   - Last session context (what was worked on, where we left off)

3. **Run consistency check if needed** — Verify `_shared/PROJECTS.md` matches reality

4. **Clarify intent if ambiguous** — If the request clearly belongs to one project, ask which project before diving in

5. **Recognize meta-level patterns:**
   - "New technique" → Document in `_shared/TECHNIQUES.md`, evaluate, log
   - "Cross-project work" → Identify which projects are involved
   - "New project setup" → Create from `_example-project/` template

**Example session start:**
```
I've reviewed the meta-level state. Here's what needs attention:

- TECHNIQUE: "Evidence-First Scoring" documented 5 days ago, not yet evaluated
- REGISTRY: All projects accounted for
- LAST SESSION: Worked on travel automation, left off at booking flow

What would you like to focus on today?
```

## Query Routing

| User Intent | Go To | Action |
|-------------|-------|--------|
| "What needs attention?" | `_shared/CURRENT_STATE.md` | Surface alerts |
| "New technique to document" | `_shared/TECHNIQUES.md` | Document → evaluate → log |
| "Evaluate technique X" | `_shared/EVALUATION_LOG.md` | Read technique, assess per project, log |
| "What projects exist?" | `_shared/PROJECTS.md` | List registry |
| "Work on [specific project]" | `[project]/CLAUDE.md` | Switch context to project |
| "Cross-project task" | Depends | Identify involved projects first |

## What NOT to Do at This Level

- **Don't dive into a specific project without asking** — If the user meant to work in `finance/`, they would have started there
- **Don't assume project-specific context** — Work at this level is meta/organizational
- **Don't skip the state check** — Always read `CURRENT_STATE.md` first

## Directory Reference

| Directory | Purpose |
|-----------|---------|
| `_shared/` | Cross-project techniques, evaluations, meta-level state |
| `_example-project/` | Template for new projects (copy and customize) |
| `.claude/commands/` | Reusable slash commands |

*Add your actual projects to this table as you create them.*

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — top-level session initialization |
| `_shared/CURRENT_STATE.md` | Meta-level dashboard with alerts |
| `_shared/TECHNIQUES.md` | Pattern catalog |
| `_shared/PROJECTS.md` | Project registry |
| `README.md` | Setup guide and philosophy |
