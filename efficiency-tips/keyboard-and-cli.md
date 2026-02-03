# Keyboard Shortcuts and CLI Deep Dive

## Essential Keyboard Shortcuts

### Interruption and Control

| Shortcut | Action | When to Use |
|----------|--------|-------------|
| `Esc` | Stop Claude | Wrong direction, taking too long |
| `Esc Esc` | Open rewind menu | Restore to previous state |
| `Ctrl+C` | Exit session | Done working |

### Mode Control

| Shortcut | Action | When to Use |
|----------|--------|-------------|
| `Shift+Tab` | Cycle modes | Switch Plan/Normal/Auto-Accept |

**Mode cycle:**
```
Normal → Auto-Accept → Plan → Normal
```

### Task Management

| Shortcut | Action | When to Use |
|----------|--------|-------------|
| `Ctrl+B` | Background current task | Long-running command |

### Editing and Input

| Shortcut | Action | Notes |
|----------|--------|-------|
| `Enter` | New line | Multi-line input |
| `Ctrl+J` | Submit prompt | Default submit key |
| `Up/Down` | History navigation | Previous commands |
| `Ctrl+R` | Search history | Find old commands |

### Model and Thinking

| Shortcut | Action | When to Use |
|----------|--------|-------------|
| `Cmd/Meta+P` | Model picker | Switch models quickly |
| `Alt+T` | Toggle extended thinking | Complex vs simple tasks |

### Advanced

| Shortcut | Action | When to Use |
|----------|--------|-------------|
| `Ctrl+G` | Edit plan externally | Modify plan in your editor |
| `Ctrl+O` | Toggle verbose mode | See more/less detail |

---

## Customizing Keybindings

### Configuration File

`~/.claude/keybindings.json`

### Example Configuration

```json
{
  "submit": "ctrl+enter",
  "newline": "enter",
  "toggle_mode": "ctrl+p",
  "background": "ctrl+b",
  "interrupt": "escape"
}
```

### Getting Help

```
/keybindings-help
```

Shows current bindings and customization options.

### Common Customizations

**Use Ctrl+Enter to submit (like chat apps):**
```json
{
  "submit": "ctrl+enter",
  "newline": "enter"
}
```

**Vim-style bindings:**
```json
{
  "submit": "ctrl+m",
  "toggle_mode": "ctrl+["
}
```

---

## CLI Flags

### Session Control

| Flag | Purpose | Example |
|------|---------|---------|
| `-c, --continue` | Resume most recent | `claude -c` |
| `-r, --resume` | Resume by name | `claude -r "auth-work"` |
| `--fork-session` | Branch off | `claude -c --fork-session` |
| `--name` | Name new session | `claude --name "feature-x"` |

### Model Selection

| Flag | Purpose | Example |
|------|---------|---------|
| `--model` | Specify model | `claude --model opus` |

### Permission Modes

| Flag | Purpose | Example |
|------|---------|---------|
| `--permission-mode` | Set mode | `claude --permission-mode plan` |
| `--allowedTools` | Whitelist tools | `claude --allowedTools "Edit,Bash(npm test)"` |

### Budget and Limits

| Flag | Purpose | Example |
|------|---------|---------|
| `--max-budget-usd` | Cost limit | `claude --max-budget-usd 5.00` |
| `--max-turns` | Turn limit | `claude --max-turns 3` |

### Input/Output

| Flag | Purpose | Example |
|------|---------|---------|
| `-p` | Print mode | `claude -p "What is this?"` |
| `--output-format` | Format output | `claude -p "List deps" --output-format json` |

### Directory Access

| Flag | Purpose | Example |
|------|---------|---------|
| `--cwd` | Working directory | `claude --cwd /path/to/project` |
| `--add-dir` | Additional access | `claude --add-dir ../shared-lib` |

---

## Print Mode (-p)

### What Is Print Mode?

One-off query, no interactive session:
```bash
claude -p "What's in package.json?"
```

Runs, prints answer, exits.

### When to Use

- Quick lookups
- Scripting
- CI/CD integration
- Non-interactive queries

### Examples

```bash
# Quick question
claude -p "What testing framework does this use?"

# Structured output
claude -p "List all API endpoints" --output-format json

# Analysis
claude -p "Summarize recent changes" < git_log.txt
```

### Combining with Other Flags

```bash
# Specific model
claude -p "Complex analysis" --model opus

# Budget limit
claude -p "Explore codebase" --max-budget-usd 2.00

# With piped input
cat error.log | claude -p "Find the root cause"
```

