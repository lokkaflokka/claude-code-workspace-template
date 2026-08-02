# Changelog

All notable changes to this template.

## [0.9.0] - 2026-08-02

Principles release. The source system this template extracts from had grown from 8 core principles to 10, and the two additions are the ones its own records rank highest by recurrence. This release closes that gap and adopts the source system's compression discipline for the always-loaded principle blocks.

### Added

- **Principle 9: Entity-of-Record Discipline.** Exactly one system holds the truth for each tracked entity; everything else references it. Covers the "before creating a second record, ask whether the first should move" gate, credential rotation as atomic with its fan-out, the owned-fix guard (new observations of a known failure class route to the owning record rather than spawning competing artifacts), and the rule that documentation describing an obligation is not the same as the obligation being tracked.
- **Principle 10: Source Before Action.** Read the underlying source before any plan, breakdown, sizing, recommendation, or state-mutating action about a tracked item. Promoted out of Principle 3 to its own number because it is the highest-recurrence anti-pattern the source system has recorded, and a rule that keeps being violated while nested inside a broader rule needs its own name and its own violation count. Includes the mechanical-not-behavioral enforcement tier and the gate-surface enumeration rule (write both lists: every surface the RULE names, every surface the GATE observes).
- **`_shared/PRINCIPLES_REFERENCE.md`:** full enforcement sections for Principles 9 and 10, matching the existing per-principle format (principle, enforcement, anti-pattern, verification, violations prevented).

### Changed

- **`_shared/CLAUDE.md` restructured to operative-rules-only** (`_context.version` 2.1 to 2.2). Each principle block now states the operative rule in the imperative and points at `PRINCIPLES_REFERENCE.md` for reasoning and history. A new House style note states the compression test explicitly: this file loads into every session, so prose that has stopped changing behavior costs tokens on every turn. Compress on that test, not on length.
- **Principle 5 now splits WRITE from RENDER.** Deciding who may read a body on which surface is a separate question from where the content may be persisted, and each fence gets priced on its own evidence. Added because a policy sentence of the form "never A, B, C, or D" is usually more than one rule, and the weaker half silently inherits the stronger half's absoluteness.
- **Principle 7 gains body-sufficiency.** An item must carry its execution payload inside whatever character budget the reading surface actually renders. A body that is sufficient in principle but truncated on a phone is not sufficient.
- **Principle 2 gains recurrence escalation.** The same gate-class failing twice inside 48 hours requires a mechanical fix at a named surface; a second occurrence never closes as a discipline insight.
- **`README.md` structure tree corrected.** It listed 2 of the 12 shipped skills and 5 of the 15 shipped `_shared/` files. Now accurate, with `consistency-check.md` marked as superseded by `/check` rather than presented as a primary command.

### Fixed

- **Backfilled the missing `[0.8.0]` CHANGELOG entry.** v0.8.0 was tagged 2026-05-16 with a full annotation, but the CHANGELOG stopped at 0.7.0, so the public release record skipped a version.

## [0.8.0] - 2026-05-16

`/end` Phase 5, the post-close addendum. Single `feat` commit since v0.7.0 (`b388328`). Backfilled 2026-08-02 from the annotated tag; this entry was missing from the original release.

### Added

- **`/end` Phase 5: Post-`/end` addendum.** Handles work that continues after session close. Captures the addendum pattern (chained sessions, late-arriving completions, mid-deferred work) into the durable session record without requiring a full second `/end` pass.

## [0.7.0] - 2026-05-10

Major release. Substantially expanded skill suite, six new design docs, and a refreshed `_shared/CLAUDE.md` that lazy-loads enforcement detail and gotchas to keep the always-loaded CLAUDE.md hierarchy under reinjection thresholds.

### Added — Skills

