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

## Session Initialization Protocol

**MANDATORY on every new session at this level — no exceptions, even if the user's first message is an explicit action request or plan.**

The user opening with "implement this plan" or "do X" does NOT skip initialization. Run init first, present the summary, THEN proceed to the user's request. Reminders due today are often directly relevant to the work being requested. Skipping init means missing this context.

**Steps (run before any other work):**

1. **Read `_shared/CURRENT_STATE.md`** — Meta-level dashboard (context for session, not reported directly)
2. **Check capture staging area** — Read `_shared/INBOX.md` (if it exists) for unrouted items. Also ask: "Anything to capture since last session?" — the user may have items from any source.
3. **Check reminders/deadlines** — Whatever reminder system you use:
   - **Anchor today's date first** from the environment `Today's date` field — all "due in X days" calculations MUST use this. Do not infer or assume the date.
   - **Recommended list architecture** (3 lists): (1) items needing session context, (2) offline/personal tasks, (3) quick capture inbox. Triage inbox items each session.
   - **Upcoming deadlines:** Surface open items due within 7 days from your "needs session context" list.
   - **Due today/tomorrow:** Surface non-recurring items due today/tomorrow from your "offline tasks" list (skip recurring habits/errands).
   - **Completed item processing:** Scan for items completed since last session. Cross-reference against CURRENT_STATE.md — if the outcome is already reflected in state files, skip silently. If not reflected, surface and ask for details. Skip obvious recurring habits.
4. **Check sync inbox** — Glob `_shared/sync/inbox/*.md`. If packets exist, surface: "📨 N sync packets pending in inbox." Process with `/sync-review`. Omit if empty.
5. **Clarify if intent seems project-specific** — If the request clearly belongs to one project, ask which before diving in.
6. **Otherwise, recognize meta-level patterns:**
   - "New technique" → Document in `_shared/TECHNIQUES.md`, evaluate, log
   - "Cross-project work" → Identify which projects are involved
   - "New project setup" → Create from `_example-project/` template

**Output format — exception-only:**

The init readout only surfaces things that need attention. If a category has nothing actionable, it doesn't appear. No "all clear" lines.

- **INBOX:** Only mention if unrouted items exist.
- **Sync inbox:** Show if packets exist in `_shared/sync/inbox/`. 📨 marker. Omit if empty.
- **Reminders:** Show items due within 7 days. 🔴 ≤1 day, ⏳ 2-3 days, • 4-7 days. Omit section if nothing due.
- **Today:** Non-recurring items due today/tomorrow. 📋 marker. Omit if none.
- **Completed since last session:** Only show completions not yet reflected in state files. Omit if all are already reflected.
- **ALERTS from CURRENT_STATE.md:** Exception-only — only surface actionable alerts. Clean categories don't appear.
- **Last session:** Always show — one line with focus and what was left off.
- **Capture prompt:** Always end with "Anything to capture since last session?"

**NOT included in init** (available separately):
- **Consistency checks** → run `/consistency-check`
- These run at their own cadence, not every session.

**Example — typical session (some things need attention):**
```
🔴 Feb 4: Submit expense report
⏳ Feb 7: Review quarterly goals
📋 Today: Pick up prescription

🔄 Last session (Feb 3): Travel automation. Left off: booking flow.
Anything to capture?
```

**Example — nothing urgent:**
```
🔄 Last session (Feb 3): Documented 2 techniques, evaluated 1.
Anything to capture?
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
| `_shared/INBOX.md` | Capture staging area (unrouted items) |
| `_shared/TECHNIQUES.md` | Pattern catalog |
| `_shared/PROJECTS.md` | Project registry |
| `README.md` | Setup guide and philosophy |
| `SYNC_PROTOCOL.md` | Cross-machine learning sync protocol |
| `MCP_SETUP.md` | MCP workspace setup guide |
| `SNIPPETS.md` | Claude Code prompt reference |
