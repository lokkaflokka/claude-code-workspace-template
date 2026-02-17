# /reflect

Session-end reflection and state persistence. Run this before closing out any session to ensure all work is captured in files.

## Checklist

Execute all items. Do not skip or compress under time pressure — that's the whole point of this skill.

### 1. Update CURRENT_STATE.md

Read `_shared/CURRENT_STATE.md`. Update:
- **Last Session** section: date, focus summary
- **Done:** what was accomplished
- **Left off:** what remains, next steps
- **Active Contexts:** reflect any status changes

### 2. Update roadmap (if applicable)

If any roadmap items were progressed this session, update their status. Skip if no roadmap work was done.

### 3. Update project-level state files

If work touched a specific project, update that project's state file. Skip if no project-specific work.

### 4. Package sync check (conditional)

**Only when code/package commits were made this session:**
- Version bump + tag + reference updates per Principle #6
- Build verification if applicable
- Skip entirely if no package work was done

### 5. Vault-first check

Every reminder or tracked item that references a vault file: verify that file contains the relevant content. No dangling pointers.

### 6. Action-reminder pairing

Every item in state file "Action Required" / "Pending Actions" / "Left off" sections: verify it has a paired reminder. No orphaned actions. Create reminders for any unpaired actions.

### 7. Sync capture

Anything generalizable from this session that should cross to another machine? If yes, append to `_shared/sync/staging.md`. If no sync-worthy content: skip.

### 8. Decay check

- Any roadmap phase completed this session: collapse to summary
- Any backlog item marked done: remove from backlog
- Any alert resolved: update or remove from CURRENT_STATE.md

### 9. Backlog sync

If session work completed or invalidated any tracked backlog item, update the entry now. Stale backlog entries waste future sessions re-discovering done work.

### 10. Technique and pattern check

- Was a technique from TECHNIQUES.md applied this session? Update its `Effectiveness:` field.
- Did a novel pattern or methodology emerge worth capturing? Propose a new technique entry.
- Did a process failure or principle violation occur? Log it.
- Write a session entry to your changelog or session log.

### 11. Final scan

Quick gut check: any work discussed in this session but not persisted in files? If yes, persist now.

## Rules

- Execute all items every session. The skill exists to prevent compression/skipping.
- Items with no applicability this session are skipped with zero output (e.g., "no package work" skips step 4).
- The state file update (step 1) and session log entry (step 10) are never skippable.
- Edit discipline: when moving content between sections or files, copy original text verbatim. Only modify structural references.

## Why This Is a Skill

Session-end checklists in CLAUDE.md get compressed or skipped under context pressure — especially when sessions run long. A skill can't be compressed away: invoking `/reflect` loads the full protocol every time, ensuring compliance regardless of session state.
