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

### Routing Gate (mandatory before adding to this file)

Before adding any technique entry, answer in order:

1. **Who consumes this?** Name the specific skill files, gotcha entries, or `CLAUDE.md` sections where the rule would actually fire at a decision point. If you can't name at least one, the technique is not yet actionable — defer.
2. **How many distinct consumers?**
   - **Exactly one consumer** → inline into that consumer file. Do NOT add here.
   - **Two or more consumers across different skills/contexts** → registry entry here is justified.
3. **Has it been applied?** If the `Applied:` line would read "Not yet applied," either apply it now (inline at the consumer) or defer the entry until first use. A registry of unapplied techniques calcifies into defensive bloat.

A registry-justified technique gets its own `### ` heading with a `<!-- T-NNN -->` comment for stable cross-referencing. See `_shared/LEARNING_SYSTEM_DESIGN.md` → Data Model for the full ID scheme.

### Adding a Technique

```markdown
### [Technique Name]
<!-- T-NNN -->

**Type:** Project | Workflow
**Origin:** [Where you learned it — project, person, article, etc.]
**Added:** YYYY-MM-DD

#### What it is
[Brief description]

#### Why it works
[Key mechanics — what makes this effective]

#### Example
[Concrete example of the technique in action]

#### When to use
[Situations where this technique applies]

#### When NOT to use
[Situations where this technique is overkill or counterproductive]

**Applied:** [vault names + dates as it's adopted]
**Effectiveness:** [qualitative assessment — added/updated by Reflector]
**Connects to:** [cross-references to related T-NNN, I-NNN, P-NNN, TH-NNN]
```

### After Adding

- **Project techniques:** Evaluate relevance across all projects, log in `EVALUATION_LOG.md`
- **Workflow techniques:** Note installation/setup steps; no per-project evaluation needed
- **Effectiveness tracking:** When the technique is applied, the Reflector protocol updates the `Effectiveness:` field with qualitative assessment. See `LEARNING_SYSTEM_DESIGN.md` for the protocol.

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

### Preflight Sanity Check

**Type:** Workflow
**Origin:** Common pattern for config-dependent workflows
**Added:** YYYY-MM-DD

#### What it is
Before running a workflow that depends on configuration or external state, perform explicit checks to verify the system will behave as expected. Surface issues before wasted work, not after.

#### Why it works
- Config drift is invisible — changes may not take effect until restart
- Silent failures waste entire workflow runs on stale state
- A 10-second check before a 5-minute workflow saves time net
- Debugging from "bad output" is harder than debugging from "check failed"

#### The pattern

