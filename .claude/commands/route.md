---
description: Execute staged routing proposals from a synthesis pass
disable-model-invocation: false
---
# /route

Execute batch routing proposals from a staged synthesis output. Reads the staged-insights file, presents all actions for approval, then executes approved writes across vault files.

**Invocation:**
- `/route` — auto-discover most recent staged file
- `/route YYYY-MM-DD` — target a specific date's staged file

## Prerequisites

This skill assumes a synthesis pipeline that produces a staged-insights file (insights, position qualifiers, technique candidates, theme updates). Adapt the file paths below to wherever your pipeline writes its output.

---

## Phase 1: Load

### Step 0: Preflight

Verify before proceeding:

1. **Staged file exists:** Glob your staged-insights directory. If empty: "No staged proposals found — run synthesis first." Stop.
2. **Target files reachable:** Check existence of any vault files the staged file proposes writing to (themes file, learning log, `_shared/TECHNIQUES.md`, etc.). Report any missing — non-blocking (handled per-item in execution).
3. **Stale check:** If staged file date is 3+ weeks old, warn: "Staged file is from [date] — items may be stale. Continue?"

### Step 1: Locate Staged File

If date argument provided: glob for `*{date}*` in your staged-insights directory. Otherwise: take most recent file by modification time. If multiple candidates, show list and ask.

Read the full staged file in this step. The read happens before the approval table — source and gate in one pass.

### Step 2: Parse Action Groups

Parse staged file into action groups. Standard groups:

| Group | Content | Approval |
|-------|---------|----------|
| **A — Theme Updates** | Thread appends, last-seen bumps, appearances, trajectory | Batch |
| **B — Learning Log** | New insights (I-NNN), position qualifiers (P-NNN), cross-references | Batch |
| **C — Technique Candidates** | New entries in `_shared/TECHNIQUES.md` Candidates section | Batch |
| **D — Vault Routing** | Insights routed to vault files (destination needs resolution) | Per-item |
| **E — New Themes** | Proposed new TH-NNN entries | Per-item |
| **F — Demotions** | Themes moving to Dormant/Retired | Per-item |

Only present groups that have content. Skip empty groups silently.

**Meta-Observations are never actionable.** Skip any `## Meta-Observations` section entirely — never route from it.

**Partial prior run detection:** If items have `[COMPLETED]` markers from a previous run, skip those and present only pending items.

---

## Phase 2: Approve

### Step 3: Visible Output Gate

Present a unified approval table showing ALL concrete actions before any writes:

```
## Routing Proposals from <filename>

### Group A: Theme Updates (N items)
| Theme | Update | Detail |
|-------|--------|--------|
| TH-001 | Thread + Last seen | <brief> |

### Group B: Learning Log (N items)
| Action | ID | Detail |
|--------|-----|--------|
| New insight | I-0XX | <title> |
| Extend position | P-00X | <qualifier> |

### Group C: Technique Candidates (N items)
| Candidate | Source |
|-----------|--------|
| <name> | <source> |

### Group D: Vault Routing (N items — per-item in Phase 4)
| # | Signal | Proposed Destination | Confidence |
|---|--------|---------------------|------------|
| 1 | <insight> | <resolved path> | High/Med/Low |

### Group E/F: New Themes / Demotions (if any)
[Listed individually]
```

**This table is the visible output gate.** No file writes occur until the user explicitly acknowledges it. Structural enforcement — the table cannot be skipped under context pressure.

Ask: "Approve Groups A-C as batch? Groups D-F will be confirmed individually during execution."

---

## Phase 3: Execute Deterministic (after batch approval)

### Step 4: Themes file (automated if scripted)

If a deterministic routing script exists for theme updates, run it instead of manual edits — automation prevents drift on the most-touched file. Otherwise, edit manually.

The script should handle:
- Updating `Last seen`, `Appearances` (preserve `+ N sessions` suffix), `Trajectory` (only when changed — skip "No change")
- Appending thread entries with duplicate detection
- Updating `Last Synthesizer run:` header

**Connected Insights/Positions:** After theme updates, check if any reference new I-NNN or P-NNN IDs not yet in the theme's Connected sections. Add manually — these are judgment calls a script doesn't automate.

**Skip entirely if your themes file doesn't exist.**

### Step 5: Learning Log

Read the learning log once. Then:

**New I-NNN entries:** Count highest existing I-NNN. Assign next sequential IDs. Write each:
```
- I-NNN | YYYY-MM-DD | **Title** (Source): Summary
  Status: Active
  Connects to: [IDs from staged file]
  Vault relevance: [from staged file]
```

**P-NNN qualifier extensions:** Find the existing P-NNN entry. Append qualifier with date — never replace existing position text:
```
**Qualifier (YYYY-MM-DD):** [qualifier text]
```

