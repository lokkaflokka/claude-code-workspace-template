# Sync Protocol

Bidirectional learning sync between workspace forks (e.g., personal and work machines). Transfers generalizable techniques, system improvements, and template feedback without leaking sensitive data.

---

## The Problem

When running this workspace template on multiple machines (personal + work), each machine independently discovers techniques, process improvements, and template gaps. Without a sync mechanism, learnings stay siloed — the same mistake gets made on both machines, the same improvement gets invented twice.

---

## The Sync Packet (Unit of Transfer)

Everything transfers via markdown packets — manually copied between machines (email to self, shared note, paste). No automated sync, no shared repos, no cloud dependency.

```markdown
# Sync Packet — YYYY-MM-DD

Source: [work|personal]

## Techniques Discovered
- **[Name]:** [Description]. Category: [technique|principle-refinement|gotcha|workflow].

## System Improvements
- **[What changed]:** [Structural/process improvement applicable to template or other forks].

## Template Feedback
- **[Gap/issue]:** [What didn't work or what's missing in the template].

## Sanitization Checklist
- [ ] No company names, people, or roles
- [ ] No proprietary processes or internal tools
- [ ] No credentials, URLs, or internal references
- [ ] Placeholders used for borderline content
```

---

## Directory Structure

Both machines maintain the same structure:

```
_shared/sync/
  staging.md        # Running bullet list of learnings (cleared on packet production)
  inbox/            # Received packets (from the other machine)
  outbox/           # Produced packets (to send to the other machine)
  archive/          # Processed packets (moved from inbox after processing)
```

---

## Enforcement Points

| When | What | Where |
|------|------|-------|
| Session end | "Anything generalizable?" → append to `staging.md` | Principle #1 session-end checklist |
| Weekly review | Consolidate staging → packet in outbox, run sanitization | Weekly review step 7.5 |
| Session init | Glob `_shared/sync/inbox/*.md` → surface if non-empty | Init step 4 |
| `/sync-review` | Process inbox + produce outbox in one command | Slash command |

---

## Cadence

- **Capture:** Every session (2 min if anything, 0 if nothing)
- **Produce packet:** Weekly (5 min)
- **Transfer:** Manual — user copies packet text between machines (email to self, shared note, paste)
- **Process inbox:** Next session after transfer (5 min)

---

## Processing Received Packets

When a packet arrives in `_shared/sync/inbox/`:

1. Read the packet
2. For each **Technique Discovered:** Check if already in `_shared/TECHNIQUES.md`. If new, document it. If refinement, update the existing entry.
3. For each **System Improvement:** Evaluate applicability. If relevant, implement the change.
4. For each **Template Feedback:** Note for next template upstream opportunity.
5. Move processed packet to `_shared/sync/archive/`
6. Update `CURRENT_STATE.md` if structural changes were made

---

## Producing Outbound Packets

1. Read `_shared/sync/staging.md`
2. If non-empty, consolidate entries into a packet using the template above
3. Run the sanitization checklist — every item must be checked
4. Write packet to `_shared/sync/outbox/YYYY-MM-DD-[topic].md`
5. Clear `staging.md`

---

## Sanitization Rules

Packets transfer between machines with different security contexts. The sanitization checklist is mandatory, not advisory.

**Always replace:**
- Company names → "the company" or `COMPANY`
- People → roles ("my manager", "the CTO")
- Internal tools → generic descriptions ("the company LLM", "the ticketing system")
- URLs → remove or replace with `[internal URL]`
- Credentials → never include

**Safe to include:**
- Abstract patterns and techniques
- Process descriptions without identifying details
- Architecture decisions (abstracted)
- Template structure improvements
- Technique catalog entries

---

## Relationship to Template

Sync packets between machines are **not** the same as upstream contributions to the public template. The flow is:

```
Machine A ──packet──> Machine B     (sync protocol — this doc)
     |                    |
     └─── both ──> Template repo    (upstream extraction — separate process)
```

Sync packets share learnings between your own machines. Template upstream shares generalizable patterns with the public.
