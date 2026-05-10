---
description: Post-meeting signal extraction
disable-model-invocation: false
---
# /meeting-notes

Post-meeting signal capture — structured extraction of decisions, actions, insights, and open questions.

## When to Use

After any meeting where you want to persist key takeaways. Works best used immediately post-meeting while context is fresh.

### Delegated Mode (called by `/capture`)

When `/capture` batch mode detects meeting-structured content in a mixed dump:
- **Skip Step 0** — sensitivity lint already performed by `/capture`.
- **Skip Step 1** — meeting context already gathered or inferred by `/capture` before delegation.
- **Run Steps 2, 2b normally** — extraction + strategic synthesis. These are the value-producing steps.
- **Return** extracted items + synthesis + proposed routing to `/capture`.
- **Steps 3-5 execute via `/capture`'s unified confirmation** — not independently.

Direct invocation of `/meeting-notes` (no delegation) works exactly as below.

## Workflow

### Step 0: Sensitivity Lint

Before processing meeting content, apply the LLM Two-Lane check (see root `CLAUDE.md` LLM Usage Policy). Meeting notes often contain sensitive details. If any content is sensitive: persist to local vault files only, never to sync staging.

### Step 1: Gather Meeting Context

Ask the user for (skip fields they provide upfront):

- **Meeting type:** 1:1, team sync, cross-functional, standup, etc.
- **Date:** Default to today
- **Attendees:** Who was there (names + roles if relevant)
- **Topic/agenda:** What was discussed

### Step 2: Extract Signal Using Template

Walk through each section. The user can dictate freely — structure the output.

```
Meeting: <type> with <key attendee(s)>
Date: <date>
Attendees: <who>

Key decisions:
- <What was decided, by whom, with what rationale>

Action items:
- <WHO> → <WHAT> by <WHEN> (or "no deadline")

Insights:
- <What I learned that I didn't know before — domain knowledge, team dynamics, technical context>

Open questions:
- <What's still unclear or needs follow-up>
```

**Cross-reference proper nouns during extraction** (not during routing — do it once here):
- People names → check team directory via Grep before routing
- Company/product/system names → check against known vault entries via Grep

### Step 2b: Strategic Synthesis (mandatory — run before routing)

After factual extraction, run this pass. The goal is not more facts — it's patterns and implications. These items often drive the highest-value vault updates.

**Five questions to answer for every meeting:**

1. **Structural gap/disconnect:** What systemic problem does this reveal? (Not a new fact — a pattern or constraint that affects how work gets done.)

2. **Role opportunity:** Is there something you could propose, build, or drive based on what was learned? Concrete and actionable — not "do more of X."

3. **Cross-system connections:** Does this connect to active projects, ongoing roadmap work, or other stakeholders' concerns? Explicitly check active project directories and cross-reference against active work streams.

4. **New OQs beyond prep:** What questions emerged that weren't in the meeting prep? These often reveal where the meeting went beyond its agenda — high signal.

5. **Stakeholder framing context:** How does this person frame their work? What matters to them operationally? This is the lens for future proposals and follow-ups.

**Routing rules for synthesis output:**

| Output type | Destination |
|-------------|-------------|
| Structural gap / disconnect | Domain reference files (inline, not Pending Merge) |
| Role opportunity | Task list + OQ table |
| Cross-system connections | Active project state files |
| New OQs | OQ table + person's 1:1 prep / playbook section |
| Stakeholder framing context | Meeting file debrief — **not state files** |

**Why the last rule matters:** Stakeholder framing (how to approach someone, what framing lands, communication style) belongs in meeting files — same tier as communication style notes in playbooks. State files and OQ tables are factual/actionable only.

---

### Step 2c: Routing Preview (hard gate)

**Before any file writes,** present a routing preview table:

| # | Content Summary | Target File | Target Section |
|---|----------------|-------------|----------------|

User confirms or adjusts before proceeding. Never skip straight to writing — preview-first catches misroutes before they happen (fixing after writes requires re-reading files to undo).

**Published artifact gate (update mode):** When routing items that touch a maintained/published artifact, classify each as:
- **"Artifact (substance)"** — the reader learns something new about the system. Routes to the artifact.
- **"Progress log (tracking)"** — internal tracking, status updates, operational detail. Routes to state file progress log.

Default: progress log. Test: "Does the reader learn something new about the system?" This prevents over-routing operational findings to published artifacts.

---

### Step 3: Route — Tiered by Read Cost

Route each extracted item to the appropriate tier. **Do all Tier 0 and Tier 1 writes first, then all Tier 2 appends in one pass.**

#### Tier 0 — Session Memory (free, already loaded at /start)

Files loaded during `/start`. Edit directly — no pre-read required.

| Content | Target | Method |
|---------|--------|--------|
| Action items | state file → Action Items | Direct edit |
| Open questions | state file → Open Questions | Direct edit |
| Decisions | decision log | Append (small file, append-only by design) |

