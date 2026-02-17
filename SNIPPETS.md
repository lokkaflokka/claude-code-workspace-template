# Claude Code Snippets Reference

Quick-reference for common prompts. Type or paste these into Claude Code.

For auto-expansion, import into your text expander (Raycast, macOS Text Replacements, etc.). Without a text expander, this file is your cheat sheet — the keywords are just labels.

## Session Flow

| Keyword | Prompt |
|---------|--------|
| `ccstart` | Read CURRENT_STATE.md and surface any alerts. Show me what needs attention and where we left off last session. |
| `ccend` | Before we end: Update CURRENT_STATE.md with what we accomplished and where we left off. Check if any techniques emerged worth documenting. Verify no uncommitted work. Surface one specific, actionable improvement from this session's work. What should I know for next session? |
| `ccreflect` | Pause and reflect on the work chunk we just completed. What did we learn? Are there patterns worth extracting, mistakes to document, or improvements to capture? Don't wait for session end — capture it now. |

## Operations

| Keyword | Prompt |
|---------|--------|
| `cccommit` | Review my staged and unstaged changes, then create a well-structured commit with a message that captures the 'why' not just the 'what'. |
| `ccpr` | Create a pull request for this branch. Summarize all commits since branching, write a clear description, and include a test plan. |
| `cctest` | Run the test suite. If there are failures, analyze them and fix the issues. Re-run until green. |
| `ccplan` | Before implementing, explore what's involved. What files need to change, what are the dependencies, what could go wrong? Design the approach before writing code. |
| `cccapture` | Capture this to the appropriate surface (Reminders Inbox, INBOX.md, or directly to the relevant vault): |
| `ccsync` | Check _shared/sync/staging.md for this session's generalizable learnings. If anything worth syncing, append a bullet. If nothing, skip. |

## Slash Commands

These are already available as `/command` — no snippet needed:

| Command | Purpose |
|---------|---------|
| `/consistency-check` | Registry, versions, tags, drift scan |
| `/new-technique` | Document a new technique in TECHNIQUES.md |
| `/new-project` | Create a new project from template |
| `/sync-review` | Process sync inbox + produce outbound packets |

## Systems Thinking

| Keyword | Prompt |
|---------|--------|
| `ccstate` | Update CURRENT_STATE.md to reflect the work we just completed. Update Last Session, Active Work Streams, and clear any resolved alerts. Add any new working memory items if needed. |
| `ccimprove` | Review the current workflow/system we just worked on. Are there patterns that could be extracted, inefficiencies to address, or improvements that would make this more robust? Be specific and actionable. |
| `ccdrift` | Check for drift or inconsistencies: Are there any mismatches between documentation and reality? Registry entries that don't match actual state? Version numbers out of sync? Surface anything that needs reconciliation. |
| `cchealth` | Perform a system health check: Run /consistency-check, then go beyond the mechanical checks — are Active Contexts in CURRENT_STATE.md still accurate? Any pending evaluations in EVALUATION_LOG.md? Anything feeling stale or neglected across the system? |
| `ccpattern` | I notice a pattern emerging from this work. Help me evaluate: Is this worth extracting as a reusable technique? If so, document it in TECHNIQUES.md following the standard format, then evaluate it against existing projects. |

## Typical Workflow

```
Session Start:     ccstart
                        |
Plan Complex Work: ccplan -> design approach before coding
                        |
During Work:       cccommit, cctest, cccapture as needed
                        |
After Work Chunk:  ccreflect -> capture learnings mid-session
                        |
Notice Pattern:    ccpattern -> evaluate -> document if valuable
                        |
Review:            ccimprove -> surface improvements to what we built
                        |
After Chunk:       ccstate -> keep CURRENT_STATE.md fresh
                        |
Session End:       ccsync -> capture generalizable learnings
                   ccend -> state + learnings + one improvement
```
