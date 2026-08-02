# CLAUDE.md

```yaml
_context:
  tier: 1  # Root of inheritance hierarchy
  version: 2.2
  last_updated: YYYY-MM-DD
  inherits_from: []
  techniques_applied:
    - name: "Session Initialization Protocol"
      applied: YYYY-MM-DD
    - name: "CLAUDE.md as Living Knowledge Base"
      applied: YYYY-MM-DD
```

Cross-project techniques and patterns workspace.

## Purpose & References

| Reference | File |
|-----------|------|
| Vault registry, boundaries, sync | `PROJECTS.md` |
| Technique catalog + applied tracking | `TECHNIQUES.md` |
| Cross-pollination evaluations | `EVALUATION_LOG.md` |
| Technical gotchas | `TECHNICAL_GOTCHAS.md` |
| Full principle enforcement detail | `PRINCIPLES_REFERENCE.md` |
| Skill inventory + evaluation framework | `SKILL_INDEX.md` |
| Tracking system architecture (lists, zones, body tags) | `TRACKING_SYSTEM_DESIGN.md` |
| Signal capture pipeline | `SIGNAL_CAPTURE_PATTERN.md` |
| Silent-failure safeguards (M1-M4) | `SILENT_FAILURE_SAFEGUARDS.md` |
| Learning system (Synthesizer/Advisor/Reflector/Challenger) | `LEARNING_SYSTEM_DESIGN.md` |
| Proactivity (push-based, always-on) | `PROACTIVITY_DESIGN.md` |
| System roadmap | `SYSTEM_ROADMAP.md` |
| Methodology: simulated expert panels | `guides/simulated-expert-panels.md` |
| Methodology: synthetic testing | `guides/synthetic-testing.md` |

**Technique workflow:** Document in `TECHNIQUES.md` with `Applied:` line → done. Full eval in `EVALUATION_LOG.md` only on structural triggers.

**Directory boundaries:** Code in distributable repos (e.g., `~/mcp_personal_dev/`). Personal data in vaults (`~/Projects/`). Config in dotfiles (`~/.config/`).

## Session Initialization Protocol

`/start` runs every session. Non-negotiable, even when the user opens with an action request. See `.claude/commands/start.md` for the full protocol.

**Output format:** Exception-only. Sections only appear if actionable. No "all clear" lines. Hard cap: 8 lines (excluding capture prompt).

## Discovery Protocol

> **Search first, explore as last resort.** Use search/grep tools. Only list directories or read files speculatively if search returns nothing.

This prevents the "list → read → filter → repeat" loop that burns tokens on large codebases. See **Search-First Discovery** in `TECHNIQUES.md` for full details.

---

## Core Principles (Enforced)

**The meta-rule:** documentation without enforcement is aspirational. Every principle names a verification step. Full enforcement detail and violation history: `PRINCIPLES_REFERENCE.md`.

*Customize these for your workflow. The structure (operative rule + enforcement surface + pointer to detail) is the pattern; the specific rules are yours to define.*

**House style, operative-rules-only.** Each block states what to DO, in the imperative, and points at the reference file for reasoning and history. Rationale, war stories, and superseded wording live in `PRINCIPLES_REFERENCE.md` or an archive file, never here. This file is loaded into every session, so prose that has stopped changing behavior is costing tokens on every turn. Compress on that test, not on length.

### 1. Files Are The Work Product
Knowledge lives in files, not conversation. Before declaring done: verify the artifacts exist in files. Run `/end` at session close for all persistence. Moving content between files means copy verbatim, changing only structural references. **State-file composition:** entries in always-read files (`CURRENT_STATE.md` Active Contexts, `INBOX.md`) are pointers of two sentences or fewer; detail routes to domain files.

### 2. Verify, Don't Assume
Verify claims with commands, not memory. Today's date comes from the SessionStart hook, never derived mentally. User corrections override state files. **Tool-before-question:** never ask the user a factual question a tool can answer. **Mechanical-fix-first:** a surfaced process failure gets a structural fix (script, hook, file rule) before a behavioral note. **Recurrence escalation:** the same gate-class failing twice inside 48 hours requires a mechanical fix at a named surface; a second occurrence never closes as a discipline insight. Detail: `PRINCIPLES_REFERENCE.md` Principle 2.

### 3. Understand Before Acting
Read context before producing artifacts. `/start` runs every session, non-negotiable, even when the user opens with an action request. Check `CLAUDE.md` and `CURRENT_STATE.md`; multi-repo work maps the dependency graph first. **Cross-vault awareness:** an entity mentioned in one vault but tracked in another gets grepped and presented as connected. **First-use gate:** a step that mutates external state (mail, APIs, databases) and has never run end-to-end on real data requires a planning gate before batching.

### 4. Apply Before Inventing
Search `TECHNIQUES.md` and existing vaults for a proven structure before designing a new pattern. Invent only when nothing applies.

