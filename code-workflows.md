# Code Workflows

This is a quick reference. For detailed notes, see the [code-workflows/](code-workflows/) subfolder:

- [Plan-Implement-Test](code-workflows/plan-implement-test.md) - The core development cycle
- [Code Review](code-workflows/code-review.md) - Review patterns, multi-session workflows
- [Refactoring](code-workflows/refactoring.md) - Safe refactoring, TDD, large-scale migrations
- [Debugging](code-workflows/debugging.md) - Investigation, root cause analysis, test-first debugging

---

## Plan → Implement → Test Cycle

### The Four Phases

**Phase 1: Explore** (Plan Mode - read-only)
```bash
claude --permission-mode plan
```
```
Read src/auth/ and understand how we handle sessions.
Also look at how we manage environment variables.
```

**Phase 2: Plan** (Still Plan Mode)
```
I want to add Google OAuth. What files need to change?
What's the session flow? Create a plan.
```
Press `Ctrl+G` to edit plan in your editor.

**Phase 3: Implement** (Normal Mode)
```
Shift+Tab  # Switch to Normal Mode
```
```
Implement the OAuth flow from your plan.
Write tests for the callback handler.
Run the test suite and fix any failures.
```

**Phase 4: Commit**
```
Commit with a descriptive message and open a PR
```

### When to Skip Planning

**Skip for:**
- Small tasks (typo, log line, rename)
- Clear requirements (one-sentence diff)
- Familiar code with straightforward changes

**Use planning for:**
- Multi-file changes
- Unfamiliar systems
- Complex refactoring
- Breaking changes

### Verification-First

**Always include verification criteria:**

| Before | After |
|--------|-------|
| "implement validateEmail" | "implement validateEmail. tests: user@example.com→true, invalid→false. run tests after" |
| "make dashboard look better" | "[screenshot] implement this. compare result to original, fix differences" |

---

## Code Review Assistance

### Direct Review

```
Review my recent code changes for security issues

Review @src/auth/oauth.ts for edge cases, race conditions,
and consistency with our existing patterns
```

### PR Review Workflow

```
/pr-summary  # Summarize PR changes
```

### Create a Security Reviewer Subagent

```markdown
.claude/agents/security-reviewer.md
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
Review code for:
- Injection vulnerabilities (SQL, XSS, command)
- Authentication flaws
- Secrets in code
- Insecure data handling

Provide line references and fixes.
```

### Multi-Session Writer/Reviewer Pattern

| Session A (Writer) | Session B (Reviewer) |
|---|---|
| Implement rate limiter | |
| | Review @src/middleware/rateLimiter.ts for edge cases |
| Address feedback: [paste review] | |

---

## Refactoring Strategies

### Safe Refactoring Pattern

```
1. Find deprecated API usage in our codebase
2. Suggest how to refactor utils.js to modern JavaScript
3. Refactor while maintaining same behavior and passing tests
4. Run tests for the refactored code
```

### Large-Scale Refactors (Fan-Out Pattern)

**1. Generate task list:**
```
List all 200 Python files that need migrating from Django 2.0 to 4.0
```

**2. Script the migration:**
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from Django 2.0 to 4.0" \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**3. Test on few files first, then scale**

### Test-Driven Refactoring

```
1. Before refactoring utils.js, write comprehensive tests
   covering all existing behavior including edge cases
2. Run tests - establish baseline (should pass)
3. Refactor utils.js to use modern patterns
4. Run tests again - verify behavior preserved
5. Commit only when tests pass
```

### Backwards Compatibility

```
Refactor this auth module to use async/await while
maintaining backwards compatibility with existing code
that calls the old callback-based API.

Create an adapter layer that wraps the new async API
to support the old callback pattern.
```

---

## Debugging with Claude

### Bug Investigation Workflow

```
I'm seeing "TypeError: Cannot read property 'email' of undefined"
when I run npm test in the auth module.

Steps to reproduce:
1. Run npm test
2. Look for failing test "should handle missing user"
The error occurs when middleware doesn't receive user object

Suggest ways to fix. Hint: src/middleware/auth.js around line 45
```

### Provide Full Context

