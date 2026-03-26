---
description: Pre-session-end checklist - ensures documentation, learning capture, and intelligent branch recommendations
argument-hint: [--quick]
---

# Pre-Clear Session End Checklist

Comprehensive pre-session-end command to ensure nothing is lost before ending a Claude Code session. Captures generalizable learnings, verifies documentation, and provides intelligent branch recommendations.

---

## Phase 1: Documentation Verification

### 1.1 Check Core Documentation Files

Verify documentation reflects current state:

```bash
# Get list of recently modified files (last 7 days)
git log --since="7 days ago" --name-only --pretty=format:"" | sort | uniq

# Check if README.md was updated recently
git log --since="7 days ago" --oneline -- README.md | wc -l

# Check if CLAUDE.md was updated recently
git log --since="7 days ago" --oneline -- CLAUDE.md | wc -l

# Check if design docs were updated
git log --since="7 days ago" --oneline -- design_docs/ | wc -l
```

**Analysis:**
- Identify which components were modified (algorithms/, cli/, compressor/, tests/, scripts/, design_docs/)
- Check if corresponding documentation files were updated
- Flag mismatches (code changed but docs didn't)

**Documentation Mapping:**
- `compressor/algorithms/*` → README.md (Features), COMPRESSION_ALGORITHM_DESIGN.md
- `compressor/database/*` → DATABASE_DESIGN_PHILOSOPHY.md, DATABASE_OPERATIONS_GUIDE.md
- `cli/*` → CLI_AND_JOB_SYSTEM_INTEGRATION.md, USAGE_GUIDE.md, README.md
- `tests/*` → TESTING.md
- `scripts/*` → BENCHMARKING_GUIDE.md, README.md
- `compressor/pipeline.py` → SYSTEM_ARCHITECTURE.md, DATA_FLOW_ARCHITECTURE.md

---

### 1.2 Validate todo.md Status

Check for orphaned or stale tasks:

```bash
# Check for in_progress tasks
grep -i "status.*in_progress\|in.progress" todo.md | head -5 || echo "[OK] No in_progress tasks"

# Check for recently completed tasks
grep -i "status.*completed\|completed" todo.md | head -5 || echo "[OK] No completed tasks to archive"

# Check todo.md last modified
git log -1 --format="%ar" -- todo.md || echo "Never modified"
```

**Validate:**
- [OK] No orphaned "in_progress" tasks (or they match current work)
- [OK] Completed tasks ready to be archived
- [WARN] todo.md modified recently (good sign of active tracking)

---

## Phase 2: Git Status & Branch Management

### 2.1 Working Tree Status

Check current git state:

```bash
# Show current branch
BRANCH=$(git branch --show-current)
echo "Current branch: $BRANCH"

# Check for uncommitted changes
git status --short

# Full status
git status
```

**Validate:**
- [OK] Working tree clean (no uncommitted changes)
- [WARN] Uncommitted changes exist (need to commit or stash)

---

### 2.2 Commits Analysis

Review commits to be pushed:

```bash
BRANCH=$(git branch --show-current)

# Count commits ahead of remote
git rev-list --count origin/$BRANCH..$BRANCH 2>/dev/null || echo "New branch or no remote"

# List commits to push
echo "Commits to push:"
git log origin/$BRANCH..$BRANCH --oneline --no-merges 2>/dev/null || git log $BRANCH --oneline --no-merges -5

# Show files changed
git diff origin/$BRANCH..$BRANCH --stat 2>/dev/null || echo "New branch - all files are new"
```

**Display:**
- Number of commits to push
- Commit messages (check quality)
- Files changed summary

---

### 2.3 Remote Sync Status

Check relationship with remote:

```bash
BRANCH=$(git branch --show-current)

# Fetch latest remote state (quietly)
git fetch origin --quiet 2>/dev/null || echo "[WARN] Could not fetch from remote"

# Check if behind remote
BEHIND=$(git rev-list --count $BRANCH..origin/$BRANCH 2>/dev/null || echo "0")
AHEAD=$(git rev-list --count origin/$BRANCH..$BRANCH 2>/dev/null || echo "0")

echo "Behind remote: $BEHIND commits"
echo "Ahead of remote: $AHEAD commits"

# Check if remote branch exists
git rev-parse origin/$BRANCH >/dev/null 2>&1 && echo "[OK] Remote branch exists" || echo "[INFO] New branch - no remote yet"
```

**Validate:**
- [OK] Not behind remote (or only ahead)
- [WARN] Behind remote (need to pull/rebase)
- [INFO] New branch (no remote yet)

---

### 2.4 Branch Recommendation

Based on analysis, suggest next action:

**Decision Logic:**

```bash
BRANCH=$(git branch --show-current)
UNCOMMITTED=$(git status --short | wc -l)
AHEAD=$(git rev-list --count origin/$BRANCH..$BRANCH 2>/dev/null || echo "0")
BEHIND=$(git rev-list --count $BRANCH..origin/$BRANCH 2>/dev/null || echo "0")
IS_MAIN=$(echo $BRANCH | grep -E "^(main|master)$" && echo "yes" || echo "no")
```

**Recommendations:**

| Condition | Recommendation |
|-----------|----------------|
| Uncommitted changes > 0 | "Commit or stash changes first" |
| Behind remote > 0 | "Pull/rebase from remote first: git pull origin $BRANCH" |
| Ahead > 0, tests pass, not main | "Ready to push: git push origin $BRANCH" |
| Ahead > 0, on main | "WARNING: Pushing to main - consider feature branch" |
| Ahead = 0, on feature branch | "Consider: Create PR or merge to main" |
| Ahead = 0, on main, pushed | "Session complete - branch is current" |

---

## Phase 3: Security & Cleanup Validation

### 3.1 Debug Statement Scan

Check for debug code in modified files:

```bash
BRANCH=$(git branch --show-current)

# Check for debug statements in changed files
echo "Checking for debug statements..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -E "^\+.*print\(|^\+.*breakpoint\(|^\+.*import pdb|^\+.*console\.log" | head -10 || echo "[OK] No debug statements"

# Check for TODO comments
echo "Checking for unresolved TODOs..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -i "^\+.*TODO" | head -10 || echo "[OK] No new TODOs"
```

**Validate:**
- [OK] No print() debug statements
- [OK] No breakpoint() or pdb imports
- [WARN] TODO comments found (acceptable if intentional)

---

### 3.2 Security Scan

Check for sensitive data:

```bash
BRANCH=$(git branch --show-current)

# Scan for potential secrets
echo "Scanning for secrets..."
git diff origin/$BRANCH..$BRANCH 2>/dev/null | grep -i -E "^\+.*(password|secret|api_key|token|private_key)" | head -5 || echo "[OK] No secrets detected"

# Check for database files
echo "Checking for database files..."
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | grep "\.db$" || echo "[OK] No .db files"

# Check for .env files
git diff origin/$BRANCH..$BRANCH --name-only 2>/dev/null | grep "\.env" || echo "[OK] No .env files"
```

**Validate:**
- [OK] No secrets or credentials
- [OK] No .db result files
- [OK] No .env configuration files
- [ERROR] If secrets found → CRITICAL STOP

---

### 3.3 Database Cleanup Check

Identify database files needing attention:

```bash
# Find test databases (small, temporary)
echo "Test databases (candidates for deletion):"
find . -name "test*.db" -o -name "*_test.db" -size -10M 2>/dev/null | head -5 || echo "[OK] No test databases"

# Find large result databases (may need backup)
echo "Large databases (consider backup):"
find . -name "*.db" -not -path "*/venv/*" -size +100M 2>/dev/null | while read f; do
    SIZE=$(ls -lh "$f" | awk '{print $5}')
    echo "$f ($SIZE)"
done || echo "[OK] No large databases"

# Check for orphaned databases (no recent access)
echo "Old databases (not accessed in 30 days):"
find . -name "*.db" -not -path "*/venv/*" -mtime +30 2>/dev/null | head -5 || echo "[OK] No old databases"
```

**Suggestions:**
- Test DBs: `rm test_*.db` (if safe)
- Large DBs: `python scripts/backup_database.py results.db --compress`
- Old DBs: Move to `data/db_backups/` or delete

---

### 3.4 Temporary File Cleanup

Check for temporary artifacts:

```bash
# Find temp directories
find . -type d -name "temp" -o -name "tmp" -o -name "__pycache__" | head -10 || echo "[OK] No temp dirs"

# Find cache files
find . -name "*.pyc" -o -name ".pytest_cache" -o -name "htmlcov" | head -10 || echo "[OK] No cache files"

# Check for untracked files
echo "Untracked files:"
git ls-files --others --exclude-standard | head -10 || echo "[OK] No untracked files"
```

**Validate:**
- [OK] No temp files to clean
- [WARN] Cache files exist (usually okay, ignored by git)
- [INFO] Untracked files listed (review if needed)

---

## Phase 4: Learning Capture & Generalization

### 4.1 Interactive Learning Prompts

**Display this prompt to user (skip if --quick flag provided):**

```
================================================================================
                          LEARNING CAPTURE
================================================================================

Session insights help improve the entire project!

1. Problem-Solving Pattern
   What approach or technique worked well this session?
   Examples: debugging strategy, workflow optimization, tool usage

   [User input or "none"]

2. Architectural Insights
   Any design decisions or component interactions to document?
   Examples: performance patterns, integration approaches

   [User input or "none"]

3. Code Conventions
   Any new patterns to standardize across the project?
   Examples: naming conventions, error handling, testing patterns

   [User input or "none"]

4. Efficiency Improvements
   What would make future sessions faster or smoother?
   Examples: missing documentation, unclear workflows

   [User input or "none"]

================================================================================
```

---

### 4.2 Documentation Update Suggestions

Based on user responses, suggest specific file updates:

**If Problem-Solving Pattern provided:**
- Suggest adding to: `CLAUDE.md` → "Development Workflow" section
- Suggest adding to: `TESTING.md` → "Best Practices" section (if testing-related)

**If Architectural Insights provided:**
- Analyze which design doc is relevant:
  - Performance → `design_docs/PERFORMANCE_MONITORING_DESIGN.md`
  - Database → `design_docs/DATABASE_DESIGN_PHILOSOPHY.md`
  - Pipeline → `design_docs/DATA_FLOW_ARCHITECTURE.md`
  - Algorithms → `design_docs/COMPRESSION_ALGORITHM_DESIGN.md`
  - CLI → `design_docs/CLI_AND_JOB_SYSTEM_INTEGRATION.md`

**If Code Conventions provided:**
- Suggest adding to: `CLAUDE.md` → "Core Development Rules" section
- Consider creating new section if major convention

**If Efficiency Improvements provided:**
- Suggest updating: `CLAUDE.md` → Documentation or tooling sections
- Create issue/task in `todo.md` if significant improvement needed

**Action:**
```
[LEARNING] Suggestions based on your input:

1. Add to CLAUDE.md (Core Development Rules):
   "Use manager classes for expensive operations requiring caching"

2. Create design doc: design_docs/CACHING_PATTERNS.md
   Document the cached pivot table pattern (95% speedup)

3. Update todo.md:
   Add task: "Document caching patterns for future reference"

Would you like me to make these updates? (yes/no/review)
```

---

## Phase 5: Pre-Push Validation (Light)

### 5.1 Determine if Validation Needed

```bash
AHEAD=$(git rev-list --count origin/$BRANCH..$BRANCH 2>/dev/null || echo "0")

if [ "$AHEAD" -gt 0 ]; then
    echo "[INFO] Unpushed commits detected - running pre-push validation..."
else
    echo "[INFO] No unpushed commits - skipping pre-push validation"
    # Skip to Phase 6
fi
```

---

### 5.2 Quick Test Run

If unpushed commits exist, run core tests:

```bash
# Activate virtual environment
source data-compressor-env/bin/activate 2>/dev/null || echo "[WARN] Could not activate venv"

# Run core algorithm tests (fast, reliable subset)
echo "Running core tests..."
pytest tests/test_algorithms.py::TestGzipCompressor tests/test_algorithms.py::TestLzmaCompressor \
  --tb=short -x --maxfail=2 --disable-warnings -q 2>&1

# Capture exit code
TEST_EXIT=$?

if [ $TEST_EXIT -eq 0 ]; then
    echo "[OK] Core tests passed"
else
    echo "[ERROR] Core tests failed - fix before pushing"
fi
```

**Validate:**
- [OK] Tests pass
- [ERROR] Tests fail → Block push recommendation

---

### 5.3 Commit Message Quality Check

Review commit messages for quality:

```bash
BRANCH=$(git branch --show-current)

echo "Reviewing commit messages..."
git log origin/$BRANCH..$BRANCH --pretty=format:"%h - %s" --no-merges 2>/dev/null | while read commit; do
    # Check for bad patterns
    if echo "$commit" | grep -iq "created by claude\|generated by\|wip\|temp\|fix bug\|update files"; then
        echo "[WARN] Poor message: $commit"
    fi
done || echo "[OK] Commit messages look good"
```

**Message Quality Criteria:**
- [OK] Clear and descriptive
- [OK] Focuses on "why" not just "what"
- [WARN] Generic messages ("fix bug", "update files")
- [ERROR] Contains "created by claude" or "generated by"

---

### 5.4 Push Recommendation

Combine all validation results:

**If all checks pass:**
```
[PASS] PRE-PUSH VALIDATION
  [OK] Core tests pass
  [OK] Security scan clear
  [OK] Commit messages quality good

  --> All checks passed
  --> SAFE TO PUSH: git push origin [branch]
```

**If warnings exist:**
```
[WARN] PRE-PUSH VALIDATION
  [OK] Core tests pass
  [OK] Security scan clear
  [WARN] 1 commit message could be improved

  --> All checks passed with warnings
  --> Safe to push (review warnings first)
  --> Suggest: git push origin [branch]
```

**If blockers exist:**
```
[FAIL] PRE-PUSH VALIDATION
  [ERROR] Core tests failed
  [OK] Security scan clear
  [WARN] Commit messages need review

  --> BLOCKED: Fix test failures before pushing
  --> Do not push until all errors resolved
```

---

## Phase 6: Next Session Preparation

### 6.1 Interactive Session Planning

**Prompt user:**

```
================================================================================
                        NEXT SESSION PREPARATION
================================================================================

Help set up for a smooth continuation:

1. What's your top priority for the next session?
   Examples: "Complete dashboard refactoring", "Fix failing tests", "Add feature X"

   [User input]

2. Any blockers or open questions?
   Examples: "Waiting on data", "Need to research approach", "Unclear requirements"

   [User input or "none"]

3. Environment setup needed?
   Examples: "Install new dependency", "Download dataset", "Update config"

   [User input or "none"]

================================================================================
```

---

### 6.2 Update todo.md (Optional)

**If priority provided, suggest:**

```
[INFO] Would you like to add this to todo.md?

Priority: [user's priority]
Status: pending
Context: [brief context from session]

(yes/no)
```

---

## Phase 7: Summary Checklist Output

### 7.1 Generate Comprehensive Report

Compile all results into terminal checklist:

```
================================================================================
                PRE-CLEAR SESSION END CHECKLIST
================================================================================

[PASS] DOCUMENTATION
  [OK] README.md current (updated 2 days ago)
  [OK] CLAUDE.md up to date (updated 5 days ago)
  [WARN] SYSTEM_ARCHITECTURE.md not updated (modified code in compressor/)
  [OK] todo.md status clean (no orphaned tasks)

[PASS] GIT STATUS
  [OK] Working tree clean
  [WARN] 3 unpushed commits on feature/dashboard
  [OK] Synced with remote (not behind)

  Branch Recommendation:
  --> Ready to push and create PR
  --> Suggest: git push origin feature/dashboard

[PASS] SECURITY & CLEANUP
  [OK] No debug statements
  [OK] No secrets detected
  [WARN] 2 TODO comments in new code (review if intentional)
  [OK] Database cleanup not needed
  [OK] No temp files

[PASS] LEARNING CAPTURED
  --> Problem-solving: "Used manager class for caching expensive operations"
  --> Architecture: "Pivot table caching reduces analysis from 500ms to 5ms"
  --> Convention: "Prefer @cached_property for expensive read-only computations"

  Suggested Documentation Updates:
  --> Add to CLAUDE.md: Caching pattern for expensive operations
  --> Create design_docs/CACHING_PATTERNS.md with pivot table example
  --> Update TESTING.md: Document performance testing for cached operations

[WARN] PRE-PUSH VALIDATION
  [OK] Core tests pass
  [OK] Security scan clear
  [WARN] 1 commit message could be improved: "fix issue" -> be more specific

  --> All checks passed with warnings
  --> Safe to push (review warnings first)

[PASS] NEXT SESSION
  Priority: Complete dashboard refactoring with manager class integration
  Blockers: None
  Setup needed: None

================================================================================

SUMMARY: 5/6 sections clear, 4 warnings

SUGGESTED ACTIONS:
1. Update SYSTEM_ARCHITECTURE.md with dashboard caching architecture
2. Resolve 2 TODO comments or move to todo.md tracking
3. Improve commit message: "fix issue" -> "Fix dashboard analysis performance with caching"
4. Apply suggested documentation updates (CLAUDE.md, design_docs/)
5. Push to remote: git push origin feature/dashboard
6. Consider creating PR: "Dashboard performance optimization with cached pivot tables"

================================================================================
```

---

### 7.2 Final Decision Point

**Ask user:**

```
Would you like to:
1. Push to remote now (if validated)
2. Review warnings/suggestions first
3. Make documentation updates
4. Exit without pushing

Your choice? (1/2/3/4)
```

**Handle response:**
- **1 (Push)**: Execute `git push origin $BRANCH` if validation passed
- **2 (Review)**: Display detailed warning/error messages
- **3 (Updates)**: Offer to make suggested doc updates
- **4 (Exit)**: Clean exit with summary

---

## Usage Examples

```bash
# Full interactive session-end checklist
/preclear

# Quick mode (skip learning prompts, just run validations)
/preclear --quick
```

---

## Command Execution Flow

1. [FAST] Documentation verification (~2s)
2. [FAST] Git status analysis (~1s)
3. [MEDIUM] Security & cleanup scans (~3s)
4. [INTERACTIVE] Learning capture prompts (user-dependent)
5. [SLOW] Pre-push validation (~5-10s if tests run)
6. [INTERACTIVE] Next session prep (user-dependent)
7. [FAST] Summary generation (~1s)
8. [INTERACTIVE] Final decision point

**Total time:** ~15-20 seconds (excluding interactive prompts)

---

## Safety Features

- **Non-destructive**: Only reads and suggests, never auto-commits or auto-pushes
- **Flexible**: Can proceed with warnings (user choice), blocks on critical errors
- **Cross-platform**: No emojis, uses [OK]/[WARN]/[ERROR] markers
- **Learning-focused**: Captures generalizable patterns, not just personal notes
- **Intelligent**: Context-aware branch recommendations
- **Fast**: Lightweight checks, optional full validation

---

## Important Notes

- **Learning capture** is the key differentiator from /prepush and /end-of-day
- **Branch recommendations** help clarify next steps (push, PR, merge, delete)
- **Documentation hygiene** ensures insights don't get lost
- **Quick mode** (--quick) skips interactive prompts for fast validation
- **Safe by default**: Never modifies files without user confirmation

---

## Symbol Legend

**Section Status:**
- `[PASS]` - Section passed all checks
- `[WARN]` - Section has warnings (non-blocking)
- `[FAIL]` - Section has critical errors (blocking)

**Individual Checks:**
- `[OK]` - Check passed
- `[WARN]` - Check has warning
- `[ERROR]` - Check failed
- `[INFO]` - Informational message
- `-->` - Action or recommendation

---

*Generated by /preclear command - Pre-session-end checklist for Claude Code sessions*
