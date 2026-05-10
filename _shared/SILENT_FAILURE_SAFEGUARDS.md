# Silent Failure Safeguards

A framework for catching failures that report success but didn't actually do the work.

## The Failure Class

**Definition:** *Silent infrastructure failure that escapes notice until the verification window arrives.*

The shape: an operation reports success (or no error), but the work it was supposed to do didn't happen — or happened against false preconditions. The failure is invisible at the moment of occurrence; it only becomes visible at a downstream verification point, by which time the system has accumulated drift that's expensive to unwind.

This pattern crosses subsystem boundaries. It's not a launchd bug, a reminders bug, or a digest bug. It's the same epistemic failure expressed in different mechanical surfaces.

## Example Instances

Concrete cases that share the failure shape — useful for reasoning about which mechanism applies. Replace with your own as you encounter them.

| # | Instance | What appeared OK | What actually failed | How it was caught |
|---|----------|------------------|----------------------|-------------------|
| 1 | Multi-step pipeline post-step skip | Pipeline run reported success; primary artifact written | Final close-loop steps (routing/notification/persistence) skipped due to thrown exception in cleanup phase | Downstream verification pass surfaced the gap days later |
| 2 | Batch-completion against future state | Bulk-mark-done succeeded | Items referenced hardware/preconditions not yet present — couldn't have been done | Spot review found the contradiction the next week |
| 3 | Long-running job PID-loss after auto-update | Scheduler reported `runs=N`, exit 0 | Auto-updated binary swapped underneath; subsequent fires errored silently | Six days of silence noticed during periodic review |
| 4 | Headless run budget-killed mid-execution | Primary output written to disk | Budget cap killed run before close-loop steps (success marker, downstream trigger, status bump, completion) | Same-session investigation; would have been invisible at next session start |
| 5 | Multi-source pool lost a source silently | Each source branch reported `status=success`; overall pipeline `success` | One source's endpoints returned 403 (IP-based block); 0 items returned per branch; pool ran truncated for 5+ days | Pool-composition diff during a periodic review caught the rolling drop |

The pattern: in every instance, there was a moment where reaching for the verification signal would have caught the failure. None were caught at that moment. Each was discovered later, when ambient cost of recovery was higher.

## Four Safeguard Mechanisms

Four ways to close the verification window earlier. Each has a different cost profile and applicable scope.

### M1: Precondition Gate (cheapest, catches at completion time)

**Definition:** Before marking an item complete, verify that completion was *possible* given current state.

**Implementation pattern:**
- Before bulk-complete, scan target items for completion-blockers (referenced hardware not yet present, blocked-by-other-task tags, due dates in the future without explicit override)
- Refuse the completion (or warn + require explicit confirmation) when blockers exist
- Stays in the operation surface — no separate watchdog

**Best fit for:**
- Instance #2 — a precondition check for `[hw-arrives:]` tag would refuse early bulk-completion
- Instance #4 — orchestrators already have empty-pool / source-concentration gates; needs one more for "all close-loop steps completed?" before declaring success

**Cost:** Low — operation surface already has the data it needs to check.

### M2: Verification Window (medium, catches at drift detection time)

**Definition:** A scheduled re-check that compares declared state against observed state at a known-distant verification point.

**Implementation pattern:**
- For long-running infrastructure (scheduled jobs, recurring reminders), add a scheduled "is this thing actually working?" probe that runs *independent* of the thing itself
- Probe checks for liveness signals (recent log entries, expected artifacts, plausible runtime intervals) and alerts on anomaly
- Often externalized to a different runtime (e.g., a monitoring workflow tool) so failure of the primary doesn't take the probe down with it

**Best fit for:**
- Instance #1 — a probe that checks for routing artifacts after each pipeline run
- Instance #3 — a probe that checks "did the job log activity in the last N hours during expected run window?"

**Cost:** Medium — separate probe to design, schedule, and surface.

### M3: Contradiction Alert (most thorough, catches at downstream surface)

**Definition:** When a downstream operation reads state, if that state contradicts a recent declaration, surface it.

**Implementation pattern:**
- A session-init or periodic-scan surface already reads many state files
- Add: cross-check recent completion claims against subsequent observation
- Example: if an item was marked complete on date X but its body references hardware/event arriving on date Y > X, flag

