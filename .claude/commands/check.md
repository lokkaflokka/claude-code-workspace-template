---
description: System health check
disable-model-invocation: false
---
# /check

System health check — mechanical consistency + semantic health.

**Invoke:** `/check` (full), `/check merge-pending` (run only the Pending Merge step)

## Mechanical Checks

### 1. Registry Drift
- List all directories in your code repos location (e.g., `~/mcp_personal_dev/mcp-authored/`)
- Compare against `_shared/PROJECTS.md` vault registry
- Flag packages that exist but aren't in registry, and vice versa

### 2. Version Sync
- For each vault-package link in `PROJECTS.md`, verify versions match
- Check `package.json` (or equivalent) version against `PROJECTS.md` and any vault `_context.mcp_package.version` markers

### 3. Technique Applied Lines
- Identify real technique entries in `_shared/TECHNIQUES.md`: a `### ` header is a technique entry only if `**Type:**` appears within the next 3 lines (excludes code blocks and sub-sections)
- For each, check that `**Applied:**` line exists before next technique entry
- Flag any missing

### 4. Tag Drift
For each repo:
- Find latest git tag, count commits past it
- Flag if commits > 0 past latest tag (version bump needed)
- Check `package.json` version matches latest tag

```bash
for dir in ~/path/to/your/repos/*/; do
  pkg=$(basename "$dir")
  cd "$dir"
  tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "NONE")
  past=$(git rev-list "${tag}"..HEAD --count 2>/dev/null || echo "n/a")
  ver=$(node -p 'require("./package.json").version' 2>/dev/null || echo 'n/a')
  types=$(git log "${tag}"..HEAD --format="%s" 2>/dev/null | sed 's/(.*//' | sed 's/:.*//' | sort -u | tr '\n' ',' | sed 's/,$//')
  echo "$pkg: tag=$tag past=$past pkg=$ver types=[$types]"
done
```

The `types` field shows commit prefixes. `feat:` = minor bump, `fix:` = patch bump, `docs:/chore:` = no bump.

### 5. File Surface Health

Coverage-by-discovery sweep across primary surfaces. Find all files, classify by surface class, score against three-tier thresholds, surface remediation prompts for flagged files.

**Discovery sweep:**

```bash
find ~/Projects \( -path "*/node_modules" -o -path "*/.git" -o -path "*/_scratch" -o -path "*/archived" \) -prune -o -type f -name "*.md" -print 2>/dev/null | xargs -I {} wc -c -l "{}" 2>/dev/null
```

Discovery is path-based — no fixed file list. New `CLAUDE.md` or skill files are picked up automatically.

**Surface classes** (bytes primary — assistant context reinjection is byte-bounded; lines secondary):

| Surface class | Path pattern | 🟢 OK | 🟡 Watch | 🔴 Hard |
|---|---|---|---|---|
| Vault `CLAUDE.md` | `*/CLAUDE.md` | ≤4KB | 4–5KB | >5KB |
| Vault `CURRENT_STATE.md` | `*/CURRENT_STATE.md` | ≤80 lines | 80–100 lines | >100 lines |
| Skill files | `.claude/commands/*.md` | ≤25KB | 25–30KB | >30KB |
| Active design specs | `_shared/*_DESIGN.md`, `_shared/*_SPEC.md`, `_shared/*_PATTERN.md` | ≤40KB | 40–60KB | >60KB |
| Reference / append-only | `_shared/{CHANGELOG,TECHNIQUES,EVALUATION_LOG,RESOURCES,TECHNICAL_GOTCHAS}.md`, vault learning logs | exempt — entries are append-only by design; growth managed by archive policies, not compression |

Vault content files (trip files, INBOX, project decision logs) are out of scope — they grow legitimately and don't hit reinjection paths.

**Three-tier output:**

```
🔴 Hard (action required):
  - <file> — <metric> (over by <delta>)
🟡 Watch (compress soon):
  - <file> — <metric>
🟢 OK: N files within bounds
```

**Remediation surface** (offered for 🔴 always; for 🟡 `CLAUDE.md` since reinjection cost is real):

Prompt: "Want me to execute [action] for [file] now?"