**Cross-reference additions:** For each `Connects to:` proposal, find the referenced entry and add the new ID to its `Connects to:` line (bidirectional linking).

### Step 6: TECHNIQUES.md

Read `_shared/TECHNIQUES.md`. Locate the `## Candidates` section. Append each Group C entry:
```
- **Pattern Name** (from source, YYYY-MM-DD): Description. Applicability context.
```

### Step 7: Status Gist Update (optional)

If you maintain an external staleness gist (e.g., for cross-machine awareness or session-init signals), update it after Step 4 completes. Patch in `last_run` + current theme state. **Do not** touch `last_digest` — that field is owned by the autonomous digest runner.

Use a fetch-merge-write pattern that preserves fields you don't own:
1. Fetch current gist
2. Merge: preserve `last_digest`, replace `last_run` + `themes`
3. PATCH the gist

Construct nested JSON via Python rather than shell (avoids manual escaping for em-dashes, quotes, unicode). Skip this step if your themes file doesn't exist or you don't run such a gist.

---

## Phase 4: Execute Judgment (per-item)

### Step 8: Vault Routing (Group D)

For each Group D item, sequentially:

1. **Show:** Signal summary + proposed destination + resolved file path + confidence
2. **Read** the resolved target file (Principle #2 — source-before-action)
3. **Show** proposed insertion point + exact text to be written
4. **User confirms**, skips, or redirects to a different file
5. **Write** on confirm only

**Destination resolution:** Read vault `CLAUDE.md` files and directory structure to resolve vague destinations (e.g., "career vault" → specific file path). When ambiguous, present 2-3 candidate files. User always confirms.

**Sensitive-data boundary check:** If content contains walled-off domain information, it must route to that domain's vault only — never reminders, never git-tracked public files. Flag and confirm.

### Step 9: New Themes and Demotions (Groups E/F)

**New themes:** Show proposed TH-NNN entry with all fields (Status, First seen, Trajectory, initial Thread entry, Vault Routing). User confirms or rejects. Write confirmed entries to themes file.

**Demotions:** Show theme being moved and reason. User confirms. Update status field.

---

## Phase 5: Complete

### Step 10: Summary and Archive

Print execution summary:
```
| Group | Proposed | Executed | Skipped |
|-------|----------|----------|---------|
| A — Themes | N | N | N |
| B — Learning Log | N | N | N |
| C — Techniques | N | N | N |
| D — Vault Routes | N | N | N |
| E — New Themes | N | N | N |
| F — Demotions | N | N | N |
```

**All handled:** Move staged file to your archive directory. Rename to preserve original filename.

**Partial execution:** Keep staged file in staged-insights. Mark completed items with `[COMPLETED]` prefix so next `/route` invocation skips them.

---

## Fallback Behavior

| Dependency | Detection | Fallback | Impact |
|-----------|-----------|----------|--------|
| Staged file not found | Glob returns empty | "No staged proposals found — run synthesis first." Stop. | No execution |
| Target file missing | Read fails | "Target [path] not found. Create? Skip? Redirect?" | Per-item decision |
| Stale staged file | Date 3+ weeks old | Warn: "Items may be stale." User decides. | Lower confidence |
| Partial prior run | `[COMPLETED]` markers | Skip completed, present only pending | Resumable |
| Vault not on machine | Path doesn't resolve | Skip those items with note | Partial execution |

---

## Rules

- **Source-before-action (Principle #2):** Read every target file before writing. Staged file read before approval table.
- **Staged file is the contract.** Only execute what the file proposes. No inferred writes from memory or conversation.
- **Visible output gate.** No file writes before the approval table is acknowledged. Structural, not advisory.
- **Meta-Observations are never actionable.** Never route from `## Meta-Observations`.
- **P-NNN updates are additive.** Append qualifier with date — never replace position text.
- **I-NNN numbering is sequential.** Count highest existing ID before assigning.
- **Status gist (if used) runs after themes file is fully updated.**
- **Group D items are never batch-executed** — resolve and read each target individually, even if user approves all at once.
- **Partial execution is designed behavior.** Archive only when all items are handled (executed or explicitly skipped).
- **No hardcoded vault paths.** Discover structure from local `CLAUDE.md` files and file existence.
- **Sync packet directory enforcement:** If `/route` generates a sync packet for cross-machine flow, write to your outbox directory only. Never write to your inbox — that's inbound from the other machine.
- **Sync packet frontmatter (mandatory if cross-machine):** All sync packets must use standard YAML frontmatter (`source`, `date`, `session`, `type`) — never freeform body headers. Receivers key on the frontmatter.
