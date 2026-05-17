---
description: Session-end persistence
disable-model-invocation: false
---
# /end

Session-end reflection and state persistence. Run before closing any session to ensure all work is captured in files.

**Invoke:** `/end` (full), `/end plan` (force Plan Tomorrow phase)

## Why Phases Matter

Session-end work involves reading current state, deciding what changed, and writing updates. Without phase discipline, reads and writes interleave — causing stale reads, missed updates, and forgotten persistence. The Gather-Process-Emit pattern prevents this: scan first, batch all reads, reason with zero tool calls, then batch all writes.

## Phases

Execute all phases every session. The skill exists to prevent compression/skipping. Phases with no applicability are skipped with zero output (e.g., "no MCP work" skips MCP sync check). **Never skippable:** Phase 0 (Session Scan), Phase 2 Turn 1 (state + changelog).

### Phase 0 — Session Scan (1 turn, no tools)

Scan the session and produce a short structured list — explicit input to subsequent phases.

Produce these lists:

- **Process failures:** anything that went wrong, got corrected, or violated a principle this session
- **Verification misses:** any instance where the user was asked a factual question that a tool could have answered. Did I ask "did you...?", "have you...?", "is X still...?" when vault files, reminders, email, code, or calendar could have answered it? Empty is good.
- **Techniques applied:** techniques from `TECHNIQUES.md` applied this session
- **Generalizable patterns:** anything worth syncing to another machine — workflow improvements, gotchas, skill changes, template feedback
- **Novel insights:** methodology discoveries, cross-domain connections, reusable patterns worth persisting to a learning log
- **Position reviews:** if a position-review skill ran this session — which positions were reviewed, evolution decisions made
- **Value moment:** one-line description of the highest-value thing the system did this session — connected dots, caught something, saved time. Write "—" if nothing notable.

Keep each list short and specific. Empty lists are fine. The point is that these are *decided before writing begins* — not discovered after the changelogs are already closed.

---

### Phase 1 — Gather (1 turn)

**Make ALL of these tool calls in a single response. Do not reason about results between calls.**

1. Read `_shared/CURRENT_STATE.md` — skip if session < 25 turns (still in context from `/start`)
2. Read `_shared/CHANGELOG.md` (limit: 80 lines)
3. Read learning log / themes file — if Phase 0 flagged "novel insights" or position changes
4. Bash: list current open items from your task system (Strategic + Personal lists)
5. Any vault-specific state files touched this session

**Same-session read elision:** When `/start` and `/end` run in the same session, files read during `/start` are already in context. Skip those. Only read changelogs and files not yet loaded this session.

**If any tool call fails, proceed with available data.**

---

### Phase 2 — Write (2 turns)

Using Phase 0 scan + Phase 1 data, write all updates. Split across 2 turns to maintain edit quality.

**Do NOT re-read files after editing.** The Edit tool shows the result. Compose complete edits using Phase 1 data. Exception: if an Edit fails (`old_string` not found), re-read that specific file and retry.

**Turn 1 — Core state files (2-3 edits):**
- Edit `_shared/CURRENT_STATE.md` — Last Session (date, focus, outcome with process failures, left off), Active Contexts, decay cleanup, completions-processed-through. Trim Updates to 2 entries max — route older content to project state files. **Left off convention:** only items with remaining work. Completed items belong in Outcome, not Left off — a completed item in Left off creates a false signal that re-surfaces next session.
- Edit `_shared/CHANGELOG.md` — session entry + principle violations table (if applicable)
- Edit roadmap or system-design files — only if relevant work done

**PHI / sensitive-data redaction gate** (if your system tracks any sensitive domain — health, finance, work):
When writing state files, never include: medication names, conditions, provider names, insurer/PBM names, account numbers, customer/employer identifiers, or any other category your CLAUDE.md designates as walled-off. Use generic domain labels + vault pointer (e.g., `Health coverage path → see health/MEDICATION_ISSUES.md`). Operational detail lives in domain vault files only — `_shared/` carries pointers, not content. **Audit before saving:** grep candidate text against your domain's restricted vocabulary; non-empty match = redaction pass before final Edit.

