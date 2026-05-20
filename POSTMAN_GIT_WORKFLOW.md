# Postman + Git Workflow Guide

This guide explains how to manage Postman v3 collections with git while minimizing cloud sync conflicts.

## The Conflict Problem

With Postman v3, you have TWO sources of truth:
1. **Git Repository** - Your local files committed to version control
2. **Postman Cloud** - Synced via `.postman/resources.yaml`

When changes occur in both places, conflicts arise.

## Our Solution: Git-First Workflow

We use **git as the primary source** with these configurations:

### 1. Git Merge Strategy

The `.gitattributes` file includes:
```
.postman/resources.yaml merge=ours
```

This tells git to **prefer your local version** during merge conflicts.

### 2. Workflow Rules

Follow these rules to minimize conflicts:

#### ✅ DO:
- **Always pull before editing** in Postman UI
- **Commit changes frequently** to git
- **Push to origin** regularly
- **Let Postman sync from git** (it reads the files)
- **Use git as source of truth** for collection structure

#### ❌ DON'T:
- Don't make concurrent edits in Postman UI and git
- Don't edit in Postman Cloud web UI (use desktop app)
- Don't ignore `resources.yaml` changes
- Don't force-push without pulling first

### 3. Recommended Workflow

**Daily Workflow:**
```bash
# 1. Start of day - pull latest
git pull origin main

# 2. Make changes in Postman Desktop
# (Postman auto-saves to local files)

# 3. Review changes
git status
git diff

# 4. Commit and push
git add .
git commit -m "Description of changes"
git push origin main
```

**Team Collaboration:**
```bash
# Before editing
git pull origin main

# After editing (check for conflicts)
git status

# If resources.yaml shows as modified:
git add .postman/resources.yaml
git commit -m "Update Postman sync config"
git push origin main
```

## Handling Sync Conflicts

### Scenario 1: resources.yaml Conflict

**Symptom:** Git shows `.postman/resources.yaml` as modified

**Solution:**
```bash
# Option A: Accept the change
git add .postman/resources.yaml
git commit -m "Update Postman cloud sync"

# Option B: Discard the change (if minor)
git restore .postman/resources.yaml
```

### Scenario 2: Collection Out of Sync

**Symptom:** Postman shows "out of sync" warning

**Solution:**
1. **In Postman Desktop:**
   - Right-click collection
   - Choose "Pull" to sync from git
   
2. **Or restart Postman:**
   - Close Postman completely
   - Reopen (it will resync from files)

### Scenario 3: Merge Conflict on Pull

**Symptom:** Git merge conflict in collection files

**Solution:**
```bash
# 1. See what changed
git status

# 2. For resources.yaml (use ours strategy)
git checkout --ours .postman/resources.yaml
git add .postman/resources.yaml

# 3. For collection files (.yaml in Prisma AIRS/)
# Manually review and merge, OR:
git checkout --ours "Prisma AIRS/.resources/definition.yaml"
git add "Prisma AIRS/.resources/definition.yaml"

# 4. Complete the merge
git commit
```

## Best Practices

### 1. Single Source Workflow

**Choose ONE primary editing location:**
- ✅ **Postman Desktop App** (recommended) - Changes auto-save to git files
- ❌ Postman Web UI - Can cause sync issues
- ❌ Direct YAML editing - Harder to validate

### 2. Variable Management

**For sensitive variables (API keys, passwords):**
- Use **Current Value** in Postman (never synced)
- Keep **Initial Value** as placeholder `{{variable_name}}`
- See `.env.example` for reference values

**Example in definition.yaml:**
```yaml
variables:
  x-pan-token: "{{x-pan-token}}"        # Initial: placeholder
  scm_username: "{{scm_username}}"      # Initial: placeholder
  scm_password: "{{scm_password}}"      # Initial: placeholder
```

**In Postman UI:**
| Variable | Initial Value | Current Value (local only) |
|----------|---------------|----------------------------|
| x-pan-token | `{{x-pan-token}}` | `your-actual-api-key` |
| scm_username | `{{scm_username}}` | `actual-username` |
| scm_password | `{{scm_password}}` | `actual-password` |

### 3. Commit Hygiene

**Good commit messages:**
```bash
git commit -m "Add OAuth2 token automation for SCM API"
git commit -m "Update DLP detection test cases"
git commit -m "Fix: Correct malicious URL endpoint"
```

**Bad commit messages:**
```bash
git commit -m "update"
git commit -m "changes"
git commit -m "sync"
```

### 4. Pre-Commit Checklist

Before committing:
- [ ] No API keys in Initial Values
- [ ] No hardcoded credentials
- [ ] Test scripts are functional
- [ ] Variable placeholders are correct
- [ ] All requests have descriptions

## Troubleshooting

### "Collection has unsaved changes"

**Cause:** Postman UI changes not yet written to files

**Fix:**
1. Save the request (Ctrl+S / Cmd+S)
2. Wait 2-3 seconds for auto-save
3. Check `git status` to confirm file changed

### "Pull rejected - uncommitted changes"

**Cause:** You have local changes not committed

**Fix:**
```bash
# Option 1: Commit your changes first
git add .
git commit -m "Your changes"
git pull origin main

# Option 2: Stash, pull, then apply
git stash
git pull origin main
git stash pop
```

### "Postman showing old data"

**Cause:** Postman cached old version

**Fix:**
1. Close Postman completely (quit application)
2. Pull latest from git: `git pull origin main`
3. Reopen Postman
4. Or: Right-click collection → "Pull from folder"

## Advanced: Disabling Cloud Sync (Optional)

If you want **git-only** (no Postman Cloud):

**Option 1: Remove Cloud Link**
```bash
# Remove cloud sync configuration
rm .postman/resources.yaml

# Add to .gitignore
echo ".postman/resources.yaml" >> .gitignore

# Commit
git add .gitignore
git commit -m "Disable Postman Cloud sync - use git only"
```

**Option 2: Keep Link (Current Setup)**
- Keep `.postman/resources.yaml` in git
- Use as backup/sharing via Postman web
- Git remains primary source

## Summary

**Our Current Setup:**
- ✅ Git is primary source of truth
- ✅ Postman Desktop syncs from git files
- ✅ Cloud sync enabled as backup
- ✅ Merge conflicts auto-resolved to git version
- ✅ Sensitive variables use "Current Value" only

**Your Workflow:**
1. Pull → Edit in Postman → Commit → Push
2. Resources.yaml changes are normal, just commit them
3. If conflicts arise, git keeps your version (merge=ours)

**Questions?**
See team in GitHub Issues or contact maintainer.
