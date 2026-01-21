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
- [ ] LinkedIn post drafted and published

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
| — Defensive coding (Anxiety.md) pattern | (part of MCP guidance) | Technique: security-sensitive code review |
| — Fallback Instructions pattern | (part of MCP guidance) | Technique: dependency failure handling |
| — Preflight Sanity Check pattern | (part of MCP guidance) | Technique: config verification before work |
| **Autoskill example/guidance** | Interest in self-improving skills | Eval backfill 2026-01-20 |
| **Interactive Feedback Loop pattern** | Interest in learning capture | Eval backfill 2026-01-20 |

## Design Decisions

**Fork-not-subscribe model:** Users clone and diverge. No upgrade path expected. This is intentional — the template is a starting point, not a dependency.

**Example-heavy approach:** Each template file includes both the structure AND concrete examples. Users can see what a filled-in version looks like.

**Three starter techniques:** Session Init, Living KB, and Query Routing are foundational enough to be useful but generic enough to apply broadly. Users can delete or extend as needed.
