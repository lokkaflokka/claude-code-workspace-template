# Learning System Design

How accumulated knowledge (techniques, insights, positions, themes) compounds across sessions instead of staying as flat lists.

**Pattern:** Follows `TRACKING_SYSTEM_DESIGN.md` structure — Problem → Audit → Design → Scope limits.

---

## Problem Statement

The default state for an evolving knowledge system has 5 structural problems. Each shows up after a few months of operation.

### Problem 1: Flat Storage Kills Connections

A flat `TECHNIQUES.md` with N entries can't show that "Session Initialization Protocol" and "Pre-flight Health Check" are both instances of a broader "defensive session start" pattern. Compound learning doesn't happen.

**What good looks like:** An entry can say `Connects to: T-012, I-003` and a synthesis pass can follow chains to discover compound patterns.

### Problem 2: No Feedback Loop

When a digest surfaces an insight, that insight goes into a learning log as a standalone entry. It never flows back to update interest-profile depth, never triggers a technique relevance scan, never gets checked for staleness.

**What good looks like:** An insight triggers downstream effects — interest profile updates, technique relevance scans, theme evolution tracking.

### Problem 3: Content Doesn't Compound

Each digest starts fresh. There's no mechanism to say "this week's coverage continues a thread from 3 weeks ago" or "your position on X (formed Y) is being challenged by this new evidence."

**What good looks like:** Active themes persist across digests. The system notices when a theme is continuing, evolving, or being challenged. Positions get re-examined when new evidence arrives.

### Problem 4: No Active Application

Techniques exist but have no `Effectiveness:` field and no surfacing mechanism. Application happens only when the user remembers a technique exists and asks about it.

**What good looks like:** A lightweight scan during session init or vault entry that says "T-023 might be relevant here" — exception-only, 0-2 suggestions max, not noisy.

### Problem 5: Manual Cross-Session Continuity

Continuity files (active focus, interest taxonomy) require manual updates. They go stale.

**What good looks like:** Automated proposals for context updates based on observed engagement.

---

## Data Model

### Enhanced Markdown with Stable IDs

The system uses enhanced markdown files with stable identifiers. No database — grep is the query engine. Sufficient at small-to-mid scale and preserves the vault-based storage model.

### ID Schemes

| Entity | Format | Example | File |
|--------|--------|---------|------|
| Technique | `T-{NNN}` | `T-001` | `_shared/TECHNIQUES.md` |
| Insight | `I-{NNN}` | `I-001` | learning log file |
| Position | `P-{NNN}` | `P-001` | learning log file |
| Theme | `TH-{NNN}` | `TH-001` | active themes file |

IDs are permanent. Removed entries get a tombstone (`Status: Archived` or `Status: Superseded by T-045`) rather than deletion. Preserves cross-reference integrity.

**ID assignment:** Sequential within each type. New entries get the next available number. IDs are never reused.

### Technique Entry (Enhanced)

```markdown
### Session Initialization Protocol
<!-- T-001 -->

**Type:** Project
**Origin:** finance/
**Added:** YYYY-MM-DD

#### What it is
[content]

#### Why it works
[content]

#### Example
[content]

#### Applicability notes
[content]

**Applied:** finance (origin), travel (date), career (date), _shared (date).

**Effectiveness:** [Added by Reflector] Strong in vaults with time-sensitive state. Overhead not justified in static reference vaults.
**Connects to:** T-015 (Pre-flight Health Check), T-054 (Structural Enforcement Over Documentation)
```

New fields beyond the basic technique format:
- `Effectiveness:` — qualitative assessment, added/updated by the Reflector protocol
- `Connects to:` — cross-references to related techniques

### Insight Entry (Enhanced)

```markdown
- I-007 | YYYY-MM-DD | **<Insight Title>** (Source)
  Status: Active
  Summary: <one-line summary>
  Connects to: I-005, TH-002, P-001
  Vault relevance: <vault names>
```

Fields:
- `I-{NNN}` ID prefix
- `Status:` — `Active` (current), `Integrated` (absorbed into practice/positions), `Superseded`, `Archived`
- `Connects to:` — cross-references
- `Vault relevance:` — which vaults this insight maps to (used by routing)

### Position Entry

