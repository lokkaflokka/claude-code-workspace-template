# /consistency-check

Verify that the workspace registry matches reality and flag any drift.

## What This Command Does

1. **Registry Drift Check**
   - List all project directories in the workspace
   - Compare against `_shared/PROJECTS.md` registry
   - Flag projects that exist but aren't in the registry
   - Flag registry entries for projects that don't exist

2. **CLAUDE.md Coverage Check**
   - For each project in the registry, verify CLAUDE.md exists
   - Flag projects missing CLAUDE.md

3. **Technique Freshness Check**
   - Scan `_shared/TECHNIQUES.md` for documented techniques
   - Cross-reference with `_shared/EVALUATION_LOG.md`
   - Flag techniques documented but never evaluated
   - Flag techniques documented more than 14 days ago without evaluation

4. **Tag Drift Check** (for MCP packages or versioned artifacts)
   - For each package: find latest git tag, count commits past it
   - Flag if commits > 0 past latest tag (version bump needed)
   - Flag if no tags exist at all (baseline tag needed)
   - Check that `package.json` version matches latest tag

5. **State Freshness Check**
   - Check `_shared/CURRENT_STATE.md` last updated date
   - Flag if more than 7 days stale

6. **Template Staleness Check** (if using a workspace template)
   - Compare template's last tag date against source system's last modification dates
   - Flag if source files (CLAUDE.md, consistency-check, TECHNIQUES.md) are significantly newer than the template's last release

7. **File Size Check** (prevents context bloat)
   - Check `CURRENT_STATE.md` files: flag if >100 lines (target: 50-80)
   - Check `CLAUDE.md` files: flag if >300 lines (target: 200-300)
   - Run: `wc -l */CURRENT_STATE.md _shared/CURRENT_STATE.md` and `wc -l */CLAUDE.md _shared/CLAUDE.md`

## Execution Steps

```bash
# 1. List project directories (excluding hidden, _shared, _template, etc.)
ls -d */ | grep -v "^_" | grep -v "^\."

# 2. Compare to PROJECTS.md registry

# 3. Check CLAUDE.md existence for each project

# 4. Parse TECHNIQUES.md for technique names and dates

# 5. Cross-reference with EVALUATION_LOG.md
```

## Output Format

```
## Consistency Check Results

### Registry Drift
- [OK] All projects in registry exist
- [OK] All directories are registered
  OR
- [DRIFT] Project `foo/` exists but not in PROJECTS.md
- [DRIFT] Registry lists `bar/` but directory doesn't exist

### CLAUDE.md Coverage
- [OK] All projects have CLAUDE.md
  OR
- [MISSING] Project `foo/` has no CLAUDE.md

### Technique Freshness
- [OK] All techniques have been evaluated
  OR
- [STALE] "Technique X" documented YYYY-MM-DD, not evaluated

### Tag Drift
- [OK] my-package: v0.2.0 at HEAD, matches package.json
  OR
- [DRIFT] my-package: 5 commits past v0.1.0, package.json=0.1.0 (bump needed)
- [DRIFT] my-package: no tags (baseline tag needed)

### State Freshness
- [OK] CURRENT_STATE.md updated within 7 days
  OR
- [STALE] CURRENT_STATE.md last updated YYYY-MM-DD (X days ago)

### File Sizes (Context Bloat)
- [OK] _shared/CURRENT_STATE.md: 72 lines (target: 50-80)
- [OK] finance/CLAUDE.md: 255 lines (target: 200-300)
  OR
- [WARN] _shared/CLAUDE.md: 312 lines (target: 200-300) — over limit

### Summary
X issues found / All clear
```

## After Running

**If issues found:**
1. Update `_shared/CURRENT_STATE.md` ALERTS section with findings
2. Suggest remediation actions
3. Offer to fix simple issues (e.g., add missing project to registry)

**If all clear:**
1. Note the check passed in session context
2. Update `_shared/CURRENT_STATE.md` to show "No drift detected" with timestamp

## When to Run

- At the start of sessions at the workspace root
- After creating or deleting projects
- Periodically (weekly) to catch drift
- When something feels "off" about the workspace state
