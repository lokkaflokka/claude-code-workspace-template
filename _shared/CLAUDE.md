# CLAUDE.md

```yaml
_context:
  tier: 1  # Root of inheritance hierarchy
  version: 1.0
  last_updated: YYYY-MM-DD
  inherits_from: []  # This is the root
  techniques_applied:
    - name: "Session Initialization Protocol"
      applied: YYYY-MM-DD
    - name: "CLAUDE.md as Living Knowledge Base"
      applied: YYYY-MM-DD
```

This is the cross-project techniques and patterns workspace.

## Purpose

This directory manages learnings, techniques, and patterns that span multiple projects. Use this space to:

- Document new techniques as you discover them
- Evaluate whether techniques apply to existing projects
- Track what's been evaluated and where it's been applied
- Maintain meta-level state (what needs attention across the system)

## Session Initialization Protocol

**On every session at `_shared/` level:**

1. **Read `CURRENT_STATE.md`** — The meta-level dashboard
2. **Surface ALERTS** — Pending evaluations, registry drift, stale data, working memory
3. **Check Working Memory** — Surface unexpired items; prompt to clear expired ones
4. **Show last session context** — What was worked on, where we left off
5. **Ask how to help** — Or suggest actions based on current state

**Example:**
```
I've reviewed the meta-level state:

- PENDING: "Query Routing Tables" technique needs evaluation
- REGISTRY: All projects in sync
- LAST SESSION: Documented 2 techniques, evaluated 1

What would you like to focus on?
```

## Workflow: Adding a New Technique

When you discover a useful pattern:

1. **Document it** in `TECHNIQUES.md`:
   - Name and type (Project or Workflow)
   - Origin (where you learned it)
   - What it is and why it works
   - When to use / when not to use

2. **Evaluate across projects** — For each project, ask:
   - Does this technique address a problem the project has?
   - Would applying it require significant rework?
   - Is it worth the complexity?

3. **Log the evaluation** in `EVALUATION_LOG.md`:
   - Which projects you evaluated
   - Relevance assessment (Yes/Maybe/No)
   - Whether you applied it and notes

4. **Apply where relevant** — Update project CLAUDE.md files

## Workflow: Cross-Project Evaluation

When asked to evaluate a technique:

1. Read the technique from `TECHNIQUES.md`
2. For each project in `PROJECTS.md`:
   - Read the project's `CLAUDE.md`
   - Assess fit based on purpose and current patterns
3. Update `EVALUATION_LOG.md` with findings
4. Apply to relevant projects (or note why not)

## File Index

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — workflows for cross-project work |
| `CURRENT_STATE.md` | **Meta-level dashboard with ALERTS** |
| `TECHNIQUES.md` | Catalog of patterns and techniques |
| `EVALUATION_LOG.md` | Tracking of technique evaluations |
| `PROJECTS.md` | Registry of all projects |

## Discovery Protocol (for larger projects)

When a project grows beyond ~20 files, add this constraint to reduce token waste:

> **Search first, explore as last resort.** Use search/grep tools to find what you need. Only list directories or read files speculatively if search returns nothing relevant.

This prevents the "list → read → filter → repeat" loop that burns tokens on large codebases.

---

## Core Principles (Enforced)

Consolidated behavioral rules with active enforcement mechanisms. These replace the "Common Mistakes" list below — mistakes without enforcement are aspirational.

**The meta-rule:** Documentation without enforcement is aspirational. Each principle below has a verification step.

*Customize these principles for your workflow. The structure (principle + enforcement + violations prevented) is the pattern; the specific rules are yours to define.*

---

### 1. Files Are The Work Product

**Principle:** Conversation is ephemeral. Knowledge must be in files to persist.

**Enforcement:** Before declaring anything "complete," answer:
- What artifacts should exist from this work?
- Do they exist in files right now?
- If no: persist before moving on.

**Session-end checklist** (verify before closing out):
- [ ] `CURRENT_STATE.md` — reflects what was done and where we left off
- [ ] Vault-level state files — if work touched a specific project
- [ ] **If MCP package commits were made:** version bump + tag + reference updates per Principle #5

---

### 2. Verify, Don't Assume

**Principle:** Verifiable claims must be verified. Reference explicit sources.

**Enforcement:**
- State claims (commits, file status): Run the actual command
- **Dates:** Anchor from the environment `Today's date` field. For long sessions or relative terms ("tomorrow", "next Monday"), run `date` to re-verify. NEVER derive day-of-week mentally.
- File contents: Read the file, don't assume from memory

---

### 3. Understand Before Acting

**Principle:** Read context before producing artifacts.

**Enforcement:** Before writing any artifact:
- **Run the Session Initialization Protocol if it hasn't been run yet** — non-negotiable, even if the user's first message is an action request
- Read the project's CLAUDE.md
- Check relevant CURRENT_STATE.md files

---

### 4. Apply Before Inventing

**Principle:** Check existing patterns before designing new ones.

**Enforcement:** Before proposing a new pattern:
- Search TECHNIQUES.md for relevant existing patterns
- Check existing projects for proven structures
- Only invent if nothing existing applies

---

### 5. Sync Is Immediate

**Principle:** Related updates happen in the same session.

**Enforcement:** After any project/package change, same-session checklist:
- [ ] PROJECTS.md registry updated?
- [ ] Version references match across files?
- [ ] **Version bump needed?** If feat: or fix: commits → bump package.json (feat = minor, fix = patch), update CHANGELOG, git tag.
- [ ] **Tag matches package.json?** `git tag v{version}` must exist at HEAD.
- [ ] Build step run if applicable?

---

### 6. Future Self Can Act Immediately

**Principle:** Time-delayed artifacts must be self-contained and actionable.

**Enforcement:** Every reminder, calendar event, or working memory item must include:
- What to do (specific action)
- How to do it (invocation method, not just name)
- Context needed (costs, deadlines, decision framework)

**Reminder chain rule:** When completing a reminder that has remaining follow-up actions, ALWAYS create a successor reminder before marking it done. A completed reminder with dangling TODOs is a dropped ball.

---

## Common Mistakes

*Legacy section — migrate recurring mistakes into Core Principles above with enforcement mechanisms.*

### Technique Management
- **Don't document a technique without evaluating it** — Every technique should have a corresponding EVALUATION_LOG entry
- **Don't evaluate without reading project CLAUDE.md files** — Surface-level assessment misses project-specific context
- **Don't apply techniques blindly** — "Relevant" doesn't mean "apply immediately"; consider timing and dependencies

### Registry & Sync
- **Don't create projects without updating PROJECTS.md** — Registry drift causes confusion
- **Don't forget propagation** — When techniques are applied, update both EVALUATION_LOG and project CLAUDE.md

### Session Continuity
- **Don't start sessions cold** — Always read CURRENT_STATE.md first
- **Don't end sessions without updating CURRENT_STATE.md** — Future sessions lose continuity

*Add new mistakes above this line as they're discovered.*
