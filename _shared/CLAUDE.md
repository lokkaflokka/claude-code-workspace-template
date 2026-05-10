# CLAUDE.md

```yaml
_context:
  tier: 1  # Root of inheritance hierarchy
  version: 2.1
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

**The meta-rule:** Documentation without enforcement is aspirational. Each principle has a verification step. Full enforcement detail with violation history: `PRINCIPLES_REFERENCE.md`.

*Customize these principles for your workflow. The structure (principle + enforcement + violations prevented) is the pattern; the specific rules are yours to define.*

### 1. Files Are The Work Product
Knowledge lives in files, not conversation. Before declaring done: verify artifacts exist in files. Run `/end` at session close for all persistence. Edit discipline: when moving content between files, copy verbatim — only change structural references. **State-file composition rule:** `CURRENT_STATE.md` Active Contexts and `INBOX.md` are read every session — compose entries as ≤2-sentence pointers; route detail to domain-specific files.

### 2. Verify, Don't Assume
Verify claims with commands, not memory. Today's date from SessionStart hook — never derive mentally. User corrections override state files. **Tool-before-question gate:** before asking a factual question, check if a tool can answer it (vault files, reminders, email, code, calendar). **Source-before-action gate:** Read the full source file before presenting action plans from it. **Mechanical-fix-first:** when a process failure surfaces, default to a structural fix (script edit, hook, file rule) before a behavioral note.

### 3. Understand Before Acting
Read context before producing artifacts. `/start` runs every session — non-negotiable, even for action requests. Check `CLAUDE.md`, `CURRENT_STATE.md`, understand architecture. Multi-repo: map dependency graph before writing the first file. **Cross-vault awareness:** when reading a vault file that mentions a person/event/decision tracked elsewhere, grep that entity across session context before responding. **First-use gate for destructive operations:** when a tool/skill/workflow step mutates external state (Gmail, APIs, databases) and has never been validated end-to-end on production data, require a planning gate before batching.

### 4. Apply Before Inventing
Search `TECHNIQUES.md` and existing vaults before designing new patterns. Only invent if nothing existing applies.

### 5. Boundaries Are Sacred
Code in distributable repos, data in vaults, config in dotfiles. **Structured stores (SQLite, etc.) → `~/.config/`** even for personal data — cloud sync corrupts live SQLite writes. **Sensitive-domain content (PHI, finance, work/PII) is vault-only** — never in `_shared/` files (use generic references + pointers), reminder bodies, git, or conversation as storage. **Credentials never in cloud-synced directories** — use `~/.config/` or env vars; `.gitignore` does NOT protect from cloud sync. **Public repo commits:** no real paths, no private repo links, no work emails in git author. Full enforcement: `PRINCIPLES_REFERENCE.md`.

### 6. Sync Is Immediate
Related updates happen same-session. After package/vault changes: registry, versions, orchestrator, build, tags, version propagation, distribution. MCP servers MUST go through a central toolkit-hub package — never standalone. Pre-archive: upstream all content before marking EOL.

### 7. Future Self Can Act Immediately
Every reminder: what/how/context. Correct list (Strategic = needs assistant, Personal = offline, Inbox = capture). Successor reminders before completing. Vault-first: file content before creating reminder. Action-reminder pairing: every state-file action needs a paired reminder. **`[ref:]` tags** on vault-linked reminders make the poke→context link explicit and grep-able.

### 8. Content Shared Is Content Captured
User shares content → process immediate need → route to digest pipeline (`save_for_later` or equivalent). No confirmation gate. Fetch failure doesn't block routing — save URL + available context. Both prep artifact and digest routing coexist.

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
