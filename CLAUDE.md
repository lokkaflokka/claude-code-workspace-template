# CLAUDE.md

This is the workspace root — a meta/organizational layer that spans all projects.

## Session Initialization Protocol

**MANDATORY on every new session at this level — no exceptions, even if the user's first message is an explicit action request or plan.**

The user opening with "implement this plan" or "do X" does NOT skip initialization. Run init first, present the summary, THEN proceed to the user's request.

**Steps (run before any other work):**

1. **Read `_shared/CURRENT_STATE.md`** — Meta-level dashboard
2. **Check reminders/deadlines** — Whatever reminder system you use (Apple Reminders, Things, etc.):
   - **Anchor today's date first** from the environment `Today's date` field — all "due in X days" calculations MUST use this. Do not infer or assume the date.
   - **Completed reminders since last session:** If any reminders completed since the last session, surface them and ask the user for outcomes. These represent actions taken outside the session — state files may need updating.
3. **Run consistency check** — Registry drift, technique freshness, file sizes
4. **Clarify if intent seems project-specific** — If the request clearly belongs to one project, ask which before diving in
5. **Otherwise, recognize meta-level patterns:**
   - "New technique" → Document in `_shared/TECHNIQUES.md`, evaluate, log
   - "Cross-project work" → Identify which projects are involved
   - "New project setup" → Create from `_example-project/` template

**Output format rules:**
- **Compact lines, no tables** — keep the whole summary scannable
- **Reminders:** Show open items due within 7 days. 🔴 ≤1 day, ⏳ 2-3 days, bullet for 4-7 days.
- **Completed since last session:** Surface any recently completed reminders. Ask for outcomes — these are actions taken outside the session that may require state file updates.
- **Alerts: only surface issues.** If everything is clean, print `⚠️ ALERTS: None`. Don't enumerate "all clear" categories.
- **Last session:** One line — focus and what was left off.

**Example session start:**
```
📅 REMINDERS (next 7 days):
  🔴 Tomorrow: Submit expense report
  ⏳ Feb 4: Review quarterly goals
  • Feb 7: Team sync prep

⚠️ ALERTS: None
🔄 Last session (Jan 31): Worked on travel automation. Left off: booking flow.

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
