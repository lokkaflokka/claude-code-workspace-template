---
description: Triage and route captured signals
disable-model-invocation: false
---
# /capture

Structured signal intake — triage, enrich, and route captured information to the right destination.

## When to Use

Use `/capture` when the user says things like:
- "Capture this: [something]"
- "I want to save this for later"
- "Add this to the system"
- "Remember that [X]"
- Or provides any free-text signal that needs to be persisted somewhere

## Workflow

**Note:** This skill is triggered implicitly by Principle #8 ("Content Shared Is Content Captured") when the user shares content mid-session without invoking a skill. Process the user's immediate need first, then route.

### Step 1: Accept Signal

Take the user's free-text input. If the input is vague, ask one clarifying question before proceeding.

**URL Enrichment:** When input contains URLs, attempt to fetch content before triage:
1. Check the URL against known WebFetch failures (`_shared/TECHNICAL_GOTCHAS.md`).
2. If viable: WebFetch → extract title, key facts, summary.
3. If known-failure (auth-walled sites, PDFs, etc.): WebSearch for context instead.
4. If fetch fails: proceed with whatever context is available. Fetch failure does not block routing.

### Step 2: Triage

Determine the signal type. Present your assessment and let the user confirm or override:

| Type | Description | Examples |
|------|-------------|----------|
| **Information** | Context, facts, links, articles to process later | "Interesting article about X", "Learned that Y" |
| **Action** | Something to do — has a verb and a completion state | "Need to call X", "Buy Y before Friday" |
| **Roadmap item** | System/platform improvement idea | "Should add X import", "Y scoring needs Z" |
| **Technique candidate** | Reusable pattern worth evaluating | "This approach worked well", "Found a useful workflow" |
| **Content/Resource** | Article, tool, thread, or resource for deeper evaluation | "Interesting thread about X", "Check out this tool" |
| **Backlog** | Domain-specific idea, not urgent | "Would be nice to track X in vault Y" |
| **Discard** | Not worth persisting | Confirm with user, then skip |

### Step 3: Enrich with Metadata

Add structured metadata appropriate to the type:

**For Information items (→ INBOX.md):**
```markdown
### [Brief title]
- **Captured:** [today's date]
- **Source:** [where it came from — conversation, article, meeting, etc.]
- **Tags:** [relevant domain tags]
- **Priority:** [P1/P2/P3 — P1 = time-sensitive, P2 = important, P3 = nice to have]
- **Routing hint:** [which vault or file this likely belongs in]
- **Context:** [why this matters, what prompted capture]

[Original signal content]
```

**For Action items:** Determine list placement:
- Needs assistant session context → **Strategic**
- Completable offline / physical task → **Personal**
- Include: title (short, imperative), body (steps, context, references to vault files)

**For Roadmap/Technique/Backlog items:** Include enough context that future-self can act without re-researching.

### Step 4: Assess Routing Confidence

Before routing, determine the path for each item:

**Auto-route (no confirmation needed):**
- Single clear vault destination
- Factual/domain content, not strategic
- High routing confidence — you know where it goes
- **Content/Resource type → digest/save-for-later is always auto-route.** Articles, tools, threads — save with title, notes, and tags. No confirmation gate.
- Execute immediately: one-line entry in destination file (date, source, insight)

**Human-review (present for confirmation):**
- Multi-vault relevance or ambiguous destination
- Strategic signal where framing/priority matters
- Content requiring evaluation against active contexts
- Route to `INBOX.md` for session discussion

**Both paths:** A single source can produce auto-routed vault entries (factual content) AND human-review items (strategic signal). Split and route each independently. When content is processed into a purpose-specific artifact (prep file, analysis doc), ALSO save source URLs to digest. Two independent outputs from one source.

### Step 5: Route

| Type | Destination | Method |
|------|-------------|--------|
| **Information** | Vault file (auto-route if clear) or `_shared/INBOX.md` (if ambiguous) | One-line vault append or INBOX markdown block |
| **Action** | Task system (Strategic or Personal list) | Create reminder — present proposed reminder to user first |
| **Content/Resource** | Digest pipeline / save-for-later | Save with title, rich notes, tags. For non-URL sources (Slack, conversations): notes carry the content — write as if URL will 404 |
| **Roadmap item** | `_shared/SYSTEM_ROADMAP.md` → appropriate section | Append to relevant phase/backlog |
| **Technique candidate** | `_shared/PROJECT_IDEAS.md` → Technique Candidates | Append with description |
| **Backlog** | `_shared/PROJECT_IDEAS.md` or domain vault backlog | Append to appropriate section |
| **Discard** | Nowhere | Confirm and skip |

**For work-related signals:** Route to the work-vault `INBOX.md` instead of the personal `_shared/INBOX.md`.

### Step 6: Report

After routing, report to the user:
- **Auto-routed items:** What was placed where (after the fact, not asking permission)
- **Human-review items:** Present for confirmation before executing
- Any follow-up needed (e.g., "This might need a reminder — want me to create one?")

