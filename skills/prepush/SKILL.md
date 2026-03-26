---
description: Comprehensive validation before pushing to remote - ensures everything is tidy and documented
---

# Pre-Push Validation Command

Comprehensive pre-push checklist to ensure everything is clean, tested, documented, and ready to push to remote.

---

## Phase 1: Repository Status Analysis

### 1.1 Git Working Tree Check

Verify clean working state:

```bash
# Check for uncommitted changes
git status --short

# Show current branch
git branch --show-current

# Check relationship with remote
git status
```

**Validate:**
- ✅ Working tree is clean (no uncommitted changes)
- ✅ Current branch identified
- ✅ Branch has commits to push

**If uncommitted changes exist:**
- ❌ STOP - Ask user: "You have uncommitted changes. Commit them first or stash? (commit/stash/cancel)"

---

### 1.2 Commits to Push Analysis

Review what will be pushed:

```bash
# Get remote branch name
BRANCH=$(git branch --show-current)

# List commits to push
git log origin/$BRANCH..$BRANCH --oneline --no-merges 2>/dev/null || git log $BRANCH --oneline --no-merges -5

# Show summary statistics
git diff origin/$BRANCH..$BRANCH --stat 2>/dev/null || echo "New branch - will create on remote"
```

**Display:**
- Commit count to be pushed
- Each commit hash and message
- Files changed summary
- Whether this is a new branch

---

### 1.3 Branch Safety Check

Verify branch is safe to push:

```bash
BRANCH=$(git branch --show-current)
```

**Check:**
- ✅ Not pushing directly to `main` or `master` (unless user explicitly approves)
- ✅ Branch name is descriptive
- ✅ Commits are intended for this branch

**If on main/master:**
- ⚠️ WARNING - Ask: "You're on $BRANCH. Are you sure you want to push to main? (yes/no)"

---

## Phase 2: Test & Quality Validation

### 2.1 Test Suite Execution

Run comprehensive test suite:

```bash
# Activate virtual environment
source data-compressor-env/bin/activate

# Run core tests (fast, reliable subset)
echo "Running core algorithm tests..."
pytest tests/test_algorithms.py::TestGzipCompressor tests/test_algorithms.py::TestLzmaCompressor \
  --tb=short -x --maxfail=2 --disable-warnings -q

# Optional: Run broader test suite if time permits
# pytest -v --tb=short
```

**Quality Gates:**
- ✅ Core tests pass
- ✅ No unexpected failures

**If tests fail:**
- ❌ STOP - Display failures and ask: "Tests failed. Fix before pushing? (yes/cancel)"

---

### 2.2 Code Quality Scan

Check for common issues:

```bash
BRANCH=$(git branch --show-current)

# Check for debug statements in changed files
echo "Checking for debug statements..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -E "^\+.*print\(|^\+.*breakpoint\(|^\+.*import pdb" || echo "✅ No debug statements"

# Check for TODO comments in new code
echo "Checking for unresolved TODOs..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -i "^\+.*TODO" || echo "✅ No new TODOs"

# Check for large files
echo "Checking for large files..."
git diff origin/$BRANCH..$BRANCH --stat 2>/dev/null | grep -E "Bin|[0-9]{4,} insertions" || echo "✅ No large files"
```

**Validate:**
- ✅ No print() debug statements
- ✅ No breakpoint() or pdb imports
- ✅ No unresolved TODO comments
- ✅ No unexpectedly large files

**If issues found:**
- ⚠️ WARNING - List issues and ask user to review

---

### 2.3 Security Scan

Check for security issues:

```bash
BRANCH=$(git branch --show-current)

# Check for potential secrets
echo "Scanning for secrets..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -i -E "^\+.*(password|secret|api_key|token|private_key)" || echo "✅ No secrets detected"

# Check for database files
echo "Checking for database files..."
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | grep "\.db$" || echo "✅ No .db files"

# Check for .env files
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | grep "\.env" || echo "✅ No .env files"
```

**Validate:**
- ✅ No secrets or credentials
- ✅ No .db result files
- ✅ No .env configuration files

**If secrets found:**
- 🚨 CRITICAL STOP - "SECURITY ISSUE: Potential secrets detected. Review immediately!"

---

## Phase 3: Documentation Validation

### 3.1 Changed Files Analysis

Identify what was changed:

```bash
BRANCH=$(git branch --show-current)

# Get changed files grouped by directory
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | sed 's|/.*||' | sort | uniq -c
```

