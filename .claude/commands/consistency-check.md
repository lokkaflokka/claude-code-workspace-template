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

4. **State Freshness Check**
   - Check `_shared/CURRENT_STATE.md` last updated date
   - Flag if more than 7 days stale

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

### State Freshness
- [OK] CURRENT_STATE.md updated within 7 days
  OR
- [STALE] CURRENT_STATE.md last updated YYYY-MM-DD (X days ago)

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