```
Here's the full error output:
[paste entire stack trace]

This happens when I run: npm build
It worked in version 2.1.0 but breaks in 2.2.0

What changed between those versions?
```

### Test-First Debugging

```
The login fails intermittently.
1. Write a test that reproduces the issue consistently
2. Make the test fail first
3. Then fix the underlying code to make the test pass
```

### Root Cause Analysis

**Don't just fix symptoms:**
```
The API returns 500 errors at 3 PM every day.
Check logs for that timeframe and trace back to
what's happening. Look for memory leaks, database
connection issues, or race conditions.
```

**Use subagents for investigation:**
```
Use a subagent to investigate the complete auth flow
and identify where the null reference is being introduced
```

---

## Common Workflow Patterns

### Feature Development

```
1. Explore (Plan Mode):
   Show me how features are currently structured
   What's the pattern for adding a new API endpoint?

2. Plan:
   Add a "favorites" feature. Walk me through:
   - Database changes needed
   - API endpoints to create
   - UI components to update
   Create a complete implementation plan.

3. Implement (Normal Mode):
   Shift+Tab
   Implement favorites feature according to the plan.
   Write tests for all new endpoints.
   Verify tests pass.

4. Commit:
   Commit with descriptive message and open a PR
```

### Bug Fix

```
Error when users click logout: [error]

1. Find the logout handler
2. Write a failing test that reproduces the issue
3. Fix the code to make the test pass
4. Run full test suite for regressions
5. Commit with message about why this fix was needed
```

### Migration

```
1. Plan (read-only):
   We're upgrading from React 17 to 18.
   What are the main changes needed?
   Which files most affected?
   Safest migration strategy?

2. Write tests first:
   Write tests covering all major UI interactions
   Don't change code yet—establish coverage.

3. Migrate incrementally:
   Migrate src/components/ to React 18 patterns.
   Run tests after each major component.

4. Verify end-to-end:
   Run full test suite, build, verify in browser.
```

### Codebase Onboarding

```
Give me an overview of this codebase
What are the main components?
How is authentication handled?
What are the main data models?

Trace the request flow for user login
from frontend through backend

Write a GETTING_STARTED.md for new developers
```

---

## .NET-Specific Workflows

### Feature Development

```
Add a new API endpoint for user profiles.

1. Create controller in src/Controllers/
2. Add model for UserProfile
3. Implement DI for service layer
4. Write tests in tests/Controllers.Tests/
5. Run dotnet test to verify
```

### Async Refactoring

```
Find all uses of Task.Result and Task.Wait()
and refactor to async/await.

1. Keep public API signatures the same
2. Use ConfigureAwait(false) for library code
3. Run full test suite after each component
4. Verify with CI/CD pipeline
```

### Blazor Component

```
Create a Blazor component for user authentication.

Requirements:
- Use EventCallback for login/logout (no Action)
- Accept cascading parameters if needed
- Use DI for auth service
- Include xUnit tests
- Pass all tests before finishing
```

---

## Checkpoints and Recovery

### Rewind Changes

Every file edit is snapshotted. To rewind:
```
Esc Esc  # Open checkpoint menu
```
Select a previous checkpoint to restore.

### Rewind Conversation Only

```
/rewind
# Select "Restore conversation only" to keep code changes
```

---

## Quick Reference

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Shift+Tab` | Cycle modes (Normal → Auto-Accept → Plan) |
| `Ctrl+G` | Edit plan in external editor |
| `Esc Esc` | Open checkpoint menu |
| `Esc` | Interrupt Claude |

### Commands

```
/clear           # Clear context between tasks
/rewind          # Restore to checkpoint
/context         # See context usage
/init            # Generate CLAUDE.md
```

### When to Use Each Workflow

| Situation | Approach |
|-----------|----------|
| Learning new codebase | Explore Mode + questions |
| Complex feature | Explore → Plan → Implement |
| Fixing bug | Plan analysis → failing test → fix |
| Refactoring | Plan → TDD (tests first) |
| Code review | Subagent or direct review |
| Large migrations | Fan-out pattern with scripts |
| Quick task | Skip planning, direct Normal Mode |

---

## Notes

<!-- Personal observations as you use Claude Code -->
