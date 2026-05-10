---
description: Process pending sync packets
disable-model-invocation: false
---
# /sync-review

Process pending sync packets from another machine. Invoked manually or when `/start` surfaces pending packets.

**Invoke:** `/sync-review`

## Steps

1. **Pull latest:**
   ```bash
   git -C ~/.sync-packets pull --rebase --quiet
   ```

2. **List packets:** Bash `ls ~/.sync-packets/inbox/*.md` — exclude `.gitkeep`. (Glob does not expand `~`.)

3. **For each packet, read and check `Type:` header:**

   **If `Type: replication`** (or packet has `## Replication Steps`):
   - Read Context and Prerequisites sections first
   - Follow Replication Steps sequentially — each should be executable as-written
   - At the verification step, confirm end-to-end success
   - If steps fail, check Troubleshooting section before escalating
   - Cross-reference open action items — the replication may resolve a blocked item

   **Otherwise (observation — default):**
   - For each **Technique Discovered:** Apply the **Routing Gate** below before touching `_shared/TECHNIQUES.md`. If routing concludes "registry," update or add the entry. If "inline," edit the consuming file directly and do NOT add to `TECHNIQUES.md`.
   - For each **System Improvement:** Evaluate applicability. If relevant, implement or note in `CURRENT_STATE.md`.
   - For each **Skill Learning:** Update `_shared/SKILL_INDEX.md` evaluation data.
   - For each **Shared File Change:** Apply per Shared File Registry rules in `SYNC_PROTOCOL.md`. Surface conflicts for manual resolution.
   - **Action item cross-reference (mandatory):** After reading each packet, scan open action items for any item the packet directly informs — working implementations, solved problems, reference blueprints. Update the action item before moving on.

### Step 3.5 — Per-item Disposition Gate (mandatory before archive)

Before moving the packet to `archive/`, produce a disposition table covering every item in the packet (every Technique, System Improvement, Skill Learning, Shared File Change — not just techniques). One row per item, in order:

| # | Item | Implication HERE (specific surface) | Action |

Action MUST be exactly one of:
- **Edit** — file path + change committed this session
- **Task** — Strategic/Personal reminder created with concrete venue
- **Note** — captured inline somewhere durable (`CURRENT_STATE.md`, archive commit, etc.)
- **Deferred** — must include both VENUE (where it will land) AND REASON (why not now)

**Rejected dispositions:** "already covered," "applicable," "low priority," "noted," "low impact," or any generic label without a named surface. Generic labels are the ritual-traversal failure mode this gate exists to block.

**Self-test before archive:** if you removed every file edit + task addition this session, would the archive produce a diff? If yes, the packet was actually processed. If no, you ritual-traversed — go back and produce real dispositions.

### Routing Gate (mandatory before any `TECHNIQUES.md` edit)

For every candidate technique, answer in order:

1. **Who consumes this?** Name the specific skill files, gotcha entries, or CLAUDE.md sections where the rule would actually fire at a decision point. If you can't name at least one, the technique is not yet actionable — defer.
2. **How many distinct consumers?**
   - **Exactly one consumer** → inline into that consumer file. Do NOT add to `TECHNIQUES.md`.
   - **Two or more consumers across different skills/contexts** → registry entry in `TECHNIQUES.md` is justified.
3. **Has it been applied here?** If the `Applied:` line would read "Not yet applied," that's a smell. Either apply it now (inline at the consumer) or defer the entry until first use. A registry of unapplied techniques calcifies into defensive bloat.

**Optional mechanical enforcement:** A PreToolUse hook can block Edit/Write to `_shared/TECHNIQUES.md` when a new entry is added without a routing-gate marker like `<!-- routing-gate: N consumer(s); decision: registry; rationale: ... -->`. Encodes the rule structurally instead of behaviorally. See your hooks setup.

4. **Archive processed packets:**
   ```bash
   git -C ~/.sync-packets mv inbox/<packet>.md archive/
   ```

5. **Commit and push:**
   ```bash
   git -C ~/.sync-packets add . && git -C ~/.sync-packets commit -m "archive: processed <packet>" && git -C ~/.sync-packets push --quiet 2>/dev/null
   ```

6. **Update `CURRENT_STATE.md`** if structural changes were made.

## Rules

- Process packets one at a time — each may require new technique entries or file edits.
- Archive after processing, not before. The move to `archive/` is the "processed" signal.
- If a packet contains multiple items, process all before archiving.
- Sanitization is the sender's responsibility (pre-commit hook + behavioral). Receiver trusts content.
