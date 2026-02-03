# Anti-Patterns Deep Dive

## What Are Anti-Patterns?

Common mistakes that waste time, fill context, or produce poor results. Recognizing them helps you avoid them.

---

## Anti-Pattern 1: Kitchen Sink Session

### The Problem

Mixing unrelated tasks in one session:

```
Session:
- "Fix login bug" → reads auth files
- "Update README" → reads docs
- "Add email feature" → reads notification files
- "Why is build slow?" → reads build config
- "Fix login bug again" → auth files already stale in context
```

Context fills with irrelevant file contents. Later tasks suffer.

### The Fix

**Clear between unrelated tasks:**
```
/clear
```

**Or use subagents for exploration:**
```
Use Explore to investigate the build speed issue
```

Keeps exploration isolated.

### Signs You're Doing This

- Session has been running for hours
- Claude references files from earlier, unrelated tasks
- `/context` shows high usage
- Claude seems "confused" about current task

---

## Anti-Pattern 2: Endless Correction Loop

### The Problem

```
You: Add validation to the form
Claude: [adds validation]
You: No, use Fluent Validation
Claude: [rewrites with Fluent]
You: Wrong - it should be on the DTO, not the model
Claude: [moves to DTO]
You: That's still not right, you need the validator class separate
Claude: [creates separate class but still wrong]
```

Each correction adds more context. Claude pattern-matches on all attempts, getting more confused.

### The Fix

**Two-correction rule:** After two failed corrections, `/clear` and rewrite.

```
/clear
Add form validation using FluentValidation.
- Create OrderValidator class in src/Validators/
- Validate the OrderDto, not the Order model
- Follow pattern in @src/Validators/UserValidator.cs
```

### Why This Works

- Fresh context with no failed attempts
- All constraints stated upfront
- Pattern reference guides the approach

---

## Anti-Pattern 3: Over-Specified CLAUDE.md

### The Problem

```markdown
# CLAUDE.md - 200 lines of rules

- Always use tabs
- Never use var
- Always add XML comments
- Use regions for organization
- Methods must be under 20 lines
- Classes must be under 200 lines
- Always use async/await
- Never use Task.Result
- ... 50 more rules
```

Claude ignores overwhelming instruction files. Rules compete for attention.

### The Fix

**Include only rules that prevent mistakes Claude would otherwise make.**

