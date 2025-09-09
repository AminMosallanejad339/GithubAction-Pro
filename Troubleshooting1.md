# Git Troubleshooting: Divergent Branches Resolution

## 🚨 Problem Scenario

### Initial State
- Local branch: `main` with 2 new commits
- Remote branch: `origin/main` with 2 new commits
- Branches have **diverged** (each has commits the other doesn't have)

### Error Encountered
```bash
$ git push
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/username/repo.git'
```

## 🔍 Root Cause Analysis

**Divergent Branches**: Your local branch and remote branch have different commit histories that cannot be fast-forwarded.

```
Local:  A - B - C - D - E [your new commits]
Remote: A - B - C - F - G [someone else's commits]
```

## 🛠️ Resolution Methods

### Method 1: Merge Strategy (Recommended for beginners)
```bash
# Set merge as default strategy (optional)
git config pull.rebase false

# Pull remote changes and create merge commit
git pull --no-rebase

# Resolve any merge conflicts if they occur
# Add resolved files and commit
git add .
git commit -m "Merge remote changes"

# Push the merged history
git push
```

### Method 2: Rebase Strategy (Clean history)
```bash
# Set rebase as default strategy (optional)
git config pull.rebase true

# Rebase your commits on top of remote changes
git pull --rebase

# Resolve any conflicts during rebase
# Continue rebase after conflict resolution
git rebase --continue

# Push the rebased history
git push
```

### Method 3: Force Push (Use with caution - DESTRUCTIVE)
```bash
# Only use if you're sure you want to overwrite remote
git push --force
# Or safer alternative:
git push --force-with-lease
```

## ✅ Step-by-Step Solution Applied

### What Actually Worked:
```bash
# 1. Rebase local commits on top of remote changes
git pull --rebase

# 2. Resolve any conflicts (if any occurred)
# 3. Complete the rebase process
git rebase --continue

# 4. Push the clean, linear history
git push
```

## 📊 Visual Representation

### Before Resolution:
```
Local:  A - B - C - D - E
Remote: A - B - C - F - G
```

### After Rebase:
```
Both:   A - B - C - F - G - D' - E'
```

## 🛡️ Prevention Strategies

### 1. Always Pull Before Working
```bash
git pull origin main
# or set upstream tracking
git branch --set-upstream-to=origin/main main
```

### 2. Configure Default Pull Behavior
```bash
# Set global default to merge (safe)
git config --global pull.rebase false

# Or set to rebase (clean history)
git config --global pull.rebase true
```

### 3. Use Pre-commit Hooks
Create a pre-commit hook to ensure you're up-to-date with remote.

### 4. Regular Sync Routine
```bash
# Daily workflow
git fetch origin
git merge origin/main
# or
git rebase origin/main
```

## ⚠️ Common Pitfalls & Solutions

### Merge Conflicts During Rebase
```bash
# If conflicts occur during rebase:
git status # See conflicted files
# Edit files to resolve conflicts
git add <resolved-files>
git rebase --continue
```

### Aborting Operations
```bash
# To abort a failed rebase:
git rebase --abort

# To abort a merge:
git merge --abort
```

### Checking Branch Status
```bash
# Always check before pushing
git status
git log --oneline --graph --all
```

## 🎯 Best Practices

1. **Pull before push**: Always ensure your local is updated
2. **Small commits**: Make frequent, small commits rather than large ones
3. **Regular pushes**: Push changes regularly to avoid large divergences
4. **Communication**: Coordinate with team members to avoid simultaneous changes
5. **Backup**: Always have a backup before force pushing

## 📋 Quick Reference Cheat Sheet

| Situation          | Command                       | Risk Level |
| ------------------ | ----------------------------- | ---------- |
| Divergent branches | `git pull --rebase`           | Low        |
| Merge conflicts    | `git mergetool`               | Medium     |
| Abort operation    | `git rebase --abort`          | None       |
| Force push         | `git push --force-with-lease` | High       |

 