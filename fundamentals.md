# Claude Code Fundamentals

This is a quick reference. For detailed notes, see the [fundamentals/](fundamentals/) subfolder:

- [CLI Commands](fundamentals/cli-commands.md) - Flags, session management, keyboard shortcuts
- [Context Management](fundamentals/context-management.md) - CLAUDE.md hierarchy, limits, strategies
- [Session Persistence](fundamentals/session-persistence.md) - Resume, fork, snapshots, handoff patterns
- [Configuration](fundamentals/configuration.md) - Settings files, permissions, environment variables
- [Core Workflow](fundamentals/core-workflow.md) - Explore → Plan → Implement → Verify

---

## Basic Commands and Workflow

### Essential CLI Commands

| Command | Purpose |
|---------|---------|
| `claude` | Start interactive session in current directory |
| `claude "query"` | Start with initial prompt |
| `claude -p "query"` | One-off query, exit immediately |
| `claude -c` | Continue most recent conversation |
| `claude -r "session-name"` | Resume specific session by name/ID |
| `claude --model opus` | Specify model for session |

### Session Lifecycle

- Navigate to project directory, run `claude`
- Type `/help` for available commands
- End with `exit` or `Ctrl+C`
- Resume later with `claude -c` or `claude -r`

### Important Flags

- `--permission-mode plan` - Start in read-only plan mode
- `--add-dir ../other-project` - Grant access to additional directories
- `--model sonnet` - Use Sonnet model (default)

### In-Session Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show all commands |
| `/context` | Visualize context usage |
| `/cost` | Show token usage |
| `/clear` | Clear conversation history |
| `/compact` | Summarize to free context |
| `/model opus` | Switch model mid-session |
| `/rename "name"` | Name your session |
| `/memory` | Show loaded CLAUDE.md files |

---

## Context Management

### CLAUDE.md Hierarchy (highest priority first)

1. **Managed policy** - Organization-wide (IT-deployed)
2. **Project CLAUDE.md** - `./CLAUDE.md` or `./.claude/CLAUDE.md`
3. **User CLAUDE.md** - `~/.claude/CLAUDE.md` (personal, all projects)
4. **Project local** - `./CLAUDE.local.md` (gitignored)
5. **Project rules** - `./.claude/rules/*.md` (modular)

### What to Include in CLAUDE.md

**Good:**
- Build/test/run commands
- Code style preferences that differ from defaults
- Repository conventions (branching, PR format)
- Architectural decisions
- Common gotchas

**Avoid:**
- Things Claude can figure out from code
- Standard language conventions
- Lengthy documentation
- Information that changes frequently

### Bootstrap a CLAUDE.md

```bash
claude /init
```

### Context Window Limits

- Standard: 200k tokens
- Extended (Sonnet 1m): 1 million tokens
- Performance degrades as context fills - manage aggressively

---

## Session Persistence and Memory

### How Sessions Work

- Tied to current directory
- Saves all messages, tool use, outputs
- Creates file snapshots before edits (for rewinding)
- Persists until deleted

### Key Point: Sessions Are Ephemeral

Claude doesn't remember previous sessions. Put persistent knowledge in CLAUDE.md, not conversation history.

### Resume vs Fork

```bash
claude --continue              # Resume most recent
claude --resume oauth-refactor # Resume by name
claude --continue --fork-session  # Branch off without affecting original
```

### Auto-Compaction

As context fills, Claude Code automatically:
1. Clears older tool outputs
2. Summarizes conversation
3. Preserves: requests and key code snippets
4. May lose: detailed early instructions

**Manual compaction:**
```
/compact focus on the API changes
```

---

## Configuration and Settings

### Settings File Locations

| Scope | Location | Shared? |
|-------|----------|---------|
| **User** | `~/.claude/settings.json` | No |
| **Project** | `.claude/settings.json` | Yes (git) |
| **Local** | `.claude/settings.local.json` | No |

### Example settings.json

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet test)",
      "Bash(dotnet build)"
    ],
    "deny": [
      "Read(./.env*)"
    ]
  },
  "model": "sonnet"
}
```

### Environment Variables

```bash
export ANTHROPIC_MODEL=sonnet
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-5-20251101
```

---

## Best Practices Summary

### Do's

- Put persistent knowledge in CLAUDE.md
- Provide verification (tests, expected output)
- Use `/clear` between unrelated tasks
- Reference specific files
- Use Plan Mode for complex refactors
- Interrupt and correct Claude early

### Don'ts

- Don't bloat CLAUDE.md
- Don't correct more than twice (clear and restart)
- Don't rely on conversation history for persistent rules

### The Core Workflow

1. **Explore** - `claude --permission-mode plan` to understand
2. **Plan** - Create implementation plan
3. **Implement** - `Shift+Tab` to switch modes, code with verification
4. **Commit** - When explicitly requested

---

## Notes

<!-- Personal observations as you use Claude Code -->
