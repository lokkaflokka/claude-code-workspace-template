# Simulated Expert Panels

A methodology for running multi-round validation panels with persistent AI personas in Claude Code sessions. Use this when you need structured disagreement — not confirmation bias — before making a decision.

## When to Use

- **Product decisions** — Feature prioritization, MVP scoping, go/no-go calls
- **System design reviews** — Architecture choices, API design, migration strategies
- **Career decisions** — Job offers, role transitions, negotiation strategies
- **Risk assessment** — Launch readiness, investment decisions, partnership evaluation
- **Content review** — Article drafts, documentation, pitch decks

The common thread: decisions where multiple valid perspectives exist and you want to stress-test your thinking before committing.

## Core Concepts

### Persistent Personas

Each panelist is a defined persona stored in a file that persists across sessions. Unlike one-off prompts ("pretend you're a VC"), persistent personas accumulate context — they remember what they reviewed in Round 1 when evaluating Round 3 material. Their verdicts evolve as evidence accumulates.

### Verdict Evolution

Track how each persona's position shifts across rounds. A skeptic who moves from "would not recommend" to "cautiously optimistic" after seeing your fixes is stronger signal than a supporter who was always positive. The trajectory matters more than any single verdict.

### Convergence Detection

When 2+ independent exercises (panels, pre-mortems, steelman attempts) flag the same risk from different angles, that's high-confidence signal. Convergence across methods is more reliable than unanimity within one method. If your VC persona and your end-user persona both flag the same concern from completely different lenses, pay attention.

### Panel Rotation

Different review targets need different panelists. A strategy review needs a business lens; a UX review needs design expertise. Rotate panelists in and out based on what you're reviewing, while keeping 1-2 consistent personas for continuity.

## Setup: Defining Personas

### Create a PERSONAS.md File

Store persona definitions in your project directory. This file persists across Claude Code sessions — each session reads it before running a round.

### Design 3-5 Personas with Distinct Lenses

Each persona needs:

| Element | Purpose | Example |
|---------|---------|---------|
| **Role/expertise** | Professional lens they evaluate through | "Series A investor focused on consumer products" |
| **Evaluation lens** | What they prioritize | TAM, unit economics, competitive moat |
| **Hard questions** | 3-5 questions they always ask | "Where is the moat?", "Show me a retention curve" |
| **Initial verdict state** | Starting position (updated after each round) | "Would not invest — unvalidated market" |

### Persona Design Principles

**Maximize disagreement surface.** If all your personas agree on everything, you've built a confirmation panel. Good persona sets have built-in tensions:

- **Builder vs. Buyer** — "Is this technically sound?" vs. "Would I actually use this?"
- **Optimist vs. Skeptic** — "Here's why this could work" vs. "Here's why this will fail"
- **Domain expert vs. Outsider** — "This violates field norms" vs. "Users don't care about field norms"
- **Speed vs. Quality** — "Ship something" vs. "Get it right first"

