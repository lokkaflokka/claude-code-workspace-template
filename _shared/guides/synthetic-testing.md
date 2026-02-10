# Synthetic Testing with AI Personas

A methodology for building tiered automated testing that scales from scripted flows to LLM-driven exploration. Use this when you need to test for emotional, cognitive, and experiential UX concerns that traditional testing misses.

## When to Use

- **Any app with a UI**, especially when testing for concerns like confusion, shame, frustration, or cognitive overload that automated functional tests can't catch
- **Products serving diverse user populations** where 3-5 user archetypes span meaningfully different needs
- **Apps where tone and emotional safety matter** — health, wellness, finance, education, caregiving
- **When you need to scale beyond manual QA** but want richer signal than "does the button work?"

## The 5-Tier Framework

Each tier builds on the previous. Start at Tier 0 — it's independently useful. Stack upward as your needs grow.

| Tier | What | Determinism | Speed | Cost | When to Run |
|------|------|-------------|-------|------|-------------|
| **0: Scripted flows** | Core user journeys with handcrafted fixtures | Deterministic | Seconds | Free | Every deploy (CI) |
| **1: Fuzzed inputs** | Same flows, randomized input data | Semi-random, seeded | Seconds | Free | Every deploy (CI) |
| **2: Combinatorial personas** | Generated personas from a constrained grammar | Generated, seeded | Minutes | Free | Nightly / pre-release |
| **3: LLM-as-user** | Vision LLM drives real browser sessions | Non-deterministic | Minutes | $5-15/run | Weekly / before milestones |
| **4: Multi-session arcs** | Stateful testing across simulated days | Non-deterministic | Slow | $10-30/run | Monthly / major releases |

**Tier 4 is architecture-only** — included for completeness but not detailed here. Tiers 0-3 are fully implementable.

---

## Tier 0: Scripted Flows + Rubric Evaluation

The foundation. Define 3-5 user archetypes, write Playwright/Cypress flows that model their journeys, and evaluate screenshots with a vision LLM against a structured rubric.

### Step 1: Design User Archetypes

Define 3-5 archetypes that span your user space. Each archetype should occupy a unique position in your design constraint space — not just demographics, but behavioral and cognitive differences that affect how they use your product.

**Good archetype dimensions (pick 3-5 relevant to your domain):**
- Capture style (terse vs. verbose vs. brain-dump)
- Energy/motivation level (high vs. variable vs. low)
- Tech comfort (power user vs. casual vs. hesitant)
- Engagement pattern (daily vs. sporadic vs. lapsed)
- Domain-specific traits (risk tolerance for finance, condition severity for health, etc.)