```markdown
- P-001 | YYYY-MM-DD | **<Position Name>**
  Position: <one-sentence statement>
  Evidence: I-005, I-007
  Review date: YYYY-MM-DD
  Status: Active
  Connects to: TH-002, I-005, I-007
```

**Promotion criteria** (an insight becomes a position candidate when any hold):
- The user has cited or applied the insight in 2+ distinct contexts
- The insight has been stable for 30+ days without contradicting evidence
- The user explicitly states a stance ("I believe X because Y")

Positions carry: evidence chain, review date (when to re-examine — Challenger scheduling), status tracking.

### Theme Entry (`Active Themes.md`)

```markdown
## TH-001: <Theme Name>
**Status:** Active
**First seen:** YYYY-MM-DD (digest #N)
**Last seen:** YYYY-MM-DD (digest #N)
**Appearances:** N/N digests
**Trajectory:** Strengthening | Stable | Weakening

### Thread
- YYYY-MM-DD: <event/insight>
- YYYY-MM-DD: <event/insight>

### Connected Insights
I-005, I-007, I-009

### Connected Positions
P-001 (<Position Name>)

### Vault Routing
- finance: <relevance to finance vault>
- career: <relevance to career vault>
```

### Cross-Reference Lifecycle

**Creation:** The Synthesizer proposes cross-references during synthesis passes. User confirms or rejects. Manual linking is also supported at any time.

**Maintenance:** IDs are permanent. When an entry is archived or superseded, existing cross-references remain valid — the target entry's `Status:` field tells the reader it's no longer current. No broken links.

**Querying:** `grep -r "T-012"` across vault files returns all references. Simple, fast, transparent.

**Compound learning:** During synthesis, follow cross-reference chains. If I-007 connects to TH-002 which connects to P-001, the Synthesizer can ask: "Does this new evidence strengthen or challenge P-001?" This is how insights compound.

### Graph DB Trigger Conditions

Enhanced markdown + grep is the right tool **at small scale**. Revisit when any of these trigger:

| Trigger | Threshold | Signal |
|---------|-----------|--------|
| Node count | 200+ combined entities | Manual ID management becomes burdensome |
| Multi-hop queries | Frequent 3+ hop traversals | grep can't follow A→B→C→D efficiently |
| Retrieval degradation | grep takes >2s or returns >50 results | Volume exceeds flat-file query capability |
| Synthesis bottleneck | Corpus exceeds single-pass synthesis | Need indexed retrieval for partial reads |

Until then, the overhead of a graph DB (setup, sync, query language, another tool to maintain) exceeds its benefits.

---

## Agent Protocols

Four protocols organized into two loops.

