# Evaluation Log

Tracks when techniques are evaluated against projects and whether they were applied.

---

## How to Use This File

After documenting a technique in `TECHNIQUES.md`, evaluate it against your projects:

### For Project Techniques

```markdown
## [Date]: [Technique Name]

**Type:** Project
**Origin:** [Source]

| Project | Relevant? | Applied? | Notes |
|---------|-----------|----------|-------|
| project-a | Yes/Maybe/No | Yes/No/— | [Reasoning] |
| project-b | Yes/Maybe/No | Yes/No/— | [Reasoning] |

**Summary:** [Key takeaways from the evaluation]
```

### For Workflow Techniques

```markdown
## [Date]: [Technique Name]

**Type:** Workflow
**Origin:** [Source]
**Status:** Installed / Not installed / Evaluating
**Notes:** [Setup steps, observations]
```

---

## Evaluation Criteria

When assessing relevance, consider:

1. **Problem fit** — Does this technique address a problem the project actually has?
2. **Complexity cost** — Would applying it require significant rework or add maintenance burden?
3. **Value ratio** — Is the benefit worth the cost of implementation?
4. **Timing** — Is now the right time, or should this wait?

**Relevance ratings:**
- **Yes** — Clear fit, should apply (now or soon)
- **Maybe** — Potential fit, worth revisiting later or under certain conditions
- **No** — Doesn't address a real problem for this project

---

## Evaluations

*Add new evaluations below as you assess techniques.*

### YYYY-MM-DD: Session Initialization Protocol

**Type:** Project
**Origin:** Template default

| Project | Relevant? | Applied? | Notes |
|---------|-----------|----------|-------|
| _shared | Yes | Yes | Meta-level state tracking via CURRENT_STATE.md |
| _example-project | Yes | — | Template includes it; apply when creating real projects |

**Summary:** Core technique for session continuity. Apply to any project with ongoing state.

---

### YYYY-MM-DD: CLAUDE.md as Living Knowledge Base

**Type:** Project
**Origin:** Boris Cherny / Anthropic

| Project | Relevant? | Applied? | Notes |
|---------|-----------|----------|-------|
| _shared | Yes | Yes | Common Mistakes section added |
| _example-project | Yes | — | Template includes it; customize per project |

**Summary:** Universal technique. Every project benefits from capturing mistakes.

---

### YYYY-MM-DD: Query Routing Table

**Type:** Project
**Origin:** Common pattern

| Project | Relevant? | Applied? | Notes |
|---------|-----------|----------|-------|
| _shared | Yes | Yes | In CLAUDE.md |
| _example-project | Maybe | — | Depends on project complexity; include if >5 files |

**Summary:** Most valuable for larger projects. Overkill for simple ones.

---

*Add new evaluations above this line*
