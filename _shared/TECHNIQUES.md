# Techniques Catalog

A living catalog of patterns and techniques discovered across projects.

---

## Technique Types

| Type | Description | Evaluation Scope |
|------|-------------|------------------|
| **Project** | Applies to how a specific project is structured or how Claude interacts with it | Evaluate against each project |
| **Workflow** | Applies to how you use Claude Code generally | Install globally; no per-project evaluation needed |

---

## How to Use This File

### Adding a Technique

```markdown
## [Technique Name]

**Type:** Project | Workflow
**Origin:** [Where you learned it — project, person, article, etc.]
**Added:** YYYY-MM-DD

### What it is
[Brief description]

### Why it works
[Key mechanics — what makes this effective]

### Example
[Concrete example of the technique in action]

### When to use
[Situations where this technique applies]

### When NOT to use
[Situations where this technique is overkill or counterproductive]
```

### After Adding

- **Project techniques:** Evaluate relevance across all projects, log in `EVALUATION_LOG.md`
- **Workflow techniques:** Note installation/setup steps; no per-project evaluation needed

---

## Techniques

### Session Initialization Protocol

**Type:** Project
**Origin:** Common pattern, various sources
**Added:** YYYY-MM-DD

#### What it is
On every new session, Claude reads a "current state" file and proactively surfaces alerts, time-sensitive items, and context before asking how to help.

#### Why it works
- Eliminates "catch-up" conversation at session start
- Surfaces urgent items the user might forget to ask about
- Creates consistent session experience
- Claude starts with context, not cold

#### Example
```
I've reviewed your current state. Here's what needs attention:

- ACTION: Review pending PR comments
- ALERT: API key expires in 3 days
- STALE: Project data hasn't been updated in 2 weeks

Last session you were working on the authentication flow.

What would you like to focus on today?
```

#### When to use
- Projects with time-sensitive state
- Projects with recurring tasks or deadlines
- Any project where session continuity matters

#### When NOT to use
- Simple, stateless projects
- One-off tasks with no ongoing context

---

### CLAUDE.md as Living Knowledge Base

**Type:** Project
**Origin:** Boris Cherny / Anthropic
**Added:** YYYY-MM-DD

#### What it is
Using CLAUDE.md not as static documentation but as a continuously evolving knowledge base that captures mistakes, corrections, and patterns discovered during work.

#### Why it works
- Solves "AI amnesia" between sessions
- Mistakes become permanent lessons, not repeated errors
- Knowledge compounds over time
- The file becomes a record of "how to work well in this project"

#### How to implement
1. Add a "Common Mistakes" section to each project's CLAUDE.md
2. When Claude makes an error, add it to the section
3. Keep entries specific and actionable
4. Review periodically to see if patterns emerge

#### Example "Common Mistakes" section
```markdown
## Common Mistakes

- **Don't assume data freshness** — Always check the as-of date before making recommendations
- **Don't modify config files directly** — Use the config update script to maintain formatting
- **Watch for timezone issues** — All timestamps in this project are UTC, not local
```

#### When to use
- Any project where Claude repeatedly makes the same type of error
- Projects with non-obvious conventions or constraints
- Long-running projects where context accumulates

#### When NOT to use
- Simple projects with no recurring patterns
- Projects where Claude rarely makes mistakes

---

### Query Routing Table

**Type:** Project
**Origin:** Common pattern
**Added:** YYYY-MM-DD

#### What it is
A lookup table in CLAUDE.md that maps user intents to specific files or actions, helping Claude know where to look for different types of questions.

#### Why it works
- Reduces context-gathering overhead
- Ensures consistent answers to common questions
- Makes the project structure explicit
- Prevents Claude from searching randomly through files

#### Example
```markdown
## Query Routing

| User Intent | Primary File | Secondary |
|-------------|--------------|-----------|
| "What's the current status?" | `CURRENT_STATE.md` | — |
| "How do I deploy?" | `docs/DEPLOYMENT.md` | `scripts/deploy.sh` |
| "What are the API endpoints?" | `src/routes/index.ts` | `docs/API.md` |
| "Past decisions about X" | `DECISION_LOG.md` | — |
```

#### When to use
- Projects with many files where Claude might get lost
- Projects with common recurring questions
- Projects where certain files are authoritative for certain topics

#### When NOT to use
- Small projects with only a few files
- Projects where the structure is obvious

---

### Sub-Project Pattern

**Type:** Project
**Origin:** Common pattern for nested work
**Added:** YYYY-MM-DD

#### What it is
A pattern for managing finite-scope work that lives within a parent project but has its own state tracking, lifecycle, and alert propagation.

#### Why it works
- **Right-sized structure**: Not every piece of work needs a full project; some work is bounded and will complete
- **Alert visibility**: Time-sensitive sub-project items surface at parent level via automatic scan
- **Clean lifecycle**: Sub-projects transition through `active` → `dormant` → `completed`
- **Working memory integration**: Critical sub-projects can bubble up to workspace level via manual pointer

#### Sub-project Structure

```
parent-project/
├── CLAUDE.md
├── CURRENT_STATE.md
└── sub-project-name/
    ├── CLAUDE.md           # _context.type + status
    ├── CURRENT_STATE.md    # Phase, blocking items, pending actions
    └── [domain-specific files]
```

**Required in sub-project CLAUDE.md:**
```yaml
_context:
  type: sub-project
  status: active            # active | dormant | completed
  parent: ../
  created: YYYY-MM-DD
  expected_completion: YYYY-QN
```

#### Lifecycle

| Status | Meaning | Surfaced at parent? |
|--------|---------|---------------------|
| `active` | Work in progress | Yes (via scan) |
| `dormant` | Stable, occasional reference | No |
| `completed` | Done, archive candidate | No |

#### Alert Propagation

**Parent project session init should scan for active sub-projects:**
```markdown
N. **Check sub-projects** — Scan subdirectories for CURRENT_STATE.md:
   - If sub-project has `_context.status: active`, read and surface alerts
   - If `dormant` or `completed`, skip
```

**For workspace-level visibility**, add a working memory pointer to `_shared/CURRENT_STATE.md`:
```markdown
| **Active sub-project:** parent/sub-project | YYYY-MM-DD | Brief context |
```

#### When to use
- Time-boxed initiatives (transitions, migrations, projects with end dates)
- Work clearly scoped to one domain (lives under relevant parent)
- Situations where you want tracking but not a full project

#### When NOT to use
- Ongoing domains with indefinite lifespan → use a project
- Cross-cutting work spanning multiple projects → use `_shared/` or separate project
- Quick one-off tasks → just do them, no structure needed

---

*Add new techniques above this line*