#### Tier 1 — Targeted Read (grep anchor + offset read)

Files where insertion must be person- or section-specific. Use Grep to find the anchor line, then `Read(offset=N, limit=50)` — **not full file reads**.

| Content | Target | Pattern |
|---------|--------|---------|
| 1:1 prep additions, new questions for a person | 1:1 prep / playbook file | Grep for person name → Read 50 lines from that offset → Edit |
| People/relationship context | people/context file | Grep for section → Read 30 lines → Edit |

#### Tier 1b — Project Vaults (targeted read per project)

When meeting signals connect to an active project, route to that project's state files.

**Trigger:** Any extracted item that (a) advances, validates, or changes scope of an active project, OR (b) surfaces a new open question, key resource, or stakeholder for a project.

**Discovery:** Check for active project directories. Cross-reference meeting topics against project names and state file summaries.

**For each relevant project:**

1. Read the project's `CURRENT_STATE.md` (full read — project state files are small by design)
2. Update inline:
   - **Open Questions** table: add questions the meeting surfaced
   - **Key Resources** table: add people, docs, or tools newly identified
   - **Progress Log**: add a dated entry capturing validation, scope changes, or new context
3. If action items are project-specific → project's `TASKS.md`: Grep for target section → Edit

| Content | Target | Pattern |
|---------|--------|---------|
| Open questions, scope signals, new context | project `CURRENT_STATE.md` | Full read → Edit inline |
| Project-specific action items | project `TASKS.md` | Grep for section → Edit |

**Why this matters:** Meeting signals that validate a project's priority, surface new stakeholders, or change scope belong in the project vault — not only in vault-level reference files. Without this step, project state goes stale between dedicated project sessions.

---

#### Tier 2 — Append-Only (no pre-read)

Large reference files where new content is additive. Append to a standardized `## Pending Merge` section at the bottom. The cleanup pass (`/check` Pending Merge step) merges inline on a schedule.

| Content type | Append target |
|--------------|--------------|
| Architecture updates, new systems, infrastructure, tech debt | architecture file |
| Product feature updates | product file |
| Roadmap/strategy signals | roadmap file |

**Append format:** Add this block to the bottom of the target file. If `## Pending Merge` already exists, append a new dated entry — don't create a duplicate section.

```markdown
---

## Pending Merge

*Items below are from meeting notes awaiting inline merge. Run `/check` to integrate.*

### YYYY-MM-DD (<meeting type>)
**→ <Target Section Name>:** <content to merge>
**→ <Target Section Name>:** <content to merge>
```

**Tagging rule:** The `→ <Target Section Name>` tag must match a real `###` or bolded `**` heading in the file. If no exact section exists, use the closest parent heading and add a note: `→ <Parent Section> (new subsection: <name>)`.

### Step 4: Verify Routing

- [ ] Active projects checked — any meeting signal connecting to a project is routed to that project's `CURRENT_STATE.md` (Tier 1b)
- [ ] Every open question with a named person → also in that person's 1:1 prep (Tier 1 read)
- [ ] Tier 2 items: `→ <Section Name>` tag matches a real heading in the target file
- [ ] Action items have WHO + WHAT + WHEN
- [ ] Vault persistence filter applied — skip transient/operational details

### Step 5: Confirm

Show the user what was routed where, distinguishing **written inline** (Tier 0/1) from **pending merge** (Tier 2). Ask if anything was missed.

## Rules

- **Don't force all sections.** If there were no decisions, skip that section. Meetings vary.
- **Preserve the user's words** for insights — don't over-summarize.
- **Action items need a WHO.** "Follow up on X" is incomplete. "I follow up on X by Friday" is actionable.
- **Apply vault persistence filter.** Skip administrative logistics, resolved one-time issues, transient scheduling details. Persist domain knowledge, strategic insights, relationship context, answered questions.
- **Verify before persisting.** Cross-reference proper nouns, qualify approximate quantities, classify correctly.
- **Tier 2 is not second-class.** Pending merge items are fully captured and will land in canonical files on the cleanup schedule. The tradeoff is deferred precision, not deferred capture.
- **Rescheduled meeting merge:** When a meeting is rescheduled and a new prep file created, read old file fully, identify items still relevant but missing from new file, merge them, then delete old file. Unanswered questions from the old prep should not be lost to rescheduling.
- **Prep → debrief pairing:** When a meeting prep task is marked complete, always create a paired debrief task. Prep without debrief means the conversation doesn't enter the knowledge base.
- **Boundary rule:** Capture *your takeaways*, not proprietary meeting content verbatim.
- **Sensitivity:** Meeting notes often contain sensitive content. Route to local vault files, never sync staging.
- **Reverse routing:** When answers close an open question, propagate back: remove from Open Questions, update 1:1 prep, update inline references.
- **Cross-context:** If a work meeting yields a personal career insight, route the career piece to your career vault, not the work vault.
