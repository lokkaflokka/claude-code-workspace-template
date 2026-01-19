# Claude Code Workspace Template

A starting structure for using Claude Code as a **personal operating system** — with session continuity, cross-project learning, and compounding context.

## The Problem

Claude Code sessions start cold. Every conversation begins from zero. Context doesn't persist. Lessons learned in one project don't transfer to others. You end up re-explaining the same things, re-discovering the same mistakes, and losing continuity between sessions.

This gets worse as you use Claude Code for more things. Multiple projects, different domains, evolving knowledge — without structure, it's chaos.

## The Solution

A lightweight structure that gives Claude:

- **Session continuity** — Claude knows what you were working on and what needs attention
- **Project context** — Each project has rules, patterns, and mistakes Claude should know
- **Cross-project learning** — Techniques documented once, evaluated everywhere, applied where relevant
- **Self-improvement** — The system applies its own patterns to itself

## Who This Is For

This template is for people who:
- Use Claude Code seriously, across multiple projects or domains
- Want Claude to get better over time, not start fresh every session
- Think in systems and are willing to maintain lightweight infrastructure
- May be using Claude Code for knowledge work, not just coding

If you just need Claude for quick one-off tasks, this is overkill. If you're building a long-term relationship with Claude as a thinking partner, read on.

## Quick Start

1. Clone or fork this repo into your workspace root
2. Rename `_example-project/` to your first project
3. Customize `_shared/PROJECTS.md` with your project list
4. Customize each project's `CLAUDE.md` with project-specific context
5. Start a session at the root — Claude will read `CURRENT_STATE.md` and orient itself

## Your First Session

The template is scaffolding. The value comes from using it. Here's how to bootstrap a working system:

### Session 1: Set Up Your First Project

1. **Start Claude Code at the workspace root**

   Claude will read `CLAUDE.md` and `_shared/CURRENT_STATE.md`. Since everything is placeholder text, that's fine — you're about to change that.

2. **Tell Claude about a real project you want to track**

   Example prompts:
   - "I want to set up a project for tracking my home renovation"
   - "I have a side project called X — help me create a knowledge base for it"
   - "I'm learning Spanish — let's set up a project to track resources and progress"

3. **Let Claude help you customize the project CLAUDE.md**

   Claude will rename `_example-project/`, update the Overview, Key Files, and Query Routing sections based on what you describe.

4. **Before ending the session, update state**

   Say: "Update CURRENT_STATE.md with what we worked on today."

   Claude will fill in the Last Session section with real content. Now you have state.

### Sessions 2-5: Build Momentum

Each session, Claude will read your state and tell you where you left off. As you work:

- **When Claude makes a mistake**, say "Add that to Common Mistakes" — now it won't happen again
- **When you finish something**, say "Update CURRENT_STATE.md" — now you have continuity
- **When you discover a useful pattern**, consider documenting it in `TECHNIQUES.md`

By session 5, you'll have:
- A project with real context Claude remembers
- A few mistakes Claude won't repeat
- Session history showing what you've worked on
- Maybe a technique worth tracking

This is when the system starts feeling different from a blank Claude session.

### What It Looks Like After a Few Weeks

Here's a realistic example of what `_shared/CURRENT_STATE.md` might look like once you've been using the system:

```markdown
# Meta-Level Current State

As-of date: 2026-02-15

---

## ALERTS (surface on session start)

### Pending Evaluations
| Priority | Technique | Documented | Status |
|----------|-----------|------------|--------|
| 🟡 | "Decision Log Template" | 2026-02-10 | Needs evaluation |

---

## Working Memory (expires)

| Note | Expires | Context |
|------|---------|---------|
| Contractor quote expires | 2026-02-20 | Kitchen renovation — need to decide by Friday |
| Library books due | 2026-02-18 | Spanish learning resources, can renew once |

---

## Last Session

**Date:** 2026-02-15
**Focus:** Home renovation — comparing contractor quotes
**Outcome:** Created comparison spreadsheet, identified key questions
**Left off:** Need to call references for top two contractors
```

And here's what a project's Common Mistakes section might look like after real use:

```markdown
## Common Mistakes

### Data Handling
- **Don't assume prices are current** — Always check the quote date before making comparisons. Quotes typically expire after 30 days. (discovered 2026-01-20: compared a stale quote against fresh ones)

### Recommendations
- **Don't suggest scheduling without checking the calendar** — Read CURRENT_STATE.md for any blocked dates before proposing timelines. (discovered 2026-02-01: suggested a start date during a planned vacation)
```

The system gets better because it remembers what went wrong.

### The Bootstrapping Mindset

**Week 1:** You're teaching Claude about your projects. Sessions feel like setup.

**Week 2:** Claude starts remembering context. You spend less time explaining.

**Week 3+:** Claude surfaces things you forgot, avoids mistakes it made before, and connects patterns across projects.

The structure is immediate. The value compounds.

## Structure

```
your-workspace/
├── CLAUDE.md                    # Top-level session initialization
├── _shared/
│   ├── CLAUDE.md                # Cross-project workflows
│   ├── CURRENT_STATE.md         # Meta-level dashboard (what needs attention)
│   ├── TECHNIQUES.md            # Pattern catalog
│   ├── EVALUATION_LOG.md        # Technique evaluation tracking
│   └── PROJECTS.md              # Project registry
├── _example-project/
│   ├── CLAUDE.md                # Project-level context
│   └── CURRENT_STATE.md         # Project-level state (optional)
└── .claude/
    └── commands/
        └── consistency-check.md # Registry verification command
```

