# Changelog

All notable changes to this template.

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