**Step 0 (Before work):**
- Were config files recently changed? → May need service restart
- Are dependent services running and healthy?
- Use conservative settings initially (don't clear state until confirmed good)

**Step N (After initial output):**
- Does output match expected behavior?
- Check for telltale signs of stale config
- Surface issues before proceeding

#### Example
```markdown
### Step 0: Preflight

Before running the data pipeline:
- Config changed this session → restart the MCP server
- Using `dry_run: true` to preserve state if issues arise
- Check: API token valid? `curl -s $API/health`

### Step 3: Sanity Check

Quick check on returned data:
- Expected categories present? ✓
- Old filter rules still appearing? → config not applied
- Record count in expected range? → data source healthy
```

#### When to use
- Workflows depending on config files (scoring rules, filters, thresholds)
- Tools that require service restarts to pick up changes
- Pipelines where early steps affect later steps (catch errors early)

#### When NOT to use
- Simple, stateless operations with no config dependencies
- Interactive work where you'll see issues immediately

---

### Fallback Instructions

**Type:** Workflow
**Origin:** Common pattern for robust workflows
**Added:** YYYY-MM-DD

#### What it is
Every workflow step that depends on an external tool, API, or service should document three things: how to detect it's not working, what to do instead, and what quality/capability is lost in fallback mode.

#### Why it works
- Workflows can proceed even when components fail
- Users know they're getting degraded output (transparency)
- Clear distinction between "missing" and "broken"
- Non-critical failures don't block the entire workflow

#### The three questions

For each external dependency, document:

1. **Detection:** How do you know it's not working?
2. **Fallback:** What's the degraded-mode operation?
3. **Impact:** What quality/capability is lost?

#### Example
```markdown
### Step 1: Fetch Data

Call the API with preferred parameters...

**Fallback:** If the API is unavailable:
- Use cached data from last successful run
- Note to user: "Using cached data from [date] — results may be stale"
- Skip time-sensitive analysis

### Step 3: Enrich with External Source

Query the enrichment API...

**Fallback:** If enrichment service is down:
- Proceed with base data only
- Note to user: "Enrichment unavailable — categories may be less precise"
- Impact: ~15% lower categorization accuracy
```

#### When to use
- Commands or skills that call external tools or APIs
- Workflows depending on services that can be intermittently unavailable
- Any automation where partial success is better than total failure

#### When NOT to use
- Workflows where partial output is worse than no output (e.g., financial transactions)
- Steps where the dependency IS the work (no meaningful fallback exists)

---

### Interactive Feedback Loop

**Type:** Workflow
**Origin:** Common pattern for synthesis and curation workflows
**Added:** YYYY-MM-DD

#### What it is
After generating any synthesis, curation, or summarization output, explicitly prompt the user for corrections before finalizing. Captures learning when context is fresh and the user can see exactly what to correct.

#### Why it works
- **Timing:** User's knowledge is most accessible immediately after seeing output
- **Specificity:** Corrections are grounded in concrete examples, not abstract preferences
- **Accumulation:** Each correction can inform future runs (via config, CLAUDE.md, or autoskill)
- **Engagement:** Transforms passive consumption into active refinement

#### The three prompts

After presenting draft output, ask:

1. **Filter corrections:** "Did I include something off-topic, or miss something relevant?"
2. **Contextual knowledge:** "Any insights on specific items I should know about?"
3. **Categorization accuracy:** "Are the groupings/themes right?"

#### Example
```
Here's the draft summary with 6 items across 3 themes.

Before I finalize:
1. Did I include something off-topic, or miss something relevant?
2. Any context on specific items I should factor in?
3. Are these theme groupings accurate?
```

#### When to use
- Content curation (digests, reading lists, recommendations)
- Summarization tasks (meeting notes, document synthesis, research briefs)
- Any output where the user has domain knowledge Claude lacks

#### When NOT to use
- Purely mechanical tasks with objectively correct output
- Time-sensitive work where the feedback loop adds unacceptable delay
- When the user has explicitly said "just do it, don't ask"

---

### Verification-Led Development

**Type:** Workflow
**Origin:** Boris Cherny / Anthropic
**Added:** YYYY-MM-DD

#### What it is
Always give Claude a way to verify its own work — running tests, build commands, linters, or browser automation — before considering a task complete. Claude iterates until verification passes, then submits for human review.

#### Why it works
- Self-verification catches errors before human review
- Feedback loops create iterative improvement within a single task
- Reduces back-and-forth between human and AI
- Reported to improve output quality by 2-3x

#### How to implement

Add verification instructions to your project's CLAUDE.md:

```markdown
## Verification

Before declaring any code task complete:
1. Run `npm test` and fix any failures
2. Run `npm run lint` and fix any warnings
3. Run `npm run build` and verify it succeeds
4. If UI changes: describe what you'd visually verify

Do not ask for human review until all verification passes.
```

For non-code projects, verification can mean:
- Re-reading the file you just wrote and checking for consistency
- Running a consistency check across related files
- Verifying links, references, and cross-file pointers

#### When to use
- Any coding task with testable outcomes (features, APIs, UI)
- Projects with existing test suites, linters, or build commands
- Workflows where Claude can run verification commands

#### When NOT to use
- Pure knowledge work with no mechanical verification possible
- Tasks where "correct" requires human judgment (design, tone, strategy)
- Quick one-off changes where verification overhead exceeds the task itself

---

### Convention Density (Opinionated Frameworks as LLM Multipliers)

**Type:** Workflow
**Origin:** Garry Tan (Feb 2026) + practitioner observation
**Added:** YYYY-MM-DD

#### What it is
LLMs are dramatically more productive when operating within convention-heavy, opinionated systems. The more predictable the patterns, the less ambiguity the LLM faces, and the higher quality its output. "Convention over configuration" isn't just a framework design philosophy — it's an LLM productivity multiplier.

#### Why it works
- **Reduced ambiguity:** When there's one right way to do something, the LLM doesn't waste tokens deliberating or guessing
- **Pattern matching strength:** LLMs excel at pattern completion. Dense conventions = more patterns to match against
- **Error reduction:** Strong conventions constrain the solution space, making incorrect outputs less likely
- **Compounding returns:** Each convention reinforces others. Session init reads CURRENT_STATE.md (convention) which uses a structured format (convention) with specific sections (convention). The LLM navigates all of this automatically.

#### Examples

**Rails + Claude Code (Garry Tan's observation):**
Rails prescribes directory structure, naming conventions, migration patterns, routing conventions. Claude Code generates correct Rails code because there's usually one right answer. "LLMs are sugar fiends" — they thrive on syntactic sugar and strong defaults.

**This workspace system:**
The workspace template is convention-heavy by design — and this is WHY it works:
- CLAUDE.md in every project (convention) → Claude always knows where to look for instructions
- Session init protocol (convention) → predictable startup sequence
- Query routing tables (convention) → deterministic file lookups
- CURRENT_STATE.md format (convention) → consistent state persistence
- Technique catalog format (convention) → structured knowledge capture

Each convention is a "rail" that keeps Claude Code on track. The system's productivity comes from convention density, not from any single feature.

#### Applicability notes
- **High value when:** Building systems that Claude Code will operate in repeatedly. More conventions = faster, more accurate sessions.
- **Design implication:** When choosing frameworks, libraries, or structuring projects, prefer opinionated options over flexible ones. The "flexibility" of a minimal framework is a tax on LLM productivity.
- **Anti-pattern:** Over-convention (bureaucratic overhead that slows humans without helping the LLM). Conventions should serve both human readability AND LLM predictability.
- **Tension with "avoid over-engineering":** Conventions aren't over-engineering if they reduce ambiguity. A CLAUDE.md file isn't gold-plating — it's infrastructure. The test: does this convention make the LLM's next action more predictable?

---

### Exception-Based Reporting (Frequency-Tiered Monitoring)

**Type:** Workflow
**Origin:** SRE alerting principles + practitioner observation
**Added:** YYYY-MM-DD

#### What it is
Recurring status displays should be exception-only at high frequency, with thorough audits at lower frequency. High-frequency = "only show what needs attention." Low-frequency = "check everything." This prevents alert fatigue while maintaining the same coverage.

The key insight: **"all clear" is not information.** Showing 5 green checkmarks alongside 1 red flag trains the reader to skim. Showing only the red flag commands attention.

#### Why it works
- Reduces cognitive load on high-frequency surfaces (session init reads happen every session)
- Eliminates "all clear" noise that trains readers to skip the section
- Thorough checks still happen — just at a sustainable cadence
- Mirrors SRE alerting principles: page on anomalies, dashboard for trends

#### Example
Before (verbose CURRENT_STATE.md ALERTS section):
```
### Registry Drift
| ✅ | None detected | ... |

### File Size Exceptions
| project/CURRENT_STATE.md | 134 | suppressed | ... |

### Pending Updates
| — | (none) | — | — |
```

After (exception-only):
```
### Packages Needing Attention
| my-template | Release target Feb 10 |

*Other checks via /consistency-check. Only listed here when actionable.*
```

Same coverage, far fewer lines read every session. Clean categories checked periodically via `/consistency-check`.

#### Applicability notes
- **Apply when:** Any status display read more than 1x/week. Session init, dashboards, PR check summaries, monitoring.
- **Pair with:** A periodic thorough audit at lower cadence (weekly, monthly). The exception-only display is only safe if the thorough check exists.
- **Anti-pattern:** Exception-only with no thorough audit. If you only show problems, you'll never notice things you forgot to check for.
- **Related:** Session Initialization Protocol (this technique refines its output format). Also related to "Preflight Sanity Check" — that technique is the thorough audit that complements exception-only init.

---

*Add new techniques above this line*
