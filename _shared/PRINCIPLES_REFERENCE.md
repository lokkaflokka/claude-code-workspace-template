# Principles Reference

Full enforcement detail for the 10 Core Principles. Concise forms live in `_shared/CLAUDE.md` (always loaded). This file is loaded by `/check`, `/end`, and during violation investigation.

**The meta-rule:** Documentation without enforcement is aspirational. Each principle below has a verification step.

---

## Principle 1: Files Are The Work Product

**Principle:** Conversation is ephemeral. Knowledge must be in files to persist.

**Enforcement:** Before declaring anything "complete" or "done," answer:
- What artifacts should exist from this work?
- Do they exist in files right now?
- If no: persist before moving on.

**Session-end checklist** (verify before closing out):
- [ ] **Run `/end`** — handles all session-end work: state file updates, vault-first check, action-reminder pairing, sync capture, decay check. Full protocol in the skill file.
- [ ] **Verify nothing was missed** — quick gut check: any work discussed but not persisted in files?

**Edit discipline rule:** When moving content between sections or files, copy original text verbatim. Only modify structural references (phase headers, cross-links, list placement). Never rewrite descriptions during a move operation.

**File placement rule:** When a checklist or reminder references an artifact file, locate and place the file in its permanent home at creation time. Don't reference files that haven't been verified to exist.

**State-file composition rule:** Heavily reread state files (`CURRENT_STATE.md` Active Contexts, `INBOX.md`) hit fixed byte caps and are scanned every session. Compose entries as ≤2-sentence pointers; route detail to domain-specific files (vault `CURRENT_STATE`, learning log, `CHANGELOG`). Pre-flight cap check before composing prevents mid-flow compression churn. Same rule shape as reminder-body discipline.

**Violations this prevents:**
- Designs discussed but not documented; phases declared complete without artifacts
- Dangling pointers — reminders/checklists referencing vault files that don't have the content
- Orphaned actions — state file items without paired reminders
- Content rewritten during moves — losing detail the user provided
- Files referenced before verified to exist locally
- Conversation-only persistence — work done in conversation treated as "done" without file writes

---

## Principle 2: Verify, Don't Assume

**Principle:** Verifiable claims must be verified. Reference explicit sources.

