# Skills Deep Dive

## What Are Skills?

Skills are reusable capabilities that extend Claude with:
- Domain-specific knowledge
- Step-by-step workflows
- Custom slash commands (`/skill-name`)
- Automatic invocation when relevant

Think of skills as specialized expertise packages you can add to Claude.

---

## Skill Locations

### Project Skills (Team)

```
.claude/skills/<skill-name>/SKILL.md
```

Committed to git. Shared with team.

### Personal Skills (User)

```
~/.claude/skills/<skill-name>/SKILL.md
```

Available in all your projects.

### Priority

Personal skills override project skills with the same name.

---

## Anatomy of a Skill

### Basic Structure

```markdown
---
name: my-skill
description: What this skill does and when to use it
---

# Skill Instructions

Your instructions to Claude go here.
Claude follows these when the skill is invoked.
```

### Required Elements

| Element | Purpose |
|---------|---------|
| `name` | The slash command (`/name`) |
| `description` | When Claude should use this skill |
| Content | Instructions Claude follows |

---

## Configuration Fields

### Core Fields

```yaml
---
name: deploy
description: Deploys the application to production
---
```

### Control Fields

| Field | Purpose | Values |
|-------|---------|--------|
| `disable-model-invocation` | Only manual invocation | `true` / `false` |
| `user-invocable` | Show in user menu | `true` / `false` |
| `model` | Which model to use | `sonnet`, `opus`, `haiku` |

### Execution Context

| Field | Purpose | Values |
|-------|---------|--------|
| `context` | Run in isolation | `fork` (runs as subagent) |
| `agent` | Which subagent type | `Explore`, `Plan`, etc. |
| `allowed-tools` | Restrict tool access | List of tool names |

---

## Invocation Methods

### Manual (Slash Command)

```
/deploy
/explain-code src/auth.ts
/pr-summary
```

### Automatic (Claude Decides)

Claude reads the `description` field and invokes when relevant.

```yaml
description: Use when the user asks to explain code
```

User says "explain this code" → Claude invokes the skill.

### Preventing Auto-Invocation

```yaml
disable-model-invocation: true
```

Now only `/deploy` works, Claude won't auto-invoke.

---

## Dynamic Context

### Command Execution

Use `!` to run commands before Claude sees the prompt:

```markdown
---
name: pr-summary
---

## PR Context
- Diff: !`gh pr diff`
- Files changed: !`gh pr diff --name-only`

Summarize this pull request.
```

The commands run first, output is injected into the prompt.

### File Inclusion

Include file contents:

```markdown
---
name: review-tests
---

## Test Files
!`cat tests/*.cs`

Review these tests for coverage gaps.
```

### Conditional Content

```markdown
---
name: build-check
---

## Build Output
!`dotnet build 2>&1 || echo "BUILD FAILED"`

Analyze the build result.
```

---

## Skill Patterns

### Code Explainer

```markdown
---
name: explain-code
description: Explains code with analogies and diagrams
---

When explaining code:

1. Start with a real-world analogy
2. Draw an ASCII diagram of the flow
3. Walk through step-by-step
4. Highlight common gotchas
5. Suggest improvements if any

Be concise but thorough.
```

### PR Reviewer

```markdown
---
name: review-pr
description: Reviews pull requests for issues
context: fork
agent: Explore
---

## PR Diff
!`gh pr diff`

## Changed Files
!`gh pr diff --name-only`

Review this PR for:
1. Bugs or logic errors
2. Security issues
3. Performance concerns
4. Missing tests
5. Code style violations

Be specific with line numbers.
```

### Deployment Skill

```markdown
---
name: deploy
description: Deploy to production
disable-model-invocation: true
allowed-tools:
  - Bash
  - Read
---

# Deployment Checklist

Before deploying:
1. Run `dotnet test` - all must pass
2. Check `git status` - must be clean
3. Verify on correct branch

To deploy:
1. Run `dotnet publish -c Release`
2. Execute deployment script

Always confirm with user before final step.
```

### .NET Specific Skill

```markdown
---
name: ef-migration
description: Creates and applies EF Core migrations
allowed-tools:
  - Bash
  - Read
---

# EF Migration Helper

When creating migrations:

1. Ask for migration name if not provided
2. Run: `dotnet ef migrations add <name> --project src/Data`
3. Show the generated migration file
4. Ask if user wants to apply it
5. If yes: `dotnet ef database update --project src/Data`

Always show what SQL will be generated before applying.
```

---

## Running in Isolation

### Why Fork?

```yaml
context: fork
```

Runs the skill in a subagent:
- Isolated context window
- Doesn't bloat main conversation
- Good for verbose operations

### With Specific Agent

```yaml
context: fork
agent: Explore
```

Uses the Explore agent's capabilities (read-only, fast).

### Tool Restrictions

```yaml
allowed-tools:
  - Read
  - Glob
  - Grep
```

Skill can only use these tools. Safer for risky operations.

---

## Arguments and Parameters

### Passing Arguments

```
/explain-code src/Services/AuthService.cs
```

The argument `src/Services/AuthService.cs` is available in the skill.

### Using Arguments

```markdown
---
name: explain-code
---

Read the file provided and explain it.
Focus on the main logic flow.
```

Claude receives the full command including arguments.

### Multiple Arguments

```
/compare-files src/old.cs src/new.cs
```

---

## Creating Your First Skill

### Step 1: Create Directory

```bash
mkdir -p ~/.claude/skills/my-skill
```

### Step 2: Create SKILL.md

```bash
cat > ~/.claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Does something useful
---

# My Skill

Instructions here.
EOF
```

### Step 3: Test It

```
/my-skill
```

### Step 4: Iterate

Edit SKILL.md, test again. No restart needed.

---

## Debugging Skills

### Skill Not Found

- Check file is named `SKILL.md` (exact case)
- Verify directory structure: `skills/<name>/SKILL.md`
- Check YAML frontmatter syntax

### Skill Not Auto-Invoking

- Check `description` is clear and relevant
- Verify `disable-model-invocation` is not set
- Try explicit invocation first: `/skill-name`

### Commands Not Running

- Verify `!` syntax: `!`backtick command backtick`
- Check command works in terminal first
- Look for shell escaping issues

### Wrong Output

- Add debug output to commands
- Check if running in fork (isolated context)
- Verify tool restrictions aren't blocking needed tools

---

## Best Practices

### Keep Skills Focused

One skill = one job. Don't combine unrelated functions.

### Use Descriptive Names

```
/deploy-staging     # Clear
/ds                 # Cryptic
```

### Document Expected Arguments

```markdown
---
name: explain-code
description: Explains code. Usage: /explain-code <file-path>
---
```

### Test Before Sharing

Test skills locally before adding to project `.claude/skills/`.

### Version Control Project Skills

```
.claude/skills/
├── deploy/SKILL.md
├── review/SKILL.md
└── test-coverage/SKILL.md
```

Commit these to git for team sharing.

---

## Notes

<!-- Add observations about skills as you create them -->