```
┌─────────────────────────────────────────────────┐
│  GLOBAL OUTER LOOP                               │
│  Cross-domain synthesis + bias checking          │
│                                                  │
│  ┌─────────────┐      ┌──────────────┐          │
│  │ Synthesizer  │      │  Challenger   │          │
│  │ (Connector)  │◄────►│  (Skeptic)    │          │
│  └──────┬───────┘      └──────────────┘          │
│         │                                        │
│    reads from / writes to                        │
│         │                                        │
│  ┌──────▼───────────────────────────────────┐    │
│  │  VAULT-SCOPED INNER LOOP                  │    │
│  │  Domain-expert application + reflection   │    │
│  │                                           │    │
│  │  ┌────────────┐    ┌────────────┐        │    │
│  │  │  Advisor    │    │  Reflector  │        │    │
│  │  │(Practitioner│    │ (Observer)  │        │    │
│  │  └────────────┘    └────────────┘        │    │
│  └───────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

**Rationale for the split:**
- Advisor and Reflector benefit from deep domain context. A finance Advisor should know finance techniques, finance goals, finance state. Vault-scoping makes them experts.
- Synthesizer and Challenger must be global to find cross-domain connections and prevent echo chambers. A vault-scoped Synthesizer would only see patterns within its domain — exactly the opposite of its purpose.
- **Bias mitigation:** The global Synthesizer explicitly bridges vault-scoped insights. The Challenger checks whether vault-scoped Advisors are reinforcing assumptions rather than surfacing genuine relevance.

### Protocol 1: Synthesizer ("The Connector")

**Persona:** Thinks in graphs. Looks for non-obvious relationships between ideas, techniques, and themes across domains. Biased toward connection — will sometimes see relationships that aren't there, which is a feature (generate candidates) not a bug (as long as the user confirms).

**Scope:** Global — operates across all vaults and the full knowledge corpus.

**Triggers:**
| Trigger | Context | Priority |
|---------|---------|----------|
| Post-digest | After weekly briefing synthesis | Primary — richest input |
| Explicit request | User asks for synthesis pass | On-demand |
| Quarterly review | Scheduled deep synthesis | Comprehensive |

**Operations:**

**Minimum viable run** (post-digest):
1. **Theme continuity check** (~2 min)
   - Read active themes file
   - For each theme: does this digest contain related content? Update `Last seen`, `Appearances`, `Trajectory`
   - For new clusters: propose new theme entry (user confirms)
2. **Cross-reference proposals** (~3 min)
   - For each new insight/theme: scan existing corpus for connections
   - Propose `Connects to:` additions (user confirms before writing)
   - Follow existing chains to surface compound patterns

**Verification:** After minimum run, update header: `Last Synthesizer run: YYYY-MM-DD`. Makes execution visible and lets the Advisor detect stale synthesis.

**Full run** (when session time permits):
3. **Insight promotion** — scan for insights meeting promotion criteria; write to learning log with proposed cross-references
4. **Staleness detection** — themes not seen in 4+ weeks → flag as potentially stale; positions older than review date → flag for Challenger
5. **Vault-aware routing** — for each new insight/theme: check vault learning contexts; stage insights for relevant vaults

### Protocol 2: Advisor ("The Practitioner")

**Persona:** Knows the technique catalog deeply. Biased toward application — wants to connect techniques to real work. Understands that suggesting too many techniques is worse than suggesting none (noise kills trust).

**Scope:** Vault-scoped — each invocation carries the context of whichever vault is active.

**Triggers:**
| Trigger | Context | Priority |
|---------|---------|----------|
| Session init | Lightweight scan during init | Background — non-blocking |
| Vault entry | First time touching a vault mid-session | Domain activation |
| Problem-pattern match | During active work, recognizes a technique-applicable pattern | Reactive (lowest priority) |

**Knowledge selection mechanism:**

Rather than loading the entire technique catalog for every invocation, the Advisor uses a tiered relevance filter:
1. **Vault learning context** — goals, active contexts, declared technique relevance
2. **Recently applied techniques** — techniques whose `Applied:` line includes this vault
3. **Active theme connections** — techniques connected to currently active themes
4. **Full catalog scan** — only if tiers 1-3 return <3 candidates and the problem context is novel

**Output (exception-only):**
- 0-2 suggestions max
- Format: "T-023 (Evidence-First Scoring) might apply here — [one sentence why]"
- If 0 suggestions: no output at all

**Writes:** Nothing directly — suggestions are conversational. The Reflector tracks whether suggestions were applied.

### Protocol 3: Reflector ("The Observer")

**Persona:** Meta-cognitive. Notices process patterns — what's working, what's not, what keeps recurring. Interested in the delta between "what we said we'd do" and "what we actually did." Not judgmental — observational.

**Scope:** Vault-scoped — reflects on practices within the active vault's domain.

**Triggers:**
| Trigger | Context | Priority |
|---------|---------|----------|
| Session-end | Single reflection question in `/end` skill | Primary |
| Explicit request | User asks for reflection | On-demand |

**Operations:**
1. **Technique application review** — was a technique suggested by the Advisor this session? Was it applied? Useful? Update `Effectiveness:` field in `TECHNIQUES.md`.
2. **Learning capture** — did this session produce an insight worth persisting? Propose learning log entry with appropriate connections. Check: is this genuinely new, or a restatement of an existing insight?
3. **Position check** — did this session's work reinforce or challenge an existing position? If challenge: flag for Synthesizer's next staleness pass.
4. **Pattern detection** — did the user's behavior reveal a preference or pattern? Examples: "User consistently reformats tables to X style" → technique candidate.

**Writes:** `TECHNIQUES.md` (Effectiveness updates), learning log (new entries, status updates).

### Protocol 4: Challenger ("The Skeptic")

**Persona:** Questions whether positions still hold. Checks for confirmation bias — are we only seeing evidence that supports existing views? Constructive, not adversarial.

**Scope:** Global — must cross vault boundaries to find contradictory evidence.

**Implementation:** As `/revisit` skill. Integrated with `/start` (position review date check) and `/end` (review date persistence, learning activation cache).

**Build trigger:** 4+ Positions exist in learning log. First review: 30 days from formation.

**Scheduling — spaced repetition:**

Positions are reviewed at increasing intervals unless challenged:
- First review: 30 days after formation
- Second review: 90 days
- Third review: 180 days
- Ongoing: annually

If new evidence challenges a position, the review date resets to 30 days from the challenge.

**This is spaced repetition applied to beliefs, not facts.** Anki asks "do you still remember this?" The Challenger asks "do you still believe this, and should you?"

**Operations:**
1. **Position review** — surface position + evidence chain + contradictory evidence accumulated since last review. Ask: "Does this position still hold? Update / strengthen / retire?"
2. **Confirmation bias scan** — check recent digests: are we only surfacing evidence that confirms existing positions?
3. **Position evolution** — updated positions get new evidence chain entries; retired positions get `Status: Retired` with rationale.

---

## Vault Interaction Model

The learning system is vault-aware.

### The Core Insight: File Access IS the Router

The user doesn't start separate "vault sessions." Work happens at the workspace level and the assistant's file access patterns determine which vaults are active.

- **Insights don't get pushed to vaults in real-time.** No "vault inbox" demanding attention.
- **Insights get staged and pulled contextually.** When a task touches finance files, finance-relevant staged insights become available.
- **Existing orchestration — which files get read, which state files get loaded — is the real-time signal** for what's relevant right now.

### Learning Context

Each vault can optionally maintain a `_learning_context` section in its `CLAUDE.md`. This tells the learning system what the vault "cares about."

```yaml
# In vault CLAUDE.md _context block:
_learning_context:
  last_reviewed: YYYY-MM-DD             # Staleness signal
  goals:
    - "Goal 1"
    - "Goal 2"
  active_themes: [TH-001, TH-004]       # Themes this vault is tracking
  relevant_techniques: [T-001, T-003]   # Techniques known to apply here
  expertise_areas:                       # What this vault's Advisor should be expert in
    - <area 1>
    - <area 2>
  learning_gaps:                         # What this vault needs to learn
    - <gap 1>