**Enforcement:**
- State claims (commits, file status): Run the actual command
- **Dates:** A `SessionStart` hook can automatically inject today's date + day of week into context. Use that as the primary anchor. For long sessions where the date may roll over, or when resolving relative terms ("tomorrow", "next Monday"), run `date '+%A, %B %d, %Y'` to re-verify. NEVER derive day-of-week mentally.
- File contents: Read the file, don't assume from memory
- **User corrections override state files.** When the user says a state file claim is wrong, believe them and verify the current state — don't parrot the file back. State files are snapshots that go stale between sessions; the user is the live source.
- **Interrogate stale blockers.** When an item has been "blocked" for multiple sessions, verify: (1) is the blocker still true? (2) is it actually the bottleneck, or is there a deeper gap?
- **State verification before status answers.** When asked "what's remaining," "what's the status," or "what's left" for tracked work: (1) read relevant reminders, (2) read relevant vault files, (3) cross-reference and resolve discrepancies, (4) trust the more recently updated source — for offline-completable items, reminders are typically more current than vault files. Never answer from vault files alone.
- **Phase/milestone status verification.** When assessing whether a phased plan's milestones are complete: verify the actual deliverables exist (files created, content present, features working), not planning doc checkboxes. Planning docs are claims about state, not state itself.
- **Source-before-action gate.** When presenting an action plan, execution checklist, or batch edit that references a specific source document: (1) Read the full source file in this turn — not from session memory or prior context. (2) Cite the file path in the output. An action plan that omits steps from its source document is worse than no plan — it creates false confidence.
- **Tool-before-question gate (structural).** Before asking the user ANY factual question ("did you...?", "which...?", "have you...?", "is X still...?"), check: could a tool answer this? Verification sources by type:
  - **Status/completion:** task-system list, vault state files
  - **Correspondence:** email scan (did they email someone?)
  - **Code/config:** `grep`/`read` the relevant file (don't ask "did you subscribe?" — read the code)
  - **Scheduling:** calendar tools
  - **File content:** vault files, state files (don't ask "which providers?" — read the file)
  
  If any tool can answer, use it. Asking the user for tool-verifiable facts is a Principle #2 violation. `/end` Phase 0 should audit for verification misses every session.
- **Mechanical-fix-first.** When a process failure or recurring issue surfaces, default to the structural fix (script edit, hook, file rule, schema enforcement) before a behavioral note ("remember to…"). Behavioral fixes regress under load; mechanical fixes carry forward across sessions without re-enforcement. Tier sequence: **Tier 1** (code / schema / hook — automated enforcement, no recall required) → **Tier 2** (doc rule landed at a consumer surface that fires at decision points — e.g., skill file, init protocol, gotcha entry) → **Tier 3** (behavioral-only — last resort; when chosen, name what would make it Tier 1/2 so the next recurrence has a clear upgrade target). Process-failure entries that say "remember X" or "always Y" without citing a file edit are Tier 3 by default.

**Violations this prevents:**
- Surfacing stale state or wrong dates
- Assuming files haven't changed between sessions
- Repeating stale blockers when the user has already corrected them
- Re-surfacing already-processed completions
- Surfacing expired session-scoped alerts
- Answering "what's remaining" from stale vault files without checking reminders
- Reporting phase/milestone status from stale planning doc checkboxes without verifying actual artifacts
- Presenting action plans with missing steps because source document wasn't re-read
- Rationalizing discrepancies instead of investigating — when observed count ≠ expected count, state the delta explicitly and investigate before offering explanations
- Asking user for confirmation on verifiable facts without first checking available signals

---

## Principle 3: Understand Before Acting

**Principle:** Read context before producing artifacts.

**Enforcement:** Before writing commits, evaluations, recommendations, or any artifact:
- **Run `/start` if it hasn't been run yet** — non-negotiable, even if the user's first message is an action request. An explicit plan or task does NOT bypass init.
- Read the project's `CLAUDE.md`
- Check relevant `CURRENT_STATE.md` files
- Understand the architecture and intent
- **Multi-repo dependency check:** When a work chunk touches 3+ repos, map the dependency graph before writing the first file. Which repo's changes enable the others? Do the engineering work first, then document what was built — not what's planned.

If uncertain, confirm understanding with user before proceeding.

### Cross-Vault Awareness (Principle 3 sub-rule)

When reading a vault entity file that mentions a **person, event, or decision** tracked elsewhere, MUST grep the name/entity across current-session context before responding. Verifiable step, not a suggestion.

Concrete checks:
1. **People → session context + relationships file:** If a vault file mentions a person by name, grep that name against (a) already-surfaced reminders from this session and (b) any relationships file you maintain if not yet loaded. Surface connections proactively.
2. **Events → reminders:** If reading a vault file about an upcoming event, check whether any reminders in current-session context relate to it.
3. **Decisions → affected vaults:** If a decision in one vault affects another, name the cross-vault impact explicitly.

**The test:** If the same entity appeared in two data sources during this session, they must be presented as connected — never as unrelated items across separate responses.

**First-use gate for destructive operations:** When a tool, skill phase, or workflow step mutates external state (Gmail, APIs, remote services, databases) and has NEVER been validated end-to-end on production data: (1) flag it explicitly as first use, (2) require a planning gate — what's the blast radius? what's the rollback path? what state is invisible to us? (3) test on throwaway/staging first if possible, (4) if no staging available, start with a single operation and verify before batching. A skill prescribing "execute all" does NOT override this gate.

**Violations this prevents:**
- Shallow commit messages
- Misunderstanding project purpose
- Over-filtering based on incomplete context
- Skipping session init when the user opens with an action request
- Writing artifacts in wrong locations or before prerequisite work is done (multi-repo sequencing)
- Deploying untested destructive features on production data without planning gate

---

## Principle 4: Apply Before Inventing

**Principle:** Check existing patterns before designing new ones.

**Enforcement:** Before proposing a new pattern, technique, or structure:
- Search `TECHNIQUES.md` for relevant existing patterns
- Check existing vaults for proven structures
- Only invent if nothing existing applies

**Violations this prevents:**
- Reinventing wheels
- Generic structures when proven patterns exist
- Technique fragmentation

---

## Principle 5: Boundaries Are Sacred

**Principle:** Distributable code and personal data never cross-contaminate. Sensitive data never leaves local files.

**Enforcement:** Before any git commit to a public/distributable repo:
- `grep -r "<personal-vault-name>\|<work-domain>"` for personal project references
- Check for personal naming patterns
- Verify no personal outputs in repo

**The rule:** Code in distributable repos, data in vaults, config in dotfiles. No exceptions.

**Sensitive-domain rule:** If your system tracks a walled-off domain (health/PHI, finance, work/PII), it is **vault-only**. Never in:
- `_shared/` files (`CHANGELOG.md`, `CURRENT_STATE.md`, etc.) — use generic references and pointers ("medication transition" / "see health vault for details")
- Reminder bodies (often cloud-synced) — reference vault files only
- Git repos or distributable code
- Conversation as a storage medium

Reminders reference vault files without containing the sensitive data itself. Full policy: domain `CLAUDE.md` files.

**Generic-reference patterns for cross-domain mentions in `_shared/`:**
- Specific names → generic role/category ("provider", "vendor", "stakeholder")
- Specific values → omit or bucket ("dosage details", "approximate spend")
- Specific dates tied to sensitive events → vault file pointer

**Credential rule:** API keys, secrets, and tokens NEVER live in synced directories (iCloud, Dropbox, etc.). Credentials go in `~/.config/` (local-only) or environment variables. `.env` files in synced directories must contain only placeholders. `.gitignore` does NOT protect from cloud sync.

**Public repo pre-commit checklist** (in addition to personal-data grep):
- [ ] `grep -r "<your-username>\|<work-domain>"` for personal paths/identifiers
- [ ] No links to private repos (404 links confirm private repo existence)
- [ ] Test fixtures use generic paths (`~/packages/`, `/tmp/test/`), not real directory structure
- [ ] `git log --format='%ae' | sort -u` — verify no work email addresses in commit history
- [ ] README examples use placeholders, not real config

**Violations this prevents:**
- Personal naming leaking into packages
- Personal outputs saved in git repos
- Personal examples in templates
- Sensitive data in transmitted surfaces (cloud sync, API, git)
- Sensitive data leaking from domain vaults into `_shared/` changelog/state files
- Live credentials syncing via `.env` files in synced directories
- Real directory paths and private repo links in public repositories
- Work email addresses baked into public git history

---

## Principle 6: Sync Is Immediate

**Principle:** Related updates happen in the same session.

**Structural enforcement:** `/end` Phase 3 should run an unconditional repo hygiene scan across all your code repos — checks for missing git repos, dirty working trees, and unpushed commits. Runs every session regardless of whether code work was done.

**Enforcement:** After any package/vault change, same-session checklist:
- [ ] Registry (`PROJECTS.md`) updated?
- [ ] Vault context version markers match `package.json` (or equivalent)?
- [ ] Orchestrator wrapper updated if package API changed?
- [ ] `grep -r "old/path"` if anything was moved/renamed?
- [ ] **Build:** Build + verify output is newer than source. Stale builds = stale runtime.
- [ ] **No duplicated logic:** Orchestrator must delegate to packages, never reimplement. Grep for shared function names if unsure.
- [ ] **Version bump needed?** If feat: or fix: commits → bump version (feat = minor, fix = patch), update `CHANGELOG`, git tag.
- [ ] **Tag matches version?** `git tag v{version}` must exist at HEAD.
- [ ] **Version propagation sweep?** After tagging, `grep -r "old_version"` across vault. Update live references; historical references (`CHANGELOG`, session notes) are correct as-is.
- [ ] **Distribution updated?** If package has Homebrew tap or similar: update formula, commit, push. If npm: `npm publish`.

**MCP server rule:** Every MCP server should be integrated through a central toolkit-hub package, not added as a standalone project-scoped or global config. Hub-routing ensures all tools are available from any working directory. Standalone configs create directory-scoped silos that break cross-project sessions.

**Pre-archive checklist:** Before archiving any repo or marking it EOL:
- [ ] All extractable content upstreamed to its target (template, vault, etc.)?
- [ ] All references updated (`PROJECTS.md`, `CURRENT_STATE.md`, reminders, `CLAUDE.md` files)?
- [ ] All downstream artifacts that depend on the repo's content produced and saved?
- [ ] Consumer instructions saved to a durable location before the repo becomes inaccessible?

---

## Principle 7: Future Self Can Act Immediately

**Principle:** Time-delayed artifacts must be self-contained and actionable.

**Enforcement:** Every working memory item, calendar event, or reminder must include:
- What to do (specific action)
- How to do it (invocation method, not just name)
- Context needed (costs, deadlines, decision framework)
- **Correct list placement:** Needs assistant? → Strategic. Offline? → Personal. Quick capture? → Inbox.

**Reminder chain rule:** When completing a reminder that has remaining follow-up actions, ALWAYS create a successor reminder before marking it done. A completed reminder with dangling TODOs is a dropped ball. Check: "Is there a next action? If yes, does a reminder exist for it?"

**Reminder freshness rule:** When session work changes state that an open reminder references (version numbers, completed sub-items, new prerequisites), update that reminder's body in the same session. Stale reminder bodies cause future-self to act on wrong context.

**Vault-first rule:** When creating a reminder that references a vault file, the vault file MUST contain the relevant content BEFORE the reminder is created. Create the vault entry first, then the reminder. Never create a dangling pointer.

**Entity rule:** Before creating multiple related items (e.g., flight leg reminders), identify the parent entity (e.g., "trip") and create/update the entity entry in the vault first. Individual items are components of entities, not independent items. When a reminder references a vault entity, include a `[ref: vault/path/FILE.md]` tag in the reminder body. This makes the poke→context link explicit and grep-able. Convention: `TRACKING_SYSTEM_DESIGN.md` → Body-Tag Conventions → `[ref:]`.

**Action-reminder pairing rule:** Every actionable item written to a state file's "Action Required," "Pending Actions," or equivalent section must have a paired reminder, created in the same session. State files = context; reminders = triggers. Neither is complete alone. An action without a reminder is invisible between sessions.

Test: Can future-you act without re-researching?

**Cross-device rule:** If a reminder targets a different machine than where the context lives, the reminder body must be fully self-contained — no references to files that won't exist on the target device. Test: "Can I act on this from my phone alone, with no file access?"

**Violations this prevents:**
- Vague reminders requiring context recovery
- Dangling pointers — reminder references vault file that doesn't have the content
- Completing a reminder without creating successor for remaining follow-ups
- Stale reminder bodies referencing outdated versions or completed items
- Orphaned state file actions without paired reminders
- Reminders depending on files unavailable on the target device
- Invisible Personal items due to missing `--due` on triage moves

---

## Principle 8: Content Shared Is Content Captured

**Principle:** When the user shares content (URL, screenshot, text, PDF) during a session, the default behavior is implicit `/capture`. No confirmation gate.

**Enforcement:** When the user shares content mid-session:
1. **Process the immediate need first** — analysis, discussion, prep artifact, whatever the user asked for.
2. **Then route source content to digest pipeline** (or whatever your save-for-later mechanism is). This happens automatically — no "Want me to save this?" prompt.
3. **Both outputs coexist:** A prep file (analysis, meeting prep doc) serves the immediate need. Digest routing serves the knowledge system. One does not replace the other.
4. **Fetch failure ≠ routing blocked.** If WebFetch fails, save the URL + whatever context is available (user-provided summary, conversation context). Don't drop the item because enrichment failed.

**The anti-pattern:** "Want me to route this anywhere?" after analyzing content the user shared. The answer is always yes — route it. The question wastes a turn and creates a gate where none should exist.

**Implicit `/capture` note:** This principle triggers the `/capture` skill workflow for Content/Resource items without the user needing to invoke it. See `/capture` for the full routing logic.

**Verification (`/end` Phase 3):** Content routing check — were all URLs/content pieces shared during the session routed to the digest pipeline or a vault? If not, route now.

**Violations this prevents:**
- Content analyzed but never routed
- Unnecessary confirmation gates on obvious routing decisions
- Dropped items due to fetch failures (URL still has value even without a summary)

---

## Principle 9: Entity-of-Record Discipline

**Principle:** For each tracked entity (an account, a person, a decision, a credential, a recurring obligation), exactly one system holds the truth. Every other mention references it and never duplicates it.

**Enforcement:**
1. **Before creating a second record, ask whether the first should move instead.** Duplication is the default failure mode because creating is cheap and migrating is not. The question is not "where does this belong" but "does a home already exist."
2. **Credential rotation is atomic with its fan-out.** "Find and replace every referencing site" is a step inside the rotation, never a follow-up task. A rotated secret with a stale reference somewhere is a broken system that reports success.
3. **Owned-fix guard.** When a failure class already has an owned fix in flight, new observations of that class route to the owning record. They never become a new record, a new rule, or a second attempt at the same fix. Without this, one problem grows N tracking artifacts and the fix never ships because attention splits.
4. **Documentation is not a record.** A paragraph describing an obligation reads exactly like coverage to a reader looking for coverage. Resolve "tracked elsewhere" claims to a dated, actionable item before reporting them as covered.

**The anti-pattern:** two files, both describing the same account, diverging slowly, with no marker saying which one is authoritative. Six months later both are wrong in different directions and neither reader can tell.

**Verification (`/check`):** entity fragmentation scan. Grep high-value entity names across vaults; more than one file holding mutable state for the same entity is a finding, not a style preference.

**Violations this prevents:**
- Duplicate records diverging silently
- Rotated credentials with stale references left behind
- One failure class spawning several competing tracking artifacts
- Prose about an obligation being mistaken for the obligation being tracked

---

## Principle 10: Source Before Action

**Principle:** Before producing any plan, breakdown, sizing, recommendation, or state-mutating action about a tracked item, read the underlying source content first. This covers reminder bodies, active specs, and any referenced vault file.

**Why it is its own principle:** it is a special case of Principle 3, and it was folded into Principle 3 for a long time. It earned separate status because it is the highest-recurrence anti-pattern this system has recorded. A rule that keeps being violated while nested inside a broader rule needs its own name, its own enforcement surface, and its own violation count.

**Enforcement:**
1. **Title-only execution is the failure.** A reminder title is an index entry, not a specification. The body holds the blocker, the decision already made, the phone number, the constraint that changes the answer. Acting on the title produces plans that are confidently wrong in ways the user can see immediately and you cannot.
2. **Mechanical, not behavioral.** Behavioral re-enforcement has failed repeatedly here. Enforce at every surface that can act on a title alone: a hook that gates action on tracked items until the source is read, a skill step that loads bodies before planning, a gather that ships bodies rather than titles.
3. **Name every surface the rule covers, in the gate's own docstring.** A gate's trigger surface tends to inherit the shape of the single failure that motivated it, not the shape of the rule it enforces, and the rule then reads as closed while most of its surface is uncovered. Write two lists: every surface the RULE names, and every surface the GATE observes.
4. **Where the uncovered surface has no cheap gate, say so and stop.** A gate that only flags after the fact is instrumentation. Filing one improves the counter without changing what happens.

**The anti-pattern:** the user names a tracked item, you recognize the title, and you produce a plan from what the title implies. The body said the work was blocked three weeks ago.

**Verification (`/start`, `/end`):** when the user selects a tracked item for execution, load its full body before any plan or mutation. An unresolved blocker in the body surfaces first and refuses the action until acknowledged.

**Violations this prevents:**
- Plans built on titles that contradict their own bodies
- Re-solving problems already marked resolved in the body
- Acting on items whose bodies record an unmet blocker
- Gates that cover the one remembered failure and miss the rule

---

*Principle violation log: see `CHANGELOG.md` → Principle Violations. Review enforced via Strategic reminder.*
