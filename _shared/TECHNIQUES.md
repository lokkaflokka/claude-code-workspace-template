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

### Defensive Code Generation

**Type:** Project
**Origin:** vkondrav (external)
**Added:** YYYY-MM-DD

#### What it is
Frame Claude as a senior engineer who's been burned by production incidents. Add an `ANXIETY.md` or equivalent section to your project's CLAUDE.md that lists the kinds of failures Claude should actively guard against — not as abstract principles, but as visceral "I've seen this go wrong" scenarios.

#### Why it works
- Shifts Claude from "make it work" to "make it not fail"
- Concrete failure scenarios trigger more defensive patterns than abstract security principles
- The persona framing ("you've been burned before") activates a different coding style than neutral instructions
- Results in more input validation, error handling, and edge case coverage

#### Example
```markdown
## Defensive Coding (ANXIETY.md)

You are a senior engineer who has been burned by production incidents. You've seen:
- OAuth tokens that expire mid-request and corrupt state
- Config files with valid YAML but wrong field types that pass parsing but fail at runtime
- MCP tool inputs that look valid but contain injection payloads
- Race conditions in async operations that only manifest under load

When writing code in this project, you feel the weight of these experiences.
You validate inputs even when "they should always be correct."
You handle errors even when "this path should never execute."
```

#### When to use
- Security-sensitive code (auth, payments, data access)
- Production code without extensive human review before deploy
- Code handling external inputs (APIs, user data, config files)
- Distributed systems where failure modes are non-obvious

#### When NOT to use
- Rapid prototyping or MVPs where speed matters more than robustness
- Simple CRUD operations with well-tested frameworks
- Code that will go through extensive code review anyway

---

### Route Around Unreliable Tools

**Type:** Project
**Origin:** Practitioner experience with partially reliable CLI tools
**Added:** YYYY-MM-DD

#### What it is
When a tool or dependency is partially reliable — some operations work, others fail non-deterministically — partition its operations into reliable and unreliable sets. Continue using the tool for reliable operations and route unreliable operations through alternatives.

#### Why it works
- Avoids the false binary of "use it" vs. "replace it"
- Preserves the value of the tool's working features
- Reduces debugging time by eliminating known-unreliable code paths
- The partitioning decision is explicit and documented, not implicit and discovered repeatedly

#### Example
```markdown
## Tool Reliability

**The reliability rule: `mytool` for reads, direct API for writes.**

`mytool list` and `mytool get` are reliable for reading state.
All mutation commands (`add`, `update`, `delete`) have demonstrated
non-deterministic behavior (GitHub issues #19, #42).

For mutations, use the underlying API directly:
- Create: `curl -X POST ...` or use the SDK
- Update: Same
- Delete: Same

Always verify after mutations by reading back with `mytool list`.
```

#### When to use
- Tool fails non-deterministically on specific operations (not universally broken)
- Tool is embedded enough in your workflow that replacing it entirely is costly
- Reliable operations provide genuine value worth preserving
- Alternative paths exist for the unreliable operations

#### When NOT to use
- Tool is universally unreliable (replace it entirely)
- A fix or update is available (fix the root cause instead)
- Tool is not deeply integrated (just switch to something else)

---

### Search-First Discovery

**Type:** Project
**Origin:** Common pattern for token-efficient codebase navigation
**Added:** YYYY-MM-DD

#### What it is
An explicit instruction in your project's CLAUDE.md that directs Claude to use search and grep tools before exploratory browsing when navigating the codebase. This prevents the "list → read → filter → repeat" loop that burns tokens on larger projects.

#### Why it works
- Search tools return targeted results; directory listing returns everything
- Reduces token usage significantly on projects with 20+ files
- Forces Claude to form a hypothesis ("what am I looking for?") before exploring
- Grep/search results provide immediate context vs. breadth-first browsing

#### How to implement
Add this to your project's CLAUDE.md:
```markdown
## Discovery Protocol

> **Search first, explore as last resort.** Use search/grep tools to find
> what you need. Only list directories or read files speculatively if
> search returns nothing relevant.
```

For larger projects, add specific search guidance:
```markdown
## Discovery Protocol

> **Search first, explore as last resort.** Use search/grep tools.
> Only list directories or read files speculatively if search fails.

Common search patterns:
- Config: `grep -r "configKey" src/`
- Routes: `grep -r "router\." src/routes/`
- Types: search for `interface` or `type` in `.ts` files
```

#### When to use
- Projects with 20+ files where browsing is expensive
- Token cost is a concern (API usage, long sessions)
- Project has good searchable structure (meaningful names, consistent patterns)
- Adding to CLAUDE.md for a growing project

#### When NOT to use
- Small projects (under ~10 files) where browsing is cheap
- Projects with poorly named files where search terms are hard to guess
- When exploration IS the task (learning a new codebase for the first time)

---

*Add new techniques above this line*
