# Changelog

All notable changes to this template.

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