**Outcome-drift verification:** For each Done item, verify corresponding Left off from previous session is removed/updated. Semantic matches count.

**Turn 2 — Conditional files (0-3 edits, skip turn if none apply):**
- Edit learning log — if Phase 0 flagged novel insights or position qualifiers
- Edit vault state files — only if vault work done

---

### Phase 3 — Verify + Check (1-2 turns)

Evaluate all in one turn. Execute only what applies. Skip with zero output if none apply.

**Unconditional checks (always run):**

- **Code repo hygiene scan** — every session, regardless of whether code work was done. Structural enforcement:
  ```bash
  found=""
  for dir in ~/path/to/your/code/repos/*/; do
    nm=$(basename "$dir")
    if [ ! -d "$dir/.git" ]; then found="${found}NO GIT: $nm\n"
    else
      st=$(cd "$dir" && git status --porcelain 2>/dev/null)
      up=$(cd "$dir" && git log origin/main..HEAD --oneline 2>/dev/null)
      [ -n "$st" ] && found="${found}DIRTY: $nm\n"
      [ -n "$up" ] && found="${found}UNPUSHED: $nm — $up\n"
    fi
  done
  [ -n "$found" ] && printf "$found" || echo "All repos clean."
  ```
  If findings exist: surface to user with proposed action (git init, commit, push). Do NOT silently skip. Accumulated drift across sessions is the failure mode this prevents.

**Conditional checks (batch any that apply):**

- **Package sync check** — only if package commits this session. Version bump + tag + reference updates per Principle #6. Build verification (verify build output is newer than source).
- **Vault-first check** — verify vault files contain content referenced by reminders. No dangling pointers.
- **Action-reminder pairing** — every Left off item needs a tracking home, but NOT necessarily a reminder. Apply the Task Hierarchy gate from `TRACKING_SYSTEM_DESIGN.md`:
  - Type 1 (hard deadline + needs assistant) → Strategic reminder
  - Type 3 (project-scoped, no external deadline) → vault TODO + Left-off only. **No reminder.**
  - Type 4 (evaluative/exploratory) → `PROJECT_IDEAS.md`. **No reminder.** Surfaced via spaced resurface.
  - Type 6 (personal errand, offline) → Personal reminder with real due date
  - **The gate:** "Does this have an external deadline?" If no → it's vault content, not a reminder.
- **Waiting-on pairing** — if this session created/updated a reminder with a blocker, verify `[waiting-on:]` and `[waiting-since:]` tags present. If a blocker resolved, verify tags removed and reminder updated.
- **Cross-reference maintenance** — if this session created/modified a vault entity with associated reminders:
  1. Verify reminder bodies contain `[ref: path]` tags pointing to the entity file
  2. Verify Active Contexts entry in `_shared/CURRENT_STATE.md` bridges the vault file and reminder titles (create if missing — threshold: 3+ items across both surfaces)
  3. Verify `[ref:]` tag targets still exist (catches renames)

**Verification checklist (always, using Phase 0 scan as input):**
- [ ] Process failures → logged in `CHANGELOG.md` **with structural fixes landed in enforcement files** (`PRINCIPLES_REFERENCE.md`, `TECHNICAL_GOTCHAS.md`, skill files, or init protocol). A "Fix Applied" entry that says "don't do X" or "X is now a requirement" without citing a file edit is aspirational, not structural. Verify: does the fix change what happens next time, or does it just describe what should happen?
- [ ] Techniques applied → `TECHNIQUES.md` updated (if meaningful new data)
- [ ] Generalizable patterns → handled in Phase 4
- [ ] Novel insights → learning log entry written
- [ ] Source-before-action gate check
- [ ] Verify-before-asking audit: did I ask user for status/details when verification signals were available? If yes: log as process failure with structural fix.

