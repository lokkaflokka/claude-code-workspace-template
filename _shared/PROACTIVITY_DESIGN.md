# Proactivity & Always-On Design

System-level design for push-based proactive behaviors — connecting "things that need attention" to "phone notification" without a human-initiated session.

This is a template for thinking about proactivity. Specifics (hardware, push channel, app bundle names, dollar amounts) belong to your implementation, not this doc.

---

## Problem

A reactive assistant only surfaces information when the user starts a session. Between sessions, items can go overdue, deadlines can approach, and state can go stale — all silently. The user still carries cognitive load because they can't fully trust the system to reach them.

The gap: connecting "things that need attention" to "phone notification" without a human-initiated session.

## Execution Platform

Proactive behaviors require dedicated always-on hardware. A laptop that sleeps is not a server — schedulers (launchd, cron) queue jobs on wake, producing unreliable timing that undermines trust in push notifications.

**Common patterns:**
- Always-on Mac Mini or similar (low-power, headless, on home network)
- Cloud VM (less ideal for local-data workflows but viable)
- Always-on Linux box (NUC, Raspberry Pi for lighter loads)

What runs on the always-on platform:
- Push bridge / inbound message bridge (24/7)
- Scheduled briefings / digests
- Watchdog + update watchers
- Cost monitoring agent
- Vault sync (so always-on machine stays current with interactive-machine edits)

What stays on the interactive machine:
- Interactive sessions (`/start`, deep work, `/end`)
- Direct file editing
- Git operations

---

## Design Principles

1. **Intelligence over data.** A flat reminder list pushed to a notification channel is no better than opening the task system. Push notifications earn their interruption by providing *connected, context-aware* output that the user can't get from any single native app. This requires the assistant.
2. **One push channel.** Whatever you choose (Telegram, Pushover, native iOS notification, etc.), use one. Multiple push channels split attention without adding value.
3. **Scheduler is the trigger.** Use whatever's proven on your platform (launchd on macOS, cron on Linux). Don't build new scheduling infrastructure.
4. **Push is the hook, deeper interaction is the depth.** Briefing arrives as notification. User replies → bridge handles follow-up with full vault context. The push-to-action path is the product.
5. **No false silence.** If a proactive check fails to run, the next session should know. Heartbeat tracking.
6. **Staleness-aware.** Even on dedicated hardware, scheduler timing isn't guaranteed (OS updates, restarts, network outages). Scripts detect late execution and adapt output ("afternoon catch-up" vs. "morning briefing").
7. **Learning system, not production system.** This is primarily an exploration of what proactivity means for your workflow. Build to learn, not to last.

---

## Architecture

```
                ┌──────────────────────────────────┐
                │  Always-on platform               │
                │  ┌────────────────────────────┐   │
                │  │  Proactive behaviors        │   │
                │  │  • Daily briefing            │   │
                │  │  • Digest completion notif   │   │
                │  │  • Heartbeat monitor         │   │
                │  │  • Cost monitoring           │   │
                │  └────────────┬───────────────┘   │
                │               │                    │
                │  ┌────────────┴───────────────┐   │
                │  │  Existing services          │   │
                │  │  • Push bridge (24/7)        │   │
                │  │  • Weekly digest             │   │
                │  │  • Watchdog + updater        │   │
                │  └────────────┬───────────────┘   │
                └───────────────┼───────────────────┘
                                ▼
                ┌──────────────────────────────────┐
                │  Push utility (shared)            │
                │  Reads channel config             │
                │  Sends via channel API            │
                │  Verifies delivery                │
                └───────────────┬──────────────────┘
                                ▼
                ┌──────────────────────────────────┐
                │  User's phone (notification)      │
                │  • Glance: lock-screen awareness  │
                │  • Reply: bridge handles via vault │
                │  • Capture: bullets/links/photos  │
                └──────────────────────────────────┘
```

### Why Single-Layer (Assistant-Powered Only)

A common temptation is two layers: a lightweight shell-script layer ($0) for simple state reads, with assistant sessions reserved for complex synthesis. Resist this.

A flat reminder list pushed to a notification channel creates notification noise without engagement. The value is in *connected* output — grouping related items, assessing capacity, surfacing what matters. Without intelligence, the push is just noise.

The cost difference between "3 items due today" and "Roth conversion due Tue — evening window after office, pairs with fossil fuel unwind" is the entire value proposition. Building the dumb version first poisons the evaluation.