**Archetypes that work across domains:**
- The Skeptical Investor (business viability, market size, defensibility)
- The Burned End User (has tried 5 alternatives, trust is earned not given)
- The Incumbent Competitor (strategic threat assessment, what they'd steal vs. dismiss)
- The Domain Expert (clinical, legal, technical — depends on your space)
- The Craft Specialist (design, engineering, writing — evaluates execution quality)

### Example Persona Structure

```markdown
### Persona 1: Skeptical Early-Stage Investor

**Lens:** Market size, defensibility, unit economics, venture-scale potential

**Profile:** Consumer-focused seed investor. Has seen hundreds of pitches
in this space. Respects craft but invests in markets, not products.

**Hard questions:**
1. Where is the moat? (Features are copyable in a sprint)
2. Show a retention curve (anecdotes are not data)
3. Venture-scale or lifestyle business? (Be honest)
4. Go-to-market without budget?
5. Why now? ("AI is better" is not sufficient)

**Current state (after Round N):**
- Verdict: [updated after each round]
- Key concerns: [accumulated across rounds]
- Mind-changer: [what would shift their verdict]
```

## Running a Round

### Before Each Round

1. Read `PERSONAS.md` (current persona states)
2. Read the artifact or decision being reviewed
3. Note what's new since the last round (if applicable)

### During the Round

Present the material to each persona. Each produces:

1. **Verdict shift** — How their position changed (or didn't) from the previous round
2. **Key concerns** — Specific, actionable issues (not vague worries)
3. **What would change their mind** — Concrete evidence or changes that would shift their verdict

### After the Round

1. **Cross-persona convergence analysis** — What do multiple personas flag independently?
2. **Update persona states** in `PERSONAS.md` — Verdicts, accumulated concerns, mind-changers
3. **Identify actions** — What to address before the next round

### Round Output Template

```markdown
## Round N: [Review Target]

### [Persona Name]
**Verdict shift:** [Previous] → [Current]
**Key concerns:** [Bulleted list]
**Mind-changer:** [Specific condition]

### Cross-Persona Convergence
**Universal agreement:** [What all/most agree on]
**Strongest signal:** [Concerns flagged by 2+ personas independently]
**Tensions:** [Where personas disagree — these are real trade-offs]
```

## Multi-Round Evolution

### Round 1: Surface Reactions

Present the core concept or artifact. Expect broad concerns, first impressions, and identification of the biggest risks. Don't try to address everything — catalog it.

### Rounds 2-3: Deep Dives

Address the top concerns from Round 1 with new material (revised designs, additional analysis, data). Each persona re-evaluates with accumulated context. Track which concerns were resolved vs. which persist.

### Rounds 4-5: Convergence Assessment

By now, the persistent concerns are the real ones. Transient concerns were addressed in earlier rounds. Focus on:
- Which concerns survived multiple rounds of evidence?
- Where did personas converge from initially different positions?
- What are the remaining "agree to disagree" tensions (real trade-offs, not resolvable)?

### When to Stop

- **Convergence achieved** — Personas agree on the key risks and your mitigations address them
- **Diminishing returns** — New rounds aren't surfacing new concerns
- **Decision forcing** — You have enough signal to decide, even if not everyone agrees
- **Typically 3-5 rounds.** Fewer than 3 doesn't build enough accumulated context. More than 5 usually means you're avoiding a decision.

## Panel Rotation

Not every persona belongs in every round. Rotate based on what you're reviewing:

| Review target | Good panelists | Poor fit |
|--------------|----------------|----------|
| Business strategy | Investor, Competitor, Domain Expert | UX Specialist |
| Interface design | End User, UX Specialist, Domain Expert | Investor |
| Technical architecture | Engineering Lead, Security Reviewer | Investor, End User |
| MVP scope | Investor, End User, Competitor | Design Systems Lead |
| Go-to-market | Investor, Competitor, End User | Engineering Lead |

Keep 1-2 personas consistent across all rounds for continuity. Rotate the others based on the review target.

## Convergence as Decision Signal

The most valuable output of a multi-round panel isn't any single persona's verdict — it's the convergence pattern.

**Strong signals:**
- 3+ personas independently flag the same risk from different lenses → high-confidence issue
- A skeptic shifts toward approval after seeing your response → your fix actually works
- All personas agree on one thing but disagree on everything else → that one thing is your priority

**Weak signals:**
- One persona is very concerned but others aren't → may be lens-specific, not universal
- All personas approve enthusiastically → may indicate insufficient disagreement in your panel design
- Concerns shift every round without resolving → you may be addressing symptoms, not root causes

## Example: 5-Round Product Concept Validation

A condensed illustration using a mobile app concept:

| Round | Trigger | Panel | Key finding |
|-------|---------|-------|-------------|
| 1 | Initial concept + competitive analysis | Investor, End User, Competitor, Domain Expert | Core value proposition unclear to investor; end user cautiously interested; competitor not threatened |
| 2 | + Engagement model + differentiation thesis | Same | End user upgraded to "would try"; competitor identified 2 features worth monitoring; domain expert raised safety concern |
| 3 | Detailed UX design | End User, Domain Expert, **UX Specialist** (replaces Investor) | UX specialist found 5 critical issues; domain expert flagged 3 population-specific risks; end user responded to fixes positively |
| 4 | Revised designs addressing Round 3 | End User, Domain Expert, UX Specialist, **Behavioral Designer** (new) | 4/5 critical UX issues resolved; behavioral designer identified engagement sustainability risk |
| 5 | MVP scope definition | **Investor** (returns), End User, Domain Expert, **Competitor** (returns) | Convergence: 3/4 flagged same deferred feature as trust-breaking → moved to MVP |

**The Round 5 convergence was the key decision:** Three personas independently identified the same deferred feature as the highest risk. That cross-lens agreement was stronger signal than any single persona's emphasis.

## Using This in a Claude Code Session

### Session Workflow

```
1. Read PERSONAS.md (persona states from last round)
2. Read the artifact being reviewed
3. "Run Panel Round N on [artifact]. Each persona evaluates
   through their lens. Produce verdict shifts, concerns,
   convergence analysis."
4. Review output, discuss with Claude
5. Update PERSONAS.md with new states
6. Decide: address concerns and run another round, or decide
```

### File Organization

```
your-project/
  PERSONAS.md              # Persona definitions + evolving state
  research/
    panel_round_1.md       # Full round output (archive)
    panel_round_2.md
    ...
```

### Tips

- **Persist round outputs** — Each round's full analysis goes in a file. Don't rely on conversation memory.
- **Update persona states immediately** — Before the session ends, update verdicts and concerns in PERSONAS.md.
- **Reference previous rounds** — When running Round 3, tell Claude to read Rounds 1-2 for context. Accumulated context is the whole point.
- **Don't cherry-pick** — If a persona raises a concern you disagree with, engage with it. The value is in the disagreement.

## Anti-Patterns

| Anti-pattern | Why it fails | Fix |
|-------------|-------------|-----|
| **Personas that all agree** | Confirmation theater — you learn nothing | Add a persona whose lens naturally conflicts with your thesis |
| **Resetting context between rounds** | Loses accumulated insight; every round starts from scratch | Persist persona states in a file; read it before each round |
| **Using panels for data questions** | "Will users retain?" needs data, not judgment | Use panels for judgment calls; get data for empirical questions |
| **Addressing every concern equally** | Not all concerns are equal weight | Use convergence (multi-persona agreement) as your priority signal |
| **Running panels without artifacts** | Abstract discussions produce abstract feedback | Always review a specific document, design, or decision — not a vague concept |
| **Too many personas** | Beyond 5-7, you get noise not signal | 3-5 personas with distinct lenses covers most decision surfaces |