## Key Concepts

### Session Initialization

Every session starts with Claude reading a "current state" file and proactively surfacing what needs attention — before asking how to help.

Instead of:
> "How can I help you today?"

You get:
> "I've reviewed your current state. The newsletter scoring rules haven't been evaluated in 2 weeks, and you left off yesterday working on the travel automation. What would you like to focus on?"

This eliminates catch-up conversation and ensures nothing falls through the cracks.

### CLAUDE.md as Living Knowledge Base

Each project has a `CLAUDE.md` that captures:
- What the project is and how it's structured
- Query routing (what user intents map to what files)
- Common mistakes (errors Claude has made that it shouldn't repeat)
- Techniques applied (what patterns from the catalog are in use here)

This isn't static documentation — it evolves. When Claude makes a mistake, add it. When you apply a new technique, note it. The file becomes a living record of how to work in this project.

### Technique Evaluation Workflow

When you discover a pattern that works:

1. **Document it** in `TECHNIQUES.md` — what it is, why it works, when to use it
2. **Evaluate it** against each project — is it relevant? Worth the complexity?
3. **Log the evaluation** in `EVALUATION_LOG.md` — what you decided and why
4. **Apply it** where relevant — update project CLAUDE.md files

This turns ad-hoc discovery into systematic improvement. Techniques compound across projects instead of staying siloed.

### The Meta-Layer

The `_shared/` directory is itself a project — it manages the system that manages your projects. It has its own:
- `CLAUDE.md` with workflows for cross-project work
- `CURRENT_STATE.md` tracking meta-level alerts (stale techniques, registry drift)
- Techniques applied to itself

This is "dogfooding" — the system applies its own patterns to itself. If a technique is good enough to recommend to projects, it should be good enough for the orchestration layer.

## Adapting This Template

### Minimum Viable Setup

If you want to start light:
1. Just use the top-level `CLAUDE.md` and one project `CLAUDE.md`
2. Skip `TECHNIQUES.md` and `EVALUATION_LOG.md` until you have patterns worth tracking
3. Add `CURRENT_STATE.md` when you need session continuity

### Full Setup

For serious multi-project use:
1. Use the complete structure as provided
2. Customize `_shared/PROJECTS.md` with your actual projects
3. Create a `CLAUDE.md` for each project with relevant context
4. Maintain `CURRENT_STATE.md` files for projects with time-sensitive state
5. Document techniques as you discover them

### What to Customize

| File | Customize How |
|------|---------------|
| `_shared/PROJECTS.md` | Add your actual projects |
| `*/CLAUDE.md` | Add project-specific context, rules, query routing |
| `*/CURRENT_STATE.md` | Add your actual alerts, work streams, state |
| `_shared/TECHNIQUES.md` | Add techniques you discover (or remove the examples) |

### What NOT to Put Here

- **Sensitive data** — This structure is for *context*, not secrets
- **Large content** — Keep CLAUDE.md files focused; link to detailed docs
- **Ephemeral information** — State files are for persistent state, not scratch notes

## The Personal ↔ Distributable Boundary

If you build tools (MCP packages, scripts, etc.) alongside your knowledge vaults, keep them separate:

| Type | Location | In Git? |
|------|----------|---------|
| Personal vaults | `~/Projects/*` | No |
| Distributable packages | `~/packages/*` | Yes |

Link them explicitly in `PROJECTS.md` so you can track which vaults depend on which packages, and at what version.

## Philosophy

This template embeds a few beliefs:

1. **Context is compounding** — The more Claude knows about your projects, the more useful it becomes. Investing in context pays dividends.

2. **Structure reduces friction** — A little upfront organization eliminates repeated explanation and rediscovery.

3. **Systems should self-improve** — If you're building a system for learning, apply that learning to the system itself.

4. **Explicit > implicit** — Write down the rules, the mistakes, the patterns. Don't rely on Claude (or yourself) remembering.

## FAQ

**Is this overkill?**

For simple use cases, yes. For serious multi-project knowledge work, no. Gauge by how often you find yourself re-explaining context to Claude.

**How much maintenance does this require?**

Light ongoing maintenance: update `CURRENT_STATE.md` at session boundaries, add mistakes to `CLAUDE.md` when they happen, log techniques when you discover them. Think minutes per week, not hours.

**Can I use this for coding projects?**

Yes. The patterns work for any project type. The examples lean toward knowledge management because that's where this approach was developed.

**What if I don't want the full structure?**

Start with just `CLAUDE.md` files. Add `CURRENT_STATE.md` when you need state tracking. Add `TECHNIQUES.md` when you have patterns worth documenting. Grow into it.

## Relationship to the Source System

This template was extracted from a working personal system. The relationship is:

- **This repo:** A snapshot of patterns that work
- **Your fork:** Yours to adapt and evolve
- **Sync expectation:** None — once you clone, you diverge

Unlike a library or package, you're not expected to "upgrade" to new versions of this template. Your system will evolve based on your needs. If this template improves, you might cherry-pick ideas, but there's no dependency relationship.

You're forking a pattern, not subscribing to a service.

## Credits

This template synthesizes patterns from multiple sources:
- Boris Cherny (Anthropic) — CLAUDE.md as living knowledge base, verification-led development
- Andrej Karpathy — LLM Council / multi-model deliberation concepts
- The Claude Code community — various techniques and workflows

The integration, meta-layer design, and knowledge-work application are original.

## License

MIT. Use it, adapt it, share it.
