# Plan-Implement-Test Cycle Deep Dive

## The Four Phases

```
Explore → Plan → Implement → Verify/Commit
```

This cycle maximizes quality and minimizes rework.

---

## Phase 1: Explore

### Purpose

Understand before changing. Prevents:
- Breaking existing functionality
- Duplicating existing code
- Fighting established patterns
- Missing edge cases

### How to Explore

**Start in Plan Mode:**
```bash
claude --permission-mode plan
```

Read-only. Safe to investigate without accidental changes.

**Ask targeted questions:**
```
Read src/auth/ and explain how sessions are managed
What pattern does this codebase use for validation?
How does error handling work in the API layer?
```

**Use Explore agent for broad searches:**
```
Use Explore to find all places where user permissions are checked
```

### What to Learn

Before implementing, understand:
- Where similar functionality exists
- What patterns the codebase uses
- What tests exist for related code
- What might break

### Exploration Checklist

- [ ] Found existing similar code to reference
- [ ] Understand the patterns in use
- [ ] Know what files will change
- [ ] Identified potential side effects
- [ ] Know how to verify the change works

---

## Phase 2: Plan

### Purpose

Design the approach before coding. Catches:
- Overlooked requirements
- Better alternatives
- Scope creep
- Missing dependencies

### Staying in Plan Mode

If you started with `--permission-mode plan`, you're already there.

If in Normal mode:
```
Shift+Tab  # Toggle to Plan Mode
```

### Creating a Plan

```
I want to add Google OAuth. Create a plan:
1. What files need to change?
2. What's the authentication flow?
3. What new dependencies are needed?
4. What tests should be written?
```

### Plan Components

A good plan includes:

| Component | Purpose |
|-----------|---------|
| Goal | What we're trying to achieve |
| Files | Specific paths to modify |
| Approach | How we'll make changes |
| Dependencies | New packages, services |
| Risks | What could go wrong |
| Verification | How we'll know it works |

### Editing the Plan

Press `Ctrl+G` to edit the plan in your external editor.

Useful for:
- Adding details Claude missed
- Removing unnecessary scope
- Reordering steps
- Adding your own notes

### When to Skip Planning

**Skip for:**
- Typo fixes
- Single-line changes
- Adding a log statement
- Clear, one-sentence requirements

**Use planning for:**
- Multi-file changes
- Unfamiliar code
- Complex refactoring
- Breaking changes
- Anything you're unsure about

---

## Phase 3: Implement

### Switching to Implement Mode

```
Shift+Tab  # Toggle from Plan to Normal Mode
```

Now Claude can write code.

### Starting Implementation

Reference the plan:
```
Implement the OAuth flow from the plan.
Start with the callback handler.
```

### Work Incrementally

Don't try to implement everything at once:
```
First: Add the OAuth configuration
Then: Implement the callback handler
Then: Add the session management
Finally: Write the tests
```

### Include Verification

Every implementation request should include how to verify:
```
Add the validateEmail function.
Write tests: user@example.com→true, invalid→false
Run the tests after.
```

### Verification-First Pattern

| Weak Request | Strong Request |
|--------------|----------------|
| "implement validateEmail" | "implement validateEmail. Tests: valid@test.com→true, missing-at→false. Run tests after." |
| "fix the login bug" | "fix the login bug. Write a failing test first, then fix, then verify test passes." |
| "add user profile page" | "add user profile page matching this design. Compare result to mockup when done." |

### Let Claude Work

Once you've given clear instructions:
- Let Claude complete the task
- Don't interrupt mid-edit
- Review the complete change

### Correcting Course

If Claude goes wrong:
- Press `Escape` to interrupt
- Provide specific correction
- If correcting twice doesn't work, `/clear` and restart

---

## Phase 4: Verify and Commit

### Verification Steps

**Run tests:**
```
Run dotnet test to verify nothing broke
```

**Check specific behavior:**
```
Build and run so I can test the login flow manually
```

**Review changes:**
```
Show me a summary of all files we changed
```

### Commit When Ready

Only when verification passes:
```
Commit with a descriptive message
```

Claude will:
1. Check git status
2. Review the changes
3. Write appropriate commit message
4. Create the commit

### Opening a PR

```
Create a pull request with summary of changes
```

---

## Complete Example

### Task: Add Email Validation

**1. Explore:**
```bash
claude --permission-mode plan
```
```
Show me how validation currently works in this codebase.
Where are other validators defined?
What testing patterns do they use?
```

**2. Plan:**
```
I need to add email validation to user registration.
Create a plan covering:
- Where the validator should live
- What validation rules to apply
- How to integrate with existing flow
- What tests to write
```

Review plan, edit with `Ctrl+G` if needed.

**3. Implement:**
```
Shift+Tab  # Switch to Normal Mode
```
```
Implement the email validator from the plan.
Write unit tests covering:
- Valid email formats
- Invalid emails (missing @, no domain, etc.)
- Edge cases (unicode, very long)
Run tests after implementation.
```

**4. Verify and Commit:**
```
Run the full test suite to check for regressions.
```

If passing:
```
Commit with message about adding email validation
```

---

## Mode Cycling

### The Three Modes

| Mode | Can Read | Can Write | Purpose |
|------|----------|-----------|---------|
| Plan | Yes | No | Explore and plan |
| Normal | Yes | Ask | Implement with review |
| Auto-Accept | Yes | Yes | Trusted implementation |

### Cycling Through

```
Shift+Tab
```

Cycles: Normal → Auto-Accept → Plan → Normal

### Recommended Flow

1. Start in **Plan Mode** for exploration
2. Switch to **Normal Mode** for implementation
3. Use **Auto-Accept** only for trusted, well-understood changes

---

## Session Management

### One Task Per Session

For complex work, dedicate a session:
```bash
claude --name "feature-oauth"
```

### Clear Between Tasks

When switching to unrelated work:
```
/clear
```

Prevents context confusion.

### Resume Later

```bash
claude -r "feature-oauth"
```

Pick up where you left off.

---

## Common Mistakes

### Skipping Exploration

**Problem:** Implement without understanding
**Result:** Breaks existing code, fights patterns

**Fix:** Always explore unfamiliar code first

### Vague Plans

**Problem:** "Add the feature"
**Result:** Claude makes assumptions you didn't want

**Fix:** Specify files, approach, verification

### No Verification Criteria

**Problem:** "Implement X"
**Result:** No way to know if it works

**Fix:** Always include how to verify

### Too Much at Once

**Problem:** "Implement the entire feature"
**Result:** Large changes, hard to debug

**Fix:** Work incrementally, verify each step

### Ignoring Test Failures

**Problem:** Tests fail, continue anyway
**Result:** Regressions, broken code

**Fix:** Stop, fix failures before proceeding

---

## Notes

<!-- Add observations about the plan-implement-test cycle -->

