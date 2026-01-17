# /new-project

Create a new project or sub-project with proper structure and registration.

## Gather Information

Ask the user for:

1. **Project name** (kebab-case, e.g., `my-new-project`)
2. **Project type:**
   - `project` — Top-level project in the workspace
   - `sub-project` — Finite scope, lives under a parent project
3. **If sub-project:**
   - Which parent project?
   - Expected completion timeframe (e.g., 2026-Q1)
4. **Brief description** — One sentence purpose
5. **Is it time-sensitive?** — Should it surface at workspace-level session starts?

## Create Structure

### For Projects

Create `{workspace}/{name}/` with:

**CLAUDE.md:**
```yaml
_context:
  version: 1.0
  last_updated: {today}
  inherits_from:
    - ../_shared/TECHNIQUES.md
  techniques_applied:
    - name: "Session Initialization Protocol"
      applied: {today}
```

Then add:
- Session Initialization Protocol (read CURRENT_STATE.md + ../_shared/CURRENT_STATE.md)
- Project Overview section
- Query Routing table (can be minimal to start)
- Common Mistakes section (empty, ready for entries)

**CURRENT_STATE.md:**
- ALERTS section
- Pending Actions
- Last Session
- ChangeLog

### For Sub-projects

Create `{parent}/{name}/` with:

**CLAUDE.md:**
```yaml
_context:
  type: sub-project
  status: active
  parent: ../
  created: {today}
  expected_completion: {user-provided}
```

Then add:
- Session Protocol (read CURRENT_STATE.md + ../../_shared/CURRENT_STATE.md)
- Overview section
- Key Files table
- Phase Model (if applicable)

**CURRENT_STATE.md:**
- Phase indicator (if applicable)
- Status table
- Pending Actions
- Blocking Items
- Open Questions
- Last Session
- ChangeLog

## Register

### For Projects

1. **Add to `_shared/PROJECTS.md`** registry table:
   ```
   | {name} | {description} | {notes} |
   ```

2. **Add to workspace root `CLAUDE.md`** Directory Reference (if one exists)

### For Sub-projects

Sub-projects are NOT registered in PROJECTS.md — they live under parent projects.

**If time-sensitive:**
Add working memory pointer to `_shared/CURRENT_STATE.md`:
```
| **Active sub-project:** {parent}/{name} | {expiration} | {context} |
```

## Update Parent Project (Sub-projects Only)

Check if parent project's CLAUDE.md has sub-project scan in Session Initialization Protocol.

**Look for this pattern:**
```markdown
N. **Check sub-projects** — Scan subdirectories for CURRENT_STATE.md:
   - If sub-project has `_context.status: active`, read and surface alerts
   - If `dormant` or `completed`, skip
```

**If missing:** Offer to add the sub-project scan step to parent's session init.

## Execution Checklist

```
[ ] Gathered: name, type, parent (if sub-project), description, time-sensitivity
[ ] Created: {path}/CLAUDE.md
[ ] Created: {path}/CURRENT_STATE.md
[ ] Registered: PROJECTS.md (if project) OR working memory (if time-sensitive sub-project)
[ ] Updated: Parent project session init has sub-project scan (if sub-project)
[ ] Confirmed: Structure matches Sub-Project Pattern in TECHNIQUES.md (if sub-project)
```

## Output

After creation, display:

```
Created {type}: {path}

Files:
  - CLAUDE.md (session init, query routing)
  - CURRENT_STATE.md (alerts, pending actions)

Registered:
  - {where it was registered}

Next steps:
  - {any manual steps needed}
  - Start working in {path}
```

## Reference

See `_shared/TECHNIQUES.md` → "Sub-Project Pattern" for full pattern documentation (if using sub-projects).
