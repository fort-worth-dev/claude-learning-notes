# Context Management Deep Dive

## The Context Window

### What Is It?

The context window is Claude's "working memory" - everything Claude can see and reference during a conversation:
- Your messages
- Claude's responses
- File contents read
- Tool outputs
- CLAUDE.md instructions

### Size Limits

| Model | Context Window |
|-------|---------------|
| Sonnet | 200k tokens |
| Opus | 200k tokens |
| Sonnet (extended) | 1 million tokens |

**Rough token estimates:**
- 1 token ≈ 4 characters (English)
- 1 line of code ≈ 10-20 tokens
- Average source file ≈ 500-2000 tokens

### Why It Matters

- Performance degrades as context fills
- Instructions at the start may be "forgotten"
- Large file reads consume significant space
- Tool outputs accumulate quickly

---

## CLAUDE.md Hierarchy

### Load Order (highest priority wins)

```
1. Managed policy          # Organization-wide (IT-deployed)
   └── Enterprise controls, cannot be overridden

2. Project CLAUDE.md       # ./CLAUDE.md or ./.claude/CLAUDE.md
   └── Shared team instructions, committed to git

3. User CLAUDE.md          # ~/.claude/CLAUDE.md
   └── Personal preferences, applies to all projects

4. Project local           # ./CLAUDE.local.md
   └── Personal project overrides, gitignored

5. Project rules           # ./.claude/rules/*.md
   └── Modular rules, committed to git
```

### When Instructions Conflict

Higher priority wins. Example:
- User CLAUDE.md: "Use tabs for indentation"
- Project CLAUDE.md: "Use 2 spaces for indentation"
- **Result:** 2 spaces (project overrides user)

### Checking What's Loaded

```
/memory
```

Shows all CLAUDE.md files currently in context.

---

## CLAUDE.md Best Practices

### What to Include

**Build and run commands:**
```markdown
## Commands
- Build: `dotnet build`
- Test: `dotnet test --filter "Category!=Integration"`
- Run: `dotnet run --project src/Api`
```

**Non-obvious conventions:**
```markdown
## Conventions
- All API endpoints return wrapped responses: `{ data: T, errors: [] }`
- Use `DateTimeOffset` not `DateTime` for all timestamps
- Repository methods return `null` not exceptions for not-found
```

**Architectural decisions:**
```markdown
## Architecture
- CQRS pattern: Commands in /Commands, Queries in /Queries
- No business logic in controllers - delegate to handlers
- Feature folders, not layer folders
```

**Common gotchas:**
```markdown
## Gotchas
- The `Orders` table has soft deletes - always filter by `IsDeleted = false`
- Auth tokens expire after 15 minutes in dev (8 hours in prod)
- Don't use EF migrations - we use DbUp scripts in /Database/Scripts
```

### What NOT to Include

| Avoid | Why |
|-------|-----|
| Standard language conventions | Claude already knows C# conventions |
| Information Claude can infer | Reading code is often enough |
| Lengthy documentation | Wastes context, use links instead |
| Frequently changing info | Will become stale |
| Obvious things | "Use meaningful variable names" |

### Keep It Concise

Bad:
```markdown
## Code Style
When writing C# code, please ensure that you follow the standard
C# naming conventions as documented by Microsoft. This includes
using PascalCase for public members and camelCase for private
fields. Additionally, we prefer to use...
```

Good:
```markdown
## Code Style
- Private fields: `_camelCase` (not `m_` prefix)
- Interfaces: No `I` prefix for domain interfaces
```

---

## File Organization Patterns

### Single CLAUDE.md (Simple Projects)

```
project/
├── CLAUDE.md          # All instructions here
├── src/
└── tests/
```

### Split Configuration (Team + Personal)

```
project/
├── CLAUDE.md          # Team conventions (committed)
├── CLAUDE.local.md    # Your personal preferences (gitignored)
├── src/
└── tests/
```

### Modular Rules (Large Projects)

```
project/
├── .claude/
│   ├── CLAUDE.md           # Core instructions
│   └── rules/
│       ├── api-conventions.md
│       ├── testing-guidelines.md
│       └── database-patterns.md
├── src/
└── tests/
```

Rules files are loaded alphabetically. Use prefixes for ordering:
```
rules/
├── 01-architecture.md
├── 02-coding-standards.md
└── 03-testing.md
```

---

## Managing Context During Sessions

### Monitor Usage

```
/context
```

Shows visual bar of context consumption.

### Clear Between Tasks

```
/clear
```

Use when:
- Switching to unrelated task
- Context is cluttered with irrelevant file reads
- Claude seems to be "confused" by earlier conversation

### Compact to Preserve Focus

```
/compact
```

Summarizes conversation, freeing space while retaining key points.

```
/compact focus on the API authentication changes
```

Targeted compaction - keeps specific topic details.

### When to Compact vs Clear

| Situation | Action |
|-----------|--------|
| Continuing same task, running low on context | `/compact` |
| Task complete, starting something new | `/clear` |
| Claude referencing wrong earlier context | `/clear` |
| Long session, want to preserve decisions made | `/compact` |

---

## Auto-Compaction

### What Happens Automatically

When context fills, Claude Code:
1. Clears older tool outputs first
2. Summarizes earlier conversation turns
3. Preserves: recent messages, key code, current task
4. May lose: detailed early instructions, old file contents

### Implications

- Instructions at session start may fade
- Put critical rules in CLAUDE.md, not conversation
- Re-state important constraints if session is long

### Signs of Context Pressure

- Claude "forgets" earlier instructions
- Responses become less coherent
- Claude re-reads files it already read
- `/context` shows high usage

---

## Strategies for Large Codebases

### Be Specific About Files

Bad:
```
"Update the user service"
```

Good:
```
"Update src/Services/UserService.cs to add email validation"
```

### Use the Explore Agent

For open-ended exploration, let the agent search:
```
"Use the Explore agent to find where authentication tokens are validated"
```

Avoids loading many files into your context.

### Chunk Large Refactors

Instead of:
```
"Refactor the entire data layer to use the repository pattern"
```

Break it up:
```
"Let's refactor to repository pattern. Start with UserRepository only."
# Complete, verify, then:
/clear
"Now add OrderRepository following the same pattern as UserRepository"
```

### Reference, Don't Repeat

If you've established a pattern:
```
"Add ProductRepository following the same pattern as UserRepository"
```

Claude remembers within the session - no need to re-explain.

---

## Bootstrapping CLAUDE.md

### Using /init

```
/init
```

Claude analyzes your project and generates a starter CLAUDE.md with:
- Detected build/test commands
- Project structure observations
- Framework-specific conventions

### Review and Edit

The generated file is a starting point. Edit it to:
- Remove obvious/redundant information
- Add team-specific conventions
- Include gotchas only you know about

---

## Notes

<!-- Add observations about context management as you learn -->

