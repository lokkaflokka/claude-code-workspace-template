# Roadmap

## Before Distribution

- [x] Core template structure
- [x] README with philosophy and quick start
- [x] Example techniques (Session Init, Living KB, Query Routing, Sub-Project Pattern)
- [x] Artifact sync model documented
- [x] "Relationship to Source System" section
- [x] `/new-project` and `/new-technique` commands
- [x] Dogfood in source system (ongoing)
- [x] GitHub repo created
- [x] Tagged v0.1.2

## v0.1.4 (2026-01-25) — External Review Improvements

- [x] File size guidelines (CURRENT_STATE ~1 screen, CLAUDE.md 200-300 lines)
- [x] Session closing ritual documentation
- [x] Red list (what not to store) expanded
- [x] Context rot awareness in Philosophy
- [x] Search-first discovery protocol for larger projects
- [x] Day 7 snapshot example (`_example-project-day7/`)
- [ ] LinkedIn post drafted and published

## v0.2.0 (2026-02-10) — Methodology Guides

- [x] `_shared/guides/` directory with distributable methodology guides
- [x] Simulated Expert Panels guide (persistent personas, multi-round validation, convergence detection)
- [x] Synthetic Testing guide (5-tier framework: scripted → fuzzed → combinatorial → LLM-as-user → multi-session)
- [x] Updated `_shared/CLAUDE.md` file index

## v0.4.0 (2026-02-16) — Work Context + Sync Protocol

- [x] LLM Usage Policy (two-lane model, sensitivity lint, redaction standard)
- [x] Sync Protocol for cross-machine learning transfer
- [x] `/sync-review` slash command
- [x] MCP Setup Guide (phased, isolated)
- [x] Company LLM Handoff template
- [x] Trial Evidence Log template
- [x] Snippets reference with `ccsync`
- [x] Work / Company Setup section in README
- [x] Principle #5 gains post-customization, LLM boundary, MCP isolation rules
- [x] Principle #1 gains sync capture in session-end checklist

## Post-Distribution

- [ ] Gather feedback from early users
- [ ] Refine based on friction points discovered
- [ ] Consider additional example techniques based on demand

## Future Considerations

| Item | Trigger | Source |
|------|---------|--------|
| Video walkthrough | 5+ people ask for one | Original |
| Minimal vs full setup as branches | Confusion about where to start | Original |
| Additional technique examples | Clear demand for specific patterns | Original |
| Integration guides (Obsidian, VSCode, etc.) | User requests | Original |
| **MCP Development Guidance section** | When adding MCP-related content | Eval backfill 2026-01-20 |
| — DIY Over Framework Lock-in principle | (part of MCP guidance) | Technique: simple agents > frameworks |
| ~~— Defensive coding (Anxiety.md) pattern~~ | ✅ v0.1.7 | Added as "Defensive Code Generation" technique |
| ~~— Fallback Instructions pattern~~ | ✅ v0.1.8 | Added as "Fallback Instructions" technique |
| ~~— Preflight Sanity Check pattern~~ | ✅ v0.1.8 | Added as "Preflight Sanity Check" technique |
| **Autoskill example/guidance** | Interest in self-improving skills | Eval backfill 2026-01-20 |
| ~~**Interactive Feedback Loop pattern**~~ | ✅ v0.1.8 | Added as "Interactive Feedback Loop" technique |

## Design Decisions

**Fork-not-subscribe model:** Users clone and diverge. No upgrade path expected. This is intentional — the template is a starting point, not a dependency.

**Example-heavy approach:** Each template file includes both the structure AND concrete examples. Users can see what a filled-in version looks like.

**Three starter techniques:** Session Init, Living KB, and Query Routing are foundational enough to be useful but generic enough to apply broadly. Users can delete or extend as needed.
