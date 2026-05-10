# Signal Capture Pattern

A generalized pipeline for getting ad-hoc information from any surface (Slack thread, email, article, conversation, thought) into the system, routed to the right place, and retrievable later.

## Problem Statement

The job: *"I encounter something — article, Slack thread, email, conversation, thought — and need it in the system."* Without a capture pattern, signal lives in memory or scattered notes and is unretrievable when needed.

A curated newsletter pipeline (see `../content-feed/` if present) solves a narrow version of this — one surface (email), one cadence (weekly batch), one destination (briefing file). Everything else has no capture path. This document generalizes the same architecture to any signal surface.

**Key constraint: tool-agnostic.** Capture must work whether you have a fully-featured assistant (Claude Code with MCP tools), a constrained assistant (browser-only LLM), or no assistant (manual processes). The pattern adapts; the principles don't change.

## The 5-Stage Pipeline

| Stage | What it does | Key question |
|-------|-------------|--------------|
| **Capture** | Signal enters the system from any surface | How does it get in? |
| **Triage** | Is this worth keeping? What type? What priority? | Should I keep this, and what kind of thing is it? |
| **Enrich** | Attach metadata: source, tags, routing hint, context | What is this about? |
| **Route** | Deliver to the right destination | Where does it go? |
| **Retrieve** | Find it later; confirm routing worked | Can I find this again? |

The newsletter pipeline (if you have one) is a parameterized, narrowly-scoped instance of this general pipeline. It has the most mature evaluation infrastructure (scoring, synthesis, feedback loops). The general pipeline extends the same pattern to any signal surface.

## Triage Taxonomy

Not everything captured is the same kind of thing. Triage decides both *whether* to keep something and *what kind of thing it is* — the type determines the destination.

| Type | Definition | Destination | Example |
|------|-----------|-------------|---------|
| **Information** | Facts, insights, knowledge to file for later retrieval | INBOX.md → route to vault file | "X migration target is Q2" |
| **Action** | Something with a deadline or concrete next step | Reminders/task system (session-context vs offline list) | "Send proposal by Friday" |
| **Roadmap item** | Project-level change, feature request, strategic task | `SYSTEM_ROADMAP.md` or vault-level roadmap | "Add Y connector when Y confirmed primary" |
| **Technique candidate** | A pattern, workflow, or method to evaluate for adoption | `TECHNIQUES.md` → `EVALUATION_LOG.md` | "Approach Z — could reduce overhead" |
| **Content/Resource** | Article, tool, or resource for deeper evaluation | Newsletter pipeline (if it fits) OR batch resource review | "Article — evaluate for architecture patterns" |
| **Backlog** | Undated someday/maybe item without urgency | Vault backlog files | "Try X sometime" |
| **Discard** | Not worth keeping — noise, already known, too vague | Nowhere | "Interesting but no relevance" |

**The first fork at triage:** "What kind of thing is this?" The answer determines the rest of the flow.

## Routing Confidence — Two Paths

Not all captured items need the same level of human involvement.

**Auto-route (no human gate):** High-signal items with clear vault affinity. Assistant triages, enriches, deposits directly into the destination. The user sees what was auto-routed via session-init readout — not a confirmation prompt before each action.

- **When:** Single clear vault destination, factual/domain content (not strategic), routing confidence is high.
- **Examples:** Domain fact → vault knowledge file. Tool discovery → vault backlog. Known-pattern technique → `TECHNIQUES.md`.
- **Key rule:** Auto-routed items get a one-line entry with date, source, and insight. Not a full analysis — that's over-enrichment for auto-route.

**Human-review (digest or `INBOX.md`):** Strategic signal, ambiguous routing, multi-vault relevance, or items where the "so what" matters more than the "what."

- **When:** Multi-vault relevance, strategic frameworks, items requiring evaluation against active contexts, or items where user judgment on framing/priority matters.

**Items can be both.** A single source may produce an auto-routed vault entry (the factual content) AND a human-review item (the strategic signal). Start conservative on auto-routing; expand as accuracy proves out.

## Staging Area: INBOX.md

A single file per context (personal, work) where captured items land before routing.

**Location:**
- Personal: `_shared/INBOX.md`
- Work: `{work-vault}/INBOX.md`

**Format:**

```markdown
# INBOX

Items captured but not yet routed. Review during session init.

---

## Unrouted Items

### [Short descriptive title]
- **Captured:** YYYY-MM-DD
- **Source:** slack/channel-name
- **Tags:** architecture, data-pipeline
- **Priority:** high
- **Routing hint:** → vault-name/FILE_NAME.md
- **Context:** Why this matters right now. What problem it relates to. What changes if we know this.

---

### [Next item title]
...
```