## Dump and Triage Convention

The user provides raw content with minimal context. The assistant does all the work:

1. **User dumps** — pastes text, screenshots, URLs, or a mix. No pre-sorting needed.
2. **Assistant triages** — identifies item boundaries, determines type for each, extracts key arguments/quotes, maps relevance to active contexts.
3. **Assistant presents** — summary table: item, type, proposed destination, 1-line rationale.
4. **User confirms or adjusts** — "looks good" or "move #2 to technique instead."
5. **Assistant routes** — executes all saves/appends/reminders in one pass.

**Input format guidance:** Screenshots, text paste, URLs, meeting notes, or any mix. The system handles whatever the user throws at it — don't push a specific format. Screenshots are often lowest-friction for the user; the assistant extracts content from them fine. Meeting notes are automatically detected and get full extraction + synthesis (see Meeting Content Heuristic).

## Batch Mode

When the user provides multiple signals at once (common with Slack threads, post-meeting dumps, weekly review):

1. **Identify item boundaries** — separate distinct signals from a single paste or multi-item dump.
2. **Triage all items** — determine type for each.
3. **Present routing plan** before executing:

```
[A] Auto-route (2):
1. <thread title> → digest pipeline (rationale)
2. <tool/article> → digest pipeline (rationale)

[B] Review (1):
1. "Check X" (ad-hoc) → task system / Personal? (offline task, confirm list)
```

Omit group headers for simple single-path batches (e.g., all auto-route).

4. **User confirms** — "looks good" or adjusts.
5. **Execute all routing** in parallel where possible.

**Mixed-type items:** A single source can produce multiple items of different types. A thread might yield a Content item (digest) AND an Information item (vault) AND an Action (reminder). Split and route each independently.

### Source Grouping

Between "Identify item boundaries" and "Triage all items":
- Cluster items by source context (meeting, channel, screenshot, ad-hoc).
- Source column appears in routing table when items come from 2+ distinct sources. Omit when all items share a single source context (e.g., one meeting dump with no side-channel).

### Meeting Content Heuristic

When a batch contains potential meeting content, detect before triage:

**Markers** (need 2+ to trigger delegation):
- Template markers (decisions/actions/insights/OQs sections)
- Transcript format
- User meeting-type language ("after the meeting", "we discussed", "1:1 with")
- People + actions pattern (named person → action verb → deliverable)
- Single-timeframe clustering (all content from one bounded time window)

**Threshold:**
- 2+ markers → delegate to `/meeting-notes`
- 1 marker → ask user ("This looks like it might include meeting notes — should I run full extraction+synthesis?")
- 0 markers → standard triage

**Standup override:** If content matches standup pattern (≤5 action items, no decisions section, ≤15 min timeframe), cap at 1 signal regardless of other marker matches. Standups are too thin for full extraction+synthesis.

### Meeting Content Delegation Protocol

When the heuristic triggers delegation:

1. `/capture` runs sensitivity lint once (covers all content including meeting subset).
2. Before delegating: check if meeting context (type, date, attendees) is inferrable from the dump. If not, ask one consolidated context question now — don't let it interrupt mid-delegation.
3. Pass meeting subset to `/meeting-notes` extraction + synthesis steps. Skip lint and context-gather (already done by `/capture`).
4. `/meeting-notes` returns extracted items + synthesis + proposed routing back to `/capture`.
5. Merge into unified routing table alongside capture-triaged items.
6. Meeting-derived items tagged with source meeting.
7. Execution: meeting-notes routing + capture routing in one pass.

Key rule: meeting-notes synthesis always runs — it's the value of the delegation.

## Rules

- **Always confirm type with the user** before routing non-trivial items — don't silently triage strategic signals.
- **Follow Principle #7 (Future Self Can Act):** Enriched items must be self-contained and actionable.
- **Follow Principle #5 (Boundaries):** Work signals → work INBOX. Personal signals → personal system. Sensitive data → vault files only, never in reminders.
- **Don't over-capture:** If the signal is already in a vault file or reminder, point to it instead of duplicating.
- **Mixed-content sessions:** When the user dumps meeting notes + other signals together, process everything in one pass. Meeting content gets full extraction+synthesis; everything else gets standard triage. One table, one confirmation, one execution.

## Examples

**User:** "Capture this: [domain fact about your work]"
→ Type: Information → Route to work vault domain knowledge file

**User:** "Need to check if X happened"
→ Type: Action → Route to task system "Personal" list (offline, doesn't need assistant)

**User:** "The way we structured the design doc worked really well — capture-first, then consolidate"
→ Type: Technique candidate → Route to `_shared/PROJECT_IDEAS.md` → Technique Candidates

**User:** "Add to the roadmap: we should build a Slack connector once approved"
→ Type: Roadmap item → Route to `_shared/SYSTEM_ROADMAP.md` → Backlog (blocked on access)
