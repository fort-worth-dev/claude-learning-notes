# Session Persistence Deep Dive

## How Sessions Work

### What Is a Session?

A session is a saved conversation with Claude, tied to a specific directory. It includes:
- All messages (yours and Claude's)
- Tool calls and their outputs
- File snapshots before edits
- Session metadata (name, timestamps, model used)

### Session Storage

Sessions are stored locally in `~/.claude/sessions/` (or similar platform-specific location).

Each session contains:
- Conversation history
- Checkpoints for file states
- Token usage data

### Directory Binding

Sessions are associated with the directory where you started them:

```bash
cd ~/projects/api
claude                    # Session bound to ~/projects/api

cd ~/projects/frontend
claude                    # Different session, bound to ~/projects/frontend
```

This means:
- `claude -c` in `/api` resumes the api session
- `claude -c` in `/frontend` resumes the frontend session

---

## Session Lifecycle

### Creation

A session starts when you run `claude` (without `-c` or `-r`):

```bash
claude                    # New session
claude "fix the login bug" # New session with initial prompt
```

### Active Use

During a session:
- Every exchange is saved automatically
- File changes create restore points
- No manual "save" needed

### Ending

Sessions end when you:
- Type `exit`
- Press `Ctrl+C` twice
- Close the terminal

The session persists on disk - ending just closes the interactive connection.

### Resumption

Resume anytime:

```bash
claude -c                 # Most recent session in this directory
claude -r "session-name"  # Specific named session
```

---

## Naming Sessions

### Why Name Sessions?

Unnamed sessions are identified by timestamp/ID. Hard to find later:
```
session-2026-02-01-14-32-abc123
session-2026-02-01-16-45-def456
```

Named sessions are findable:
```
oauth-refactor
bug-fix-checkout
api-v2-migration
```

### How to Name

During session:
```
/rename oauth-refactor
```

When starting:
```bash
claude --name "oauth-refactor"
```

### Naming Conventions

Use descriptive, searchable names:
- Feature: `feature-user-preferences`
- Bug fix: `fix-null-ref-orders`
- Exploration: `explore-auth-flow`
- Refactor: `refactor-data-layer`

---

## Listing and Finding Sessions

### List Recent Sessions

```bash
claude sessions list
```

Shows recent sessions with:
- Session ID/name
- Directory
- Last active timestamp
- Message count

### List All Sessions

```bash
claude sessions list --all
```

### Filter by Directory

Sessions are naturally filtered when you're in a directory:
```bash
cd ~/projects/api
claude sessions list      # Only shows api project sessions
```

---

## Resuming Sessions

### Continue Most Recent

```bash
claude -c
claude --continue
```

Resumes the most recent session for the current directory.

### Resume by Name

```bash
claude -r "oauth-refactor"
claude --resume "oauth-refactor"
```

### Resume by ID

```bash
claude -r abc123
```

Use the ID from `claude sessions list` if you didn't name the session.

### What Gets Restored?

When resuming:
- Full conversation history loads
- Claude has context of previous work
- You can continue where you left off

What doesn't restore:
- Terminal state (if you ran bash commands)
- External system state (database changes, etc.)
- Files changed outside Claude since session ended

---

## Forking Sessions

### What Is Forking?

Forking creates a copy of a session at a point in time. The original session is unchanged.

```bash
claude --continue --fork-session
```

### When to Fork

**Experimentation:**
```bash
# You've got a working state, want to try something risky
claude -c --fork-session
# Try the risky approach
# If it fails, original session is untouched
```

**Alternative approaches:**
```bash
# Fork to try approach A
claude -c --fork-session --name "approach-a"
# Try approach A...

# Fork again from original to try approach B
claude -r "original-session" --fork-session --name "approach-b"
# Try approach B...

# Compare results, pick the winner
```

**Preserving milestones:**
```bash
# Reached a good state, want to preserve it
claude -c --fork-session --name "milestone-auth-complete"
# Continue working in current session
# Can always go back to milestone if needed
```

### Fork vs Clear

| Action | Use When |
|--------|----------|
| Fork | Want to preserve current state and explore alternatives |
| Clear | Done with current context, starting fresh task |

---

## File Snapshots and Undo

### Automatic Snapshots

Before Claude edits a file, it saves a snapshot. This enables:

```
/undo
```

Reverts the last file change Claude made.

### Limitations

- Only covers Claude's changes, not your manual edits
- Snapshot history is per-session
- Deep undo (multiple steps back) may be limited

### When /undo Isn't Enough

For more control, use git:

```bash
git diff                  # See what changed
git checkout -- file.cs   # Revert specific file
git stash                 # Stash all changes
```

---

## Session Memory vs CLAUDE.md

### Key Distinction

| Aspect | Session Memory | CLAUDE.md |
|--------|---------------|-----------|
| Scope | Single conversation | All sessions in project |
| Persistence | Until session deleted | Permanent (file on disk) |
| Content | Dynamic conversation | Static instructions |
| Survives `/clear` | No | Yes |

### What Goes Where

**Put in CLAUDE.md:**
- Build commands
- Code conventions
- Architectural rules
- Things you'll need across sessions

**Keep in session:**
- Current task context
- Exploratory discussion
- Temporary decisions
- Work in progress

### The Handoff Pattern

When ending a session with unfinished work:

1. Ask Claude to summarize current state
2. Add summary to CLAUDE.md or a notes file
3. Next session, reference the notes

```
"Summarize what we've accomplished and what's left to do"
```

Save the output, then next session:
```
"Continue the auth refactor. See notes in docs/wip-auth.md"
```

---

## Auto-Compaction and Session Memory

### What Happens Over Time

Long sessions trigger auto-compaction:
1. Older tool outputs cleared first
2. Earlier conversation summarized
3. Recent context preserved
4. Key decisions retained (usually)

### Implications for Persistence

- Very long sessions may lose early details
- Important instructions should be in CLAUDE.md
- Consider `/compact` manually with focus area
- Or `/clear` and restart for fresh context

### Signs Your Session Needs Attention

- Claude "forgets" earlier decisions
- Repeating yourself frequently
- Claude re-reads files unnecessarily
- `/context` shows high usage

**Solutions:**
- `/compact focus on [current task]`
- `/clear` and restart with clear prompt
- Start fresh session for new task

---

## Session Hygiene

### Good Practices

1. **Name important sessions** - You'll want to find them later
2. **Clear between unrelated tasks** - Don't let context bleed
3. **Fork before experiments** - Preserve known-good states
4. **Use CLAUDE.md for persistent rules** - Don't rely on session memory
5. **Summarize before long breaks** - Capture state for later

### When to Start Fresh

- New feature or bug (unrelated to current work)
- Session feels "confused"
- Context is full of irrelevant history
- After completing a major task

### Cleaning Up Old Sessions

Sessions accumulate over time. Periodically:
```bash
claude sessions list --all
# Review and delete old sessions you don't need
```

---

## Practical Workflows

### Multi-Day Feature Work

```bash
# Day 1
claude --name "feature-user-prefs"
# Work on feature...
# End of day, summarize progress
/rename "feature-user-prefs-day1"

# Day 2
claude -r "feature-user-prefs-day1"
# Or fork if you want to preserve day 1 state:
claude -r "feature-user-prefs-day1" --fork-session --name "feature-user-prefs-day2"
```

### Bug Investigation

```bash
claude --permission-mode plan --name "investigate-bug-123"
# Explore, understand the issue
# Found the cause, ready to fix:
# Shift+Tab to switch to Act mode
# Fix the bug
```

### Exploratory Coding

```bash
claude --name "explore-new-pattern"
# Try things out...
# Found something promising:
claude -c --fork-session --name "explore-new-pattern-promising"
# Continue refining
```

---

## Notes

<!-- Add observations about session management as you learn -->

