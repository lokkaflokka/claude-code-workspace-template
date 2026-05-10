# CLAUDE.md

This is the workspace root — a meta/organizational layer that spans all projects.

## LLM Usage Policy (Two-Lane Model)

*Add this section if your company provides a different LLM (e.g., Gemini, ChatGPT Enterprise). Skip if Claude is the only LLM in your workflow.*

**Principle:** Company-approved LLM handles sensitive inputs. Personal Claude handles only non-sensitive work with placeholders.

**Default rule:** If unsure whether content is sensitive, use the company LLM.

### Lane A: Company LLM (Sensitive)

Use when input contains ANY of:
- Customer/member identifiers (names, emails, IDs)
- Production extracts (logs, screenshots, support tickets, raw query results)
- Secrets/credentials (API keys, tokens, passwords, private URLs)
- Security details (incident content, vulnerabilities, internal controls)
- Contracts, pricing, unreleased strategy, or partner terms

### Lane B: Claude (Non-Sensitive)

Allowed:
- Architecture/design tradeoffs, system diagrams, data modeling patterns
- Drafting PRDs, ADRs, RFCs using **placeholders** (no real names/data)
- Templates, checklists, runbooks, test plans, evaluation frameworks
- Code refactors with no secrets or sensitive payloads
- Abstracted synthesis from sensitive work (no raw snippets or identifiers)

### Sensitivity Lint (10 seconds, every prompt)

Before sending to Claude:
1. Does this contain identifiers or direct extracts (PII, logs, rows, screenshots)?
2. Could this reveal customer, partner, or production internals?
3. Would I avoid putting this in a company-wide Slack channel?

**Any YES = Company LLM.** All NO = Claude.

### Redaction Standard

When sanitizing for Claude, use these placeholders:
- **Entities:** `CUSTOMER_A`, `PAYER_B`, `VENDOR_C`
- **Tables:** `TABLE_CLAIMS`, `TABLE_ELIG`, `TABLE_PAYMENTS`
- **IDs:** `MEMBER_#`, `CLAIM_#`, `ID_1`
- **Dates:** `2026-Q1`, `T-7d`, `MONTH_YEAR`
- **Numbers:** `~Nk rows`, `p95 ~Xs`, bucketed ranges (low/med/high)

Translate once, then ask Claude for the general solution. For structured handoffs, see `templates/COMPANY_LLM_HANDOFF.md`.

---

## Session Initialization

`/start` runs every session. **Non-negotiable**, even when the user opens with an action request or plan. The user opening with "implement this plan" or "do X" does NOT skip initialization. Run `/start`, present the summary, THEN proceed to the user's request.

Reminders due today are often directly relevant to the work being requested. Skipping init means missing this context.

See `.claude/commands/start.md` for the full protocol (Phase A gather → Phase B process+output → Phase C interactive triage).

**Output format — exception-only:** sections only appear if actionable. No "all clear" lines. Hard cap: 8 lines (excluding capture prompt). Full design rationale in `_shared/TRACKING_SYSTEM_DESIGN.md` → Today Plan.

## Custom Skills

Skills live in `.claude/commands/`. See `_shared/SKILL_INDEX.md` for the full inventory + evaluation framework. Quick reference for the lifecycle skills:

| Skill | When to invoke |
|-------|---------------|
| `/start` | Every session |
| `/end` | Every session |
| `/capture` | User shares content (URL, screenshot, text, PDF) |
| `/check` | Biweekly system health pass |
| `/sync-review` | When `/start` surfaces pending sync packets |
| `/meeting-notes` | Post-meeting extraction |
| `/challenge` | Decision with stakes |
| `/route` | Post-synthesis batch routing |
| `/revisit` | Position review dates due |
| `/new-project` | Setting up a new vault |

## Query Routing

| User Intent | Go To | Action |
|-------------|-------|--------|
| "What needs attention?" | `_shared/CURRENT_STATE.md` | Surface alerts |
| "New technique to document" | `_shared/TECHNIQUES.md` | Document → evaluate → log |
| "Evaluate technique X" | `_shared/EVALUATION_LOG.md` | Read technique, assess per project, log |
| "What projects exist?" | `_shared/PROJECTS.md` | List registry |
| "Work on [specific project]" | `[project]/CLAUDE.md` | Switch context to project |
| "Cross-project task" | Depends | Identify involved projects first |
| "Where do principles live?" | `_shared/PRINCIPLES_REFERENCE.md` | Full enforcement detail |
| "Where do skills live?" | `_shared/SKILL_INDEX.md` | Skill inventory + evaluation |

## What NOT to Do at This Level

- **Don't dive into a specific project without asking** — If the user meant to work in `finance/`, they would have started there
- **Don't assume project-specific context** — Work at this level is meta/organizational
- **Don't skip the state check** — `/start` always reads `CURRENT_STATE.md` first

## Directory Reference

| Directory | Purpose |
|-----------|---------|
| `_shared/` | Cross-project techniques, evaluations, design specs, meta-level state |
| `_example-project/` | Template for new projects (copy and customize) |
| `.claude/commands/` | Custom skills (slash commands) |
| `templates/` | Reusable templates (handoff format, evidence log, etc.) |

*Add your actual projects to this table as you create them.*

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — top-level session initialization, LLM lane policy, query routing |
| `_shared/CLAUDE.md` | Cross-project principles (concise) + workflow architecture |
| `_shared/CURRENT_STATE.md` | Meta-level dashboard with alerts |
| `_shared/INBOX.md` | Capture staging area (unrouted items) |
| `_shared/PROJECTS.md` | Project registry |
| `_shared/TECHNIQUES.md` | Pattern catalog |
| `_shared/PRINCIPLES_REFERENCE.md` | Full enforcement detail for the 8 principles |
| `_shared/TECHNICAL_GOTCHAS.md` | Factual reminders about tools and environment |
| `_shared/SKILL_INDEX.md` | Skill inventory + evaluation framework |
| `_shared/TRACKING_SYSTEM_DESIGN.md` | Lists, zones, body tags, completion algorithm |
| `_shared/SIGNAL_CAPTURE_PATTERN.md` | Capture pipeline (5-stage) |
| `_shared/SILENT_FAILURE_SAFEGUARDS.md` | M1-M4 framework |
| `_shared/LEARNING_SYSTEM_DESIGN.md` | Synthesizer/Advisor/Reflector/Challenger protocols |
| `_shared/PROACTIVITY_DESIGN.md` | Push-based proactive behaviors |
| `_shared/SYSTEM_ROADMAP.md` | Platform-level roadmap |
| `README.md` | Setup guide and philosophy |
| `SYNC_PROTOCOL.md` | Cross-machine learning sync protocol |
| `MCP_SETUP.md` | MCP workspace setup guide |
