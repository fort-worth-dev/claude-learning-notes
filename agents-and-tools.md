# Agents and Tools

This is a quick reference. For detailed notes, see the [agents-and-tools/](agents-and-tools/) subfolder:

- [Subagents](agents-and-tools/subagents.md) - Explore, Plan, General-purpose, custom agents
- [Skills](agents-and-tools/skills.md) - Custom commands, workflows, slash commands
- [Core Tools](agents-and-tools/core-tools.md) - File operations, search, execution
- [Background Tasks](agents-and-tools/background-tasks.md) - Async operations, monitoring

---

## Subagents Overview

### What Are Subagents?

Specialized AI assistants that handle specific tasks in **isolated context windows**. Each subagent has:
- Custom system prompt
- Specific tool access restrictions
- Independent conversation history
- Separate context (doesn't bloat main conversation)

### Built-in Subagents

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| **Explore** | Haiku (fast) | Read-only | Codebase exploration, file discovery |
| **Plan** | Inherited | Read-only | Research during plan mode |
| **General-purpose** | Inherited | All tools | Complex multi-step tasks |
| **Bash** | Inherited | Bash only | Terminal operations in separate context |

### Subagent Locations (Priority Order)

1. **CLI flag** (`--agents`): Current session only
2. **Project** (`.claude/agents/`): This project (commit to git)
3. **User** (`~/.claude/agents/`): All your projects

### How Context Isolation Works

When Claude delegates to a subagent:
- Fresh context window created
- Subagent does NOT see parent conversation
- Large explorations stay isolated
- Results summarized when returning

---

## When to Use Which Agent

### Decision Matrix

| Task Type | Best Agent | Why |
|-----------|-----------|-----|
| Codebase exploration | Explore | Fast, read-only, isolated context |
| Planning changes | Plan | Safe research before implementation |
| Complex multi-step | General-purpose | All tools, both read and write |
| High-volume output | Custom subagent | Keeps verbose output separate |

### Explore Agent

**Use when:**
- Investigating unfamiliar codebase
- Searching for where a feature is implemented
- Understanding architecture before planning
- Need results without context bloat

**Thoroughness levels:** quick, medium, very thorough

```
Use Explore to research the authentication module
```

### Plan Agent

**Use when:**
- Multi-file changes required
- Unfamiliar with code being modified
- Want to verify approach before implementation
- Making breaking changes

**Workflow:**
1. Enter Plan Mode: `Shift+Tab` twice
2. Claude researches with Plan agent (read-only)
3. Claude creates detailed plan
4. `Ctrl+G` to edit plan in editor
5. `Shift+Tab` to switch to Normal Mode
6. Claude implements from verified plan

### General-Purpose Agent

**Use when:**
- Complex research requiring read AND write
- Multi-step workflows with dependencies
- Task needs full tool access within isolation
- Subagent must make decisions based on findings

```
Use general-purpose to investigate the performance issue
and implement a fix. Run tests when done.
```

---

## Automatic vs Manual Delegation

### How Automatic Delegation Works

Claude looks at your request and decides whether to delegate:

| Your Request | Agent Used | Why |
|--------------|------------|-----|
| "Understand how auth works" | Explore | Broad research |
| "Find where we handle payments" | Explore | Multi-file search |
| "What's the architecture?" | Explore | Codebase exploration |
| "Plan adding OAuth support" | Plan | Implementation planning |
| "Research this, then implement" | General-purpose | Needs read + write |
| "Read src/auth.ts" | None (main session) | Specific file, no exploration |

### Trigger Words for Automatic Delegation

Claude typically delegates when you use:
- **"understand"** - triggers exploration
- **"find"** / **"search"** - triggers multi-file search
- **"investigate"** / **"research"** - triggers deep dive
- **"explore"** - explicit exploration request
- **"review"** - triggers code analysis

### Stays in Main Session

Claude handles these directly without subagents:
- Reading a specific file you named
- Small, focused edits
- Direct questions it can answer immediately
- Tasks needing back-and-forth iteration with you

### Overriding the Default

**Force delegation:**
```
Use a subagent to research this
Use Explore to find all error handlers
```

**Prevent delegation:**
```
Just read src/auth.ts directly, don't use a subagent
```

---

## Research Subagent Patterns

### Why Use Subagents for Research

```
Without subagent:  Read 50 files → all in main context → bloated
With subagent:     Explore reads 50 files → returns summary → main stays lean
```

### Typical Research Flow

```
Main Session (you're here)
    │
    ├── "Use Explore to understand the auth system"
    │       └── Explore spawns, reads files, returns summary
    │
    ├── You continue with focused context
    │
    ├── "Use a subagent to check error handling"
    │       └── Another agent investigates, returns findings
    │
    └── You implement based on research
```

### Research Prompt Examples

**Quick lookup:**
```
Use Explore to quickly find where auth tokens are validated
```

**Medium investigation:**
```
Use Explore to do a medium investigation of the payment flow
```

**Deep dive:**
```
Use Explore to very thoroughly analyze the entire authentication system
```

**Parallel research:**
```
Use subagents to investigate the authentication module and the
database layer in parallel. Report findings from both.
```

**Pre-planning research:**
```
Use Explore to research how we currently handle file uploads
before I plan adding S3 support
```

### When to Use Research Subagents

| Use Subagent | Use Main Session |
|--------------|------------------|
| Exploring unfamiliar code | Small, focused lookups |
| Understanding architecture | Reading 1-2 specific files |
| Searching patterns across many files | Quick questions |
| Investigation before planning | Already know where to look |
| Keeping main context clean | Need iteration with you |

---

## Background Tasks

### Running Tasks in Background

**Method 1: Ask Claude**
```
Run the test suite in the background
```

**Method 2: Keyboard shortcut**
```
Ctrl+B  # While Claude is running a command
```

**Method 3: Bash mode**
```
! npm run build &
```

### Checking Output

```
/tasks              # List all background tasks
Check the background test results
```

### Use Cases

| Use Case | Example |
|----------|---------|
| Long builds | `npm run build` |
| Test suites | `npm test` |
| Dev servers | `npm run dev` |
| Parallel research | Multiple subagents |

---

## Skills and Custom Commands

### What Are Skills?

Reusable capabilities that extend Claude with:
- Domain-specific knowledge
- Step-by-step workflows
- Custom slash commands (`/skill-name`)
- Automatic invocation when relevant

### Skill Locations

- **Project**: `.claude/skills/<skill-name>/SKILL.md`
- **Personal**: `~/.claude/skills/<skill-name>/SKILL.md`

### Creating a Custom Skill

**1. Create directory:**
```bash
mkdir -p ~/.claude/skills/my-skill
```

**2. Write SKILL.md:**
```yaml
---
name: explain-code
description: Explains code with diagrams and analogies
---

When explaining code:
1. Start with an analogy
2. Draw ASCII diagram
3. Walk through step-by-step
4. Highlight common gotchas
```

**3. Use it:**
```
/explain-code src/auth/login.ts
```

### Skill Configuration Fields

| Field | Purpose |
|-------|---------|
| `name` | Slash command identifier |
| `description` | When to auto-invoke |
| `disable-model-invocation` | Only manual invocation (`/deploy`) |
| `user-invocable` | Hide from user menu |
| `allowed-tools` | Restrict tool access |
| `context: fork` | Run in subagent |
| `model` | Which model (`sonnet`, `opus`, `haiku`) |

### Dynamic Context with Command Execution

```yaml
---
name: pr-summary
context: fork
agent: Explore
---

## PR Context
- Diff: !`gh pr diff`
- Files: !`gh pr diff --name-only`

Summarize this pull request.
```

The `!` runs commands **before** Claude sees the prompt.

---

## Core Tools

### File Operations

| Tool | Purpose |
|------|---------|
| **Read** | View file contents |
| **Write** | Create new files |
| **Edit** | Modify existing files |
| **Glob** | Find files by pattern (`**/*.js`) |
| **Grep** | Search contents with regex |

### Execution

| Tool | Purpose |
|------|---------|
| **Bash** | Shell commands, git, package managers |
| **Task** | Background task management |

### Search & Web

| Tool | Purpose |
|------|---------|
| **Grep** | Regex search in files |
| **Glob** | Pattern matching for file discovery |
| **WebSearch** | Search the internet |
| **WebFetch** | Retrieve and analyze web content |

### When to Use Which Tool

| Task | Tool |
|------|------|
| Find files matching pattern | Glob |
| Search code for string | Grep |
| Understand file structure | Read |
| Run tests | Bash |
| Create new file | Write |
| Modify code | Edit |

### Tool Best Practices

**For searches - be specific:**
```
DON'T: "Find the authentication code"
DO:    "Glob for files in src/auth/, then Grep for 'oauth'"
```

**For bash - chain commands:**
```
DON'T: Run multiple commands separately
DO:    npm run build && npm test && npm run lint
```

---

## Context Management

### Context Window Fills Fast

Claude's context holds:
- Conversation messages
- File contents read
- Command outputs
- CLAUDE.md + skills
- System instructions

### Managing Context

| Strategy | How |
|----------|-----|
| **Skills** | Content loads only when invoked |
| **Subagents** | Isolated context, results summarized |
| **Bash** | Chain commands, background long ops |
| **CLAUDE.md** | Keep short (< 500 lines) |

### Commands

```
/context    # Check usage
/compact    # Trigger compaction
/clear      # Clear conversation
```

---

## Quick Reference

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Shift+Tab` | Cycle permission modes |
| `Ctrl+G` | Edit plan in external editor |
| `Ctrl+B` | Background current task |
| `Esc` | Interrupt Claude |

### Common Commands

```bash
claude --permission-mode plan  # Start in Plan Mode
claude --continue              # Resume conversation

/agents     # Manage subagents
/tasks      # List background tasks
/context    # Check context usage
/memory     # Show loaded CLAUDE.md files
/init       # Create CLAUDE.md
```

---

## Notes

<!-- Personal observations as you use Claude Code -->
