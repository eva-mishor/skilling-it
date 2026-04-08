---
name: repo-tidy-up
description: Use when performing repo maintenance, or when the user asks to clean up branches, dependencies, stale code, memory, or docs. Also use when "tidy up", "hygiene", "cleanup", or "maintenance" is mentioned.
---

# Repo Tidy-Up

Run through each section. Report findings, fix what's safe, flag what needs user decision.

## 1. Branch Cleanup

```bash
# Show all branches with merge status relative to main
git branch -a --sort=-committerdate

# Identify merged branches (safe to delete)
git branch --merged main | grep -v main

# Identify stale branches (no commits in 30+ days)
git for-each-ref --sort=committerdate --format='%(committerdate:short) %(refname:short)' refs/heads/ refs/remotes/origin/
```

- Delete local+remote branches fully merged into main
- Flag unmerged branches older than 30 days — ask user before deleting
- Run `git fetch --prune` to clean tracking refs

## 2. Dependency Health

```bash
npm outdated
npm audit --audit-level=moderate
```

- Report outdated packages (major vs minor/patch)
- Report security advisories
- Flag any with known breaking changes

## 3. Dead Code & Unused Exports

```bash
npx knip --no-exit-code 2>/dev/null || echo "knip not installed — run: npm i -D knip"
```

- Report unused files, exports, dependencies, devDependencies
- Don't auto-delete — present findings for user decision

## 4. Stale TODOs

```bash
# Find TODOs/FIXMEs with git blame dates
grep -rn "TODO\|FIXME\|HACK\|XXX" src/ supabase/functions/ --include="*.ts" --include="*.tsx" 2>/dev/null
```

- Flag any older than 60 days (check via `git blame`)
- Ask user: resolve, convert to issue, or remove?

## 5. CLAUDE.md Audit

- Read CLAUDE.md "Current Status" section — does it match reality?
- Check for references to deleted files, renamed functions, or completed work still marked as TODO
- Verify env var list is current

## 6. Memory Pruning

```bash
ls -la ~/.claude/projects/*/memory/*.md
```

- Read each memory file's description
- Check if referenced files/functions still exist
- Flag stale or outdated memories for removal
- Verify MEMORY.md index matches actual files

## 7. Untracked Files

```bash
git status --short
```

- Identify untracked files — should they be committed or .gitignored?
- Special attention to lockfiles (deno.lock, package-lock.json) and .env files

## Output Format

Summarize findings as a table:

| Area | Status | Action Needed |
|------|--------|---------------|
| Branches | 2 merged, 1 stale | Delete merged, review stale |
| Dependencies | 3 outdated, 0 vulns | Minor updates available |
| Dead code | knip found 2 unused exports | Review list |
| TODOs | 1 stale (>60 days) | Resolve or remove |
| CLAUDE.md | Status section outdated | Update 2 items |
| Memory | 1 stale entry | Remove |
| Untracked | deno.lock | Add to .gitignore |