| Surface class | Prescribed action |
|---|---|
| `CLAUDE.md` | Extract gotchas to `TECHNICAL_GOTCHAS.md`; compress Common Mistakes; verify workflow detail belongs in skill files, not here |
| `CURRENT_STATE.md` | Extract stale Active Contexts to `CHANGELOG`; compress to one-liners pointing to specs; drop content tables that duplicate vault state |
| Skill files | Extract verbose protocol detail to design spec; collapse repeated examples; archive removed-feature sections |
| Design specs | Extract historical decision log to `CHANGELOG`; ensure single source of truth (cross-ref, don't duplicate) |

**Drift-from-schema sub-check (emergency-bloat reversion):**

For each file flagged 🔴 OR 🟡, also check:
1. File has a documented schema (header, callout spec, template structure)
2. Current content has drifted from that schema (extra sections, oversized blocks, duplicated state)
3. Drifted content has a durable home elsewhere (`CHANGELOG`, vault `CURRENT_STATE`, learning log, etc.)

If all three — propose verify-durable-elsewhere pass before compressing. Every "loss" must be checked against its durable home before deletion.

**Forward-looking rule:** any new structural emergency response should land with reversion criteria documented inline ("revert when X durability metric reaches Y"). Without that, expect to re-detect this drift in a future `/check`.

### 6. Recurrence Tag Audit
- List open reminders from your task system (Strategic + Personal lists)
- Scan reminder bodies for `[recur:]` tags
- Check well-formedness per grammar in `_shared/TRACKING_SYSTEM_DESIGN.md` → Body-Tag Conventions
- Flag: malformed tags, missing closing bracket, non-numeric intervals, both native + `[recur:]` (mutual exclusion violation)

### 7. Completed Reminder Cleanup
- Count completed items in Strategic and Personal lists
- If any list has >20 completed items, flag for cleanup
- Cleanup = batch-clear completed items older than 30 days (with user approval)

## Semantic Health Checks

### 8. Active Context Freshness
- For each Active Context in `CURRENT_STATE.md`: is it still active?
- Flag contexts that haven't been touched in 3+ sessions
- Flag contexts where the linked reminder is already completed
- Suggest: drop, compress to one-liner, or keep with rationale

### 9. Staleness Scan
- Any pending evaluations in `EVALUATION_LOG.md`?
- Any techniques in `TECHNIQUES.md` with `Applied: Not applied` for 30+ days?
- Any vault `CLAUDE.md` files with `last_updated` older than 60 days?
- Any `CURRENT_STATE.md` `As-of date` older than 14 days?

### 10. Sync Staleness
- Check sync staging/outbox for activity
- If no sync packets produced in >14 days: flag "Sync dormant — cross-machine learning paused"
- Check sync inbox for unprocessed packets
- **Direction semantics:** `~/.sync-packets/inbox/` = inbound (read-only here, other machine writes). `~/.sync-packets/outbox/` = outbound (we write, other machine reads). Pulls only on `/start` or `/sync-review`. Pushes after sync packet approval. Stale inbox files mean either: (a) sender hasn't pushed, or (b) we haven't processed.

### 11. Action Items Freshness
- Flag items with vague or missing context
- Flag items marked "this week" from >7 days ago
- Flag items with no due date present for 3+ sessions

### 12. Pending Merge Sweep

Scan vault files for `## Pending Merge` sections (from Tier 2 meeting-notes routing — see `meeting-notes.md`).

For each file with a `## Pending Merge` section:
1. Count pending entries and find the oldest date
2. **Flag if any entries are 3+ sessions old** — stale pending items risk becoming outdated before merge
3. Report: "N pending items in [file], oldest: [date]"

**Merge execution:**

On a standard `/check` run: report pending items and ask the user whether to merge now or defer.

On `/check merge-pending` or when the user approves during a standard run:
1. For each `→ [Section Name]` tag in the pending entry:
   - Grep the file for that section heading to get its line number
   - Read with offset/limit to get context around the insertion point
   - Insert content at the appropriate point within that section
2. After all items in the entry are merged, remove that dated entry from `## Pending Merge`
3. If `## Pending Merge` is now empty, remove the entire section and its `---` separator

**Merge decision rules:**
- If a pending item's target section no longer exists: flag for manual review — do not silently drop
- If a pending item appears to duplicate existing content: flag rather than merge
- If File Surface Health (Step 5) flags a file as over-target AND that file has pending items: recommend merging in the same pass

### 13. Technique Usage Audit

Beyond metadata completeness (Step 3 checks `Applied:` lines exist), check whether techniques are actually being leveraged:
- For each technique with `Applied: Not applied`: is it relevant to any active vault? Is the system implicitly using it without tracking?
- For each technique with `Applied:` entries: is it still actively used, or has the pattern been superseded?
- Recommend per technique: Apply / Update Applied / Not applicable / Archive

Mechanical completeness ≠ operational usage.

### 14. Sensitive-Data Boundary Scan

If your system tracks any walled-off domain (health/PHI, finance, work/PII):
- List open reminders from your task system
- Scan reminder bodies for restricted-vocabulary markers (medication names, condition names, account numbers, customer/employer identifiers, etc.)
- Flag any reminders that may contain restricted data in body text (should reference vault files instead)
- Spot check, not exhaustive — catches obvious leaks

### 15. Entity Fragmentation Scan

The highest-leverage periodic check. Scan for clusters of related items across lists and vaults that lack a parent entity:
- Search for items sharing a keyword across Strategic + Personal + Left Off + vault files
- **Trigger:** 5+ related items across 2+ surfaces without a shared entity file → consolidate
- Apply the Life Event Consolidation pattern (see `_shared/TECHNIQUES.md`): create entity file → phase-based reminders → fold individuals

### 16. Gap Detection

Cross-reference upcoming deadlines against reminders:
- Items in "Left off" or vault files with deadlines but no reminder?
- Reminders pointing to vault files that have changed?
- Action-reminder pairing gaps?
- Waiting-on items with no escalation path (>14 days without follow-up)?

## Report Format

```
## System Health Check Results

### Registry Drift
- [status] ...

### Version Sync
- [status] ...

### Technique Applied Lines
- [status] ...

### Tag Drift
- [status] ...

### File Surface Health
🔴 Hard: [N files | none]
  - <file> — <metric>
🟡 Watch: [N files | none]
  - <file> — <metric>
🟢 OK: <N> files within bounds
[remediation prompts for flagged files]

### Recurrence Tags
- [status] ...

### Completed Reminder Cleanup
- [status] ...

### Active Context Freshness
- [status] ...

### Staleness Scan
- [status] ...

### Sync Health
- [status] ...

### Action Items Freshness
- [status] ...

### Pending Merge
- [status] ...

### Technique Usage
- [status] ...

### Sensitive-Data Boundary
- [status] ...

### Entity Fragmentation
- [status] ...

### Gap Detection
- [status] ...

### Summary
[X issues found / All clear]
```

## After Running

If issues found:
1. Update `_shared/CURRENT_STATE.md` with findings
2. Suggest remediation actions
3. Offer to fix simple issues

If all clear:
1. Update `_shared/CURRENT_STATE.md` with "No drift detected" + timestamp
