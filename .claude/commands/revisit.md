---
description: Position review and belief stress-testing
disable-model-invocation: true
---
# /revisit

Spaced-repetition review of accumulated positions. Stress-tests beliefs against new evidence.

**Distinct from `/challenge`:** `/challenge` stress-tests decisions/artifacts with personas. `/revisit` stress-tests accumulated positions/beliefs with evidence. `/revisit` can delegate TO `/challenge` for deeper persona-based review of a single position.

**Invoke:** `/revisit` (full audit), `/revisit P-NNN` (single position)

---

## Prerequisites

This skill assumes a learning-log architecture with named positions. See `_shared/LEARNING_SYSTEM_DESIGN.md` for the position-tracking pattern. Adapt the file paths below to wherever your positions live.

## Step 1: Gather

Read all of these in a single parallel batch:
1. Your learning log file (positions + connected insights)
2. Active themes file (theme trajectories), if you maintain one
3. The 3 most recent digest/briefing files (sorted by date desc, limit 3)

If `/revisit P-NNN` was invoked, only that position is reviewed (skip to Step 2 for that position). Otherwise, review all Active positions.

---

## Step 2: Position Review (per position)

For each position, present a structured review card:

```
## Review: P-NNN — <Title>

**Statement:** <position statement>
**Formed:** <date> | **Last reviewed:** <review date> | **Age:** <days>

### Evidence Chain
**Supporting:**
- I-NNN: <title> — <one-line relevance>
- <additional supporting evidence>

**Challenging/Qualifying:**
- <evidence from recent digests that contradicts or qualifies>
- <if none found: "No contradictory evidence found in recent digests">

### Qualifiers Added Since Formation
- <list any qualifier extensions with dates>

### Connected Themes
- TH-NNN: <title> — <trajectory: Strengthening/Stable/Weakening>
```

Present all review cards, then proceed to Step 3.

---

## Step 3: Confirmation Bias Scan

For each position:
- Count supporting vs. challenging evidence items
- Calculate ratio: `supporting / total = percentage`
- **Flag if >80% supporting** — this doesn't mean the position is wrong, but it means the system may not be surfacing disconfirming evidence

Output:
```
### Bias Scan
| Position | Supporting | Challenging | Ratio | Flag |
|----------|-----------|-------------|-------|------|
| P-001 | 5 | 1 | 83% | ⚠️ |
| P-002 | 4 | 3 | 57% | ✅ |
```

If any position is flagged, note: "Consider: are digest sources likely to surface contradictory evidence for this position? If not, what sources would?"

---

## Step 4: Position Evolution

Present evolution options for each position. **User decides** — do not auto-evolve.

Options per position:
- **Strengthen** — extend review interval (evidence continues to support)
- **Update** — add qualifier or extension (new nuance discovered)
- **Retire** — position no longer held (mark `Status: Retired` with rationale)
- **Split** — true in context A, false in context B (refactor into two positions)
- **No change** — position holds as-is, reset review date

**Review interval schedule (spaced repetition):**
- After Strengthen: current interval doubles (30d → 90d → 180d → annual)
- After Update: reset to 90d (new qualifier needs re-evaluation)
- After No change: same interval, new review date
- If contradictory evidence was flagged: reset to 30d regardless

After user decisions, write updates to the learning log:
- Update `Review date:` field with next review date (absolute date, e.g., `2026-06-15`)
- Add qualifier/extension text if Updated
- Change status if Retired

---

## Step 5: Delegate to /challenge (optional)

After all positions are reviewed, offer:

"Any positions worth deeper stress-testing? I can run `/challenge` panel mode on a position — e.g., challenge P-001 with three personas representing the strongest opposing views."

Only offer if ≥1 position had contradictory evidence or a bias flag. Otherwise skip silently.

---

## Integration with /start and /end

### /start integration

`/start` checks for due position reviews using the `Position Reviews:` line in `CURRENT_STATE.md` (maintained by `/end`). When reviews are due, `/start` surfaces:
```
⚠️ Position review due: P-001, P-003 — run /revisit
```
This counts toward the `/start` output line cap as an Alert line.

### /end integration

After `/revisit` runs in a session, `/end` Phase 0 captures:
- Which positions were reviewed
- Evolution decisions made
- Any bias flags

`/end` Phase 2 updates the `Position Reviews:` line in `CURRENT_STATE.md` with next review dates.

---

## Rules

- **User decides evolution.** Never auto-strengthen or auto-retire.
- **Evidence over opinion.** Present evidence chain, not interpretive summaries.
- **Contradictory evidence is the highest-value signal.** Finding it is success, not failure.
- **Spaced repetition resets on challenge.** Any contradictory evidence resets interval to 30d.
- **No `TECHNIQUES.md` reads.** `/revisit` examines beliefs and evidence, not workflow patterns.
