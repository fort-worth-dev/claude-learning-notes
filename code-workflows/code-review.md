# Code Review Deep Dive

## Review Approaches

### Direct Review

Ask Claude to review code directly:
```
Review src/Services/AuthService.cs for:
- Security issues
- Edge cases
- Error handling
- Code style
```

### Targeted Review

Focus on specific concerns:
```
Review the new OAuth code for race conditions
Check src/api/handlers.ts for SQL injection vulnerabilities
Review this PR diff for breaking API changes
```

### Comparative Review

Compare against patterns:
```
Review this code against our existing patterns in src/Services/
Does this follow the same conventions as UserService.cs?
```

---

## Review Prompts by Goal

### Security Review

```
Review src/auth/ for security vulnerabilities:
- Injection (SQL, XSS, command)
- Authentication flaws
- Authorization bypasses
- Secrets in code
- Insecure data handling

Provide line numbers and fixes.
```

### Performance Review

```
Review src/Services/OrderService.cs for performance issues:
- N+1 queries
- Missing async/await
- Unnecessary allocations
- Missing caching opportunities
- Inefficient LINQ

Suggest optimizations with examples.
```

### Correctness Review

```
Review the checkout logic for edge cases:
- Empty cart
- Negative quantities
- Concurrent modifications
- Null/missing data
- Boundary conditions

Write tests for any issues found.
```

### Style/Consistency Review

```
Review this code for consistency with our codebase:
- Naming conventions
- Error handling patterns
- Async patterns
- DI usage
- Test structure

Reference existing code as examples.
```

---

## PR Review Workflow

### Quick Summary

```
/pr-summary
```

Gets diff and summarizes key changes.

### Detailed Review

```
Review this PR for issues:

gh pr diff 123

Focus on:
1. Logic errors
2. Security issues
3. Missing tests
4. Breaking changes
```

### Review with Context

```
Review PR #123 in context of our auth system.
Check if changes are consistent with src/auth/ patterns.
```

---

## Multi-Session Review Pattern

### Why Two Sessions?

Separate writer and reviewer prevents bias:
- Writer session has implementation context
- Reviewer session sees code fresh

### Setup

**Terminal 1 - Writer:**
```bash
cd project
claude --name "feature-writer"
```

**Terminal 2 - Reviewer:**
```bash
cd project
claude --name "feature-reviewer"
```

### Workflow

| Writer Session | Reviewer Session |
|----------------|------------------|
| Implement rate limiter | |
| "Done with initial implementation" | |
| | Review src/middleware/rateLimiter.ts |
| | "Found 3 issues: [details]" |
| Address feedback: [paste issues] | |
| "Fixed, ready for re-review" | |
| | Re-review the changes |

### Handoff

Copy review findings from reviewer to writer:
```
Address this feedback:
1. Race condition in line 45
2. Missing error handling for timeout
3. Tests don't cover concurrent access
```

---

## Creating Review Subagents

### Security Reviewer Agent

Create `.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
model: opus
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Security Reviewer

Review code for OWASP Top 10 and common vulnerabilities:

1. **Injection** - SQL, XSS, command injection
2. **Broken Auth** - Weak passwords, session issues
3. **Sensitive Data** - Exposure, weak encryption
4. **XXE** - XML external entities
5. **Access Control** - Missing authorization
6. **Misconfig** - Default credentials, verbose errors
7. **XSS** - Reflected, stored, DOM-based
8. **Deserialization** - Unsafe object handling
9. **Components** - Known vulnerable dependencies
10. **Logging** - Missing audit trails

Provide:
- File path and line numbers
- Severity (Critical/High/Medium/Low)
- Explanation of the risk
- Recommended fix with code example
```

### Performance Reviewer Agent

Create `.claude/agents/perf-reviewer.md`:

```markdown
---
name: perf-reviewer
description: Reviews code for performance issues
model: sonnet
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Performance Reviewer

Analyze code for performance problems:

## Database
- N+1 query patterns
- Missing indexes (suggest based on queries)
- Large result sets without pagination
- Missing .AsNoTracking() for read-only

## Async
- Blocking calls (.Result, .Wait())
- Missing ConfigureAwait(false)
- Sync-over-async patterns

## Memory
- Large object allocations in loops
- Missing disposal of IDisposable
- String concatenation in loops
- Unbounded caches

## General
- Unnecessary computation
- Missing caching
- Inefficient algorithms

Provide line numbers and optimized code examples.
```

### Using Review Agents

```
Use security-reviewer to analyze src/Controllers/
Use perf-reviewer to check the order processing code
```

---

## Review Checklist Approach

### Define a Checklist

```
Review this code against our checklist:

[ ] No hardcoded secrets
[ ] All inputs validated
[ ] Errors logged properly
[ ] Async methods used correctly
[ ] Null checks present
[ ] Tests cover happy path and errors
[ ] Follows naming conventions
[ ] No commented-out code
```

### Automated Checklist Agent

Create a skill for standard reviews:

```markdown
---
name: review-checklist
description: Standard code review checklist
---

Review the specified files against this checklist:

## Security
- [ ] No secrets in code
- [ ] Inputs validated
- [ ] SQL parameterized
- [ ] XSS prevented

## Quality
- [ ] Error handling complete
- [ ] Logging appropriate
- [ ] No dead code
- [ ] Tests exist

## Style
- [ ] Naming conventions followed
- [ ] Async patterns correct
- [ ] DI used properly

Report: PASS, WARN, or FAIL for each item.
```

---

## Review Findings Format

### Structured Output

Ask for consistent format:
```
Review and report findings as:

## Critical
- [file:line] Description of critical issue
  Fix: suggested code

## High
- [file:line] Description
  Fix: suggested code

## Medium
...

## Suggestions
- Nice-to-have improvements
```

### Actionable Feedback

Good review finding:
```
[src/auth/login.cs:45] SQL Injection vulnerability
Current: var sql = $"SELECT * FROM Users WHERE email = '{email}'";
Fix: Use parameterized query:
  var sql = "SELECT * FROM Users WHERE email = @email";
  cmd.Parameters.AddWithValue("@email", email);
```

Bad review finding:
```
There might be a security issue in the login code.
```

---

## Iterative Review

### First Pass: Automated Checks

```
Run these checks and report issues:
1. dotnet build - any warnings?
2. dotnet test - all passing?
3. Grep for TODO/FIXME/HACK
4. Check for .Result or .Wait() calls
```

### Second Pass: Logic Review

```
Now review the actual logic in src/Services/OrderService.cs:
- Does it handle all edge cases?
- Is the business logic correct?
- Are there race conditions?
```

### Third Pass: Architecture Review

```
Step back and review the architecture:
- Is this the right place for this code?
- Does it follow our patterns?
- Will it be maintainable?
```

---

## Self-Review Before Commit

Before committing, ask Claude to review its own work:

```
Review all the changes we just made:
1. Security issues?
2. Missing error handling?
3. Tests adequate?
4. Any regressions?
```

---

## Notes

<!-- Add observations about code review patterns -->