**Categorize changes:**
- `compressor/algorithms/` → Algorithm implementation
- `compressor/database/` → Database schema/operations
- `cli/` → CLI interface
- `tests/` → Test infrastructure
- `scripts/` → Benchmarking tools
- `design_docs/` → Documentation

---

### 3.2 Documentation Coverage Check

Based on changed components, verify required docs are updated:

**Mapping:**

| Component Changed | Required Documentation |
|------------------|------------------------|
| `compressor/algorithms/*` | COMPRESSION_ALGORITHM_DESIGN.md, README.md |
| `compressor/database/*` | DATABASE_DESIGN_PHILOSOPHY.md, DATABASE_OPERATIONS_GUIDE.md |
| `cli/*` | CLI_AND_JOB_SYSTEM_INTEGRATION.md, USAGE_GUIDE.md, README.md |
| `tests/*` | TESTING.md |
| `scripts/*` | BENCHMARKING_GUIDE.md, README.md |
| `compressor/pipeline.py` | SYSTEM_ARCHITECTURE.md, DATA_FLOW_ARCHITECTURE.md |
| `compressor/metrics/*` | PERFORMANCE_MONITORING_DESIGN.md |

**Check each relevant doc:**

```bash
BRANCH=$(git branch --show-current)

# Check if docs were updated in this push
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | grep -E "\.md$|README"
```

**Verify:**
- ✅ Relevant documentation files were updated
- ✅ README.md reflects new features/changes
- ✅ Architecture docs align with code changes

**If documentation missing:**
- ⚠️ WARNING - List which docs should be updated: "Consider updating: [list]"

---

### 3.3 Todo.md Status Check

Verify todo.md reflects current state:

```bash
# Check for in-progress tasks
grep -A 1 '"status".*"in_progress"' todo.md || echo "No tasks in-progress"

# Check for completed tasks
grep -A 1 '"status".*"completed"' todo.md || echo "No completed tasks to archive"
```

**Validate:**
- ✅ No orphaned "in_progress" tasks
- ✅ Completed tasks match work being pushed
- ✅ Todo list is up to date

**If inconsistencies:**
- ⚠️ WARNING - "Consider updating todo.md to reflect current state"

---

## Phase 4: Commit Message Quality

### 4.1 Review Commit Messages

Analyze commit message quality:

```bash
BRANCH=$(git branch --show-current)

# Show all commit messages to be pushed
git log origin/$BRANCH..$BRANCH --pretty=format:"%h - %s" --no-merges 2>/dev/null || git log $BRANCH --pretty=format:"%h - %s" --no-merges -5
```

**Check each message for:**
- ✅ Clear and descriptive (not "fix bug", "update files")
- ✅ Focuses on "why" not just "what"
- ✅ Professional tone
- ❌ NO "created by claude" or "generated by"
- ❌ NO "WIP" or "temp" messages

**Message Quality Score:**
- **Good**: "Add performance indexes to eliminate O(n) table scans"
- **Good**: "Fix CLI verify command mock patterns for test reliability"
- **Bad**: "Update database" ❌
- **Bad**: "Fix tests" ❌
- **Bad**: "Created by claude: Add feature" ❌

**If poor messages found:**
- ⚠️ WARNING - "Some commit messages could be improved. Amend before pushing? (yes/no)"

---

## Phase 5: Remote Sync Check

### 5.1 Fetch Latest Remote State

Update remote tracking:

```bash
echo "Fetching latest from remote..."
git fetch origin --quiet

BRANCH=$(git branch --show-current)
```

---

### 5.2 Check for Divergence

Verify branch relationship:

```bash
# Check if behind remote
git rev-list --count $BRANCH..origin/$BRANCH 2>/dev/null || echo "0"

# Check if ahead of remote
git rev-list --count origin/$BRANCH..$BRANCH 2>/dev/null || echo "New branch"
```

**Validate:**
- ✅ Branch is up-to-date with remote (or new branch)
- ✅ No divergence (behind remote)

**If behind remote:**
- ⚠️ WARNING - "Your branch is behind origin/$BRANCH. Pull/rebase first? (yes/cancel)"

---

## Phase 6: Pre-Push Summary

### 6.1 Generate Validation Report

Create comprehensive summary:

