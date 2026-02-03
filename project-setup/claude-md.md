# CLAUDE.md Deep Dive

## What Is CLAUDE.md?

A memory file Claude reads at session start. Provides persistent context that:
- Claude cannot infer from code alone
- Applies across all sessions in the project
- Survives `/clear` and context compaction

Think of it as instructions for a new team member who knows the language but not your project.

---

## File Locations

### Hierarchy (highest to lowest priority)

| Location | Scope | Shared | Purpose |
|----------|-------|--------|---------|
| Managed policy | Organization | Yes | IT-enforced rules |
| `./CLAUDE.md` | Project root | Yes (git) | Main project instructions |
| `./.claude/CLAUDE.md` | Project | Yes (git) | Alternative location |
| `~/.claude/CLAUDE.md` | All projects | No | Personal defaults |
| `./CLAUDE.local.md` | Project | No | Personal overrides |

### Priority Rules

- Higher priority overrides lower
- Project-level beats user-level
- Local overrides everything (for personal use)

### Checking Loaded Files

```
/memory
```

Shows all CLAUDE.md files currently in context.

---

## What to Include

### Build and Test Commands

```markdown
## Commands
- Build: `dotnet build`
- Test: `dotnet test`
- Run: `dotnet run --project src/WebApp`
- Watch: `dotnet watch run --project src/WebApp`
```

Claude needs to know how to build and test your project.

### Code Style (Differences Only)

```markdown
## Code Style
- Private fields: `_camelCase` (not `m_` prefix)
- Interfaces: No `I` prefix for domain interfaces
- Prefer expression-bodied members for one-liners
```

Only include what differs from standard conventions.

### Architectural Decisions

```markdown
## Architecture
- CQRS: Commands in /Commands, Queries in /Queries
- No business logic in controllers
- Feature folders, not layer folders
```

Decisions that affect how Claude writes code.

### Project-Specific Gotchas

```markdown
## Gotchas
- Orders table uses soft deletes (always filter IsDeleted)
- Auth tokens expire after 15 minutes in dev
- Don't use EF migrations - we use DbUp scripts
```

Things Claude would get wrong without being told.

### Repository Conventions

```markdown
## Git
- Commit messages: imperative mood ("Add" not "Added")
- Branch naming: feature/description, fix/issue-number
- Don't commit unless explicitly asked
```

---

## What NOT to Include

### Standard Conventions

Claude already knows:
- C# naming conventions
- Common design patterns
- Language best practices

Don't repeat what Claude knows.

### Things Claude Can Infer

Claude can see:
- File structure
- Dependencies (package.json, .csproj)
- Existing code patterns

Let Claude read the code.

### Lengthy Documentation

Bad:
```markdown
## API Documentation
The UserService class provides methods for managing users.
The GetUser method accepts an integer ID parameter and returns
a User object. If the user is not found, it returns null...
[500 more lines]
```

Good:
```markdown
## API Documentation
See `/docs/api.md` for full API reference.
```

Link, don't paste.

### File-by-File Descriptions

Claude can explore the codebase. Don't duplicate the file tree.

---

## Keeping It Concise

### The Test

For each line, ask:
> "Would removing this cause Claude to make mistakes?"

If no, remove it.

### Target Size

- Ideal: Under 100 lines
- Acceptable: Under 200 lines
- Too long: Over 500 lines

Long files cause Claude to ignore rules.

### Pruning Techniques

**Before:**
```markdown
## Code Style
When writing C# code, please ensure that you follow modern C#
conventions. This includes using null-conditional operators like ?.
and ??, using pattern matching where appropriate, and preferring
async/await over blocking calls like .Result and .Wait()...
```

**After:**
```markdown
## Code Style
- Use modern C#: ?., ??, pattern matching
- Async/await over .Result, .Wait()
```

---

## Example CLAUDE.md Files

### Minimal (.NET Project)

```markdown
# Build
- `dotnet build`
- `dotnet test`

# Conventions
- Private fields: `_camelCase`
- Always use async/await

# Gotchas
- Soft deletes on Orders table
```

### Standard (.NET + Blazor)

```markdown
# Commands
- Build: `dotnet build`
- Test: `dotnet test`
- Run: `dotnet run --project src/WebApp`

# Code Style
- Use modern C#: ?., ??, pattern matching
- Prefer async/await over .Result, .Wait()
- Use var when type obvious
- Prefer DI over static methods

# Blazor
- Use EventCallback over Action
- Cascading parameters sparingly
- Extract sub-components when complex

# Azure
- Blob Storage for files; SAS tokens on-demand
- Never commit connection strings

# Git
- Commit messages: explain "why" not "what"
- Don't commit unless explicitly asked
```

### Full (Large Project)

```markdown
# Build and Test
- Build: `dotnet build`
- Test: `dotnet test --filter "Category!=Integration"`
- Integration tests: `dotnet test --filter "Category=Integration"`
- Run API: `dotnet run --project src/Api`
- Run Worker: `dotnet run --project src/Worker`

# Code Style
- Private fields: `_camelCase`
- Async all the way - no .Result or .Wait()
- Use ConfigureAwait(false) in libraries
- Prefer records for DTOs

# Architecture
- CQRS pattern in /Commands and /Queries
- Mediator pattern via MediatR
- Feature folders, not layer folders
- Domain events for cross-aggregate communication

# Database
- EF Core with DbUp for migrations
- Always use .AsNoTracking() for reads
- Soft deletes on Users, Orders, Products
- UTC timestamps everywhere

# API Conventions
- Wrapped responses: { data: T, errors: [] }
- Use ProblemDetails for errors
- Versioning via URL: /api/v1/

# Gotchas
- Legacy orders (pre-2023) have nullable Customer
- Auth service has 5-second timeout in dev
- Don't modify audit tables directly

# Git
- Branch naming: feature/, fix/, release/
- Squash merge to main
- Don't commit unless asked
```

---

## Importing External Files

### @-References

```markdown
See @README.md for project overview.
Git workflow: @docs/git-workflow.md
API reference: @docs/api.md
```

Claude reads these files when referenced.

### When to Use @-References

- Long documentation that shouldn't be inline
- Files that change frequently
- Shared docs used by humans too

---

## Personal vs Project

### Project CLAUDE.md (Committed)

Team-wide rules:
- Build commands
- Code conventions
- Architecture decisions
- Git workflow

### CLAUDE.local.md (Gitignored)

Personal preferences:
- Your model preference
- Your editor settings
- Local environment quirks
- Shortcuts for your workflow

### User CLAUDE.md (~/.claude/)

Defaults for all projects:
- Your general preferences
- Common tools you use
- Personal style preferences

---

## Bootstrapping

### Using /init

```
/init
```

Claude analyzes the project and generates a starter CLAUDE.md.

### Post-Init Checklist

1. [ ] Review generated content
2. [ ] Remove obvious/redundant items
3. [ ] Add project-specific gotchas
4. [ ] Verify build commands work
5. [ ] Commit to git

---

## Maintenance

### When to Update

- Adding new conventions
- After architectural changes
- When Claude repeatedly makes mistakes
- After onboarding feedback

### Review Periodically

Every few months:
1. Read through CLAUDE.md
2. Remove outdated rules
3. Add missing context
4. Verify commands still work

---

## Notes

<!-- Add observations about CLAUDE.md -->

