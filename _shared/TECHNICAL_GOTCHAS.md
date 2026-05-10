# Technical Gotchas

Factual reminders (not behavioral principles). Reference as needed.

Add new gotchas as they're discovered. Each entry should describe a concrete failure mode and the fix — not advice.

---

## How to read "wrong tool for X" entries

Before accepting any categorical "wrong tool for X" exclusion below, distinguish *this operation failed* from *this tool doesn't handle this class of operation*. Tools typically have a safe envelope and a failure envelope; the original lesson may have only tested the failure envelope. Run a small atomic probe of the tool's simplest operation before propagating the exclusion.

If a probe recovers a usable envelope, refine the entry with ✅ safe / ❌ fails / ⚠️ caveat rows rather than leaving the categorical version in place.

---

## Claude Code Hooks
- **SessionStart date injection:** `~/.claude/settings.json` can have a `SessionStart` hook that echoes the current date + day of week. Structurally solves recurring day-of-week derivation failures. If the hook stops working, fall back to running `date` manually.
- **Hook stdout = context:** Exit code 0 + stdout from hook commands is automatically injected into Claude's context. Use this for any "always-available" information.

## CLAUDE.md Reinjection Cost
- **CLAUDE.md is reinjected on every tool call.** Files in the CLAUDE.md hierarchy (root + project) are sent with every inference request. A 5KB CLAUDE.md across 20 tool calls = 100KB of context consumed. Target ≤5KB per CLAUDE.md file. Use the lazy-load pattern (pointer in CLAUDE.md → detail in referenced file) for anything beyond core routing and principles.
- **Custom command descriptions:** Add YAML front matter to `.claude/commands/*.md` to display descriptions in the `/help` dialog:
  ```
  ---
  description: One-line description shown in /help
  ---
  ```
  Without this, commands show with no description in the help UI.
- **`disable-model-invocation` frontmatter:** Controls whether the Skill tool can invoke a command — does NOT affect system context size. Skill bodies load as user-role messages only when explicitly invoked, not at startup.
- **`claude auth status`:** Verify which account Claude Code is authenticated as. Returns JSON with email, authMethod, apiProvider, orgId, subscriptionType. Stored in macOS Keychain — no readable config file.

## Background Agents (`claude -p`)
- **Background agents cannot outlive `claude -p` parents.** When the main session in print/non-interactive mode exits, spawned background subagents (`run_in_background: true`) are killed mid-flight before they can write outputs. Invisible in interactive sessions but fatal for headless/scheduled pipelines. **Rule:** Any step in a `-p` pipeline whose output is required for the final artifact must run **foreground** (blocking). Shared step templates between interactive and headless modes must carry explicit mode overrides.
- **Success markers for multi-step pipelines must verify the final step's artifact, not any artifact.** A loose glob (`*.md`) matches transient/intermediate files and writes a false success marker, hiding silent step failures under "idempotency" (the next run sees the marker and skips). Match the specific expected output pattern (`*synth*.md`, `*-report.json`, etc.).

## Tool Result Reliability
- **Glob/Grep can silently return incomplete results.** Known issues: tilde non-expansion (below), file count cache ignoring patterns, and server-side KV cache stale context after compaction. **Rule:** for critical gate checks ("are there items to process?", "is directory empty?"), verify with `Bash ls` — never trust a zero-result Glob/Grep as proof of absence.
- **Glob does not expand `~`:** Any Glob pattern starting with `~` returns no results silently. Use `Bash ls` for home directory paths, or use the full expanded path.

## WebFetch Known Failures

**Skip WebFetch entirely — use WebSearch instead:**
- `x.com/*`, `twitter.com/*` — JS-rendered, WebFetch returns empty/broken content
- `*.pdf` direct URLs — binary content, WebFetch can't process. Use Read tool for local PDFs
- LinkedIn posts/articles — auth-walled, always returns login page

**One attempt, one fallback, then ask:**
- Standard web articles: WebFetch → if fails, WebSearch → if still insufficient, ask user to paste key content
- Never cascade through 3+ failing fetch attempts on the same URL

**Fetch failure ≠ routing blocked:** When content can't be fetched, still route the URL with whatever context is available. The pipeline can re-attempt later or the user can enrich manually.

## JS-Rendered Portal Escalation
- After 2-3 failed WebFetch attempts on JS-rendered portals (Framer, Azure APIM, React SPAs), stop and ask the person to share a download/export directly. Don't try URL variations past that.
- **Trigger signals:** WebFetch returns only CSS/layout/navigation with no content; 404s on spec URL guesses; search confirms portal exists but doesn't expose content.

## LLM Tool Boundaries
- **Bulk tabular data (>100 rows):** Conversational LLMs receive file "snippets" (first/last rows) and produce wrong results (e.g., reporting 3 distinct values when there are 8). Use local Python for CSV/tabular analysis. MCP data access reduces *workflow friction* (no manual export), but analysis still runs locally.