```markdown
# Pre-Push Validation Report

## 📋 Repository Status
- **Branch:** [current-branch]
- **Status:** Clean working tree ✅ / Uncommitted changes ❌
- **Commits to push:** [N]
- **Files changed:** [N]

## 🧪 Quality Gates
- **Tests:** PASS ✅ / FAIL ❌
- **Debug statements:** Clean ✅ / Found ⚠️
- **Security scan:** Clear ✅ / Issues 🚨
- **Code quality:** Good ✅ / Needs review ⚠️

## 📚 Documentation
- **Relevant docs updated:** Yes ✅ / Needs update ⚠️
- **README.md:** Updated ✅ / Check ⚠️
- **todo.md:** Current ✅ / Stale ⚠️

## 📝 Commits to Push
[List each commit with hash and message]

## 📊 Files Changed
[Summary by component: algorithms/, cli/, tests/, etc.]

## 🔍 Validation Checks
- [✅/⚠️/❌] Working tree clean
- [✅/⚠️/❌] Tests pass
- [✅/⚠️/❌] No debug statements
- [✅/⚠️/❌] No secrets detected
- [✅/⚠️/❌] Documentation updated
- [✅/⚠️/❌] Commit messages quality
- [✅/⚠️/❌] Synced with remote

## ⚠️ Warnings
[List any warnings or issues that need review]

## 🚨 Blockers
[List any critical issues that must be fixed before pushing]
```

---

### 6.2 User Decision Point

**If ALL checks passed (all ✅):**
```
🎉 All validation checks passed!

Ready to push [N] commits to origin/[branch]?
(yes/no)
```

**If warnings exist (some ⚠️):**
```
⚠️ Validation completed with warnings:
[List warnings]

These are non-blocking. Proceed with push anyway?
(yes/no/review)
```

**If blockers exist (any ❌ or 🚨):**
```
❌ Critical issues must be resolved before pushing:
[List blockers]

Push is blocked. Fix these issues first.
```

---

## Phase 7: Execute Push

### 7.1 Push to Remote

After user approval:

```bash
BRANCH=$(git branch --show-current)

echo "Pushing to origin/$BRANCH..."
git push origin $BRANCH

# Capture result
if [ $? -eq 0 ]; then
    echo "✅ Push successful!"
else
    echo "❌ Push failed - check error messages above"
    exit 1
fi
```

---

### 7.2 Post-Push Verification

Verify push succeeded:

```bash
BRANCH=$(git branch --show-current)

# Verify branch is up-to-date
git fetch origin --quiet
git status | grep "Your branch is up to date"
```

**Confirm:**
- ✅ Push completed successfully
- ✅ Local branch matches remote
- ✅ No errors or warnings

---

### 7.3 Post-Push Summary

Display results:

```markdown
# Push Complete! ✅

## Summary
- **Branch:** origin/[branch]
- **Commits pushed:** [N]
- **Files changed:** [N]
- **Remote URL:** [git remote URL]

## Next Steps
Consider:
- [ ] Create pull request (if feature branch)?
- [ ] Merge to main (if on dev)?
- [ ] Notify team members?
- [ ] Update project board/issues?
- [ ] Tag a release (if appropriate)?
```

---

## Usage Examples

```bash
# Basic usage - validate and push current branch
/prepush

# The command will:
# 1. Check working tree is clean
# 2. Run tests
# 3. Scan for code quality issues
# 4. Verify documentation updated
# 5. Review commit messages
# 6. Sync with remote
# 7. Show comprehensive summary
# 8. Wait for approval
# 9. Push to remote
```

---

## Safety Features

- ✅ **Multiple checkpoints**: User approval required before push
- ✅ **Rollback safe**: Only pushes, no destructive local operations
- ✅ **Security scanning**: Prevents accidental secret commits
- ✅ **Documentation enforcement**: Ensures docs stay in sync
- ✅ **Test validation**: Catches broken code before it reaches remote
- ✅ **Commit quality**: Encourages professional commit messages
- ✅ **Branch protection**: Warns before pushing to main/master

---

## Important Notes

- **Non-destructive**: This command only reads and pushes, never modifies local state
- **Flexible**: Can proceed with warnings (user choice), but blocks on critical errors
- **Fast tests**: Uses core algorithm tests for speed, not full suite
- **Git hooks compatible**: Works alongside pre-commit hooks if installed

---

## Execution Flow

1. ✅ Check working tree is clean
2. ✅ Analyze commits to push
3. ✅ Verify branch safety
4. ✅ Run test suite
5. ✅ Scan code quality
6. ✅ Security validation
7. ✅ Documentation coverage check
8. ✅ Commit message quality review
9. ✅ Sync with remote
10. 📋 Generate validation report
11. ❓ Wait for user approval
12. 🚀 Push to remote
13. ✅ Post-push verification & summary
