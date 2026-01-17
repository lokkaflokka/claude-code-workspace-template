# /new-technique

Document a new technique in TECHNIQUES.md and evaluate it across projects.

## Gather Information

Ask the user for:

1. **Technique name** (e.g., "Evidence-First Scoring")
2. **Type:**
   - `Project` — Applies to specific project structure/interaction (evaluate per-project)
   - `Workflow` — Applies globally to Claude Code usage (no per-project evaluation)
3. **Origin:**
   - Internal project name
   - External source (name + URL)
4. **Brief description** — What it is in 1-2 sentences
5. **Why it works** — Key mechanics that make it effective
6. **Example** — Concrete usage from the origin
7. **When to use / when NOT to use** — Applicability notes

## Document in TECHNIQUES.md

Add to `_shared/TECHNIQUES.md` before the `*Add new techniques above this line*` marker:

```markdown
---

### [Technique Name]

**Type:** Project | Workflow
**Origin:** [project or external source]
**Added:** [today's date]
**Source:** [URL if external]

#### What it is
[Brief description]

#### Why it works
[Key mechanics]

#### Example
[Concrete example from origin]

#### When to use
[Situations where this applies]

#### When NOT to use
[Situations where this is overkill or counterproductive]
```

## Evaluate (Project Techniques Only)

For **Project** techniques, immediately evaluate against each project:

1. **Read each project's CLAUDE.md** to understand its purpose and current patterns

2. **For each project, assess:**
   - Does this technique address a problem the project has?
   - Would applying it require significant rework?
   - Is it worth the complexity?

3. **Log in `_shared/EVALUATION_LOG.md`** before the `*Add new evaluations above this line*` marker:

```markdown
---

### [Date]: [Technique Name]

**Type:** Project
**Origin:** [where it came from]

| Project | Relevant? | Applied? | Notes |
|---------|-----------|----------|-------|
| project-1 | Yes/Maybe/No | —/Yes/No | [reasoning] |
| project-2 | Yes/Maybe/No | —/Yes/No | [reasoning] |
| ... | ... | ... | ... |

**Summary:** [Overall assessment and any action items]
```

## Log Setup Status (Workflow Techniques Only)

For **Workflow** techniques, log in `_shared/EVALUATION_LOG.md`:

```markdown
---

### [Date]: [Technique Name]

**Type:** Workflow
**Origin:** [where it came from]
**Status:** Installed / Not installed / Evaluating
**Notes:** [setup steps taken, observations, or blockers]
```

## Update CURRENT_STATE.md

After documenting and evaluating:

1. **Remove any "Pending Evaluations" alert** for this technique
2. **Update Data Freshness** if tracking freshness dates

## Execution Checklist

```
[ ] Gathered: name, type, origin, description, mechanics, example, applicability
[ ] Documented: Added to _shared/TECHNIQUES.md
[ ] Evaluated: (Project) Assessed against all projects, logged in EVALUATION_LOG.md
[ ] Logged: (Workflow) Setup status in EVALUATION_LOG.md
[ ] Cleared: Removed pending evaluation alert from CURRENT_STATE.md (if applicable)
```

## Output

After completion, display:

```
Documented technique: [name]

Type: [Project | Workflow]
Added to: _shared/TECHNIQUES.md

Evaluation:
  - [For Project: summary of relevance across projects]
  - [For Workflow: setup status]

Logged in: _shared/EVALUATION_LOG.md

Next steps:
  - [Any "Yes" projects that should have technique applied]
  - [Any setup steps remaining for Workflow techniques]
```

## Edge Cases

### Technique already documented, not evaluated
If a technique exists in TECHNIQUES.md but not in EVALUATION_LOG.md:
- Skip the documentation step
- Proceed directly to evaluation
- Clear any pending alert

### Applying a technique during evaluation
If during evaluation you determine a technique should be applied immediately:
- Ask the user if they want to apply now or defer
- If applying: make the changes, update `Applied?` to `Yes` in the log
- If deferring: log as `Yes | No | [reasoning for deferral]`

## Reference

- Template format: `_shared/TECHNIQUES.md` → "How to Use This File"
- Evaluation format: `_shared/EVALUATION_LOG.md` → "How to Use This File"