## External Services
- **CDN cache headers ignored:** Some CDNs (GitHub Gist, others) ignore `Cache-Control` headers. Use `?_cb=${Date.now()}` for cache-busting.
- **Cross-system schemas:** When System A writes data for System B, verify schema match explicitly. Don't trust "it worked before."

## macOS Filesystem
- **TCC (Full Disk Access):** Claude Code cannot access TCC-protected paths (`~/Library/Mail/`, Photos, Contacts) without Full Disk Access granted to Terminal. `dangerouslyDisableSandbox` does NOT help — this is macOS kernel-level enforcement. Workaround: prepare commands for user to run in Terminal (which must have FDA).
- **iCloud Drive exception:** `~/Library/Mobile Documents/` (iCloud Drive) IS accessible — both read and write.

## macOS launchd + Unsigned `.app` Bundles
- **`open -W -a` on unsigned .app bundles is one Gatekeeper refresh away from silent fail.** When a launchd plist's `ProgramArguments` is `[/usr/bin/open, -W, -a, /path/to/Foo.app]` and the .app has no usable code signature, `open` returns exit 0 without launching the app. launchd records `last exit code = 0` — looks healthy. No log entry from the app's script. Works fine until macOS does a Gatekeeper signature-cache refresh, at which point cached green verdicts on unsigned bundles get invalidated and ALL machines using the same shared bundle silent-fail simultaneously.
- **Diagnostic order when `open -W -a` silent-fails:** Run `spctl -a -vv /path/to/Foo.app` and `codesign -dv /path/to/Foo.app` FIRST. If signature is rejected/missing, do not attempt `lsregister -f` re-registration as the fix — Gatekeeper signature enforcement is the cause.
- **Structural fix:** Switch plist `ProgramArguments` from the `.app`-via-`open` envelope to the inner binary directly: `[/path/to/Foo.app/Contents/MacOS/foo-binary]`. Skips Launch Services entirely. Trade-off: TCC attribution may shift from the bundle ID to the binary's path/identity — verify per-app TCC grants survive.

## Bash idioms
- **`VAR=$(grep -c PATTERN FILE 2>/dev/null || echo 0)` is broken when FILE exists with 0 matches.** `grep -c` prints `0\n` AND exits 1 when no matches. The `||` then triggers `echo 0`, which prints another `0\n`. Command substitution captures both: `VAR="0\n0"`. Then `[ "$VAR" -ge 3 ]` fails with `[: 0\n0: integer expression expected`.
- **Fix:** `VAR=$(grep -c PATTERN FILE 2>/dev/null); VAR=${VAR:-0}` — parameter expansion handles the file-missing case (empty output → 0) without double-counting the file-exists-with-0-matches case.

## Shell (zsh)
- **`?` in URLs is a glob wildcard:** zsh treats `?` as a glob pattern. Running `gh api repos/OWNER/REPO/git/trees/HEAD?recursive=1` fails with "zsh: no matches found." Fix: single-quote the URL argument.
- **`$` in double-quoted strings:** Dollar signs get interpreted as variable references. Use single quotes for literal `$` in strings.

## AppleScript
- **Sandbox path resolution:** `path to home folder` inside `tell application "X"` resolves to the app's container directory, NOT the actual `~`. Fix: use `do shell script` with a hardcoded POSIX path.
- **Script Editor re-encodes files:** Opening a UTF-8 `.applescript` file in Script Editor re-saves it as UTF-16. If iterating on an AppleScript file, rewrite the full file each time rather than editing.
- **No heredoc in `do shell script`:** Use `echo` with `quoted form of` instead.
- **Date string parsing produces corrupted dates:** Never pass date strings like `date "2026-02-18 09:00:00 AM"` — locale-dependent parsing silently produces wildly wrong dates. Always construct dates field-by-field (set year, month, day, hours, minutes, seconds individually).
- **`body` returns "missing value" as a value, not an error:** When a reminder has no notes, `body of r` returns the literal `missing value` rather than throwing. Test with `if b is not missing value` not a try/catch.
- **Apple Contacts "1604" year:** When a contact has a birthday stored without a year, Contacts uses year 1604. Month/day are correct; ignore the year.

## Reminders / Task System
- **Read/write tool reliability:** Many reminder CLIs and AppleScript interfaces have asymmetric reliability — reads work but writes (add, complete, edit) may silently fail or target wrong items. Establish a single reliable tool for both reads and writes. Always verify after mutations.
- **Confirm before mutating:** Present proposed reminder changes (title, list, body, recurrence) and wait for user approval before executing mutations. Reads don't need confirmation.
- **`--due` is mandatory** for any move into a list filtered by date. A move/add without `--due` creates an item invisible to date-based filters.
- **Smart punctuation in shell:** Voice/iOS captures commonly produce smart quotes (`"` `"`), smart apostrophes (`'`), and em dashes. Standard double-quote interpolation may store literal escape sequences (`“` text) instead of the actual characters. Prefer `move` over `add` when copying titles to bypass shell quoting entirely. For new items, use `$'...'` ANSI-C quoting or paste the actual characters.

