# Subagents Deep Dive

## What Are Subagents?

Subagents are specialized AI assistants that Claude spawns to handle specific tasks. They run in **isolated context windows**, meaning:

- They don't see your main conversation
- Their file reads don't bloat your context
- Results are summarized when they return
- Each has specific tool access and capabilities

Think of them as focused specialists Claude can delegate to.

---

## Built-in Subagents

### Explore Agent

| Attribute | Value |
|-----------|-------|
| Model | Haiku (fast, cheap) |
| Tools | Read-only (Glob, Grep, Read) |
| Purpose | Codebase exploration and search |

**Best for:**
- Finding where something is implemented
- Understanding architecture
- Searching across many files
- Quick reconnaissance

**Thoroughness levels:**
- `quick` - Basic search, first matches
- `medium` - Moderate exploration
- `very thorough` - Comprehensive analysis

```
Use Explore to quickly find where auth tokens are validated
Use Explore to do a medium investigation of the payment flow
Use Explore to very thoroughly analyze the authentication system
```

### Plan Agent

| Attribute | Value |
|-----------|-------|
| Model | Inherited from parent |
| Tools | Read-only |
| Purpose | Research during planning phase |

**Best for:**
- Understanding code before changing it
- Gathering context for implementation plans
- Safe exploration (can't modify anything)

Used automatically in Plan Mode. You rarely invoke it directly.

### General-Purpose Agent

| Attribute | Value |
|-----------|-------|
| Model | Inherited from parent |
| Tools | All tools |
| Purpose | Complex multi-step tasks |

**Best for:**
- Tasks requiring both read and write
- Multi-step workflows
- When agent needs to make decisions and act

```
Use general-purpose to investigate the bug and implement a fix
```

### Bash Agent

| Attribute | Value |
|-----------|-------|
| Model | Inherited from parent |
| Tools | Bash only |
| Purpose | Terminal operations in isolation |

**Best for:**
- Long-running commands
- Commands with verbose output
- Keeping terminal noise out of main context

---

## How Context Isolation Works

### Without Subagents

```
Your Context Window
├── Your messages
├── Claude's responses
├── File 1 contents (read)
├── File 2 contents (read)
├── File 3 contents (read)
├── ... (50 more files)
├── Command outputs
└── Context filling up fast
```

### With Subagents

```
Your Context Window                 Subagent Context (isolated)
├── Your messages                   ├── Subagent prompt
├── Claude's responses              ├── File 1 contents
├── Summary from subagent ◄─────────├── File 2 contents
└── Context stays lean              ├── ... (50 more files)
                                    └── Subagent analysis
```

The subagent reads 50 files, but your context only gets the summary.

---

## Automatic vs Manual Delegation

### Automatic Delegation

Claude decides to use subagents based on your request:

| Your Request | Likely Delegates To | Why |
|--------------|---------------------|-----|
| "Understand how auth works" | Explore | Broad exploration |
| "Find where payments are handled" | Explore | Multi-file search |
| "What's the architecture?" | Explore | Codebase analysis |
| "Research this, then fix it" | General-purpose | Needs read + write |

### Trigger Words

Words that often trigger automatic delegation:
- **understand** - exploration
- **find** / **search** - file discovery
- **investigate** / **research** - deep analysis
- **explore** - explicit exploration
- **review** - code analysis

### Stays in Main Session

Claude handles directly (no delegation):
- Reading a specific file you named
- Small, focused edits
- Direct questions
- Tasks needing back-and-forth with you

### Manual Override

**Force delegation:**
```
Use a subagent to research this
Use Explore to find all error handlers
Use general-purpose to investigate and fix
```

**Prevent delegation:**
```
Just read src/auth.ts directly, don't use a subagent
Read the file yourself, no need for an agent
```

---

## Subagent Locations

### Priority Order

1. **CLI flag** - `--agents path/to/agents/` (session only)
2. **Project** - `.claude/agents/*.md` (committed to git)
3. **User** - `~/.claude/agents/*.md` (all your projects)

### Project Agents

Create in `.claude/agents/`:
```
.claude/
└── agents/
    └── my-agent.md
```

Share with team via git.

### Personal Agents

Create in `~/.claude/agents/`:
```
~/.claude/
└── agents/
    └── my-personal-agent.md
```

Available in all your projects.

---

## Creating Custom Subagents

### Basic Structure

```markdown
---
name: code-reviewer
description: Reviews code for best practices and potential issues
model: sonnet
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Code Review Agent

You are a code reviewer. When given code to review:

1. Check for security issues
2. Look for performance problems
3. Verify error handling
4. Assess code clarity

Be specific about line numbers and provide examples.
```

### Configuration Fields

| Field | Purpose | Values |
|-------|---------|--------|
| `name` | Agent identifier | Any string |
| `description` | When to use (for auto-delegation) | Any string |
| `model` | Which model | `sonnet`, `opus`, `haiku` |
| `allowed-tools` | Restrict tool access | List of tool names |

### Tool Restrictions

```yaml
allowed-tools:
  - Read      # Can read files
  - Glob      # Can search for files
  - Grep      # Can search in files
  # No Write, Edit, or Bash = read-only agent
```

### Example: Test Runner Agent

```markdown
---
name: test-runner
description: Runs tests and analyzes failures
model: haiku
allowed-tools:
  - Bash
  - Read
  - Grep
---

# Test Runner

Run the test suite and analyze any failures:

1. Execute the test command
2. If tests fail, read the failing test files
3. Grep for related code
4. Summarize what's failing and likely causes

Test command: `dotnet test`
```

---

## Subagent Communication Patterns

### One-Way Delegation

```
You → Claude → Subagent → Results → Claude → You
```

Standard pattern. Subagent does work, returns summary.

### Parallel Subagents

```
You → Claude ─┬→ Subagent A → Results A ─┬→ Claude → You
              └→ Subagent B → Results B ─┘
```

Multiple subagents work simultaneously:
```
Use subagents to investigate auth and payments in parallel
```

### Sequential Subagents

```
You → Claude → Subagent A → Results → Claude → Subagent B → Results → You
```

Chain of investigations:
```
Use Explore to understand the current implementation,
then use general-purpose to refactor based on findings
```

---

## Best Practices

### When to Use Subagents

| Situation | Use Subagent? |
|-----------|---------------|
| Exploring unfamiliar code | Yes - keeps context clean |
| Reading many files | Yes - summarized results |
| Simple file lookup | No - direct is faster |
| Need back-and-forth | No - subagents are one-shot |
| Complex multi-step task | Yes - general-purpose |

### Effective Prompts

**Be specific about scope:**
```
Use Explore to find authentication code in src/auth/ only
```

**Specify thoroughness:**
```
Use Explore to do a quick search for the config file
Use Explore to very thoroughly analyze the entire API layer
```

**Chain actions:**
```
Use general-purpose to find the bug, fix it, and run tests
```

---

## Notes

<!-- Add observations about subagents as you use them -->

