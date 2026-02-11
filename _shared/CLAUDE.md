# CLAUDE.md

```yaml
_context:
  tier: 1  # Root of inheritance hierarchy
  version: 2.0
  last_updated: YYYY-MM-DD
  inherits_from: []  # This is the root
  techniques_applied:
    - name: "Session Initialization Protocol"
      applied: YYYY-MM-DD
    - name: "CLAUDE.md as Living Knowledge Base"
      applied: YYYY-MM-DD
```

This is the cross-project techniques and patterns workspace.

## Session Initialization Protocol

**On every session at `_shared/` level:**

1. **Read `CURRENT_STATE.md`** — The meta-level dashboard
2. **Read `INBOX.md`** — Capture staging area. Only surface if unrouted items exist.
3. **Check reminders** — Surface upcoming deadlines, completed items, due-today items per top-level CLAUDE.md protocol.
4. **Show last session context** — What was worked on, where we left off
5. **Ask "Anything to capture since last session?"** — Always.

**Output format:** Exception-only — see top-level `CLAUDE.md` → Session Initialization Protocol for full rules. Sections only appear if actionable: no "all clear" lines. Consistency checks are NOT part of init (run `/consistency-check` separately).

---

## Purpose & References

| Reference | File |
|-----------|------|
| Vault registry, boundaries | `PROJECTS.md` |
| Technique catalog + applied tracking | `TECHNIQUES.md` |
| Cross-pollination evaluations | `EVALUATION_LOG.md` |
| Methodology: simulated expert panels | `guides/simulated-expert-panels.md` |
| Methodology: synthetic testing | `guides/synthetic-testing.md` |

**Technique workflow:** Document in TECHNIQUES.md with `Applied:` line → done. Full cross-pollination eval in EVALUATION_LOG.md only on structural triggers (new vault, major restructuring, quarterly review). See EVALUATION_LOG.md header for trigger definitions.

## Discovery Protocol

> **Search first, explore as last resort.** Use search/grep tools. Only list directories or read files speculatively if search returns nothing.

This prevents the "list → read → filter → repeat" loop that burns tokens on large codebases. See the **Search-First Discovery** technique in `TECHNIQUES.md` for full details.

---

## Core Principles (Enforced)

Consolidated behavioral rules with active enforcement mechanisms.

**The meta-rule:** Documentation without enforcement is aspirational. Each principle below has a verification step.

*Customize these principles for your workflow. The structure (principle + enforcement + violations prevented) is the pattern; the specific rules are yours to define.*

---

### 1. Files Are The Work Product

**Principle:** Conversation is ephemeral. Knowledge must be in files to persist.

**Enforcement:** Before declaring anything "complete," answer:
- What artifacts should exist from this work?
- Do they exist in files right now?
- If no: persist before moving on.

**Session-end checklist** (verify before closing out):
- [ ] `CURRENT_STATE.md` — reflects what was done and where we left off
- [ ] Vault-level state files — if work touched a specific project
- [ ] **If code/package commits were made:** version bump + tag + reference updates per Principle #6
- [ ] **Vault-first check:** Every reminder/tracked item that references a vault file → that file contains the relevant content. No dangling pointers.
- [ ] **Action-reminder pairing:** Every item in state file "Action Required" or "Pending Actions" sections has a paired reminder. No orphaned actions.

**Violations this prevents:**
- Designs discussed but not documented
- Phases declared complete without artifacts
- Reminders pointing to vault files that don't have the content
- Actionable items written to state files without a corresponding reminder

---

### 2. Verify, Don't Assume

**Principle:** Verifiable claims must be verified. Reference explicit sources.

**Enforcement:**
- State claims (commits, file status): Run the actual command
- **Dates:** Anchor from the environment `Today's date` field. For long sessions or relative terms ("tomorrow", "next Monday"), run `date` to re-verify. NEVER derive day-of-week mentally — this is a recurring failure mode across sessions. Structurally solve with a SessionStart hook (see Technical Gotchas).
- File contents: Read the file, don't assume from memory

