# Changelog

All notable changes to this template.

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