- **`/start`** — Full session-init protocol. Phases A (parallel gather → cache JSON) → B (process + zone-based Today Plan output, capacity signal, exception-only) → C (interactive triage). Mode tokens (`quick`, `system`, `today`), vault aliases, activity tokens (`triage`, `plan`, `prep`, etc.), `then` free-text continuation. Hard 8-line cap on output. `/start system` adds Phase D system audit.
- **`/check`** — System health check. 16 mechanical + semantic checks (registry drift, version sync, tag drift, file-surface health with three-tier thresholds, recurrence tag audit, sensitive-data boundary scan, entity fragmentation, gap detection, etc.). `/check merge-pending` runs only the pending-merge step.
- **`/capture`** — Signal triage and routing. 7-type triage taxonomy (Information, Action, Roadmap, Technique, Content, Backlog, Discard), routing-confidence assessment (auto-route vs human-review), batch mode with source grouping, mixed-content orchestration that delegates meeting content to `/meeting-notes`.
- **`/route`** — Batch execution of staged synthesis proposals. 5-phase protocol (load → approve → execute deterministic → execute judgment → complete) with visible-output approval gate. Handles theme updates, learning log writes, technique candidates, vault routing, new themes, demotions.
- **`/revisit`** — Position review with spaced repetition. Evidence chain presentation, confirmation-bias scan, position evolution options (strengthen / update / retire / split / no change), interval doubling on strengthen, reset-to-30d on contradictory evidence. Integrates with `/start` (review-date check) and `/end` (cache update).
- **`/meeting-notes`** — Post-meeting structured extraction. Meeting context gather, four-section template (decisions / actions / insights / OQs), strategic synthesis pass (5 questions), routing-preview hard gate, tiered routing (Tier 0 session memory → Tier 1 targeted read → Tier 1b project vaults → Tier 2 append-only with Pending Merge).

### Changed — Skills

- **`/end`** — Expanded from 4-step framework to 5-phase Gather-Process-Emit + verification. Phase 0 (Session Scan, no tools) added as an explicit input list — process failures, verification misses, techniques applied, generalizable patterns, novel insights, position reviews, value moment. Phase 1 batches all reads in parallel. Phase 2 splits writes across two turns to maintain edit quality. Phase 3 verification adds unconditional repo hygiene scan + conditional vault-first / action-reminder pairing / waiting-on / cross-reference checks. Phase 3.5 (optional Plan Tomorrow). Phase 4 mandatory sync-worthiness gate before drafting packets.
- **`/sync-review`** — Adds the **Disposition Gate** (mandatory per-item table before archive — actions must be Edit/Task/Note/Deferred, no generic "noted/applicable/already covered" labels) and the **Routing Gate** for technique candidates (one consumer → inline into that file; two+ → registry entry justified). Optional mechanical-enforcement note for PreToolUse hook on `TECHNIQUES.md`.

### Added — Design Docs (`_shared/`)

