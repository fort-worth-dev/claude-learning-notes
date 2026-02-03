# Modular Rules Deep Dive

## What Are Modular Rules?

Rules stored in separate files under `.claude/rules/`:
- Organize rules by topic
- Apply rules to specific paths
- Easier to maintain than one large CLAUDE.md

---

## Directory Structure

```
.claude/
└── rules/
    ├── code-style.md
    ├── testing.md
    ├── security.md
    ├── api-design.md
    └── database.md
```

All `.md` files in `rules/` are loaded automatically.

### Loading Order

Files load alphabetically. Use prefixes for explicit ordering:

```
rules/
├── 01-architecture.md
├── 02-code-style.md
├── 03-testing.md
└── 04-security.md
```

---

## Basic Rule Files

### Simple Rule File

`rules/code-style.md`:
```markdown
# Code Style Rules

- Use modern C# features (?., ??, pattern matching)
- Prefer async/await over .Result, .Wait()
- Private fields: `_camelCase`
- Use var when type is obvious
```

### With Explanation

`rules/testing.md`:
```markdown
# Testing Guidelines

## Unit Tests
- One assertion per test (when practical)
- Name: MethodName_Scenario_ExpectedResult
- Use xUnit with FluentAssertions

## Integration Tests
- Tag with [Category("Integration")]
- Use test database, not production
- Clean up test data after each test
```

---

## Path-Specific Rules

### Targeting Specific Files

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Rules

These rules apply only to files in src/api/:

- All endpoints must validate input
- Use standard error response format
- Include OpenAPI documentation comments
```

### Multiple Path Patterns

```markdown
---
paths:
  - "src/controllers/**/*.cs"
  - "src/api/**/*.cs"
---

# Controller Rules

- No business logic in controllers
- Delegate to services
- Return appropriate HTTP status codes
```

---

## Glob Pattern Syntax

### Common Patterns

| Pattern | Matches |
|---------|---------|
| `*.cs` | .cs files in current directory |
| `**/*.cs` | .cs files in any subdirectory |
| `src/**/*` | All files under src/ |
| `**/*.{ts,tsx}` | .ts and .tsx files anywhere |
| `src/api/**/*.ts` | .ts files under src/api/ |
| `tests/**/*Test.cs` | Test files in tests/ |

### Pattern Examples

**All TypeScript:**
```yaml
paths:
  - "**/*.ts"
  - "**/*.tsx"
```

**Source only (not tests):**
```yaml
paths:
  - "src/**/*"
```

**Specific module:**
```yaml
paths:
  - "src/auth/**/*"
```

---

## Rule Organization Patterns

### By Domain

```
rules/
├── auth.md         # Authentication rules
├── orders.md       # Order processing rules
├── payments.md     # Payment handling rules
└── users.md        # User management rules
```

### By Concern

```
rules/
├── code-style.md   # Formatting, naming
├── testing.md      # Test conventions
├── security.md     # Security requirements
├── performance.md  # Performance guidelines
└── database.md     # Data access patterns
```

### By Layer

```
rules/
├── controllers.md  # API layer rules
├── services.md     # Business logic rules
├── repositories.md # Data access rules
└── models.md       # Domain model rules
```

---

## Example Rule Files

### Security Rules

`rules/security.md`:
```markdown
---
paths:
  - "src/**/*.cs"
---

# Security Rules

## Input Validation
- Validate all user input at API boundaries
- Use parameterized queries (never string concatenation)
- Sanitize output to prevent XSS

## Authentication
- Never log passwords or tokens
- Use secure password hashing (bcrypt, Argon2)
- Implement rate limiting on auth endpoints

## Secrets
- No hardcoded secrets in code
- Use environment variables or Key Vault
- Never commit .env files
```

### API Design Rules

`rules/api-design.md`:
```markdown
---
paths:
  - "src/Controllers/**/*.cs"
  - "src/Api/**/*.cs"
---

# API Design Rules

## Response Format
All responses use standard wrapper:
```json
{
  "data": T,
  "errors": [],
  "meta": {}
}
```

## Status Codes
- 200: Success with data
- 201: Created
- 204: Success, no content
- 400: Bad request (validation)
- 401: Unauthorized
- 404: Not found
- 500: Server error

## Versioning
- URL-based: /api/v1/resource
- Support previous version for 6 months
```

### Database Rules

`rules/database.md`:
```markdown
---
paths:
  - "src/Data/**/*.cs"
  - "src/Repositories/**/*.cs"
---

# Database Rules

## Entity Framework
- Use .AsNoTracking() for read-only queries
- Avoid N+1 queries (use Include())
- Use projection (Select) for large entities

## Migrations
- Don't use EF migrations
- Create DbUp scripts in /Database/Scripts
- Name: YYYYMMDD_Description.sql

## Soft Deletes
Tables with soft deletes:
- Users (IsDeleted)
- Orders (IsDeleted)
- Products (IsDeleted)

Always filter by IsDeleted = false.
```

### Testing Rules

`rules/testing.md`:
```markdown
---
paths:
  - "tests/**/*.cs"
---

# Testing Rules

## Naming
- MethodName_Scenario_ExpectedResult
- Example: GetUser_WithInvalidId_ReturnsNull

## Structure
- Arrange, Act, Assert sections
- One logical assertion per test
- Use FluentAssertions for readability

## Categories
- [Category("Unit")] - Fast, isolated
- [Category("Integration")] - Uses database
- [Category("E2E")] - Full stack

## Mocking
- Use NSubstitute for mocks
- Only mock external dependencies
- Don't mock the system under test
```

---

## Combining Rules

### CLAUDE.md + Rules

Use CLAUDE.md for:
- Build commands
- Quick reference
- Project overview

Use rules/ for:
- Detailed guidelines
- Path-specific rules
- Extensive documentation

### Example Structure

**CLAUDE.md:**
```markdown
# Project

Build: `dotnet build`
Test: `dotnet test`

See .claude/rules/ for detailed guidelines.
```

**rules/overview.md:**
```markdown
# Architecture Overview

- CQRS pattern
- Feature folders
- Domain-driven design

See other rule files for specifics.
```

---

## Conditional Rules

### Environment-Specific

```markdown
---
paths:
  - "src/**/*.cs"
---

# Environment Rules

## Development
- Verbose logging OK
- Can use test data

## Production
- Minimal logging (no PII)
- No test/debug code
```

### Team-Specific

Some rules for specific teams:
```markdown
---
paths:
  - "src/payments/**/*"
---

# Payment Team Rules

- All changes require security review
- Use PCI-compliant patterns
- Audit log all transactions
```

---

## Best Practices

### Keep Rules Focused

One topic per file. Don't combine unrelated rules.

### Use Path Restrictions

Apply rules only where relevant:
```yaml
paths:
  - "src/api/**/*"  # Only API code
```

### Document Why

Explain the reason for rules:
```markdown
## No .Result or .Wait()

These cause deadlocks in ASP.NET Core.
Always use async/await.
```

### Review Periodically

- Remove outdated rules
- Update for new patterns
- Verify rules are followed

---

## Notes

<!-- Add observations about modular rules -->