If any item from Phase 0 didn't land: fix now.

---

### Phase 3.5 — Plan Tomorrow (optional)

**Trigger:** Activates when (a) user invoked `/end plan`, OR (b) Phase 1 data shows tomorrow has ≥2 calendar events OR ≥2 reminders due within 2 days. Otherwise **silently skipped** — no question asked. Respects the anti-ceremony principle.

**If triggered:**

**Step 1 — Gather (parallel, no user interaction):**
- Calendar tool: tomorrow's events (5s timeout, proceed without if fails)
- From Phase 1 data: Strategic reminders due within 3 days, Personal due tomorrow, Habits due tomorrow
- Read `_shared/RESURFACE_STATE.md` — eligible backlog items for resurface

**Step 2 — Propose (assistant does the work, user reviews):**
- Identify free zones around fixed calendar events
- Place tasks in zones by type: habits → 💪 Movement, session work → 🧠 Deep Work, errands → 📋 Admin
- Surface 1-2 resurface-eligible items as "Consider?" appendix (priority-weighted 70%, random 30%)
- Present zone layout in standard format:

```
📅 Tomorrow: <Day, Date>

  🧠 Deep Work: <task>
  💪 Movement: <workout details>
  📋 Admin: <errands, email, scheduling>
  ⏰ Anchored: <calendar events>
  📆 Ahead: <deadlines within 7 days>
  💡 Consider: <1-2 resurface items from backlog>
```

Zones always present. ⏰ Anchored row only if calendar data available. 💡 Consider row only if eligible items exist.

**Step 3 — Approve (target: one user response):**
- User says "good" → plan accepted, persist as Plan Tomorrow draft in `CURRENT_STATE.md` Last Session section
- User says "swap X and Y" or "drop Z" → single adjustment, accepted
- User says "skip" or just proceeds → plan not saved, no consequence

**Plan Tomorrow persistence format** (in `CURRENT_STATE.md` Last Session):
```
**Plan Tomorrow:**
📅 <Day, Date> — <capacity from calendar if available>
🧠 <task> | 💪 <workout> | 📋 <admin items> | ⏰ <anchored events>
```
Keep to 2 lines max. Draft for `/start` to validate against live data — not a detailed spec. `/start` always regenerates the full Today Plan from scratch; the draft provides intent context that helps with priority ordering.

**Step 4 — Resurface bookkeeping (no user interaction):**
- Update `_shared/RESURFACE_STATE.md`: mark surfaced items with today's date, increment skip count for items surfaced but not accepted
- If any item hits 5 skips: prompt "Archive [item]? You've skipped it 5 times." User can archive (move to Frozen section) or reset count.

**Design constraints:**
- Target <2 minutes of human decision-making. Assistant proposes the full plan; user approves or makes one swap.
- Skipping Plan Tomorrow has zero cost. `/start` generates a Today Plan from scratch regardless. No question, no guilt, no tracking of skips.

---

### Phase 4 — Sync + Close

**Sync-worthiness check (mandatory before skipping):**
- [ ] Were any shared files modified? (`.claude/commands/*.md`, `_shared/TECHNIQUES.md`, `_shared/SKILL_INDEX.md`, `_shared/TECHNICAL_GOTCHAS.md`, etc.)
- [ ] Did Phase 0 identify generalizable patterns?
- [ ] Any skill improvements, convention changes, or workflow discoveries?

All genuinely empty → skip to skill evaluation. But "I ran out of context" or "this phase is late" is not the same as "nothing qualifies." Shared file changes are the strongest signal — if files were edited, a packet is almost certainly warranted.