**Violations this prevents:**
- Surfacing stale state
- Wrong day-of-week claims ("due today" for tomorrow's items)
- Giving advice based on pre-action state

---

### 3. Understand Before Acting

**Principle:** Read context before producing artifacts.

**Enforcement:** Before writing any artifact:
- **Run the Session Initialization Protocol if it hasn't been run yet** — non-negotiable, even if the user's first message is an action request
- Read the project's CLAUDE.md
- Check relevant CURRENT_STATE.md files
- If uncertain, confirm understanding with user before proceeding

---

### 4. Apply Before Inventing

**Principle:** Check existing patterns before designing new ones.

**Enforcement:** Before proposing a new pattern:
- Search TECHNIQUES.md for relevant existing patterns
- Check existing projects for proven structures
- Only invent if nothing existing applies

---

### 5. Boundaries Are Sacred

**Principle:** Distributable code and personal data never cross-contaminate.

**Enforcement:** Before any git commit to a distributable package:
- Search for personal project references, naming patterns, personal data
- Verify no personal outputs in repo

**The rule:** Code in repos, data in local vaults, config in dotfiles. No exceptions.

**PHI/PII rule:** If your system tracks sensitive data (health, financial, credentials), designate it as **local-files-only**. Never in: reminder bodies synced to cloud, git repos, or conversation as storage. References point to vault files without containing the data.

**Violations this prevents:**
- Personal naming leaking into packages
- Personal outputs saved in git repos
- Sensitive data in transmitted surfaces

---

### 6. Sync Is Immediate

**Principle:** Related updates happen in the same session.

**Enforcement:** After any project/package change, same-session checklist:
- [ ] PROJECTS.md registry updated?
- [ ] Version references match across files?
- [ ] **Build:** If applicable, run build and verify output is newer than source. Stale builds = stale runtime.
- [ ] **No duplicated logic:** If you have an orchestrator wrapping packages, it must delegate, never reimplement. Grep for shared function names if unsure.
- [ ] **Version bump needed?** feat: commits → minor, fix: commits → patch. Update CHANGELOG, git tag.
- [ ] **Tag matches version?** `git tag v{version}` must exist at HEAD.

---

### 7. Future Self Can Act Immediately

**Principle:** Time-delayed artifacts must be self-contained and actionable.

**Enforcement:** Every reminder, calendar event, or working memory item must include:
- What to do (specific action)
- How to do it (invocation method, not just name)
- Context needed (costs, deadlines, decision framework)
- **Correct list placement:** Quick capture → inbox list. Completable without AI session → offline list. Needs session context → session list.

**Reminder chain rule:** When completing a reminder that has remaining follow-up actions, ALWAYS create a successor reminder before marking it done. A completed reminder with dangling TODOs is a dropped ball.

**Reminder freshness rule:** When session work changes state that an open reminder references, update that reminder's body in the same session. Stale reminder bodies cause future-self to act on wrong context.

**Vault-first rule:** When creating a reminder that references a vault file, the vault file MUST contain the relevant content BEFORE the reminder is created. Create the vault entry first, then the reminder. Never create a dangling pointer.

**Entity rule:** Before creating multiple related items (e.g., flight leg reminders for a trip), identify the parent entity and create/update the entity entry in the vault first. Individual items are components of entities, not independent items.

**Action-reminder pairing rule:** Every actionable item written to a state file must have a paired reminder, created in the same session. State files = context; reminders = triggers. Neither is complete alone.

Test: Can future-you act on this without re-researching?

**Violations this prevents:**
- Vague reminders that require context recovery
- Reminders referencing vault files that don't contain the content
- Completing a reminder without creating the next one in the chain
- Orphaned state file actions with no corresponding reminder

---

## Technical Gotchas

Factual reminders about tools and environment. Not behavioral principles — just things to know. Add new gotchas as they're discovered.

### Claude Code Hooks
- **SessionStart hook:** Add a `SessionStart` hook in `~/.claude/settings.json` to run commands at session start with stdout injected into context. Example: `"SessionStart": [{"command": "echo Today is $(date '+%A, %B %d, %Y')"}]` — structurally solves day-of-week derivation failures.
- **Hook stdout = context:** Exit code 0 + stdout from hook commands is automatically injected. Use this for any "always-available" information.

### Reminder Systems
- **Three-list architecture:** Separate (1) items needing AI/session context ("Strategic"), (2) offline habits/errands/tasks ("Personal"), (3) quick capture inbox ("Inbox"). This prevents session init from drowning in habit reminders while still catching time-sensitive offline tasks.
- **Read/write tool reliability:** Many reminder CLIs and AppleScript interfaces have asymmetric reliability — reads work but writes (add, complete, edit) may silently fail or target wrong items. Establish which tool is reliable for reads vs writes and document it. Always verify after mutations.
- **Confirm before mutating:** Present proposed reminder changes and wait for user approval before executing mutations. This applies to creates, completes, edits, and deletes — not reads.

### AppleScript (if using Apple ecosystem)
- **Sandbox path resolution:** `path to home folder` inside `tell application "X"` resolves to the app's container, not actual `~`. Use `do shell script` with hardcoded POSIX paths.
- **Date construction:** Never pass date strings like `date "2026-02-18"` — locale-dependent parsing silently produces wrong dates. Always construct dates field-by-field (set year, month, day, hours, minutes, seconds individually).
- **Script Editor re-encodes files:** Opening a UTF-8 `.applescript` file in Script Editor re-saves it as UTF-16. Rewrite the full file each time rather than editing.
- **No heredoc in `do shell script`:** Use `echo` with `quoted form of` instead.
- **`body` returns "missing value":** When a reminder has no notes, `body of reminder` returns the literal `missing value` rather than throwing an error. Test with `if b is not missing value` not a try/catch.

### MCP Development
- **Tool names:** Only `A-Z, a-z, 0-9, _, -, .` allowed. No colons.
- **Orchestrator sync:** When adding features to domain packages, update orchestrator wrapper in same session.
- **Build after commit:** Always run build after committing source changes. Verify output is newer than source. Without this, the MCP server runs stale code.

### External Services
- **API caching:** Some CDNs ignore Cache-Control headers. Use cache-busting query parameters (e.g., `?_cb=${Date.now()}`) when freshness matters.
- **Cross-system schemas:** When System A writes data for System B, verify schema match explicitly. Don't trust "it worked before."

### macOS Filesystem
- **TCC restrictions:** Claude Code cannot access certain protected paths (`~/Library/Mail/`, Photos, Contacts). `dangerouslyDisableSandbox` does NOT help — this is macOS kernel-level enforcement. Workaround: prepare commands for user to run in Terminal (which must have Full Disk Access).
- **iCloud Drive exception:** `~/Library/Mobile Documents/` (iCloud Drive) IS accessible for both read and write.

### File Management
- **CURRENT_STATE.md size:** Target 50-80 lines. Extract history to CHANGELOG.md.
- **CLAUDE.md size:** Target 200-300 lines. Extract gotchas or reference sections if approaching 320.
- **Environment date:** Use the date from environment context or hooks, not UTC timestamps from external systems.