```

**Design properties:**
- **Optional.** Vaults without `_learning_context` still participate — Synthesizer routes based on file content analysis. Optimization, not requirement.
- **Human-maintained with machine proposals.** The Synthesizer can propose updates; user confirms.
- **Lightweight.** 5-10 lines.
- **Staleness-aware.** `last_reviewed` field bumps when human confirms. Session init flags contexts not reviewed in 30+ days.

### Routing Mechanics

```
Synthesizer identifies vault-relevant insight
         │
         ▼
ROUTING STAGE (one batch file per synthesis run)
   <staged-insights>/YYYY-MM-DD-routed.md
   ## For finance
   - I-007: <insight>
     Relevance: <why>
     Action: <suggested>
         │
         ▼
File access by the assistant during normal work
         │
         ▼
Advisor (vault-scoped) surfaces relevant staged insights
         │
         ▼
User acts → Reflector notes → feedback loop closes
```

**Key design decisions:**
1. **Staging, not direct writes.** Synthesizer doesn't write to vault files directly. It stages recommendations.
2. **Batch files, not per-vault inboxes.** One dated file per synthesis run. Simple to scan and clean up.
3. **File access IS the pull signal.** When a session touches a vault, that's the signal that vault context is active.
4. **Staged insights expire.** If a staged insight hasn't been acted on after 3 synthesis runs, the Synthesizer marks it stale.

### Bi-Directional Flow

**Capture direction (vault → learning system):**
```
Work in vault produces insight → Reflector proposes log entry → Synthesizer connects to themes → Cross-references link to other vaults
```

**Application direction (learning system → vault):**
```
Synthesis identifies vault-relevant insight → Staged for routing → Advisor surfaces when vault is active → User applies → Reflector notes effectiveness
```

The application direction is the harder one to bootstrap.

### Vault Onboarding

When a new vault is created:
1. Vault `CLAUDE.md` includes `_context` block (already standard).
2. Add `_learning_context` section — goals, initial relevant techniques, expertise areas. Minimal at creation; grows organically.
3. Update `PROJECTS.md` registry.
4. First Synthesizer pass after vault creation picks it up from the registry.

The system doesn't need to be "fully warmed up" to be useful. Cold-start is handled by the global protocols.

---

## Persistence Model

| Knowledge Type | File | Created By | Read By | Update Trigger |
|----------------|------|------------|---------|----------------|
| Techniques | `_shared/TECHNIQUES.md` | Manual + Reflector | Advisor, Synthesizer | Discovery, effectiveness update |
| Insights | learning log | Synthesizer + Manual | All protocols | Post-digest synthesis, session reflection |
| Positions | learning log | Manual (promoted from insights) | Challenger, Synthesizer | Position formation, review date |
| Themes | active themes file | Synthesizer | Synthesizer, Advisor | Post-digest, theme evolution |
| Staged routing | `staged-insights/` | Synthesizer | Advisor, Manual | Post-synthesis routing |
| Vault learning context | vault `CLAUDE.md` | Manual + Synthesizer proposals | Advisor, Synthesizer | Goal/context changes |

---

## Feedback Mechanisms

Three feedback loops, each closing a different gap:

### Loop 1: Synthesis Write-Back (closes Problems 1, 3)

```
Weekly digest
  → Synthesizer: theme updates, insight promotion, cross-references
  → Active Themes updated
  → Learning Log entries with connections
  → Vault-routed staged insights
