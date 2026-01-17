# Projects Registry

The authoritative list of projects in this workspace.

---

## Project Index

| Project | Purpose | Has CLAUDE.md | Has CURRENT_STATE |
|---------|---------|---------------|-------------------|
| `_shared` | Cross-project techniques and meta-level state | Yes | Yes |
| `_example-project` | Template for new projects | Yes | Yes |

*Add your actual projects to this table as you create them.*

---

## Adding a New Project

1. Copy `_example-project/` to a new directory
2. Rename and customize `CLAUDE.md` for your project
3. Decide if you need `CURRENT_STATE.md` (recommended for stateful projects)
4. Add the project to this registry
5. Run `/consistency-check` to verify

---

## Source Control Boundaries

Define what goes in version control vs. stays local.

| Location | In Git? | Notes |
|----------|---------|-------|
| Personal knowledge vaults | No | Personal content, never distribute |
| Distributable packages/tools | Yes | Versioned, shareable code |
| `_shared/` metadata | Your choice | Templates yes; personal state no |

---

## Project ↔ Package Links

If projects depend on external packages or tools, track the relationships here.

| Project | Package | Version | Notes |
|---------|---------|---------|-------|
| — | — | — | — |

*Example:*
| Project | Package | Version | Notes |
|---------|---------|---------|-------|
| newsletters | mcp-content-reader | 0.2.0 | Gmail integration |
| finance | — | — | No external packages |

---

## Sync Protocol

When certain events happen, update this registry:

| Event | Update Required |
|-------|-----------------|
| New project created | Add to Project Index |
| Project deleted/archived | Remove from Project Index |
| Package dependency added | Add to Project ↔ Package Links |
| Package version updated | Update version in Links table |

This keeps the registry as the single source of truth.

---

*Last updated: YYYY-MM-DD*