---

## Piping and Redirection

### Piping Input

```bash
# Pipe file contents
cat src/auth/login.cs | claude -p "Explain this code"

# Pipe command output
git diff | claude -p "Review these changes"

# Pipe error logs
cat errors.log | claude -p "What's causing these errors?"
```

### Piping Output

```bash
# Save response to file
claude -p "Generate a README" > README.md

# Pipe to another command
claude -p "List test commands" | sh
```

### Combined

```bash
# Read, process, write
cat old.cs | claude -p "Modernize this C# code" > new.cs

# Git integration
git diff HEAD~5 | claude -p "Summarize changes" > CHANGELOG_ENTRY.md
```

---

## Bash Mode

### Inline Commands

```
> ! npm run build
```

The `!` prefix runs command directly.

### Background Execution

```
> ! npm run dev &
```

The `&` runs in background.

### Use Cases

- Quick commands without asking Claude
- Running scripts
- System commands

---

## In-Session Commands

### Navigation and Info

| Command | Purpose |
|---------|---------|
| `/help` | Show all commands |
| `/status` | Session info |
| `/context` | Context usage |
| `/cost` | Token costs |
| `/memory` | CLAUDE.md files loaded |

### Session Management

| Command | Purpose |
|---------|---------|
| `/clear` | Clear conversation |
| `/compact` | Summarize conversation |
| `/compact [focus]` | Targeted summary |
| `/rename "name"` | Name session |
| `/rewind` | Restore checkpoint |

### Model and Mode

| Command | Purpose |
|---------|---------|
| `/model opus` | Switch model |
| `/model sonnet` | Switch model |
| `/model haiku` | Switch model |

### Tools and Config

| Command | Purpose |
|---------|---------|
| `/mcp` | MCP server management |
| `/config` | Open settings |
| `/doctor` | Diagnose issues |
| `/init` | Generate CLAUDE.md |

### Tasks

| Command | Purpose |
|---------|---------|
| `/tasks` | List background tasks |

---

## Scripting with Claude

### Basic Script

```bash
#!/bin/bash
claude -p "Generate a .gitignore for a .NET project" > .gitignore
```

### Processing Multiple Files

```bash
#!/bin/bash
for file in src/*.cs; do
  claude -p "Add XML documentation to $file" --allowedTools "Edit"
done
```

### CI/CD Integration

```yaml
# GitHub Actions example
- name: Code Review
  run: |
    git diff HEAD~1 | claude -p "Review for issues" > review.md
```

### Error Handling

```bash
#!/bin/bash
result=$(claude -p "Analyze this code" < code.cs)
if [ $? -ne 0 ]; then
  echo "Claude failed"
  exit 1
fi
echo "$result"
```

---

## Common Workflows

### Quick Code Review

```bash
git diff | claude -p "Review for bugs and style issues"
```

### Generate Documentation

```bash
claude -p "Generate API documentation" --model sonnet > API.md
```

### Explore Then Work

```bash
# Start read-only
claude --permission-mode plan

# Inside session, when ready:
# Shift+Tab to switch to Normal mode
```

### Continue Previous Work

```bash
# List sessions
claude sessions list

# Resume specific session
claude -r "auth-refactor"

# Or just continue most recent
claude -c
```

### Multi-Directory Project

```bash
claude --add-dir ../shared-lib --add-dir ../common-types
```

---

## Environment Variables

### Model Defaults

```bash
export ANTHROPIC_MODEL=sonnet
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-5-20251101
```

### API Configuration

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

### Thinking Budget

```bash
export MAX_THINKING_TOKENS=10000
```

### Setting Permanently

Add to `~/.bashrc` or `~/.zshrc`:
```bash
export ANTHROPIC_MODEL=sonnet
export MAX_THINKING_TOKENS=10000
```

---

## Tips and Tricks

### Quick Model Switch

```
Cmd+P → select model
```

Faster than `/model`.

### Abort Gracefully

`Esc` stops Claude but keeps context.
Use when:
- Wrong direction
- Taking too long
- Want to clarify

### Background Long Commands

Start command, then `Ctrl+B` to background it.
Check later with `/tasks`.

### History Search

`Ctrl+R` then type to search previous commands.

### Multi-Line Input

Just press `Enter` for new lines.
`Ctrl+J` (or your submit key) to send.

---

## Notes

<!-- Add observations about keyboard shortcuts and CLI -->