### Shared Infrastructure: Push Utility

A single utility script called by all proactive behaviors to send messages.

**Requirements:**
- Read channel credentials from a fixed local config path (NOT the synced vault)
- Send via the channel's API
- Verify delivery: check for an explicit success signal (message ID in response, etc.) — not just HTTP 200
- Retry once on failure, log failures to a known location
- Respect channel message length limits — truncate gracefully if needed
- Plain text by default; rich formatting only if the channel supports it well

---

## Proactive Behaviors

### B1: Daily Morning Briefing (primary)

**What:** Assistant-powered morning briefing pushed to the channel. Connected, context-aware — not a flat list.

**Architecture:** Wrapper script bundles the assistant binary, sets a per-run budget cap, writes an idempotency marker (one briefing per day).

**Execution flow:**
1. **Preflight gates:** idempotency (success marker for today exists? skip), required permissions accessible, working directory accessible, binary exists.
2. **Staleness gate:** Check current time vs scheduled time. If >4 hours late: catchup mode (changes output framing). If >12 hours late: skip entirely (evening briefing has no value). Write heartbeat with `skipped-late` status.
3. **Main execution:** Headless assistant call with budget cap, prompt for compact phone-optimized output.
4. **Push:** Call the push utility with the briefing content.
5. **Heartbeat:** Write success heartbeat with timestamp, status, cost.
6. **Success marker:** Write today's marker file.

**Output format (compact, phone-optimized):** plain text, max ~500 words, no markdown tables. Capacity signal first line, due-today items, coming-up items, ahead summary, action affordance ("Reply here to discuss anything").

**Key differences from `/start` output:**
- Shorter — word cap, no rigid 8-line structure
- Plain text, no markdown tables
- No algorithm trace
- Catchup mode prefix when running late

### B2: Digest Completion Notification

**What:** After autonomous digest runs, push result to the channel.

**Logic:** At end of digest script, after success marker is written, count staged insight files and push a one-line summary. On failure, push a one-line failure notification ("Digest failed: [reason]. Check in next session.").

**Cost:** Negligible (curl call, no assistant session).

### B3: Heartbeat Monitor

**What:** Track that proactive behaviors are running. Surface gaps at next session start.

**Mechanism:**
- Each behavior writes `~/.proactive-heartbeat/<behavior-name>` with:
  ```
  timestamp: <ISO 8601>
  status: success | skipped-late | push-failed | error
  cost_usd: <amount>
  details: <optional context>
  ```
- `/start` Phase A reads heartbeat directory.
- Phase B: if a heartbeat is older than expected cadence + tolerance (e.g., briefing >28 hours old → "Morning briefing hasn't run since [date]").
- Push failure log reviewed during `/check`.

**Heartbeat checks delivery, not just execution.** A script that runs but fails to push writes `status: push-failed`.

### B4: Cost Monitoring

**What:** Track API spend across all autonomous behaviors. Push weekly summary. Alert on anomalies.

**Mechanism:**
1. Read all heartbeat files from the past 7 days
2. Sum `cost_usd` across all behaviors
3. Compare against expected weekly budget
4. Push weekly summary
5. If weekly total exceeds threshold (e.g., 1.5x expected), push immediately

**Cost data source:** Per-run heartbeat data. Each wrapper script extracts token usage from assistant output or estimates from budget cap.

---

## Surface Interaction Model

The push notification creates a moment. What happens next depends on the user's context:

### Glance (most common)
User sees notification on lock screen or in the channel. Reads the briefing. Gets oriented for the day. **No action taken.** This is the baseline value — awareness without a session.

### Reply (when something needs attention)
User replies to the briefing message. The bridge receives the reply as a new message and handles it with full vault context.

**Example flows:**
- "Mark X done" → bridge processes via task system
- "What's Y about?" → bridge reads vault, explains
- "Add: do Z by Friday" → bridge creates Personal reminder
- "Run /start" → bridge runs full session init

**Capture flow:** User sends freeform text, bullets, links, or photos to the channel at any time (not just in response to briefing). Bridge routes via existing `/capture` skill. Async mobile capture surface — no new infrastructure.

### Session (when deep work is needed)
User opens laptop, starts interactive session. `/start` runs as usual. The briefing was context-setting, not a replacement — `/start` has the full zone structure, learning activation, triage, and Phase C/D capabilities.

