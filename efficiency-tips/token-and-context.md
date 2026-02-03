# Token and Context Management Deep Dive

## Understanding Context

### What Consumes Context

| Component | Typical Size | Notes |
|-----------|--------------|-------|
| System prompt | ~5-10k tokens | Fixed overhead |
| CLAUDE.md files | Variable | Keep under 500 lines |
| Conversation | Grows over time | Primary consumer |
| File reads | ~500-2k per file | Accumulates quickly |
| Tool outputs | Variable | Command results, searches |
| MCP tools | ~1-5k per server | Schema definitions |

### Checking Usage

```
/context
```

Shows breakdown:
- Conversation percentage
- CLAUDE.md percentage
- MCP/tools percentage
- Available space

### Context Window Sizes

| Model | Standard | Extended |
|-------|----------|----------|
| Sonnet | 200k tokens | 1M tokens |
| Opus | 200k tokens | - |
| Haiku | 200k tokens | - |

---

## Reducing Token Usage

### Strategy 1: Clear Between Tasks

```
/clear
```

**When to clear:**
- Switching to unrelated work
- Completed current task
- Context feels cluttered
- Starting fresh investigation

**Don't clear when:**
- Continuing same task
- Need previous context
- Mid-conversation on same topic

### Strategy 2: Compact Instead of Clear

```
/compact
```

Summarizes conversation while preserving key points.

```
/compact focus on the API authentication changes
```

Targeted compaction - keeps specific details.

**When to compact:**
- Long session, still working on same thing
- Need to preserve decisions made
- Want to free space without losing context

### Strategy 3: Keep CLAUDE.md Lean

**Bad:**
```markdown
# CLAUDE.md - 800 lines of instructions
... extensive documentation ...
... copy-pasted style guides ...
... verbose explanations ...
```

**Good:**
```markdown
# CLAUDE.md - 50 lines
## Build
- `dotnet build`
- `dotnet test`

## Conventions
- Private fields: `_camelCase`
- Async all the way

## Gotchas
- Orders table has soft deletes
```

### Strategy 4: Move to Skills

Instead of putting everything in CLAUDE.md, create skills:

```
~/.claude/skills/deploy/SKILL.md     # Loaded only when deploying
~/.claude/skills/review/SKILL.md     # Loaded only when reviewing
```

Skills load on-demand, not always.

### Strategy 5: Use Subagents

```
Use Explore to investigate the auth module
```

Subagent reads 50 files, your context gets summary.

**Without subagent:** 50 files × 1000 tokens = 50k tokens in your context
**With subagent:** 500 token summary in your context

### Strategy 6: Disable Unused MCP

```
/mcp
```

Shows MCP servers and their overhead. Disable unused ones.

Prefer CLI tools:
- `gh` over GitHub MCP
- `aws` over AWS MCP
- `az` over Azure MCP

---

## Monitoring Context Health

### Warning Signs

| Sign | Problem | Fix |
|------|---------|-----|
| Slow responses | Context near limit | `/clear` or `/compact` |
| Claude "forgets" | Important info compacted out | Re-state or add to CLAUDE.md |
| Repeated questions | Context pollution | `/clear` and restart |
| `/context` >80% | Running out of space | Clear or compact |

### Proactive Monitoring

Check `/context` periodically:
- After reading many files
- After long tool outputs
- Before starting complex task
- When responses slow down

### Context Budgeting

For a 200k token context:
- ~10k: System + CLAUDE.md
- ~50k: Comfortable conversation
- ~100k: Extended work session
- ~150k: Getting tight
- ~180k+: Time to clear/compact

---

## Compaction Deep Dive

### How Auto-Compaction Works

When context fills, Claude Code automatically:
1. Clears old tool outputs first
2. Summarizes older conversation turns
3. Preserves recent messages
4. Keeps key code snippets

### What Gets Lost

- Detailed early instructions
- Verbose tool outputs
- Exploration tangents
- Nuanced preferences stated early

### What Gets Preserved

- Recent conversation
- File paths mentioned
- Key decisions
- Current task context

### Manual Compaction Options

**General:**
```
/compact
```

**Focused:**
```
/compact focus on database schema changes
/compact preserve the API endpoint decisions
/compact keep the test results
```

### Compaction Instructions in CLAUDE.md

```markdown
# Compact instructions
When compacting, preserve:
- All modified file paths
- Test results and failures
- API contract changes
- Architecture decisions
```

---

## Extended Thinking

### What It Is

Extended thinking lets Claude "think longer" on complex problems.

### Cost Implications

More thinking = more tokens = higher cost

### Controlling It

**Environment variable:**
```bash
export MAX_THINKING_TOKENS=10000
```

**Toggle in session:**
```
Alt+T
```

### When to Use

**Enable for:**
- Complex architecture decisions
- Tricky debugging
- Multi-step planning
- Nuanced code review

**Disable for:**
- Simple file edits
- Running commands
- Straightforward tasks
- Cost-sensitive work

---

## Cost Optimization

### Model Selection for Cost

| Task | Model | Why |
|------|-------|-----|
| Quick lookup | Haiku | Cheapest |
| Normal coding | Sonnet | Good balance |
| Complex planning | Opus | Best quality |
| Subagent exploration | Haiku | Fast and cheap |

### Budget Limits

```bash
claude -p "Explore the codebase" --max-budget-usd 5.00
```

Stops when budget reached.

### Turn Limits

```bash
claude -p "Fix this bug" --max-turns 3
```

Limits back-and-forth iterations.

### Print Mode for One-Offs

```bash
claude -p "What's in package.json?"
```

No session overhead, quick answer, exit.

---

## Efficient Prompting

### Be Specific

**Expensive (explores everything):**
```
Find the authentication code
```

**Cheaper (targeted):**
```
Read src/auth/login.cs and explain the token validation
```

### Batch Related Requests

**Expensive (multiple turns):**
```
Read file A
Now read file B
Now read file C
```

**Cheaper (one turn):**
```
Read files A, B, and C and summarize how they interact
```

### Use References

**Expensive:**
```
Change the validation the same way we did before
(Claude re-reads previous conversation)
```

**Cheaper:**
```
Add email validation to UserService following the pattern in AddressValidator.cs
```

---

## Session Hygiene

### Name Sessions

```
/rename feature-oauth
```

Easier to find, easier to know what's in them.

### One Task Per Session

Don't mix unrelated work:
- Session A: OAuth feature
- Session B: Bug fix in orders
- Session C: Documentation update

### Clean Up Old Sessions

```bash
claude sessions list --all
```

Delete sessions you won't need.

### Fork for Experiments

```bash
claude --continue --fork-session
```

Try risky changes without polluting main session.

---

## Quick Reference

### Commands

| Command | Purpose |
|---------|---------|
| `/context` | Check usage |
| `/clear` | Reset conversation |
| `/compact` | Summarize conversation |
| `/compact [focus]` | Targeted summary |
| `/mcp` | Manage MCP servers |

### Flags

| Flag | Purpose |
|------|---------|
| `--max-budget-usd N` | Cost limit |
| `--max-turns N` | Turn limit |
| `-p "query"` | One-off, no session |
| `--model haiku` | Cheap model |

### Environment

```bash
export MAX_THINKING_TOKENS=10000
export ANTHROPIC_MODEL=sonnet
```

---

## Notes

<!-- Add observations about token management -->