```

**Frequency:** Weekly (post-digest), plus on-demand.

### Loop 2: Technique Effectiveness (closes Problem 4)

```
Advisor suggests technique
  → User applies (or doesn't)
  → Reflector notes outcome
  → TECHNIQUES.md Effectiveness field updated
  → Advisor uses effectiveness data for future suggestions
```

**Frequency:** Per-session (via Reflector's session-end step).

### Loop 3: Position Freshness (closes Problem 2)

```
Position formed (from promoted insights)
  → Review date set (spaced repetition schedule)
  → Synthesizer monitors for contradictory evidence
  → At review date: Challenger surfaces position for re-examination
  → Position updated, strengthened, or retired
```

**Frequency:** Per-position schedule (30d → 90d → 180d → annual).

---

## Background Processing

Roadmapped capabilities for asynchronous/automated support.

### Staleness Monitoring (Gist + cron pattern)

Externalize theme dates to a public gist (or equivalent monitoring surface). A scheduled job (cron, n8n, GitHub Actions) reads the gist daily, runs date math, sends a notification when:

- Any active theme with `last_seen` >28 days ago → "Theme TH-003 hasn't appeared in 5 weeks"
- `last_digest` >9 days ago → "No autonomous digest has run in N days — check pipeline"
- `last_run` >14 days ago → "No synthesis routing has run in N days — check the queue"

Field split rationale: original single `last_run` conflated "pipeline fired" with "user engaged with output." Failing pipeline and backlogged queue are different failure modes with different thresholds.

### Background Subagent Synthesis

After digest completion, spawn background subagent to run Synthesizer operations. Subagent writes results to staged-insights/ and active themes file. User reviews at next session start. Subagents can read/write files, which is all the Synthesizer needs (no MCP access required).

---

## Skill Integration

How protocols map to actual skills:

| Protocol | Skill |
|----------|-------|
| **Synthesizer** | Background subagent post-digest + `/route` for execution |
| **Advisor** | Init parallel batch + `/end` coverage check |
| **Reflector** | `/end` Phase 0 (session scan) + Phase 3 (verify) |
| **Challenger** | `/revisit` skill |

Behavioral instructions in `CLAUDE.md` files are insufficient for high-frequency protocols — they get skipped under context pressure. Move to skill form (structural enforcement) once the protocol's behavior stabilizes.

---

## Phased Implementation

### Phase 0: Foundation (1 session)
- Add IDs to existing artifacts (`T-{NNN}`, `I-{NNN}`)
- Create active themes file
- Create `staged-insights/` directory

### Phase 1: Synthesizer + Vault Routing (2-3 sessions)
- Synthesizer behavioral instructions in digest workflow
- Implement vault learning context: add `_learning_context` to vault `CLAUDE.md` files
- Run first Synthesizer pass on existing digests to seed themes and cross-references

### Phase 2: Advisor + Reflector (2 sessions)
- Advisor instructions in init parallel batch
- Reflector as `/end` skill
- First effectiveness annotations on actively-used techniques
- Set up staleness monitoring

### Phase 3: Organic Accumulation (ongoing)
- Cross-references accumulate through normal Synthesizer runs
- Theme tracking matures (dormant/retired themes emerge)
- Effectiveness data accumulates on techniques

### Phase 4: Challenger (when 4+ Positions exist)
- Implement Challenger as `/revisit` skill
- Set up spaced-repetition review schedule
- `/start` integration: position review date check
- `/end` integration: review date persistence, learning activation cache

---

## Scope Boundaries

| Scoped Out | Why | Revisit When |
|------------|-----|-------------|
| Graph DB | Enhanced markdown + grep sufficient at small scale | 200+ nodes, frequent multi-hop queries, grep >2s |
| Automated cross-references | Manual linking + Synthesizer proposals sufficient | Manual linking >5 min per synthesis |
| Per-vault learning stores | Centralized store with vault-aware routing avoids duplication | Vault-aware routing proves insufficient |
| Real-time routing | Staged batch routing matches actual usage patterns | User starts vault-specific sessions where real-time matters |
| Automated context updates | Proposals require user confirmation | Proposal acceptance rate >90% over 3 months |
| NLP-based connection detection | Keyword/ID-based grep sufficient | Corpus exceeds keyword matching capability |

---

## Error Model and Regression Triggers

### Error/Degradation Handling

| Failure Mode | Detection | Recovery |
|-------------|-----------|----------|
| Bad cross-references | User rejects during confirmation | Unconfirmed links never written |
| Misrouted staged insight | Insight expires after 3 synthesis cycles without action | Passive cleanup |
| Stale active themes | `Last Synthesizer run:` timestamp | Session init flags if >2 weeks stale |
| Stale vault learning context | `last_reviewed:` field | Session init flags if >30 days stale |
| Partial Synthesizer execution | Timestamp only updates after minimum viable run completes | Missing operations accumulate for next full run |

### Regression Triggers (When to Simplify)

These are **scale-down** signals — indicating the system is adding overhead without proportional value:

| Signal | Threshold | Action |
|--------|-----------|--------|
| Active themes not updated | 6+ weeks | Synthesizer not running. Suspend until digest cadence stabilizes. |
| Advisor suggestions ignored | 10+ consecutive sessions | Relevance is poor. Review knowledge selection or suspend. |
| Staged insights expiring without action | 3+ consecutive synthesis cycles | Routing wrong or content not relevant. Simplify to theme tracking. |
| Reflector question skipped | 5+ consecutive sessions | Overhead too high. Reduce to monthly. |
| Cross-references not followed during synthesis | Synthesizer runs but never traverses chains | Connections exist but aren't compounding. Investigate. |

**The principle:** The learning system must earn its maintenance cost. If a component is not producing visible value after its expected ramp-up period, simplify or suspend — don't persist out of sunk cost.

### Reflector-to-Synthesizer Handoff

Clear boundary between vault-scoped observation (Reflector) and cross-domain synthesis (Synthesizer):

1. **Reflector captures domain-specific observations** within the active vault.
2. **When the same observation appears across 2+ vault Reflectors**, the Synthesizer detects the cross-domain pattern on its next run.
3. **The Reflector is the sensor; the Synthesizer is the aggregator.** Reflectors never propose cross-vault connections directly.

---

## Minimum Viable Adoption

For a new system adopting this design:

1. **ID scheme on techniques/insights** — start adding `T-{NNN}` and `I-{NNN}`.
2. **Active themes file** — even with one or two themes, makes the pattern real.
3. **Synthesizer protocol** (simplest version) — theme continuity check + cross-reference proposals after each digest.
4. **Session-end reflection question** — Reflector at its simplest.

Advanced components (Advisor, Challenger, vault routing, familiarity tracking) are opt-in for multi-vault setups with established knowledge bases.