**Relationship between briefing and `/start`:**
- Briefing is a *subset* of `/start` output, optimized for phone
- `/start` is not degraded by a prior briefing — always runs fresh from live data
- If briefing surfaced something the user already handled (via channel reply), `/start` reflects the updated state

---

## Security Considerations

Industry vocabulary: this is the **"permission hungry"** mitigation pattern (per Thoughtworks Tech Radar). Autonomous outbound push expands the permission surface — credentials become more sensitive.

| Threat | Risk | Mitigation |
|--------|------|------------|
| Channel credentials compromised → attacker sends fake system alerts | Low if creds in local config (not synced) | Store credentials in a non-synced location. Monitor channel for unexpected messages. |
| Attacker sends phishing messages that look like briefings | Low (1:1 chat with known bot identity) | Bot only sends to allowlisted user. |
| Briefing leaks sensitive data via channel | Medium (briefing includes reminder titles, calendar events) | Same content visible at session start. Channel encryption status is what it is. **No PHI/sensitive data in reminders** — enforced by Principle #5. |
| Autonomous session accesses/modifies vault data without oversight | Low if headless prompt is read-heavy | Budget cap, headless prompt is read-only by design. |

**Accepted risk:** Most push-channel APIs are not E2E encrypted. Reminder titles and calendar event names transit servers. Acceptable when no sensitive data appears in reminders (enforced by boundary principle), and content is low-sensitivity (task titles, not credentials or financial details).

---

## Implementation Phasing

| Phase | What | When |
|-------|------|------|
| **Phase 0a** | Build deployable artifacts on the interactive machine: push utility, briefing wrapper, briefing prompt, cost monitoring script. Test push utility against live channel API. | Before always-on hardware exists |
| **Phase 0b** | Always-on hardware setup. OS, sync, toolchain, migrate existing scheduled jobs, permissions, OAuth re-auth. Disable interactive-machine scheduled jobs. | When hardware arrives |
| **Phase 1** | Deploy briefing app + scheduled trigger + heartbeat integration on always-on platform. | After 0b |
| **Phase 2** | Add digest notification + cost monitoring. | After 1 |
| **Phase 3** | Evaluate after 2 weeks. Does the briefing change behavior? Track: items already handled when `/start` runs, channel replies, heartbeat reliability, actual cost per briefing. | After 1 |
| **Phase 4** | Iterate based on Phase 3 learnings. Candidates: local inference trial, briefing prompt tuning, timing adjustments, weekend cadence. | TBD |

---

## Cost Controls

- **Per-run budget cap:** Hard limit per scheduled run (e.g., $1.50 per briefing).
- **Idempotency guard:** One briefing per day max, one digest per week max.
- **Staleness skip:** No briefing if >12h late.
- **Weekly cost summary:** Automated tracking pushed to channel.
- **Alert threshold:** Immediate push if weekly spend > expected × 1.5.
- **Escalation:** alert → check provider dashboard → unload scheduled job if anomalous.

---

## Success Criteria

1. **Briefing fires reliably for 2 weeks** — reaches phone every day on schedule.
2. **User acts on briefing content** — replies to channel, handles items before `/start`, or uses briefing to plan the day.
3. **Heartbeat gaps are rare** — on dedicated hardware, any gap is an anomaly.
4. **No notification fatigue** — user doesn't mute or ignore the briefing channel.
5. **Reply path works** — reply → bridge receives → handles with vault context.
6. **Connection pass adds value** — briefing groups items the user wouldn't have connected from a flat list.

---

## Open Questions

These are the questions you'll likely face when implementing:

1. **Briefing prompt tuning.** First version will be simple; expect 2-3 revisions based on output quality.
2. **Reply-path verification.** Does the bridge receive replies to bot-sent messages? Verify before Phase 1 launch. If not, push is read-only (still valuable, but reduces action affordance).
3. **Timing.** Initial schedule is a guess. Adjustable. Iterate based on wake patterns.
4. **Weekend vs. weekday.** Same briefing cadence? Or lighter weekend format? Start with same cadence, adjust if weekends feel noisy.
5. **Platform absorption timeline.** Vendors are moving toward native proactivity. Monitor: if your platform provides daily briefing capability, migrate rather than maintain custom infrastructure. The insights transfer even if the code doesn't.
6. **Local inference for briefings.** If your hardware can run a sufficient local model, you can replace API costs with electricity. Quality gap is the question — local models often handle structured tasks but may lose connection-pass quality. Trial after the API version is running.
