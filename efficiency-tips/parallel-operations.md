# Parallel Operations Deep Dive

## Why Parallelize?

### Serial vs Parallel

**Serial (slow):**
```
Task A (5 min) → Task B (5 min) → Task C (5 min) = 15 min total
```

**Parallel (fast):**
```
Task A (5 min) ─┐
Task B (5 min) ─┼→ = 5 min total
Task C (5 min) ─┘
```

### When to Parallelize

- Independent tasks (no dependencies)
- Multiple investigations
- Large-scale migrations
- Build + test cycles

### When NOT to Parallelize

- Tasks depend on each other
- Modifying same files
- Shared resources (database, etc.)
- Sequential logic required

---

## Multiple Claude Sessions

### Basic Multi-Session

**Terminal 1:**
```bash
cd project
claude --name "feature-auth"
# Work on authentication
```

**Terminal 2:**
```bash
cd project
claude --name "feature-orders"
# Work on orders (different files)
```

### Problem: File Conflicts

Two sessions editing the same file = trouble.

**Solutions:**
1. Work on different files
2. Use git worktrees
3. Coordinate manually

---

## Git Worktrees

### What Are Worktrees?

Multiple working directories from same repo:
```
project/           # main branch
project-feature-a/ # feature-a branch
project-bugfix/    # bugfix-123 branch
```

Each is a full checkout. Each can have its own Claude session.

### Creating Worktrees

```bash
# From main project directory
git worktree add ../project-feature-a -b feature-a
git worktree add ../project-bugfix -b bugfix-123
```

### Using with Claude

**Terminal 1:**
```bash
cd ../project-feature-a
claude --name "feature-a"
# Full isolation, can edit any file
```

**Terminal 2:**
```bash
cd ../project-bugfix
claude --name "bugfix-123"
# Completely independent
```

### Worktree Workflow

```bash
# 1. Create worktree for feature
git worktree add ../project-feature -b feature-xyz

# 2. Work with Claude in that worktree
cd ../project-feature
claude

# 3. When done, merge back
git checkout main
git merge feature-xyz

# 4. Clean up worktree
git worktree remove ../project-feature
```

### Listing Worktrees

```bash
git worktree list
```

### Removing Worktrees

```bash
git worktree remove ../project-feature-a
```

---

## Background Tasks

### Starting Background Tasks

**Method 1: Ask Claude**
```
Run the test suite in the background
```

**Method 2: Keyboard shortcut**
While a command is running:
```
Ctrl+B
```

**Method 3: Bash mode**
```
> ! npm run build &
```

### Checking Background Tasks

```
/tasks
```

Lists all background tasks with status.

### Getting Results

```
Check the test results
What was the output of the build?
```

### Multiple Background Tasks

```
Run these in the background:
1. dotnet test
2. npm run lint
3. docker build .
```

All three run simultaneously.

### Good Background Candidates

| Task | Background? |
|------|-------------|
| Full test suite | Yes |
| Production build | Yes |
| Dev server | Yes |
| Docker builds | Yes |
| Quick compile | No |
| Single unit test | No |

---

## Parallel Subagents

### Single Request, Multiple Agents

```
Use subagents in parallel to investigate:
1. The authentication module
2. The database access layer
3. The API endpoint handlers

Report findings from all three.
```

Claude spawns three agents simultaneously.

### When to Use Parallel Agents

- Independent areas to research
- Broad codebase exploration
- Multiple questions to answer
- Gathering context quickly

### Combining Results

```
Use parallel subagents to:
1. Find all API endpoints
2. Find all database models
3. Find all validation rules

Then summarize how they relate to each other.
```

---

## Fan-Out Pattern

### For Large-Scale Changes

When you need to change many files the same way:

**1. Generate file list:**
```
List all Python files that use the old logging API.
Save to migration-files.txt
```

**2. Script the migration:**
```bash
#!/bin/bash
for file in $(cat migration-files.txt); do
  claude -p "Migrate $file from old logging to new logging API" \
    --allowedTools "Edit,Bash(python -m py_compile *)" &
done
wait
```

**3. Verify:**
```bash
python -m pytest
```

### Fan-Out Best Practices

**Start small:**
```bash
# Test on 3 files first
head -3 migration-files.txt | while read file; do
  claude -p "Migrate $file" --allowedTools "Edit"
done

# Verify those work, then scale up
```

**Include verification:**
```bash
claude -p "Migrate $file AND verify it compiles" \
  --allowedTools "Edit,Bash(dotnet build)"
```

**Use budget limits:**
```bash
claude -p "Migrate $file" --max-budget-usd 1.00
```

### Fan-Out Patterns

**Independent files:**
```bash
for file in *.cs; do
  claude -p "Add null checks to $file" &
done
wait
```

**With dependencies:**
```bash
# Process in order, not parallel
for file in $(cat ordered-files.txt); do
  claude -p "Migrate $file" --wait
done
```

---

## Writer/Reviewer Pattern

### Two-Session Code Review

**Session A (Writer):**
```bash
claude --name "feature-writer"
```
```
Implement the rate limiting feature
```

**Session B (Reviewer):**
```bash
claude --name "feature-reviewer"
```
```
Review src/middleware/rateLimiter.ts for:
- Edge cases
- Race conditions
- Security issues
```

### Handoff

Copy reviewer's findings to writer:
```
Address this feedback:
1. Line 45 has a race condition
2. Missing timeout handling
3. Tests don't cover concurrent requests
```

### Why Two Sessions?

- Fresh perspective (no implementation bias)
- Parallel work (review while writing more)
- Clear separation of concerns

---

## Coordination Strategies

### File-Based Coordination

Divide by files:
- Session A: src/auth/*
- Session B: src/orders/*
- Session C: src/users/*

No conflicts if files don't overlap.

### Branch-Based Coordination

Use git worktrees:
- Worktree A: feature-auth branch
- Worktree B: feature-orders branch

Merge when both complete.

### Task-Based Coordination

Split by task type:
- Session A: Implementation
- Session B: Testing
- Session C: Documentation

Implementation finishes → Testing picks up → Documentation follows

---

## Monitoring Parallel Work

### Check All Sessions

Switch terminals to check each session.

### Background Task Status

```
/tasks
```

In any session, shows all background tasks.

### Git Status

```bash
git status
git diff
```

See what changed across all work.

---

## Common Patterns

### Build and Test Parallel

```
In the background:
1. Run the build
2. Start the dev server

While those run, let me know when ready to test
```

### Parallel Investigation

```
I need to understand three systems. Use subagents to investigate in parallel:
1. How authentication works
2. How orders are processed
3. How notifications are sent

Give me a summary when done.
```

### Multi-Branch Development

```bash
# Create worktrees
git worktree add ../proj-api -b feature-api
git worktree add ../proj-ui -b feature-ui

# Work in parallel (different terminals)
cd ../proj-api && claude --name "api-work"
cd ../proj-ui && claude --name "ui-work"

# Later, merge both
git checkout main
git merge feature-api
git merge feature-ui
```

---

## Notes

<!-- Add observations about parallel operations -->