Otherwise:
1. **Hard gate before drafting:** Read `SYNC_PROTOCOL.md` → observation sanitization checklist. Do not draft from session memory alone. Observation type bars: company names, people names, roles, project names, proprietary processes, internal tools, credentials, URLs, internal references.
2. Write sync packet using the observation template from `SYNC_PROTOCOL.md`. Run checklist against draft before writing to disk.
3. Display full content inline for user approval. Wait for explicit approval before committing.
4. After approval: commit and push (depending on your sync setup).

**Skill evaluation (always, final output):**
- Skills invoked: [list with modes]
- Effectiveness: [good / neutral / needed adjustment]
- Turn counts per phase: Phase 0=X, 1=X, 2=X, 3=X, 4=X
- Any work discussed but not persisted? Fix now.

---

### Phase 5 — Post-/end addendum (when work continues after close)

`/end` is a checkpoint, not a hard close. Real sessions often continue with user-directed follow-ups (commit + tag + push, security patches, late fixes, "while we're at it" cleanups). Without explicit handling, this work accumulates untracked and gets lost between sessions.

**Trigger** — any of the following after Phase 4 close:
- A new git commit in a tracked repo
- A new tag pushed
- Substantive Edit/Write work on shared files, vault, or skill files (≥3 edits OR any single edit ≥50 lines)
- User-directed multi-step operations (deploy, audit-fix, migration)

**Don't trigger on:** read-only exploration, conversation, single small edits to in-progress work tracked elsewhere, or operations that have their own state surface (`/digest`, `/route`, etc. each persist independently).

**Persistence shape** — append a `**Post-/end addendum (S<NNN>, <N> turns past /end Phase 4 close):**` block to the existing Last Session section in the state file. Do NOT create a new Last Session block — that would orphan the original /end output. The addendum block sits under "Completions processed through" and includes:

- Brief framing sentence (when the addendum work started, what triggered it)
- Per-item: 1-2 sentences capturing what landed, commits referenced by short SHA, packages/versions touched
- `**Post-/end Files touched:**` bullet list (paths + short SHAs)
- `**Post-/end generalizable patterns:**` bullet list (sync-worthy patterns, gotcha candidates)
- `**Post-/end discipline note:**` (optional) — if the addendum surfaces a recurring shape worth codifying

**Also update Left off:** mark closed items with `✅ S<NNN> post-/end` framing. If new follow-ups emerged, add to Left off as fresh `[ ]` entries.

**When to invoke:**
- User says "we're past /end" or "don't lose track of this stuff" — explicit
- Agent recognizes trigger conditions hit ≥3 turns past Phase 4 — proactive surface to user ("we've landed [N items] since /end — capture as Post-/end addendum?")
- Before any subsequent `/start` (the natural next-session checkpoint)

**Edit-quality rules** (same as Phase 2): no re-read after Edit unless old_string fails; copy verbatim when moving content; any redaction gates active for Phase 2 apply to addendum content just as to Last Session.

**Cross-session lifecycle:** Post-/end addendums from previous sessions get integrated into the main Last Session prose during the next session's Phase 2 Turn 1 rewrite — the addendum is a working surface, not a permanent structure. If the next session also accumulates a post-/end addendum, append fresh; the prior addendum has already been folded.

---

## Rules

- Execute all phases every session. The skill exists to prevent compression/skipping.
- **Never skippable:** Phase 0 (Session Scan), Phase 2 Turn 1 (state file + changelog).
- **Execution enforcement:** Every phase must produce visible evidence of execution — tool calls (Read, Grep, Bash, Edit) that verify or act. Asserting completion in prose without tool calls is assertion, not execution. This failure mode is most likely during long sessions, API error recovery, or context compaction.
- Edit discipline: when moving content between sections or files, copy original text verbatim. Only modify structural references.
- **Audit trail hygiene:** State files document substantive work output — not communication strategy, relationship dynamics, or stakeholder management framing.

## Common Mistakes Format

Common Mistakes entries include session number and date: `[S12, 2026-03-10: description of what happened]`. If a mistake hasn't been violated in 10+ sessions, it's a candidate for removal — the behavior may be learned or context may have changed.