**Rules:**
- Items appended chronologically (newest at bottom within Unrouted).
- Each item is a self-contained block with all metadata.
- Routed items are deleted from `INBOX.md` (they live at their destination).
- If `INBOX.md` has >10 unrouted items, something is wrong — either routing is too slow or capture threshold is too low.
- Session init reads `INBOX.md` and proposes routing for pending items.

## Metadata Schema

Every captured item carries this metadata, in `INBOX.md` or at its final destination:

```yaml
captured: YYYY-MM-DD            # When captured (ISO date)
source: slack/engineering        # Where it came from (surface/channel)
tags: [architecture, migration]  # Topic tags for retrieval
priority: high | medium | low    # Triage assessment
routing_hint: vault-name/file    # Suggested destination (may be overridden)
context: |                       # Why this matters right now
  Connects to active stream X.
  Bears on decision Y.
```

**Source format:** `{surface}/{channel}` — e.g., `slack/engineering`, `email/manager`, `meeting/1on1`, `web/article`, `thought/walking`, `voice/siri`.

**Tags:** Free-form but convergent. Use existing vault section names when possible. New tags emerge naturally; prune quarterly.

**Priority:**
- **High:** Time-sensitive or blocks other work. Route within the session.
- **Medium:** Valuable but not urgent. Route within the week.
- **Low:** Interesting but speculative. Route when convenient or drop after 2 weeks.

## Routing Rules

Manual-first with routing hint suggestions. The routing table maps tags and sources to likely destinations. Personalize per workflow.

**Example personal routing (adapt to your vaults):**