- **`PRINCIPLES_REFERENCE.md`** — Full enforcement detail for the 8 Core Principles (concise forms remain in `_shared/CLAUDE.md`). Includes tool-before-question gate, source-before-action gate, mechanical-fix-first tier sequence (Tier 1 code/schema/hook → Tier 2 doc rule at consumer surface → Tier 3 behavioral last resort), cross-vault awareness sub-rule, first-use destructive-operation gate, sensitive-domain wall, public-repo pre-commit checklist.
- **`TECHNICAL_GOTCHAS.md`** — Curated factual reminders. Extracted from `_shared/CLAUDE.md` for size management — keeps the always-loaded CLAUDE.md hierarchy small while preserving the gotcha catalog. Covers Claude Code hooks, CLAUDE.md reinjection cost, background agents (`claude -p`), tool result reliability, WebFetch failures, JS-rendered portals, macOS launchd + unsigned `.app` bundles, bash idioms, AppleScript, reminders/task systems, MCP development, Obsidian race conditions, terminal input truncation, more.
- **`TRACKING_SYSTEM_DESIGN.md`** — Three-list architecture (Strategic / Personal / Inbox) + zone-based planning (Deep Work / Movement / Admin) + Today Plan output format with 8-line cap + completion processing algorithm + body-tag conventions (`[recur:]`, `[waiting-on:]`, `[ref:]`) + cross-vault entity references + phase-based checklist convention. Task-system-agnostic; includes a "Tooling Note" describing reference-implementation properties (single tool for reads/writes, list-scoped operations, `--dry-run`, verify-after-save, JSON output with `completionDate`).
- **`SIGNAL_CAPTURE_PATTERN.md`** — 5-stage capture pipeline (capture / triage / enrich / route / retrieve), triage taxonomy, INBOX.md staging format with metadata schema, routing confidence (auto-route vs human-review), capture surfaces (assistant session, Slack, email, meeting notes, web links, mobile), implementation playbooks (Playbook A full assistant / Playbook B constrained / Playbook C hybrid), cross-context flows.
- **`SILENT_FAILURE_SAFEGUARDS.md`** — M1-M4 framework for catching failures that report success but didn't actually do the work. M1 Precondition Gate, M2 Verification Window, M3 Contradiction Alert, M4 Source-Coverage Gate. Five anonymized example instances + mechanism × instance matrix + adoption-order guidance.
- **`SKILL_INDEX.md`** — Skill inventory + evaluation framework. Two-tier structure (Core/structural + Common/specialized), routing table (when to invoke), skill boundaries (owns vs. doesn't own), native Claude Code (no custom skill needed), evaluation framework (collection / contrast / synthesis), skill candidates section, two-tier-reality concept.
- **`LEARNING_SYSTEM_DESIGN.md`** — Knowledge architecture for compounding insights. Stable IDs (T-NNN, I-NNN, P-NNN, TH-NNN), enhanced markdown entries with cross-references, dual-loop architecture (global outer: Synthesizer + Challenger; vault-scoped inner: Advisor + Reflector), staged-routing model, vault learning context (`_learning_context` block), spaced-repetition position review, 3 feedback loops. Phased rollout from foundation through Challenger.
- **`PROACTIVITY_DESIGN.md`** — Push-based proactive behaviors design. Always-on execution platform pattern, single-layer (assistant-powered) vs two-layer rejection, shared push utility, four behaviors (B1 daily briefing, B2 digest completion notification, B3 heartbeat monitor, B4 cost monitoring), surface interaction model (glance / reply / session), security considerations ("permission hungry" framing), phased implementation, cost controls, success criteria.
- **`SYSTEM_ROADMAP.md`** — Roadmap skeleton (template the user fills in) covering completed phases / product-pass arsenal / current phase / future phases / tier-based sequencing / opportunistic backlog / changelog.

### Changed — Existing Files

- **Root `CLAUDE.md`** — Refreshed Session Initialization section to point at `.claude/commands/start.md` for the full protocol. Added a Custom Skills quick-reference table with all 10 lifecycle skills. Expanded Query Routing table with PRINCIPLES_REFERENCE / SKILL_INDEX entries. Expanded Key Files table to cover the new design docs.
- **`_shared/CLAUDE.md`** — Added Principle #8 (Content Shared Is Content Captured). Tightened Principles 1-7 with new sub-rules (state-file composition, tool-before-question gate, mechanical-fix-first, cross-vault awareness, first-use destructive-operation gate, credential rule, `[ref:]` tags). Replaced inline Technical Gotchas section with a pointer to `TECHNICAL_GOTCHAS.md` (significantly reduces always-loaded CLAUDE.md weight). Replaced inline Session Initialization Protocol with a pointer to `.claude/commands/start.md`. Added cross-references to the new design docs in Purpose & References.
- **`_shared/PROJECTS.md`** — Added Cross-References section (`SYSTEM_ROADMAP.md`, `ARCHITECTURE.md`, `SYNC_PROTOCOL.md`).
- **`_shared/TECHNIQUES.md`** — Added Routing Gate (mandatory before adding new entries: name consumers, count them, defer if not yet applied) and `<!-- T-NNN -->` ID scheme, with new `Applied:` / `Effectiveness:` / `Connects to:` fields per entry. Existing technique entries unchanged.

### Conventions Added

- **Stable IDs** for techniques (`T-NNN`), insights (`I-NNN`), positions (`P-NNN`), and themes (`TH-NNN`). Permanent — never reused, never reassigned. Removed entries get a tombstone (`Status: Archived` or `Status: Superseded by T-045`).
- **Body tags** for task-system reminders: `[recur: N days]` / `[recur: +N days]` / `[recur: stop]` for completion-based recurrence, `[waiting-on:]` + `[waiting-since:]` for blocked items, `[ref: vault/path/FILE.md]` for cross-vault entity references.
- **Routing Gate** — applied to both `/sync-review` and `_shared/TECHNIQUES.md` editing. Prevents the registry from calcifying with unapplied techniques.
- **Mechanical-fix-first principle** — when a process failure surfaces, default to a Tier 1 (code/schema/hook) or Tier 2 (doc rule at consumer surface) fix. Tier 3 (behavioral-only "remember to...") is last-resort and must name what would make it Tier 1/2.

## [0.6.0] - 2026-03-10

### Added
- **Gather-Process-Emit (GPE) skill architecture** — New section in `_shared/CLAUDE.md` documenting the phase-separated skill pattern: batch reads → reason with zero tool calls → batch writes. Prevents interleaved read/write errors.
- **File Staleness targets** — New section in `_shared/CLAUDE.md` with per-file staleness thresholds and overflow actions. CURRENT_STATE.md: 7 days, CLAUDE.md: 30 days, TECHNIQUES.md: 30 days.
- **Session-attributed Common Mistakes** — All Common Mistakes entries now include session number and date (`[S#, YYYY-MM-DD: what happened]`). Enables pruning: no violation in 10+ sessions = candidate for removal. Updated in both example projects.

### Changed
- **`/reflect` → `/end`** — Session-end skill renamed and restructured around Gather-Process-Emit phases. Phase A batches all reads, Phase B reasons about updates with zero tool calls, Phase C batches all writes. Replaces the 11-step interleaved checklist.
- **README updated** — Session closing ritual references `/end` and GPE. Common Mistakes examples show session attribution format. Structure diagram updated.
- **Principle #1** session-end checklist references `/end` instead of `/reflect`.

### Removed
- **SNIPPETS.md** — Text expander shortcuts superseded by slash commands. Every snippet was a worse version of an existing skill or CLAUDE.md instruction.
- **ROADMAP.md** — Internal planning doc with stale future considerations. Not user-facing value.

## [0.5.0] - 2026-02-28

### Added
- **`/challenge` slash command** — Decision challenge using named personas. Three stakeholder perspectives with specific concerns, not generic archetypes.

## [0.4.0] - 2026-02-16

### Added
- **LLM Usage Policy** (Two-Lane Model) — Company-approved LLM handles sensitive inputs, personal Claude handles non-sensitive work with placeholders. Includes sensitivity lint, redaction standard, and lane definitions. Added to root CLAUDE.md.
- **Sync Protocol** (`SYNC_PROTOCOL.md`) — Bidirectional learning sync between workspace forks (personal + work). Sync packets, staging/inbox/outbox directory structure, 4 enforcement points, sanitization checklist.
- **`/sync-review` slash command** — Process sync inbox and produce outbound packets in one command.
- **MCP Setup Guide** (`MCP_SETUP.md`) — Phased approach to MCP tools at work. Plugin manifest interface, work/personal isolation, security checklist.
- **Company LLM Handoff Template** (`templates/COMPANY_LLM_HANDOFF.md`) — Structured template for passing abstracted findings from company LLM to Claude.
- **Trial Evidence Log** (`templates/TRIAL_LOG.md`) — Template for tracking Claude Code wins, time savings, and quality improvements to build the case for company-credentialed access.
- **Snippets Reference** (`SNIPPETS.md`) — Quick-reference for common Claude Code prompts. Session flow, operations, and systems thinking snippets with `ccsync` entry.
- **Work / Company Setup** section in README — Setup lifecycle, steps, verification, and what stays on the work machine.

### Changed
- **Principle #5 (Boundaries Are Sacred)** gains three new rules:
  - Post-customization rule (remove remote after filling in company details)
  - LLM boundary rule (sensitivity boundary between company and personal LLMs)
  - MCP isolation rule (separate configs, packages, credentials)
- **Principle #1 (Files Are The Work Product)** gains sync capture in session-end checklist
- **Session init** gains sync inbox check (new step 4)
- **Key Files table** expanded with SYNC_PROTOCOL.md, MCP_SETUP.md, SNIPPETS.md

## [0.3.0] - 2026-02-11

### Changed
- **Session Initialization Protocol** — Full rewrite synced with production system:
  - Exception-only output format (sections omitted when clean, no "all clear" lines)
  - Capture staging area (INBOX.md) with "Anything to capture?" prompt
  - Three-list reminder architecture (session context / offline tasks / quick capture inbox)
  - Completed item processing with state file cross-referencing
  - Due today/tomorrow surfacing for non-recurring offline items
  - Consistency checks removed from init (run `/consistency-check` separately)
- **Core Principles expanded from 6 to 7:**
  - New **#5: Boundaries Are Sacred** — code/data separation, PHI/PII rule
  - **#1** gains vault-first check and action-reminder pairing in session-end checklist
  - **#6 (was #5)** gains build verification, no-duplicated-logic check
  - **#7 (was #6)** gains vault-first rule, entity rule, action-reminder pairing rule, list placement guidance
  - All principles now include "Violations this prevents" sections
- **Technical Gotchas significantly expanded** — 6 categories (Claude Code Hooks, Reminder Systems, AppleScript, MCP Development, External Services, macOS Filesystem) with 17 specific gotchas from 40+ production sessions
- **`_context` version** bumped to 2.0

### Removed
- "Common Mistakes" section — fully migrated into Core Principles with enforcement mechanisms
- Consistency check from session init (now runs at its own cadence via `/consistency-check`)

## [0.2.1] - 2026-02-10

### Improved
- **Synthetic Testing guide** — Validated by building from scratch against a fresh app. Patches:
  - Added Prerequisites section (Playwright install, config template, directory structure)
  - Rewrote Step 2 with concrete app-specific helpers pattern
  - Replaced pseudocode property assertions with real Playwright implementations
  - Added concrete generator function pattern with word banks and batch generation
  - Scrubbed all domain-specific references to keep examples fully generic

## [0.2.0] - 2026-02-10

### Added
- **Methodology Guides** — New `_shared/guides/` directory with two distributable methodology guides:
  - **Simulated Expert Panels** (`simulated-expert-panels.md`) — Multi-round validation panels with persistent AI personas. Covers persona design, round execution, verdict evolution, convergence detection, and panel rotation. Generic methodology applicable to product decisions, system design reviews, career decisions, and risk assessment.
  - **Synthetic Testing** (`synthetic-testing.md`) — Tiered automated testing framework scaling from scripted flows to LLM-driven exploration. 5-tier architecture (scripted → fuzzed → combinatorial → LLM-as-user → multi-session), with implementation patterns for seeded randomness, persona generation, vision provider abstraction, tiered model selection, and cost modeling.
- Updated `_shared/CLAUDE.md` file index with guides references

## [0.1.9] - 2026-02-08

### Added
- **Convention Density** technique — Opinionated, convention-heavy frameworks multiply LLM productivity by reducing ambiguity and decision surface
- **Exception-Based Reporting** technique — Frequency-tiered monitoring: exception-only at high frequency, thorough audits at low frequency. Prevents alert fatigue.
- **File Size Check** in `/consistency-check` — Flags CURRENT_STATE.md >100 lines and CLAUDE.md >300 lines to prevent context bloat

## [0.1.8] - 2026-02-05

### Added
- **Preflight Sanity Check** technique — Verify config and external state before running workflows to catch issues early
- **Fallback Instructions** technique — Document detection, fallback, and impact for every external dependency in workflows
- **Interactive Feedback Loop** technique — Prompt for corrections after synthesis/curation while context is fresh
- **Verification-Led Development** technique — Give Claude verification mechanisms (tests, lints, builds) and iterate until they pass
- **Technical Gotchas** section in `_shared/CLAUDE.md` — Factual reminders about tools and environment (separate from behavioral principles)
- **Hooks guidance** — SessionStart hook pattern for date injection and environment setup

## [0.1.7] - 2026-02-05

### Added
- **Defensive Code Generation** technique — Frame Claude as a senior engineer burned by production incidents to induce defensive coding patterns (input validation, error handling, edge case coverage)
- **Route Around Unreliable Tools** technique — When a tool is partially reliable, partition reliable vs unreliable operations and route the broken ones through alternatives
- **Search-First Discovery** technique — Promotes the existing Discovery Protocol blurb into a full technique with examples, implementation guidance, and applicability criteria
- Cross-reference from Discovery Protocol in `_shared/CLAUDE.md` to the full Search-First Discovery technique

## [0.1.6] - 2026-02-02

### Added
- **Reminder freshness rule** — When session work changes state referenced by an open reminder, update that reminder's body in the same session. Prevents stale reminder bodies from causing future-self to act on wrong context.

## [0.1.5] - 2026-02-02

### Added
- **Core Principles framework** — Enforced behavioral rules replacing passive "Common Mistakes" list. Template structure (principle + enforcement + violations prevented) for users to customize with their own rules.
- **Compact session init output** — Output format rules: compact lines, no tables, visual priority indicators (🔴/⏳/•), suppress all-clear categories
- **Completed reminders since last session** — Init protocol now surfaces recently completed reminders and asks for outcomes (state files may need updating)
- **Date anchoring rule** — Always anchor today's date from environment; never derive day-of-week mentally
- **Session-end artifact checklist** — Explicit checklist: CURRENT_STATE, vault state files, MCP version bumps
- **Reminder chain rule** — Completing a reminder with remaining actions → create successor before marking done
- **Versioning policy** — When/how to bump packages (feat=minor, fix=patch), tag drift awareness in Principle #5
- **Tag drift check** — New section in `/consistency-check` detecting commits past latest tag, missing tags, version/tag mismatches
- **Template staleness check** — New section in `/consistency-check` comparing template release dates against source system modifications

### Changed
- Session init protocol is now mandatory (no exceptions, even for action requests)
- Init example uses compact format with emoji indicators instead of verbose text

## [0.1.4] - 2026-01-25

### Added
- **File Size Guidelines** — Explicit targets: CURRENT_STATE ~1 screen, CLAUDE.md 200-300 lines max
- **Session Closing Ritual** — Documented pattern for consistent state updates
- **Red List expanded** — Explicit list of what NOT to store (credentials, PII, raw transcripts, etc.)
- **Context Rot awareness** — Added to Philosophy section explaining why modular files and session boundaries matter
- **Search-First Discovery Protocol** — Added to `_shared/CLAUDE.md` for larger projects to reduce token waste
- **Day 7 snapshot example** — `_example-project-day7/` showing a populated project after a week of real use (home renovation example with deadlines, working memory, discovered mistakes)

### Changed
- Structure section now describes both example projects (empty scaffolding vs. lived-in snapshot)

## [0.1.3] - 2026-01-19

### Added
- **"Your First Session" guide** in README — step-by-step instructions for bootstrapping a working system from the template
- **"What It Looks Like After a Few Weeks"** — realistic examples showing populated CURRENT_STATE.md and Common Mistakes sections
- **"The Bootstrapping Mindset"** — sets expectations for how value compounds over time (Week 1 = setup, Week 2 = remembering, Week 3+ = compounding)

## [0.1.2] - 2026-01-17

### Added
- **`/new-project` command** — Automates creation of projects and sub-projects with proper structure and registration
- **`/new-technique` command** — Automates documenting techniques and evaluating them across projects
- **Sub-Project Pattern** in TECHNIQUES.md — Pattern for nested finite-scope work with lifecycle management and alert propagation

## [0.1.1] - 2026-01-17

### Added
- **Working Memory** section in CURRENT_STATE.md templates — short-term notes with expiration dates
- Session initialization protocols now surface unexpired working memory items and prompt to clear expired ones
- **Cross-level working memory** — project sessions now read both project-level AND system-level state, so system-wide items surface everywhere

## [0.1.0] - 2026-01-17

### Added
- Initial template structure
- README with philosophy, quick start, key concepts, FAQ
- Top-level CLAUDE.md with session initialization protocol
- `_shared/` directory with:
  - CLAUDE.md (cross-project workflows, common mistakes)
  - CURRENT_STATE.md (meta-level dashboard template)
  - TECHNIQUES.md (pattern catalog with 3 example techniques)
  - EVALUATION_LOG.md (technique evaluation tracking)
  - PROJECTS.md (project registry template)
- `_example-project/` with:
  - CLAUDE.md (project-level template with examples)
  - CURRENT_STATE.md (project state template)
- `.claude/commands/consistency-check.md` (registry verification command)
- "Relationship to Source System" section explaining fork-not-subscribe model