### 5. Boundaries Are Sacred
Code in distributable repos, data in vaults, config in dotfiles. **Structured stores (SQLite and similar) belong in a config directory**, never a cloud-synced tree: sync corrupts live SQLite writes. **Sensitive-domain content (PHI, finance, work PII) is vault-only for WRITES**, never `_shared/`, git, or any durable shared artifact. **RENDER is a separate axis from WRITE.** Decide separately who may read a body, on which surface, and price each fence on its own evidence. A policy sentence of the form "never A, B, C, or D" is usually more than one rule, and the weaker half will inherit the stronger half's absoluteness unless you split them. **Credentials never in cloud-synced directories**; `.gitignore` does not protect against cloud sync. **Public repo commits:** no real paths, private repo links, or work emails. Detail: `PRINCIPLES_REFERENCE.md` Principle 5.

### 6. Sync Is Immediate
Related updates happen in the same session: registry, versions, orchestrator, build, tags, propagation, distribution. MCP servers MUST route through a central toolkit-hub package, never standalone. Pre-archive: upstream all content before marking EOL.

### 7. Future Self Can Act Immediately
Every reminder carries what, how, and context, on the correct list (Strategic = needs the assistant, Personal = offline, Inbox = capture). Write successor reminders before completing the parent. Vault-first: file content before creating the reminder. Pair every state-file action with a reminder. **`[ref:]` tags** make the poke-to-context link explicit and grep-able. **Body-sufficiency:** the item carries its execution payload in the body, and inside whatever character budget the reading surface actually renders. A body that is sufficient in principle but truncated on the phone is not sufficient.

### 8. Content Shared Is Content Captured
User shares content, so process the immediate need and route it to the digest pipeline (`save_for_later` or equivalent) with no confirmation gate. A fetch failure does not block routing: save the URL plus whatever context you have. Prep artifact and digest routing coexist.

### 9. Entity-of-Record Discipline
For each tracked entity (account, person, decision, credential), exactly one system holds the truth; every other mention references it and never duplicates it. **Before creating a second record, ask whether the first should move instead.** Credential rotation includes "find and replace every referencing site" as an atomic step, not a follow-up. **Owned-fix guard:** when a failure class already has an owned fix, new observations route to the owning record, never into a new record, a new rule, or a re-attempted fix. Detail: `PRINCIPLES_REFERENCE.md` Principle 9.

### 10. Source Before Action
Before producing any plan, breakdown, sizing, recommendation, or state-mutating action about a tracked item (a reminder body, an active spec, a referenced vault file), **read the underlying source first**. Acting from a title or a one-line summary is the highest-recurrence anti-pattern this system has recorded, which is why it is stated as its own principle rather than folded into Principle 3. Behavioral re-enforcement has repeatedly proven insufficient: enforce it mechanically at every surface that can act on a title alone. Detail: `PRINCIPLES_REFERENCE.md` Principle 10.

---

## Skill Architecture: Gather-Process-Emit

All multi-step skills (especially `/end`) follow the **Gather-Process-Emit** pattern:

1. **Gather** — Batch all reads into a single parallel tool call. Load everything you'll need.
2. **Process** — Reason about what needs to happen. **Zero tool calls** in this phase. All data was loaded in Gather.
3. **Emit** — Batch all writes. Execute the plan from Process.

**Why this matters:** Without phase separation, reads and writes interleave. This causes stale reads (you read file A, write file B, then make decisions about A using pre-B state), missed updates, and forgotten persistence. The pattern is simple but the discipline prevents a class of errors that documentation alone doesn't catch.

**When to apply:** Any skill or workflow that reads state, makes decisions, and writes updates. The `/end` skill enforces this structurally.

---

## File Staleness

Files that aren't updated become misleading. Stale context is worse than no context — it causes confident wrong answers.

**Staleness targets:**

| File | Threshold | What to do when stale |
|------|-----------|----------------------|
| `CURRENT_STATE.md` | 7 days | Update Last Session + Active Work Streams |
| `CLAUDE.md` (per project) | 30 days | Review Common Mistakes, Key Files, routing |
| `TECHNIQUES.md` | 30 days | Check for unevaluated techniques |

**Enforcement:** `/check` flags files past their staleness threshold. The `/end` skill updates `CURRENT_STATE.md` every session, which naturally prevents staleness for the most critical file.

**Common Mistakes entries** include session numbers and dates. If a mistake hasn't been violated in 10+ sessions, it's a candidate for removal — the behavior may be learned or the context may have changed.

---

## Technical Gotchas

See `TECHNICAL_GOTCHAS.md`. Covers: Claude Code hooks, Apple Reminders/eventkit CLI, AppleScript, MCP development, macOS filesystem, file management, and more.

---

*Principle violation log: see `CHANGELOG.md` → Principle Violations.*
