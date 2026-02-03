# Efficiency Tips

This is a quick reference. For detailed notes, see the [efficiency-tips/](efficiency-tips/) subfolder:

- [Token and Context](efficiency-tips/token-and-context.md) - Managing context, compaction, cost optimization
- [Parallel Operations](efficiency-tips/parallel-operations.md) - Multiple sessions, worktrees, subagents
- [Model Selection](efficiency-tips/model-selection.md) - When to use each model, switching strategies
- [Keyboard and CLI](efficiency-tips/keyboard-and-cli.md) - Shortcuts, flags, piping, scripting

---

## Reducing Token Usage

### Monitor Context

```
/context    # See what's consuming space
/mcp        # Check MCP server overhead
```

### Key Strategies

| Strategy | How |
|----------|-----|
| Clear between tasks | `/clear` before unrelated work |
| Keep CLAUDE.md concise | Under 500 lines, prune ruthlessly |
| Move to skills | Specialized workflows load on-demand |
| Use subagents | Exploration stays isolated |
| Disable unused MCP | `/mcp` to disable idle servers |

### /clear vs /compact

- **`/clear`**: Best between unrelated tasks. Resets everything.
- **`/compact <focus>`**: Long sessions on related work. Preserves important context.

```
/compact Focus on the API changes
```

### Reduce MCP Overhead

Prefer CLI tools over MCP servers when possible:
- `gh` instead of GitHub MCP
- `aws` instead of AWS MCP
- `sentry-cli` instead of Sentry MCP

---

## Parallel Operations

### Multiple Claude Sessions

**Git worktrees for isolation:**
```bash
git worktree add ../project-feature-a -b feature-a
cd ../project-feature-a
claude

# Another terminal
git worktree add ../project-bugfix -b bugfix-123
cd ../project-bugfix
claude
```

### Background Tasks

```
Ctrl+B     # Background current task
/tasks     # Check background progress
```

### Subagents for Parallel Research

```
Use subagents to investigate authentication and database
modules in parallel. Report findings.
```

### Fan-Out for Large Migrations

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue" \
    --allowedTools "Edit,Bash(git commit *)" &
done
wait
```

---

## When to Start Fresh vs. Continue

### Continue When:

- Working on same task
- Context still relevant
- Just stepped away briefly

### Start Fresh When:

- Switching to unrelated work
- Corrected Claude 2+ times on same issue
- Context feels cluttered with failed attempts

### Session Commands

```bash
claude --continue              # Resume most recent
claude --resume auth-refactor  # Resume by name
claude --continue --fork-session  # Branch off
```

### Signs of Context Pollution

- Claude suggests same wrong approach repeatedly
- Questions answered earlier get asked again
- Large output from unrelated exploration

**Prevention:** Use `/rename` to name sessions by task.

---

## Model Selection

### When to Use Each

| Model | Use For | Cost |
|-------|---------|------|
| **Sonnet** (default) | 90% of tasks | Fast, cheap |
| **Opus** | Complex architecture, planning | Slow, expensive |
| **Haiku** | Simple tasks, subagents | Very fast, very cheap |
| **opusplan** | Opus plans, Sonnet executes | Balanced |

### Switching Models

```bash
claude --model opus    # At startup
/model sonnet          # Mid-session
/model haiku           # For quick tasks
```

### Environment Defaults

```bash
export ANTHROPIC_MODEL=sonnet
```

---

## Permission Management

### Pre-Approve Safe Commands

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet test)",
      "Bash(dotnet build)",
      "Bash(git commit *)",
      "Read",
      "Edit"
    ]
  }
}
```

### Permission Modes

| Mode | Behavior |
|------|----------|
| Default | Prompts for side effects |
| Auto-Accept | Allows edits, prompts Bash |
| Plan Mode | Read-only |

**Cycle with:** `Shift+Tab`

### CLI Flags

```bash
claude --permission-mode auto-accept
claude --allowedTools "Edit,Bash(npm test)"
```

---

## Keyboard Shortcuts

### Essential

| Shortcut | Action |
|----------|--------|
| `Esc` | Stop Claude (context preserved) |
| `Esc Esc` | Open rewind menu |
| `Shift+Tab` | Cycle permission modes |
| `Ctrl+B` | Background task |
| `Ctrl+G` | Edit plan in external editor |
| `Ctrl+O` | Toggle verbose mode |
| `Ctrl+R` | Search command history |

### Model & Thinking

| Shortcut | Action |
|----------|--------|
| `Cmd/Meta+P` | Open model picker |
| `Alt+T` | Toggle extended thinking |

---

## CLI Tricks

### Common Flags

```bash
# Continue with query
claude --continue -p "Run tests"

# Print mode (non-interactive)
claude -p "Summarize project"

# Structured output
claude -p "List endpoints" --output-format json

# Specific model
claude --model opus

# Budget limit
claude -p "Explore" --max-budget-usd 5.00

# Max turns
claude -p "Fix this" --max-turns 3
```

### Bash Mode

```
> ! npm run build    # Run in background
```

### Piping

```bash
cat errors.log | claude -p "Find root cause"
cat build-error.txt | claude -p "Fix this" > fix.md
```

---

## Context Management

### Aggressive Pruning

Check `/context`:
- Conversation >50%: Use `/clear` between tasks
- CLAUDE.md >500 lines: Prune or move to skills
- MCP tools >15%: Disable unused servers

### Compaction Instructions

Add to CLAUDE.md:
```markdown
# Compact instructions
When compacting, preserve:
- All modified file paths
- Test output and results
- API changes
```

### Extended Thinking

```bash
# Limit token budget
export MAX_THINKING_TOKENS=10000

# Toggle per-session
Alt+T
```

Use for complex decisions. Disable for simple tasks.

---

## Workflow Efficiency Patterns

### Bug Fix (5-10 min)

1. Paste error
2. Run tests to see failures
3. Search codebase for error
4. Fix and verify

### Feature Implementation

1. **Plan Mode**: Understand architecture
2. **Normal Mode**: Implement with tests
3. **Subagent**: Review in parallel
4. **PR**: Create and submit

### Large Refactor (Multi-Session)

**Session A (Writer):** Implements following plan
**Session B (Reviewer):** Reviews without bias
**Session A:** Incorporates feedback

---

## Quick Checklist

Before starting:
- [ ] `/clear` if switching tasks
- [ ] `Shift+Tab` twice for Plan Mode (complex changes)
- [ ] Start with Sonnet
- [ ] Use subagents for research
- [ ] Check `/context` periodically
- [ ] `/rename` sessions by task
- [ ] Include verification in prompts

---

## Notes

<!-- Personal observations as you use Claude Code -->
