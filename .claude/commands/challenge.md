# /challenge

Run a persona-based challenge against a decision, plan, or artifact. Supports two modes: quick challenge (individual personas) and review panel (sequential with veto power).

## Inputs

The user provides:
1. **The artifact** — a decision, plan, proposal, spec, code design, or document to challenge
2. **Mode** (ask if not specified):
   - **Quick** — 2-3 independent persona perspectives, single round
   - **Panel** — Sequential review with veto power, for technical/distributable artifacts

## Quick Challenge Mode

For decisions, proposals, plans, career moves, and general stress-testing.

### Step 1: Select Personas (2-3)

Choose personas whose lenses create productive tension with the artifact. If the user doesn't specify, select from these archetypes or invent domain-appropriate ones:

| Archetype | Lens | Good for |
|-----------|------|----------|
| Skeptical Investor | Market size, defensibility, ROI | Business decisions, product strategy |
| Burned End User | Trust, friction, alternatives tried | UX, product, feature scoping |
| Incumbent Competitor | Strategic threat, what to steal/dismiss | Positioning, differentiation |
| Domain Expert | Field norms, clinical/legal/technical risk | Regulated domains, novel approaches |
| Craft Specialist | Execution quality, maintainability | Code, design, writing |
| Skeptical Hiring Manager | Evidence, signal vs. noise | Career moves, pitches |
| Devil's Advocate | Weakest assumptions, failure modes | Any high-stakes decision |

### Step 2: Run Challenge

For each persona, produce:
- **Verdict:** Support / Cautious / Oppose (with one-line rationale)
- **Top concerns:** 2-3 specific, actionable issues
- **Mind-changer:** What evidence or change would shift their verdict

### Step 3: Convergence Analysis

- **Cross-persona agreement:** Concerns flagged by 2+ personas independently = strongest signal
- **Tensions:** Where personas disagree = real trade-offs to acknowledge
- **Recommended actions:** Prioritized by convergence strength

### Output Format

```
## Challenge: [artifact name]

### [Persona 1 Name] ([Archetype])
**Verdict:** [Support/Cautious/Oppose] — [one line]
**Concerns:** [bulleted]
**Mind-changer:** [specific condition]

### [Persona 2 Name]
...

### Convergence
**Strongest signal:** [concerns with multi-persona agreement]
**Tensions:** [real trade-offs]
**Actions:** [what to fix/test/defer]
```

## Review Panel Mode

For technical artifacts, system designs, specs, and distributable code. Sequential review with veto power.

### Step 1: Assemble Panel (2-3 reviewers)

Default panel for technical artifacts:
1. **Staff Engineer** — architecture, maintenance burden, over-engineering, abstraction level
2. **Security/DevOps** — injection vectors, access scope, failure modes, operational safety
3. (Optional) **Product/UX** — learnability, user-facing error messages, degradation paths

Swap reviewers based on artifact type:
- UI feature → replace Security with Designer
- Data pipeline → add Data Engineer
- API design → add API Consumer persona

### Step 2: Sequential Review

Each reviewer evaluates independently, then sees prior reviewers' concerns. Each produces:
- **Hard requirements:** Must fix before shipping (veto power)
- **Recommendations:** Add to Known Risks or Future Enhancements
- **Verdict:** Ship / Ship with fixes / Block

### Step 3: Iterate

If any hard requirements exist:
1. Address hard requirements
2. Re-run affected reviewers (not full panel)
3. Repeat until all hard requirements resolved

### Output Format

```
## Review Panel: [artifact name]

### Reviewer 1: Staff Engineer
**Verdict:** [Ship/Ship with fixes/Block]
**Hard requirements:**
- [must fix — veto]
**Recommendations:**
- [nice to have]

### Reviewer 2: Security/DevOps
...

### Panel Summary
**Hard requirements (must fix):** [count]
**Recommendations (defer OK):** [count]
**Ship decision:** [Ship/Blocked — list blockers]
```

## Multi-Round Panels

For high-stakes decisions needing deeper exploration, use persistent personas across rounds. Full methodology: `_shared/guides/simulated-expert-panels.md`.

Setup:
1. Create `PERSONAS.md` in the project directory with persona definitions and evolving state
2. Run rounds, updating persona states after each
3. Track verdict evolution — a skeptic moving to approval is stronger signal than a supporter staying positive
4. Stop when convergence is achieved or diminishing returns (typically 3-5 rounds)

## Rules

- **Maximize disagreement.** If all personas agree, the panel is too friendly. Add a contrarian lens.
- **Convergence > individual emphasis.** When 2+ personas flag the same concern from different angles, that's the priority — stronger than any single persona's strong opinion.
- **Be specific.** "This might not work" is not a concern. "The auth token refresh has no retry logic, which will cause silent failures on flaky connections" is.
- **Don't cherry-pick.** Engage with uncomfortable feedback. The value is in the disagreement.