| Signal Pattern | Destination |
|----------------|-------------|
| Tags: portfolio, investment, tax | `finance/` vault |
| Tags: career, skills | `career/` vault |
| Tags: travel, booking | `travel/` vault |
| Tags: technique, pattern | `_shared/TECHNIQUES.md` |
| Tags: tool, setup, config | `_shared/` or vault-specific |
| Source: voice/* or thought/* | Often stays in `INBOX.md` for triage |

**Example work routing (adapt per employer):**

| Signal Pattern | Destination |
|----------------|-------------|
| Tags: architecture, system | Architecture notes file |
| Tags: stakeholder, relationship | Relationships file |
| Tags: decision | Decision log |
| Tags: learning, domain | Learning log |
| Tags: process, workflow | Operating system / runbook file |

**Routing is a suggestion, not automation.** The human confirms or overrides. Over time, if accuracy is consistently high for a tag pattern, that pattern becomes a candidate for auto-routing.

## Retrieval Design

Three retrieval paths, each serving a different need:

**1. Search-based (grep across vaults)** — for "I know I captured something about X":
```bash
grep -r "search term" ~/Projects/
```
Plain text in known locations. No special infrastructure needed.

**2. Time-based (`INBOX.md` is chronological)** — for "what did I capture this week":
`INBOX.md` entries are chronological. Routed items carry their `captured` date at their destination.

**3. Vault-linked (routed items live in their destination)** — for "what do I know about X":
Routed items land in the file matching their topic. Reading the destination file gives all captured insights about that domain. Session init reads these files, so captured knowledge surfaces automatically when working in that domain.

**What this deliberately skips:** Full-text search indexing, semantic search, cross-vault search UI. Grep works. Revisit with a search tool only if retrieval becomes a bottleneck.

## Capture Surfaces

Each surface needs a concrete spec: what triggers capture, what the input looks like, what the output looks like in `INBOX.md`.

### Assistant Session

**Trigger:** User says "capture this" or session produces an insight worth preserving.

**Input:** Free text in conversation — a realization, decision rationale, useful fact.

**Implementation:** Convention-based — when the user says "capture this," append a formatted block to `INBOX.md`. Could be formalized as a `/capture` skill (structured input → formatted append).

### Slack Threads / DMs

**Manual input:** Copy-paste relevant message(s) into a session, or screenshot.

**Tips:**
- Screenshots are first-class input. Cmd+Shift+4 → paste is often the lowest-friction capture. Assistants handle screenshot extraction well — participants, timestamps, arguments, quotes, links all come through.
- Text copy-paste also works and preserves clickable links. Use whichever is faster in the moment.
- Don't optimize for assistant comfort over user's capture friction.

**Output depends on triage type:** Content/resource (most common for industry signal) → digest/save-for-later. Information (domain knowledge) → `INBOX.md` → route to vault file. A thread can produce both if it has both current-relevance signal and durable knowledge.

**Future tooling:** Slack MCP connector — `conversations.history`, `search.messages`. Build when access is granted; manual paste works in the meantime.

### Email

**Manual input:** Copy-paste relevant email content into a session.

**Future tooling:** Gmail/IMAP MCP connector — reuse newsletter pipeline auth if it exists. Build when non-newsletter email capture is regular (>1×/week); otherwise manual paste is fine.

### Meeting Notes

**Capture template:**
```
Meeting: [name/type]
Date: [date]
Attendees: [who]
Key decisions: [what was decided]
Action items: [who owes what by when]
Insights: [what I learned that I didn't know before]
Open questions: [what's still unclear]
```

**Output:** Each row routes by triage type — decisions to a decision log, action items to reminders, insights to a learning log, open questions to a follow-up file.

### Web Links / Articles

**Input:** Paste URL into session. Optionally use a fetch tool for summary extraction.

**Output:** Standard `INBOX.md` block with URL in the context field.

### Notion / Docs

**Manual input:** Copy-paste relevant content. Source: `notion/page-name` or `gdoc/doc-name`.

**Future tooling:** Notion API MCP — `pages.retrieve`, `databases.query`. Build when Notion is the primary documentation tool for a context.

### Mobile / Away-from-Desk

**Design principle:** Don't prescribe the capture mechanism — prescribe the **processing** mechanism. People already have habits for quick-capturing (Siri, notes app, text themselves, jot on paper). Imposing a specific tool just adds friction. Use whatever capture method feels natural; process at the next session.

**Capture — any of these work:**
- Voice assistant ("Hey Siri, remind me to X" → lands in default reminders list)
- Notes app quick note from phone
- Text/email yourself
- Write on paper
- Voice memo

**Processing — this is the structured part:**

Session init includes the prompt: *"Anything to capture since last session?"* The user brings whatever they have — reads off their phone, checks their notes, recalls from memory. The assistant handles triage → enrich → route per the pipeline.

**Why this works better than a strict single-list flow:**
- Zero new habits to learn.
- No single point of failure — if voice misroutes, it still gets processed at session start.
- Works identically across runtimes (work vs personal).

## Implementation Playbooks

### Playbook A: Full-Featured Assistant (Preferred)

**Setup:**
- `INBOX.md` lives in the vault (personal: `_shared/INBOX.md`, work: `{work-vault}/INBOX.md`)
- `/capture` skill handles triage + enrichment + append
- Session init reads `INBOX.md`, proposes routing for unrouted items
- MCP tools for Slack/email/Notion when available

**Capture flow:**
1. User encounters signal → says "capture this" or `/capture`
2. Assistant triages, enriches, appends to `INBOX.md`
3. At session start, assistant reads `INBOX.md` → "You have 3 unrouted items. [Item 1] looks like it belongs in vault X. Route it?"
4. User confirms or overrides → assistant moves content to destination, removes from `INBOX.md`

**Advantages:** File persistence, session context, MCP connectors, formalizable skill patterns.

### Playbook B: Constrained Assistant / No File Access

Fallback when no file persistence is available.

**Setup:**
- INBOX lives in a shared doc accessible to both the assistant and the user
- Capture is copy-paste into the conversation + manual append to the shared doc
- Routing is fully manual

**Work INBOX location options:**

| Option | Pros | Cons | Choose when |
|--------|------|------|-------------|
| Shared doc (Google Doc, Confluence) | Easy create/share/search | No structured metadata; manual formatting | Doc-suite is primary; simple is enough |
| Notion page | Structured fields possible via DB | Requires Notion access; more setup | Notion is primary docs tool |
| Notes app | Already synced to phone; zero setup | Not readable by work assistant; no metadata | Tooling unclear; need immediate capture |
| Local markdown | Same format as personal INBOX; greppable | Requires file access from work machine | Full assistant approved (upgrade to Playbook A) |

**Recommendation:** Start with the simplest option accessible from both the work assistant and your phone. Migrate to local markdown when access permits.

**Limitations:** No session persistence, no automated routing, no MCP connectors, higher friction at every stage.

**Mitigation:** Keep the INBOX doc simple. Use the metadata schema so items are consistently structured even without automation. The key value is the *habit* of capturing — even a plain doc with timestamped entries beats nothing.

### Playbook C: Hybrid (Most Likely)

Personal system runs the full assistant. Work system runs whatever's available.

**Setup:**
- Personal: Full Playbook A — `_shared/INBOX.md`, `/capture` skill, session init reads INBOX
- Work: Playbook B until full assistant approved, then upgrade to A
- Cross-context sync: manual at session boundaries

## Cross-Context Flows

Capture happens across four quadrants. Each needs a path from signal to destination.

### Context Model

| | Belongs in personal vault | Belongs in work vault |
|---|---|---|
| **Originates at work** | Work → Personal | Work → Work |
| **Originates in personal life** | Personal → Personal | Personal → Work |

### Quadrant Details

**Personal → Personal (most common, easiest)** — full assistant, file persistence. `_shared/INBOX.md`.

**Work → Work (highest volume)** — runtime depends on what's available. Work INBOX (Playbook B or A).

**Work → Personal (career, techniques)** — requires cross-context transfer.
- Capture at work: note it during work session (mental note, personal notes app, or flag in work INBOX as "personal")
- Transfer: at next personal session, mention it during "anything to capture?"
- **Boundary rule:** Transfer the *insight*, not the work artifact. "I learned that batching reduces error rates" is fine. Copy-pasting proprietary content is not.

**Personal → Work (rare)** — relevant article on personal time, transfers at next work session.

### Cross-Context Transfer Mechanics

Hardest moment is **work → personal** — at work, deferring a personal capture. From lowest to highest friction:

1. **Mental note** — works for low-volume, high-salience items
2. **Notes app quick note** — type a 1-liner from phone during break
3. **Voice assistant** — lands in default reminders, processed at personal session
4. **Direct INBOX append** — if personal session is open in another window

For **personal → work**, the mirror applies.

**Volume expectation:** Cross-context transfers are low-volume — maybe 2-5 per week. The system doesn't need automation here. It needs a clear convention: "at session start, bring items from the other context."

## Volume & When to Automate

Realistic volume estimate for an active capture habit:

| Source | Frequency | Items/week |
|--------|-----------|-----------|
| Slack/chat threads | Daily | 5-10 |
| Post-meeting captures | Daily | 3-5 |
| Random thoughts / mobile | Daily | 2-4 |
| Links / articles | Few times/week | 2-3 |
| Emails | Few times/week | 1-2 |
| Doc captures | Weekly | 1-2 |
| Cross-context transfers | Weekly | 2-5 |
| **Total** | | **~15-30/week** |

At 15-30 items/week, the pipeline needs to be low-friction but doesn't need automation. Manual triage + routing in sessions handles this volume. If it consistently exceeds ~30/week, that's the trigger to consider auto-routing — see deprioritized tooling below.

## Deprioritized Tooling

Not vague backlog — each entry has enough detail to implement without re-researching.

### Slack MCP Connector
- Slack Web API: `conversations.history`, `search.messages`
- OAuth scopes: `channels:history`, `groups:history`, `im:history`, `search:read`
- MCP tool: takes channel/query, returns formatted messages → `INBOX.md`
- Build when: assistant approved at work + Slack API access granted

### Email Capture MCP
- Gmail/IMAP API — reuse existing auth flow if newsletter pipeline exists
- MCP tool: takes search query (sender, subject, date range), returns content
- Build when: regular non-newsletter email capture (>1×/week)

### Notion Reader MCP
- Notion API: `pages.retrieve`, `databases.query`, `blocks.children.list`
- Auth: integration token
- MCP tool: page URL or DB ID + filter → markdown
- Build when: Notion is the primary documentation tool

### Auto-Routing
- Config file (`routing-rules.yaml`) mapping tag patterns to destinations
- Assistant proposes, user confirms, assistant moves content
- Build when: `INBOX.md` regularly has >10 unrouted items at session start

## What This Deliberately Does NOT Include

- **No cross-vault search infrastructure.** Grep works. Revisit only if retrieval degrades.
- **No real-time push notifications.** Capture is async. Items land in `INBOX.md` and process at next session start. If something is truly urgent, it stays in the source system where notifications already work.
- **No automated triage.** Human decides what's worth keeping. Automate when patterns emerge, not before.
- **No multi-user / sharing patterns.** This is a personal capture system. Shared capture (team knowledge bases, shared inboxes) is a different problem with different constraints.
- **No knowledge graph or entity linking.** Items are flat text blocks with tags. Tags provide loose coupling. Graph structure is premature.

## Cross-References

| Document | Relationship |
|----------|--------------|
| `TRACKING_SYSTEM_DESIGN.md` | Sister design — same format pattern (Problem → Audit → Design → Scope limits) |
| `TECHNIQUES.md` | Where technique candidates land |
| `SYSTEM_ROADMAP.md` | Where roadmap-type items land |