**Each archetype needs:**
- A name and 2-3 sentence profile
- Fixture data matching their behavioral pattern (what they'd actually type/click)
- Expected system responses for their inputs
- Multi-day sequences (Day 1 behavior, Day 2, Day 7 return)

### Step 2: Write Playwright Flows

Each archetype gets a scripted journey through your app. Use fixture data that matches their profile.

```typescript
// Example: scripted Day 1 flow for a "power user" archetype
test('Power User — Day 1: Rapid multi-item capture', async ({ page }) => {
  await setupTest(page, '2026-03-01', 8); // Mock date + time

  await page.goto('/');
  await captureItem(page, 'Q1 board deck, review hiring pipeline, 1:1 with Jamie');
  await waitForAnimations(page);

  // Verify items were split and stored correctly
  const items = await getStoredItems(page);
  expect(items).toHaveLength(3);

  // Verify timing inference
  const deck = items.find(i => i.text.includes('board deck'));
  expect(deck.timing).toBe('today');
});
```

**Key patterns:**
- Mock date/time so tests are deterministic (`addInitScript` for Date override)
- Use helper functions (`captureItem`, `getStoredItems`, `waitForAnimations`) to keep flows readable
- Test the full loop: input → processing → display → persistence → cross-day carry-forward
- Each archetype tests different paths (voice capture, text capture, multi-item, single-item, etc.)

### Step 3: Build an Evaluation Rubric

Don't ask the LLM "is this good?" — give it a structured rubric with dimensions and scoring criteria.

**Example rubric dimensions:**
- **Clarity** (1-5): Can the user understand what's happening at each step?
- **Emotional safety** (1-5): Does the interface avoid shame, guilt, or pressure?
- **Cognitive load** (1-5): Is the decision burden appropriate for the target user?
- **Information density** (1-5): Is text readable, appropriately sized, well-spaced?
- **Navigation** (1-5): Can the user always tell where they are and how to proceed?
- **Accessibility** (1-5): Are elements labeled, tappable areas adequate, contrast sufficient?

**Per-archetype weighting:** Not all dimensions matter equally for all archetypes. A power user cares about efficiency; a hesitant user cares about clarity. Weight the rubric per archetype.

### Step 4: Vision LLM Evaluation

Send screenshots to a vision-capable LLM with the rubric. Get structured scores, not open-ended opinions.

```typescript
const evaluationPrompt = `
Evaluate this screenshot for the "${archetype.name}" user archetype.
Score each dimension 1-5:

${rubric.dimensions.map(d => `- ${d.name}: ${d.description}`).join('\n')}

For each dimension, provide:
- Score (1-5)
- Evidence (what you see in the screenshot that supports the score)
- Red flags (anything that would concern this specific user type)

Output as JSON.
`;
```

**Property assertions (apply to ALL archetypes, ALL screenshots):**
- App never crashes or shows an error state
- App never loses previously entered data
- App never displays shame language, urgency indicators, or guilt-inducing copy
- Text entered by the user is visible somewhere after submission
- Navigation always works (no dead ends, back button always available)

---

## Tier 1: Fuzzed Inputs

Keep the Playwright flow structure from Tier 0 but replace hardcrafted fixture data with input generators. Same flows, wildly different data each run.

### Seeded Randomness

Use a deterministic PRNG (like Mulberry32) so every run is reproducible from its seed. Log the seed — when a test fails, re-run with the same seed to reproduce.

```typescript
class SeededRandom {
  private state: number;
  readonly seed: number;

  constructor(seed: number) {
    this.seed = seed;
    this.state = seed;
  }

  next(): number {
    // Mulberry32 algorithm
    this.state |= 0;
    this.state = (this.state + 0x6d2b79f5) | 0;
    let t = Math.imul(this.state ^ (this.state >>> 15), 1 | this.state);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  }

  int(min: number, max: number): number {
    return Math.floor(this.next() * (max - min + 1)) + min;
  }

  pick<T>(arr: readonly T[]): T {
    return arr[this.int(0, arr.length - 1)];
  }
}
```

### Generator Categories

Build generators for each input type your app accepts:

| Category | Examples | What it tests |
|----------|----------|--------------|
| **Timing language** | "tomorrow-ish", "before 3 if possible", "whenever" | Natural language parsing robustness |
| **Delimiter mixing** | "do laundry and also text mom, oh and return books" | Item splitting with mixed separators |
| **Item count extremes** | 0 items (empty submit), 1 word, 50 comma-separated items | Boundary handling |
| **Abbreviations + slang** | "tmrw", "2nite", "nvm", "idk maybe gym?" | Informal input handling |
| **Unicode + emoji** | Items with emoji, non-Latin characters, smart quotes | Character encoding |
| **Very long strings** | 500-character single items, items with URLs | Overflow and truncation |
| **Typos** | "gorceries", "tex mom" | Graceful degradation |

### Universal Property Assertions

These must hold for ANY generated input — they're your invariants:

```typescript
// After any fuzzed input submission:
expect(appDidNotCrash).toBeTruthy();
expect(previousDataIntact).toBeTruthy();
expect(noShameLanguage).toBeTruthy();
expect(capturedTextVisibleSomewhere).toBeTruthy();
expect(noNavigationDeadEnds).toBeTruthy();
```

---

## Tier 2: Combinatorial Persona Generation

Scale from 5 hand-crafted archetypes to 50+ synthetic personas. Each generated persona is a plausible combination of traits, filtered by domain knowledge to avoid nonsensical combinations.

### Define a Persona Grammar

Identify the trait dimensions relevant to your domain:

```typescript
// Example dimensions — customize for your product
type CaptureStyle  = 'terse' | 'brain-dump' | 'verbose' | 'mixed';
type EnergyPattern = 'consistent' | 'variable' | 'low-baseline';
type TechComfort   = 'low' | 'medium' | 'high';
type LifeContext   = 'student' | 'parent' | 'professional' | 'retired';
type ReturnPattern = 'daily' | 'sporadic' | 'lapsed';
// ... add domain-specific dimensions
```

The full combinatorial space is enormous (thousands of combinations). Most are meaningless.

### Domain Constraints: Rejection + Affinity Rules

Not all trait combinations are valid. Use two types of rules:

**Rejection rules** eliminate implausible combinations:
```typescript
// Example: low energy + verbose capture is rare
const REJECTION_RULES = [
  (t) => t.energyPattern === 'low-baseline' && t.captureStyle === 'verbose',
  (t) => t.techComfort === 'low' && t.captureStyle === 'brain-dump',
  // ... domain-specific rules
];
```

**Affinity rules** boost clinically or empirically validated pairings:
```typescript
// Example: high anxiety + terse capture is a well-documented pattern
const AFFINITY_RULES = [
  { check: (t) => t.anxiety && t.captureStyle === 'terse', boost: 2.0 },
  // ... domain-specific rules
];
```

### Generation via Rejection Sampling

```
1. Generate random trait combination
2. Check against rejection rules → discard if implausible
3. Calculate affinity score → accept probabilistically (higher affinity = more likely)
4. Map to nearest core archetype for evaluation weighting
5. Generate profile text, capture examples, behavioral config
6. Repeat until you have N personas
```

### Archetype Affinity Mapping

Each generated persona maps to its nearest core archetype. This determines which rubric weights apply during evaluation — a generated persona similar to your "power user" archetype gets evaluated with power-user rubric weights.

### Seeded Generation

Use the same seeded PRNG from Tier 1. Log the seed. Same seed = same 50 personas = reproducible results. Change the seed to explore different regions of your persona space.

---

## Tier 3: LLM-as-User

The paradigm shift. Instead of scripted `page.fill()` / `page.click()`, a vision LLM drives the browser session — seeing screenshots, deciding what to do, and narrating what the persona is thinking.

### The Decision Loop

```
┌──────────────────────────────────────────┐
│  Persona Profile + Constraints           │
└────────────────┬─────────────────────────┘
                 │
                 v
┌──────────────────────────────────────────┐
│  Screenshot → Vision LLM                 │
│  Input: screenshot + persona + history   │
│  Output: monologue + action              │
└────────────────┬─────────────────────────┘
                 │
                 v
┌──────────────────────────────────────────┐
│  Browser Executor                        │
│  Translates semantic action → DOM        │
│  Takes screenshot after each action      │
└────────────────┬─────────────────────────┘
                 │
                 v
┌──────────────────────────────────────────┐
│  Session Transcript                      │
│  Action log + screenshots + monologue    │
└──────────────────────────────────────────┘
```

### Action Schema

Keep the action surface area small and semantic. The LLM produces user-facing actions; the executor translates to DOM interactions.

```typescript
type LLMAction =
  | { action: 'type_text'; text: string }
  | { action: 'tap_button'; label: string }
  | { action: 'tap_item'; text: string }
  | { action: 'scroll'; direction: 'up' | 'down' }
  | { action: 'wait'; seconds: number }
  | { action: 'give_up'; reason: string }   // persona abandons — this is signal
  | { action: 'done'; reason: string };      // natural session end
```

**Key design choice:** `give_up` is a valid outcome, not a test failure. When a persona gives up, the reason in the monologue tells you exactly what went wrong from their perspective.

### Persona Prompt: Cognitive Constraints, Not Preferences

The persona prompt encodes how the person thinks, not just what they want. This is what makes LLM-as-user different from scripted testing.

```
You are roleplaying as [Name], a real person using this app for the
first time. You are NOT a tester — you are a real human with real
constraints.

Your cognitive constraints:
- You get distracted mid-task and might forget what you were about to do
- You have limited activation energy — if this asks too much, you'll close it
- Ambiguity makes you uncomfortable — you need things to feel "done"

Rules:
- You can give up at ANY time. That is valuable data, not failure.
- If you're confused, say so. Don't figure it out like a tester would.
- If something feels shaming or pressuring, react honestly.
- Act naturally for your persona — impulsive people tap fast, low-energy
  people might quit early.
```

### The Internal Monologue: The Key Innovation

At each step, the LLM narrates what the persona is thinking. This is evaluation data that static screenshot analysis cannot produce.

```
MONOLOGUE: I just typed three things but I can't tell if they saved.
The screen looks the same. Am I supposed to do something else? I feel
a little anxious that my stuff might be lost.

ACTION: {"action": "scroll", "direction": "down"}
```

The monologue captures:
- **Confusion** — "I don't know what this button does"
- **Frustration** — "I already did this, why is it asking again?"
- **Shame** — "Seeing all these undone items makes me feel bad"
- **Delight** — "Oh, it remembered what I said yesterday. That's nice."
- **Cognitive overload** — "There's too much on this screen"

### Finding Extraction

Pattern-match the monologue for signal categories:

```typescript
const FINDING_PATTERNS = [
  { pattern: /confus/i,           category: 'confusion' },
  { pattern: /frustrat/i,         category: 'frustration' },
  { pattern: /shame|ashamed/i,    category: 'shame' },
  { pattern: /overwhelm/i,        category: 'overload' },
  { pattern: /lost|missing|gone/i, category: 'data-loss-fear' },
  { pattern: /nice|love|good/i,   category: 'delight' },
  // Add patterns relevant to your product's concerns
];
```

Aggregate findings across all personas to identify systemic issues vs. one-off reactions.

### Provider Abstraction

Abstract the vision LLM behind an interface so you can swap models without changing test logic:

```typescript
interface VisionProvider {
  name: string;
  sendScreenshot(
    systemPrompt: string,
    userText: string,
    screenshotBase64: string
  ): Promise<{ text: string; inputTokens: number; outputTokens: number }>;
  estimateCost(inputTokens: number, outputTokens: number): number;
}
```

Implement for each provider (Anthropic, Google Gemini, OpenAI, etc.). This enables the tiered model selection described below.

---

## Tiered Model Selection

Scale model quality with testing scope. You don't need the best model for every iteration.

### Test Profiles

| Profile | Model | Personas | Steps | Parallelism | Cost | Use Case |
|---------|-------|----------|-------|-------------|------|----------|
| **dev** | Free tier (e.g., Gemini Flash) | 3 | 8 | Serial | $0 | Quick iteration — "did I break something?" |
| **sweep** | Mid-tier (e.g., Haiku-class) | 15 | 12 | Parallel | ~$0.50-1 | Broader coverage before a decision |
| **full** | High-tier (e.g., Sonnet-class) | 50 | 12 | Parallel | ~$8-15 | Comprehensive eval for milestones |

### Profile Configuration

```typescript
interface TestProfile {
  name: string;
  provider: VisionProvider;
  personaCount: number;
  maxSteps: number;
  parallel: boolean;
}

// Resolve from environment: LLM_TIER=dev|sweep|full
// Override individual settings: LLM_PERSONA_COUNT=5, LLM_PROVIDER=gemini
```

### The Key Insight

You don't need rich emotional monologues on every dev iteration. A free-tier model can tell you "persona got stuck on step 3" for zero cost. Save the expensive models for evaluation runs that feed product decisions or documentation.

---

## Cost Model

### Per-Session Estimates

| Component | Cost per step | Steps/session | Cost/session |
|-----------|--------------|---------------|-------------|
| Free tier (Gemini Flash) | $0 | 8-12 | $0 |
| Mid-tier (Haiku-class) | ~$0.002-0.005 | 12 | ~$0.03-0.06 |
| High-tier (Sonnet-class) | ~$0.01-0.03 | 12 | ~$0.12-0.36 |

### Budget Planning

| Run type | Personas | Model | Est. cost |
|----------|----------|-------|-----------|
| Dev iteration | 3 | Free tier | $0 |
| Pre-decision sweep | 15 | Mid-tier | $0.50-1 |
| Milestone eval | 50 | High-tier | $8-15 |
| Weekly budget (1 dev + 1 sweep) | — | Mixed | $0.50-1/week |
| Monthly budget (4 weekly + 1 full) | — | Mixed | $10-20/month |

---

## Implementation Order

**Build Tier 0 first.** It's independently useful — you get deterministic regression tests and rubric-evaluated screenshots even if you never build the other tiers. Each subsequent tier compounds the previous.

```
Tier 0: Scripted flows + rubric evaluation
  └─ Foundation: archetype fixtures, Playwright flows, evaluation rubric
  └─ Value: "these 5 paths always work"

Tier 1: Fuzzed inputs
  └─ Add: seeded PRNG, input generators, property assertions
  └─ Value: "the app handles messy real-world input"

Tier 2: Combinatorial personas
  └─ Add: persona grammar, constraint rules, generation pipeline
  └─ Value: "50 plausible users can complete Day 1"

Tier 3: LLM-as-user
  └─ Add: vision provider, action executor, monologue extraction
  └─ Value: "we know what confused/frustrated/delighted users feel"
```

**Recommended build triggers:**
- Tier 0: Build immediately when you have a testable UI
- Tier 1: Build when you have a working Tier 0 and want input robustness
- Tier 2: Build when your archetype count feels limiting (~5-10 isn't enough)
- Tier 3: Build when you need unknown-unknown discovery or emotional/cognitive UX signal

---

## Using This in a Claude Code Session

### Getting Started (Tier 0)

```
1. Define 3-5 archetypes in a PERSONAS.md or fixtures file
2. Write Playwright flows for each archetype's core journey
3. Build an evaluation rubric (dimensions + scoring + per-archetype weights)
4. "Evaluate these screenshots against the rubric for [archetype]"
5. Iterate on the UI until all archetypes pass
```

### Scaling Up (Tiers 1-3)

```
1. "Add fuzz generators for [input type] — timing language, delimiters, etc."
2. "Build a persona generator with these trait dimensions: [list]"
3. "Set up LLM-as-user with the dev profile (Gemini free tier, 3 personas)"
4. "Run a sweep (15 personas, mid-tier model) before we ship this"
```

### File Organization

```
your-project/
  tests/synthetic/
    drivers/
      vision-provider.ts     # Provider interface
      anthropic-provider.ts  # Anthropic implementation
      gemini-provider.ts     # Gemini implementation
      llm-user.ts            # LLM-as-user driver
      test-profiles.ts       # dev/sweep/full profiles
    generators/
      input-generators.ts    # Seeded fuzz generators (Tier 1)
      persona-generator.ts   # Combinatorial personas (Tier 2)
    flows/
      helpers.ts             # Shared test utilities
      archetype-1.spec.ts    # Tier 0 scripted flows
      archetype-2.spec.ts
      fuzz.spec.ts           # Tier 1 fuzzed inputs
      combinatorial.spec.ts  # Tier 2 batch persona testing
      llm-as-user.spec.ts    # Tier 3 LLM-driven sessions
    evaluation/
      rubric.ts              # Scoring dimensions + weights
      evaluate.ts            # Vision LLM evaluation engine
    results/                 # Transcripts, screenshots, reports
```

---

## Anti-Patterns

| Anti-pattern | Why it fails | Fix |
|-------------|-------------|-----|
| **Starting at Tier 3** | No regression foundation; no way to tell if LLM findings are real or artifacts | Build Tier 0 first — scripted tests are your ground truth |
| **Evaluating without a rubric** | LLM opinions without structure are noise | Define dimensions, scores, and per-archetype weights before evaluating |
| **Testing only happy paths** | Real users send "tmrw", mistype, paste URLs, submit empty forms | Tier 1 fuzz testing exists for exactly this reason |
| **Non-reproducible generators** | "It failed yesterday but I can't reproduce it" | Seed everything; log the seed; same seed = same run |
| **LLM navigating core test paths** | Non-deterministic; can't tell if failures are app bugs or LLM confusion | Use Playwright for deterministic flows (Tier 0); reserve LLM for exploration (Tier 3) |
| **One-size-fits-all evaluation** | A power user and a hesitant user have different rubric priorities | Weight rubric dimensions per archetype |
| **Ignoring "give up" outcomes** | Persona abandonment is your most valuable signal | Read the monologue — it tells you exactly where the UX broke |
| **Running full-tier on every commit** | Expensive and slow; blocks CI | Dev tier on every commit ($0); full tier weekly or at milestones |