Good candidates:
- Build commands (Claude can't guess these)
- Non-obvious conventions specific to your team
- Critical constraints (security, architecture)

Remove:
- Standard language conventions
- Things obvious from code inspection
- Preferences that don't affect correctness

### Example: Focused CLAUDE.md

```markdown
# Build
- Build: `dotnet build src/Api`
- Test: `dotnet test --filter "Category!=Integration"`

# Conventions
- Private fields: `_camelCase` (not m_ prefix)
- Use DateTimeOffset, never DateTime
- Repositories return null, never throw for not-found

# Don't Touch
- Migration files in /Database/Migrations
- Shared contracts in /Contracts (breaking changes need approval)
```

---

## Anti-Pattern 4: Vague Problem Statements

### The Problem

```
The build is failing
```

```
Something's wrong with authentication
```

```
The tests are broken
```

Claude has to guess what you mean, often wrong.

### The Fix

**Include symptoms, errors, and reproduction steps.**

```
Build fails with:
error CS0246: The type or namespace name 'UserDto' could not be found

This started after I added the new Order model.
Run 'dotnet build src/Api' to reproduce.
```

```
Users report being logged out after exactly 5 minutes.
I expect sessions to last 30 minutes.
Check the token expiry in src/Auth/TokenService.cs
```

### Template

```
Problem: [what's happening]
Expected: [what should happen]
Error/Symptoms: [paste actual error or describe behavior]
Reproduce: [command or steps]
Likely location: [where to look]
```

---

## Anti-Pattern 5: Infinite Exploration

### The Problem

```
Tell me everything about this codebase
```

```
Explore the entire authentication system
```

Claude reads file after file, filling context before any useful work happens.

### The Fix

**Scope explorations:**
```
Explore just the token refresh flow in src/Auth/
```

**Use subagents:**
```
Use Explore to understand the auth system. Report:
- Where tokens are stored
- How refresh works
- What triggers logout
```

Subagent's exploration stays isolated.

**Set stopping conditions:**
```
Find where we validate user permissions.
Stop once you've found the main validation method - don't read everything.
```

---

## Anti-Pattern 6: Token Wasters

### The Problem

Prompts filled with unnecessary words:

```
Hi Claude! I hope you're doing well. I have a question for you.
I was wondering if you could possibly help me with something.
I need your assistance with implementing a feature. The feature
I'm talking about is email validation. Would you be so kind as
to help me add email validation to the registration form? I would
really appreciate it if you could do this for me. Thank you so
much in advance for your help!
```

This wastes tokens and buries the actual request.

### The Fix

**Direct and concise:**
```
Add email validation to the registration form in src/Forms/RegisterForm.cs
```

**Skip:**
- Greetings and pleasantries
- "Could you please..." / "I need your help with..."
- Explaining what Claude already knows
- Restating the obvious

### Before and After

**Before (47 words):**
```
Hey Claude, I'm working on this project and I've noticed that
we have a bug in our authentication system. The bug causes users
to be logged out unexpectedly. Could you please take a look at
the code and help me figure out what's going wrong?
```

**After (18 words):**
```
Users are logged out unexpectedly. Investigate token refresh
in src/Auth/TokenService.cs. Write a test that reproduces the issue.
```

---

## Anti-Pattern 7: Fire and Forget

### The Problem

```
Implement the entire payment system
```

Then walking away expecting perfection.

Large, unverified tasks often go wrong halfway through.

### The Fix

**Build in checkpoints:**
```
Implement the payment system. Steps:
1. Add IPaymentGateway interface - show me before continuing
2. Implement StripeGateway - run tests after
3. Add to DI container - verify it resolves
4. Wire up in OrderService - run integration test
```

**Or use verification:**
```
Implement payment system. After each major change,
run relevant tests. Stop if any fail.
```

---

## Anti-Pattern 8: Premature Optimization

### The Problem

```
Make this code as efficient as possible
```

```
Optimize for maximum performance
```

Claude adds caching, parallelization, memory pooling... for code that runs once per day.

### The Fix

**State the actual requirement:**
```
This endpoint needs to handle 1000 requests/second.
Currently it's at 100/second. Profile and fix the bottleneck.
```

**Or explicitly constrain:**
```
Add basic caching for repeated lookups. Keep it simple -
IMemoryCache with 5-minute expiry. Don't over-engineer.
```

---

## Anti-Pattern 9: Implicit Context

### The Problem

```
Fix it like we discussed
```

```
Use the same pattern as before
```

```
You know what I mean
```

Claude's memory of "before" may have been compacted or summarized.

### The Fix

**Be explicit every time:**
```
Fix the null reference in GetOrder, same as we fixed GetUser:
add null check before accessing properties.
```

**Reference files instead of memory:**
```
Follow the pattern in @src/Services/UserService.cs
```

---

## Anti-Pattern 10: Unclear Scope

### The Problem

```
Refactor the data layer
```

How much of it? Which parts? What's the target architecture?

### The Fix

**Explicit boundaries:**
```
Refactor OrderRepository only.
- Convert to async/await
- Add IOrderRepository interface
- Don't touch other repositories yet
- Don't change the Order model
```

**Clear success criteria:**
```
When done:
- OrderRepository is fully async
- All existing tests pass
- New interface is registered in DI
```

---

## Quick Reference

| Anti-Pattern | Fix |
|--------------|-----|
| Kitchen sink session | `/clear` between tasks |
| Endless correction | Two corrections max, then restart |
| Over-specified CLAUDE.md | Only non-obvious constraints |
| Vague problems | Include error, symptoms, steps |
| Infinite exploration | Scope or use subagents |
| Token wasters | Direct, concise prompts |
| Fire and forget | Add checkpoints and verification |
| Premature optimization | State actual requirements |
| Implicit context | Be explicit, reference files |
| Unclear scope | Explicit boundaries and success criteria |

---

## Notes

<!-- Add anti-patterns you discover as you work -->

