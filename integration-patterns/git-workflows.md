# Git Workflows Deep Dive

## Commit Workflow

### Basic Commit

```
Look at my staged changes and create an appropriate commit
```

Claude will:
1. Run `git status` and `git diff --staged`
2. Analyze the changes
3. Write a descriptive commit message
4. Create the commit

### Commit Message Style

Claude follows repository conventions. For consistency, add to CLAUDE.md:

```markdown
## Git Conventions
- Commit messages: imperative mood ("Add feature" not "Added feature")
- Format: "type: description" (feat:, fix:, refactor:, docs:, test:)
- Keep first line under 72 characters
```

### Pre-Approving Git Commands

In `.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git status *)",
      "Bash(git diff)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git branch *)",
      "Bash(git checkout *)"
    ],
    "deny": [
      "Bash(git push --force*)",
      "Bash(git reset --hard*)"
    ]
  }
}
```

---

## Pull Request Workflow

### Creating PRs

```
Create a PR with a detailed description
```

Claude will:
1. Check current branch and changes
2. Push to remote if needed
3. Create PR with title and description
4. Return the PR URL

### PR Summary Skill

```
/pr-summary
```

Summarizes changes for quick review.

### Starting from a PR

```bash
claude --from-pr 123
```

Loads PR context into session.

### PR Review

```
Review PR #456 for issues:
- Security concerns
- Performance problems
- Missing tests
```

---

## Git Worktrees

### What Are Worktrees?

Multiple working directories from one repo:
```
project/           # main branch
project-feature/   # feature branch (separate directory)
project-hotfix/    # hotfix branch (separate directory)
```

Each has its own files, own Claude session.

### Creating Worktrees

```bash
# Create worktree with new branch
git worktree add ../project-feature -b feature-auth

# Create worktree from existing branch
git worktree add ../project-hotfix hotfix-123
```

### Worktree + Claude Workflow

**Terminal 1:**
```bash
cd ../project-feature
claude --name "feature-auth"
# Work on auth feature
```

**Terminal 2:**
```bash
cd ../project-hotfix
claude --name "hotfix-123"
# Fix urgent bug
```

No conflicts - completely separate directories.

### Managing Worktrees

```bash
# List all worktrees
git worktree list

# Remove worktree (after merging)
git worktree remove ../project-feature

# Prune stale worktrees
git worktree prune
```

---

## Git Hooks

### Claude Code Hooks

Create in `.claude/hooks/`:

**protect-branches.sh** - Prevent commits to protected branches:
```bash
#!/bin/bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
PROTECTED=("main" "master" "production")

for protected in "${PROTECTED[@]}"; do
  if [[ "$BRANCH" == "$protected" ]]; then
    echo "Cannot commit to protected branch: $BRANCH" >&2
    exit 2  # Exit 2 = block the action
  fi
done
exit 0
```

### Hook Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success, continue |
| 1 | Error, but continue |
| 2 | Block the action |

### Configuring Hooks

In `.claude/settings.json`:
```json
{
  "hooks": {
    "pre-commit": ".claude/hooks/protect-branches.sh"
  }
}
```

---

## Protected Actions

Claude avoids by default:
- `git push --force`
- `git reset --hard`
- `git clean -f`
- Commit to main/master/production

---

## Notes

<!-- Add observations about git workflows -->

