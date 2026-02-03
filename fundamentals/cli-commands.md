# CLI Commands Deep Dive

## Starting Claude Code

### Basic Invocation

```bash
claude                    # Interactive session in current directory
claude "your prompt"      # Start with initial prompt
claude -p "query"         # Print mode - one-off query, no session
```

### Model Selection

```bash
claude --model sonnet     # Use Sonnet (default, fastest)
claude --model opus       # Use Opus (most capable)
claude --model haiku      # Use Haiku (lightweight tasks)
```

**When to use each:**
- **Sonnet** - Day-to-day coding, good balance of speed/capability
- **Opus** - Complex architecture decisions, nuanced refactoring
- **Haiku** - Quick lookups, simple questions, cost-conscious

### Permission Modes

```bash
claude --permission-mode default    # Ask before risky operations
claude --permission-mode plan       # Read-only, no writes/executes
claude --permission-mode full-auto  # Trust all operations (use carefully)
```

**Plan mode is ideal for:**
- Exploring unfamiliar codebases
- Understanding before changing
- Reviewing Claude's approach before giving write access

---

## Session Management

### Resuming Sessions

```bash
claude -c                      # Continue most recent session
claude --continue              # Same as -c

claude -r "session-name"       # Resume by name or ID
claude --resume "session-name" # Same as -r
```

### Naming Sessions

Inside a session:
```
/rename oauth-refactor
```

Or when starting:
```bash
claude --name "feature-xyz"
```

Named sessions are easier to find and resume later.

### Forking Sessions

```bash
claude --continue --fork-session
```

Creates a branch of the conversation. Original session untouched. Useful for:
- Trying alternative approaches
- "What if" exploration
- Preserving a known-good state

### Listing Sessions

```bash
claude sessions list           # Show recent sessions
claude sessions list --all     # Show all sessions
```

---

## Directory and Context Flags

### Working with Multiple Directories

```bash
claude --add-dir ../shared-lib
claude --add-dir ~/projects/common
```

Grants read/write access to directories outside the current folder. Useful for:
- Monorepos with shared code
- Referencing another project's patterns
- Cross-project refactoring

### Specifying Working Directory

```bash
claude --cwd /path/to/project
```

Starts Claude in a different directory without `cd`-ing first.

---

## Input/Output Modes

### Piping Input

```bash
cat error.log | claude "explain this error"
git diff | claude "review these changes"
```

Pipes content directly into the prompt context.

### Print Mode (-p)

```bash
claude -p "what version of node is recommended for this project?"
```

- Runs query, prints response, exits
- No interactive session created
- Good for scripting and quick lookups

### Output Formats

```bash
claude -p "list dependencies" --output-format json
claude -p "explain this" --output-format text
claude -p "analyze" --output-format stream-json
```

Useful for automation and CI/CD integration.

---

## In-Session Commands

### Navigation and Info

| Command | Purpose |
|---------|---------|
| `/help` | Show all available commands |
| `/status` | Current session info |
| `/context` | Visualize context window usage |
| `/cost` | Show token usage and costs |
| `/memory` | Show loaded CLAUDE.md files |

### Context Management

| Command | Purpose |
|---------|---------|
| `/clear` | Clear conversation, start fresh |
| `/compact` | Summarize conversation to free space |
| `/compact [focus]` | Summarize with specific focus area |

### Session Control

| Command | Purpose |
|---------|---------|
| `/model opus` | Switch model mid-session |
| `/rename "name"` | Name current session |
| `/undo` | Revert last file change |

### Special Commands

| Command | Purpose |
|---------|---------|
| `/init` | Bootstrap CLAUDE.md for current project |
| `/doctor` | Diagnose configuration issues |
| `/config` | Open settings |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Add newline |
| `Ctrl+J` | Submit prompt (default) |
| `Escape` | Cancel current operation |
| `Shift+Tab` | Toggle Plan/Act mode |
| `Ctrl+C` | Interrupt Claude or exit |
| `Up/Down` | Navigate prompt history |

Customize shortcuts in `~/.claude/keybindings.json`.

---

## Environment Variables

```bash
# Default model
export ANTHROPIC_MODEL=sonnet

# Specific model versions
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-5-20251101
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-20250514

# API configuration
export ANTHROPIC_API_KEY=sk-ant-...
```

---

## Practical Examples

### Quick Code Review

```bash
git diff HEAD~3 | claude -p "review for bugs and style issues"
```

### Explore Before Changing

```bash
claude --permission-mode plan "explain the authentication flow"
# Then Shift+Tab to switch to Act mode when ready to code
```

### Continue Yesterday's Work

```bash
claude sessions list
claude -r "auth-refactor"
```

### Multi-Project Task

```bash
claude --add-dir ../api-contracts "update client to match new API schema"
```

---

## Notes

<!-- Add observations as you use these commands -->