## Gmail API
- **Snooze state is invisible and fragile.** Gmail's snooze-returned threads do NOT appear in `threads.list?labelIds=INBOX` responses. The API consistently undercounts inbox by the number of snoozed-returned threads. `messages.modify removeLabelIds:INBOX` destabilizes snoozed threads' internal display state.
- **Label cycling causes drift.** Repeated add/remove INBOX label on a thread destabilizes Gmail's internal thread index. Validate any archive feature on a throwaway account before using on production inbox.

## Google Sheets
- **Always write raw numbers, never formatted strings.** When updating cells that feed into formulas (SUM, subtraction, etc.), write the numeric value (`32946.52`) not a currency string (`"$ 32,946.52"`). Strings break downstream formulas.

## GCal
- **GCal holds for time-bound reservations:** When processing reservation confirmations (restaurants, appointments, building access) with specific date+time slots, ALWAYS create GCal calendar holds — not just reminders. The calendar hold prevents double-booking. Reminder is optional/secondary.

## MCP Development
- **Tool names:** Only `A-Z, a-z, 0-9, _, -, .` allowed. No colons. Use underscores for namespacing.
- **Orchestrator sync:** When adding features to domain packages, update orchestrator wrapper in same session.
- **Build after commit:** Always run build after committing source changes to any MCP package. Verify `dist/` is newer than `src/`. Without this, the MCP server runs stale code.

## Obsidian Race Condition on Edit Tool
- **Problem:** Obsidian auto-saves with minor reformatting (trailing whitespace, newlines) between Read and Edit calls, causing "file has been modified since read" errors even immediately after reading.
- **Workaround:** Use Python string replacement via Bash for files actively open in Obsidian:
  ```bash
  python3 -c "
  with open('path/to/file.md', 'r') as f: content = f.read()
  content = content.replace('old string', 'new string', 1)
  with open('path/to/file.md', 'w') as f: f.write(content)
  "
  ```
- **When Edit tool is safe:** Files not currently open in the editor.

## Pandoc .md → .docx Authoring
- Blank line required before every bullet list (without it, bullets collapse into preceding paragraph)
- 4-space indent for sub-bullets (2-space doesn't work in pandoc)
- No `---` horizontal rules (render as ugly decorative lines)
- Backslash `\` for forced line breaks on stacked label lines
- Summary tables over section-per-item for parallel/comparison content
- Avoid em-dashes (AI-generated stigma in professional docs)
- **Post-processing required:** `--reference-doc` sets style defaults but pandoc can override per-element (especially tables). python-docx post-processing forces consistent borders and table width.

## n8n API
- **PUT rejects read-only fields.** The n8n REST API's PUT endpoint for updating workflows rejects `tags` and other properties that appear in GET responses but are read-only. Sending them back causes silent or confusing failures. Strip read-only fields before pushing.

## git-filter-repo
- **Reverts uncommitted changes.** Running `git filter-repo` rewrites history and drops any uncommitted work (staged or unstaged). Always commit all changes before running it.

## Terminal Input
- **Long structured input truncation.** Terminal paste handlers (Ink/React + bracketed paste mode) can silently truncate long structured input — numbered list items dropped, sub-bullets re-anchored to wrong parents. Same content pasted to a plain text editor comes through complete. Invisible to both sides without manual cross-check. **Fix:** write content to a file (e.g., `/tmp/long-prompt.md`), then Read the file via tool — bypasses the terminal paste boundary.

## Estimation
- **Nth-instance discount (~5x).** The first instance of a solved pattern costs ~10x more than the Nth instance (N ≥ 2) once the template is internalized. Calibration estimates anchored on first-instance cost stay anchored unless explicitly updated. **Rule:** after the second instance of any pattern, add explicit Nth-instance discount factor to estimates.

## File Management
- **CURRENT_STATE.md size:** Target 50-80 lines. Extract history to `CHANGELOG.md`.
- **CLAUDE.md size:** Target 200-300 lines, ≤5KB. Extract gotchas or reference sections if approaching the limit.
- **Environment date:** Use `Today's date` from environment or hooks, not UTC timestamps from external systems.

## Slack Drafting (assistant-assisted)
- **No Markdown formatting** — asterisks/italics render literally in Slack. Plain text only.
- **Multiple short messages, not one block** — draft each as a separate send. Reads like typed live, not a memo.
- **Conversational tone** — match the channel's register.
- **Source attribution via hyperlinks** — link docs directly rather than describing them.
- **Stakeholder framing check** — before drafting questions for a named recipient, check their role relative to the question. Asking someone to define their own function's role reads as confused.

## Context Tax (principle)
Every token loaded early in a conversation is re-sent (as cache_read) on every subsequent turn. Optimize init aggressively — savings compound per turn. Cache reads are 10x cheaper than base input on Claude models, but they're not free. Per-invocation cost compounds faster than per-session cost.
