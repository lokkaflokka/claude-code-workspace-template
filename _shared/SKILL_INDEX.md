# Skill Index

Authoritative reference for all custom skills in this workspace. Every skill present in `.claude/commands/` should appear here.

---

## Core Skills (Structural — High Frequency)

These run nearly every session. They ARE the system.

| Skill | Purpose | Cadence |
|-------|---------|---------|
| `/start` | Session init: parallel batch, post-processing, exception-only output | Every session |
| `/end` | Session-end persistence: scan, gather, write, verify, sync | Every session |

## Common Skills (Cadenced or On-Demand)

| Skill | Purpose | When to invoke |
|-------|---------|---------------|
| `/capture` | Signal intake: 7-type triage + routing; mixed-content orchestration (delegates to `/meeting-notes`) | User shares URL/screenshot/content |
| `/check` | System health: mechanical + semantic checks. Cadence enforced by `/start` 14-day stale alert | Biweekly or on demand |
| `/meeting-notes` | Post-meeting extraction: structured sections + tiered routing | After a meeting |
| `/sync-review` | Process pending sync packets from another machine | When `/start` surfaces pending packets |

## Specialized Skills (Lower Frequency)

| Skill | Purpose | When to invoke |
|-------|---------|---------------|
| `/challenge` | Decision vetting: persona-based stress-test (Quick or Review Panel) | Decision with stakes |
| `/route` | Batch proposal execution: read staged synthesis output, present approval table, execute writes across vault files | Post-synthesis |
| `/revisit` | Position review: spaced-repetition belief stress-testing. Full audit or single position | Review dates due |
| `/new-project` | Vault/project scaffolding + `PROJECTS.md` registration | Setting up a new vault |
| `/new-technique` | Document a technique candidate (or add inline if heavier-weight) | Pattern worth capturing |

---

## Skill Boundaries (Owns vs. Doesn't Own)

| Skill | Owns | Does NOT Own |
|-------|------|-------------|
| `/start` | Init protocol, completion processing, output format, mode selection | Vault-specific content (reads it, doesn't define it) |
| `/end` | All session-end persistence, sync capture, skill evaluation | Mid-session state updates (just ask naturally) |
| `/capture` | Inbound signal routing (typed), mixed-content orchestration (meeting detection + delegation) | Content analysis (that's the immediate task) |
| `/challenge` | Persona simulation, convergence analysis | Decision-making (presents analysis, user decides) |
| `/check` | Registry, versions, tags, techniques, file sizes, semantic health, entity fragmentation scan, gap detection | Fixing issues (reports + suggests, user approves) |
| `/meeting-notes` | Structured meeting extraction + tiered routing | Meeting scheduling or facilitation |
| `/sync-review` | Process inbound sync packets, archive after disposition | Generating outbound packets (that's `/end` Phase 4) |
| `/route` | Executing staged proposals, batch writes, archiving completed files | Generating proposals (that's the synthesis step) |
| `/revisit` | Position review and evolution | Position formation (that's the digest pipeline) |
| `/new-project` | Directory creation, registration, `CLAUDE.md` scaffolding | Project content |

---

## Native Claude Code (No Custom Skill Needed)

| Operation | How | Why no skill |
|-----------|-----|-------------|
| Commit | Describe changes or `/commit` | Built-in commit flow |
| Test | "Run tests and fix failures" | Natural language sufficient |
| Plan | EnterPlanMode | Built-in plan mode |
| PR | "Create a PR" | Built-in PR flow |
| Multi-repo | Principle #3 + explicit ask | Lightweight, not a protocol |
| State update | "Update CURRENT_STATE.md" | Natural language sufficient |

---

## Skill Candidates

Track patterns surfacing as potential new skills here. Promote to a real skill only after the routing gate passes (see `sync-review.md` → Routing Gate).

| Candidate | Origin | What it does | Status |
|-----------|--------|-------------|--------|
| | | | |

---

## Evaluation Framework

### Design Principles (Empirical)

| Principle | Implication |
|-----------|-------------|
| Trigger at decision points, not info points | Skills should force a pause + decision |
| Output must be actionable artifact | Every skill produces a file, state change, or routing decision |
| Structural triggers beat behavioral | Skills invoked every session (`/start`, `/end`) carry the most value |
| Invocation clarity over speed | Direct invocation, no intermediary tools |
| Redesign via same-session pilot | When redesigning a skill spec — especially one with load-bearing couplings — pilot in same session as design. Theoretical-design-then-defer hides workflow-level assumptions. |

### Collection (Per Session — in `/end` Phase 4)

Captured every session:
- Skills invoked (with modes): [list]
- Effectiveness per skill: [good / neutral / needed adjustment]
- Skills considered but not invoked: [list + why]
- Manual processes that could be skills: [list]

### Contrast (Automatic — in `/end`)

After collection, auto-diff against this index:
- Which skills were invoked?
- Which were NOT invoked?
- Any skill invoked only once in last 5 sessions? → candidate for redesign
- Any skill invoked zero times across 50+ sessions? → candidate for retirement

### Synthesis (Periodic — in `/check`)

Run when 50+ sessions of evaluation data have accumulated:

- **Usage heatmap:** Rank skills by frequency
- **Non-usage diagnostics:** For zero/low skills — wrong trigger? forgotten? not needed?
- **Pattern mining:** From "manual processes" entries — new skill candidates?
- **Design principle validation:** Do high-use skills share traits?
- **Two-tier reality check:** Structural (every session) vs. cadenced/situational. Both are valid; the risk is cadenced skills atrophying (untested, drifting from current patterns).

### Usage Log

Append session-level evaluation data here as `/end` runs. Optional but useful for periodic synthesis.

```
| Session | Date | Mode | Skills used | Effectiveness | Not used (why) | Manual patterns |
|---------|------|------|-------------|---------------|----------------|-----------------|
```

---

## Cross-Machine Considerations

If you maintain separate workspaces (personal + work), some skills may be machine-specific:

- **Both machines:** Core skills (`/start`, `/end`) and most common skills.
- **Machine-specific:** Skills coupled to specific data sources (e.g., a digest pipeline that only runs on the machine with the email integration).

Skill evaluation data from each machine feeds into sync packets (see `SYNC_PROTOCOL.md`). Cross-machine synthesis happens during `/check`.
