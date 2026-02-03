# Model Selection Deep Dive

## Available Models

### Overview

| Model | Speed | Capability | Cost | Best For |
|-------|-------|------------|------|----------|
| **Haiku** | Fastest | Good | Cheapest | Simple tasks, subagents |
| **Sonnet** | Fast | Very Good | Moderate | Day-to-day coding |
| **Opus** | Slower | Best | Expensive | Complex decisions |

### Model IDs

```
haiku   → claude-haiku (latest)
sonnet  → claude-sonnet-4 (latest)
opus    → claude-opus-4-5 (latest)
```

---

## When to Use Each Model

### Haiku

**Characteristics:**
- Very fast responses
- Lower capability for complex reasoning
- Cheapest option

**Best for:**
- Quick lookups
- Simple questions
- Subagent exploration
- High-volume operations
- Cost-sensitive tasks

**Examples:**
```
/model haiku
What files are in src/auth/?
Grep for "TODO" in the codebase
List all test files
```

### Sonnet (Default)

**Characteristics:**
- Fast responses
- Excellent coding ability
- Good balance of speed/quality/cost

**Best for:**
- Normal development (90% of tasks)
- Code implementation
- Debugging
- Refactoring
- Code review

**Examples:**
```
/model sonnet
Implement email validation in UserService
Fix the null reference in GetOrderById
Refactor this to use async/await
```

### Opus

**Characteristics:**
- Slower responses
- Best reasoning and planning
- Most expensive

**Best for:**
- Complex architecture decisions
- Nuanced planning
- Tricky debugging
- Security analysis
- Novel problem solving

**Examples:**
```
/model opus
Design the authentication system for our microservices
Analyze this race condition and propose solutions
Plan the migration from monolith to microservices
```

---

## Switching Models

### At Startup

```bash
claude --model opus
claude --model sonnet
claude --model haiku
```

### Mid-Session

```
/model opus
/model sonnet
/model haiku
```

### Via Keyboard

```
Cmd+P  (or Meta+P)
```

Opens model picker.

### Environment Default

```bash
export ANTHROPIC_MODEL=sonnet
```

Sets default for all new sessions.

---

## Mixed Model Strategies

### Opus Plan, Sonnet Execute

**Pattern:**
1. Use Opus to create detailed plan
2. Switch to Sonnet to implement

```bash
claude --model opus
```
```
Plan the authentication system redesign.
Include all files to modify, approach, and risks.
```

Then:
```
/model sonnet
Implement phase 1 of the plan.
```

**Why:**
- Opus excels at planning
- Sonnet executes faster and cheaper
- Best of both worlds

### Haiku for Subagents

Explore agent already uses Haiku by default. For custom agents:

```markdown
---
name: file-finder
model: haiku
---
Find files matching the pattern.
```

### Sonnet Primary, Opus for Decisions

Stay in Sonnet, switch to Opus for key moments:

```
/model sonnet
# Normal coding...

/model opus
Should we use Redis or in-memory cache for sessions?
Consider: scaling, cost, complexity, team familiarity.

/model sonnet
# Implement the decision...
```

---

## Cost Optimization

### Relative Costs

Approximate relative pricing (Haiku = 1x):
- Haiku: 1x
- Sonnet: ~10x
- Opus: ~50x

### Cost-Saving Strategies

**1. Default to Sonnet**
Good enough for most tasks.

**2. Use Haiku for exploration**
```
Use Explore agent to find all authentication code
```
Explore uses Haiku automatically.

**3. Budget limits**
```bash
claude -p "Analyze the codebase" --max-budget-usd 5.00
```

**4. Turn limits**
```bash
claude -p "Fix the bug" --max-turns 3
```

**5. Print mode for one-offs**
```bash
claude -p "What version of Node?"
```
No session overhead.

### When to Spend on Opus

Worth the cost for:
- Architectural decisions that affect months of work
- Complex debugging that's been stuck
- Security analysis of critical code
- Planning major refactors

Not worth it for:
- Simple file edits
- Running tests
- Straightforward implementation
- Tasks Sonnet handles well

---

## Model Selection by Task

### Quick Reference

| Task | Recommended Model |
|------|-------------------|
| Simple lookup | Haiku |
| List files | Haiku |
| Quick questions | Haiku |
| Normal coding | Sonnet |
| Bug fixes | Sonnet |
| Refactoring | Sonnet |
| Code review | Sonnet |
| Architecture planning | Opus |
| Complex debugging | Opus |
| Security analysis | Opus |
| Novel problems | Opus |

### By Workflow Phase

| Phase | Model | Why |
|-------|-------|-----|
| Explore | Haiku (via agent) | Fast, cheap |
| Plan | Opus | Best reasoning |
| Implement | Sonnet | Good balance |
| Review | Sonnet or Opus | Depends on criticality |
| Commit | Sonnet | Simple task |

---

## Model-Specific Tips

### Getting Best Results from Haiku

- Be very specific
- Simple, focused requests
- Don't expect complex reasoning
- Use for mechanical tasks

```
# Good for Haiku
List all files in src/auth/
Find usages of "HttpClient"
Show me the imports in this file

# Not good for Haiku
Design the authentication architecture
Analyze this complex race condition
```

### Getting Best Results from Sonnet

- Default choice
- Good for multi-step tasks
- Handles ambiguity well
- Solid code generation

```
# Great for Sonnet
Implement email validation following our existing patterns
Debug this null reference exception
Refactor to use async/await
```

### Getting Best Results from Opus

- Ask for reasoning
- Request trade-off analysis
- Use for important decisions
- Worth the wait

```
# Great for Opus
Design the caching strategy. Consider:
- Read/write patterns
- Consistency requirements
- Scaling needs
- Team expertise
Explain your reasoning and trade-offs.
```

---

## Troubleshooting Model Issues

### Haiku Giving Poor Results

Switch to Sonnet:
```
/model sonnet
```

Task may be too complex for Haiku.

### Sonnet Struggling

Try Opus for this specific question:
```
/model opus
[your complex question]
/model sonnet
# Continue normally
```

### Opus Too Slow

For time-sensitive work, accept Sonnet quality:
```
/model sonnet
```

Or be more specific to reduce thinking time:
```
Don't explain, just implement: [specific task]
```

---

## Environment Configuration

### Default Model

```bash
# In ~/.bashrc or ~/.zshrc
export ANTHROPIC_MODEL=sonnet
```

### Specific Versions

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-5-20251101
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-20250514
```

### Per-Project Defaults

In `.claude/settings.json`:
```json
{
  "model": "sonnet"
}
```

---

## Notes

<!-- Add observations about model selection -->