**Best fit for:**
- Instance #2 — natural catch could be formalized as a session-init check
- Generalizes: any completion where the body has a future-dated reference is suspicious

**Cost:** High — needs structured tagging in item bodies (e.g., `[hw-arrives:]`, `[blocked-by:]`, `[verify-on:]`) plus parsing.

### M4: Source-Coverage Gate (cheap, catches at pipeline-input time)

**Definition:** Before a pipeline that consumes a multi-source input pool starts running, verify that all *expected* sources contributed to the pool. Refuse to proceed (or fall back loudly) when the input is structurally incomplete.

**Implementation pattern:**
- Pipeline's input is composed from N declared sources (RSS feeds, API endpoints, scrapers).
- Maintain a manifest of expected sources with per-source freshness window (`max_age_days`).
- Pipeline-start health check fails closed (`ready=false`) when any expected source is missing or stale.
- Distinct from M1 (which gates *completion*) — M4 gates *input availability*. Failure mode prevented: "pool was structurally incomplete, but pipeline ran on the partial pool and reported success."

**Best fit for:**
- Instance #5 — branches each report `status=success` because `continueOnFail` returned an empty 0-item result as success; downstream pool ran truncated. M4 detects this at the pool-assembly boundary.

**Cost:** Low when pool composition is already declared in config. Medium when sources aren't schema-tracked.

**Why M4 is its own class, not a subtype of M1:**
- M1 = "is the work allowed to be marked complete?" (fires at end of operation, against operation-local state)
- M4 = "is the input structurally what we expected?" (fires at start of operation, against an external manifest)
- The two are complementary. M1 catches "we said it's done but it isn't"; M4 catches "we ran on a degraded input and didn't notice." Same failure class; opposite ends of the operation.

## Mechanism × Instance Matrix

| Instance | M1 (precondition) | M2 (verification window) | M3 (contradiction) | M4 (source-coverage) |
|----------|:-----------------:|:------------------------:|:------------------:|:--------------------:|
| #1 Pipeline post-step skip | — | **best fit** | partial | — |
| #2 Batch-complete future state | **best fit** | partial | alternative | — |
| #3 Job PID-loss after auto-update | partial | **best fit** | — | — |
| #4 Headless budget-kill | **best fit** | partial | — | — |
| #5 Multi-source silent drop | — | partial | — | **best fit** |

Two instances each best served by M1 and M2; M3 is a more general backstop; M4 covers the input-side failure class that M1 by definition can't see.

## Adoption Order (incremental)

Don't try to build all four at once. Order by leverage / cost. Each adoption step is independently shippable and independently valuable.

A workable order, generalized:

1. **M1 for orchestrator close-loop** — extend the orchestrator's success path to require all close-loop steps before declaring success. If any fail, log PARTIAL and surface at next session start.
2. **M1 for batch-completion** — add tag-aware refusal (e.g., `[hw-arrives:]` value > today blocks completion).
3. **M2 for long-running jobs** — extend whatever staleness/health surface already exists to alert when expected runs are missing past the scheduled window.
4. **M2 for pipeline steps** — formalize "if intermediate artifact exists from last run AND no archive from same period, surface as alert."
5. **M4 for any multi-source pool** — add `expected_sources` manifest with `max_age_days` per source; gate pipeline start on coverage.
6. **M3 generalization** — defer until more contradiction examples accumulate. The other mechanisms are usually cheaper. M3 is the right pattern but premature without enough data to justify the structured-tag investment.

## Open Questions

- Does the same failure class apply to learning/insight tracking? E.g., a position is marked "settled" but evidence that would falsify it accumulates without triggering a re-open. Consider a falsification window for any settled position.
- Should M2 probes be assistant-driven or simple script + alert? Cheapest fires beat richest reasoning for most cases — leans toward script + alert for routine probes.
- For M3, where do the structured tags live — body free-text or a separate metadata file? Body is the existing convention for lightweight tagging; a separate metadata file scales better but adds another surface to keep in sync.

## Closure Notes

A real safeguard mechanism can ship before its conceptual home is updated. Code-side fixes often outpace doc framing. The risk: future contributors / sessions don't see a mechanism as a class and re-invent it under another label. Naming the framework explicitly closes that gap.
