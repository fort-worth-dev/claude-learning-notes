# Prompting Techniques

This is a quick reference. For detailed notes, see the [prompting-techniques/](prompting-techniques/) subfolder:

- [Prompt Structure](prompting-techniques/prompt-structure.md) - Goal, scope, constraints, verification
- [Iteration Patterns](prompting-techniques/iteration-patterns.md) - Correcting, interrupting, restarting
- [Anti-Patterns](prompting-techniques/anti-patterns.md) - Common mistakes and how to avoid them
- [Verification-First](prompting-techniques/verification-first.md) - Testing and self-checking approaches

---

## Writing Effective Prompts

### Prompt Structure

1. **State the goal** - What you want to accomplish
2. **Provide scope** - Which files, modules, or areas
3. **Include constraints** - What NOT to do, patterns to follow
4. **Add verification** - How to know when it's done

### Vague vs. Specific

**Weak:**
```
fix the login bug
```

**Strong:**
```
Users report login fails after session timeout.
Check the auth flow in src/auth/, especially token refresh.
Write a failing test first, then fix it.
Verify the test passes and existing tests still work.
```

### Context Strategies

| Strategy | Example |
|----------|---------|
| **Scope the task** | "write tests for foo.py covering the edge case where user is logged out" |
| **Reference sources** | "look through ExecutionFactory's git history and summarize its API evolution" |
| **Show patterns** | "follow the pattern in HotDogWidget.php to implement a calendar widget" |
| **Describe symptoms** | "users report blank screen after entering wrong credentials" |

### Referencing Files

Use `@` to include files directly:
```
Explain the logic in @src/utils/auth.js
```

---

## When to Be Specific vs. Letting Claude Explore

### Be Detailed When:

- Multi-file changes
- Complex architectural decisions
- Ambiguous requirements
- Unfamiliar code being modified

**Example - Detailed:**
```
Refactor authentication to use OAuth2.
Look at sessions in src/auth/sessions.ts
Update token refresh in src/auth/token.ts
Add endpoint in src/routes/oauth.ts
Write tests in test/oauth.test.ts
Do NOT change the user model schema
```

### Be Brief When:

- Single-file changes
- Obvious fixes (typos, renaming)
- Small additions
- Task described in one sentence

**Example - Brief:**
```
The checkout flow is broken for users with expired cards.
The relevant code is in src/payments/. Investigate and fix it.
```

### Constrain vs. Trust

**Constrain when:**
- Strong style/approach opinions
- Certain patterns must be avoided
- Team conventions apply
- Security requirements exist

**Trust Claude when:**
- Problem is clear, solution flexible
- Discovering implementation options
- Learning the codebase
- Multiple valid approaches exist

---

## Iterating on Results

### Correcting Mid-Task

Press `Esc` to interrupt, then steer:
```
That's not right. The issue is in session handling, not token generation.
Look at how we refresh tokens in src/auth/refresh.ts
```

Context is preserved - you're redirecting, not restarting.

### When to Interrupt

**Interrupt when:**
- Claude is clearly wrong
- Approach contradicts your requirements
- You notice a better approach
- You can answer a clarifying question immediately

**Let Claude continue when:**
- Approach seems reasonable
- In verification phase (running tests)
- Task is exploratory

### When to Clear and Restart

Run `/clear` when:
- Switching to unrelated task
- Corrected Claude more than twice on same issue
- Context cluttered with failed attempts

**Key insight:** After two failed corrections, a fresh session with a better prompt usually works better than continuing to correct.

---

## Common Prompt Anti-Patterns

### Anti-Pattern 1: Kitchen Sink Session

Mixing unrelated tasks fills context with irrelevant exploration.

**Fix:** Use `/clear` between unrelated tasks.

### Anti-Pattern 2: Correcting Over and Over

After two corrections, context is polluted with failed approaches.

**Fix:** `/clear` and rewrite with better specificity:
```
/clear
Implement payment flow. Key challenge is expired cards.
Look at src/payments/cardHandler.ts for existing pattern.
Write a test first that reproduces the issue.
```

### Anti-Pattern 3: Over-Specified CLAUDE.md

50+ rules = Claude ignores the whole file.

**Fix:** Keep only rules that prevent mistakes Claude would otherwise make.

### Anti-Pattern 4: Vague Problems

```
The build is failing
```

**Fix:**
```
Build fails with: [paste error]
Run 'npm test' to reproduce
Fix root cause, don't suppress the error
```

### Anti-Pattern 5: Infinite Exploration

Unbounded investigation fills context before implementation.

**Fix:** Scope investigations or use subagents with separate context.

### Token Wasters

Skip:
- "I need your help with..."
- "Could you please..."
- Explaining what Claude already knows

---

## Verification-First Prompting

**Highest-leverage technique:** Include tests so Claude can check its own work.

### Without Verification:
```
Implement validateEmail function
```
Claude produces plausible code that may not handle edge cases.

### With Verification:
```
Implement validateEmail function.
Test cases: 'user@example.com' → true, 'invalid' → false, 'user@.com' → false
Run tests after implementing and fix any failures.
```

Claude can now:
- Run tests immediately
- See what broke
- Fix issues itself
- Verify before reporting done

### Self-Checking Pattern

```
Refactor @src/utils.js to use modern ES2024 features
Run tests after refactoring
List any behavior changes (should be none)
Fix any failing tests
```

---

## Prompt Checklist

### Clarity
- [ ] Goal is specific, not vague
- [ ] Could I describe this diff in one sentence?

### Context
- [ ] Files referenced with `@`
- [ ] Relevant patterns/examples shown
- [ ] Constraints listed

### Scope
- [ ] Which file(s) affected
- [ ] What's in/out of scope

### Verification
- [ ] How will Claude know it's done?
- [ ] Tests to write or run?
- [ ] Commands for verification?

---

## Quick Reference: Good vs. Bad

| Bad | Good |
|-----|------|
| "Add email notifications" | "Add email notifications on purchase. Follow pattern in @src/services/notifications/" |
| "Fix the payment bug" | "Payment form shows 'declined' but doesn't clear on retry. Check PaymentForm.tsx" |
| "Modernize this code" | "Refactor to async/await. Keep same API. Verify tests pass." |
| "Explore authentication" | "Use subagents to investigate token refresh. Report: storage, timing, failure handling." |

---

## Notes

<!-- Personal observations as you use Claude Code -->
