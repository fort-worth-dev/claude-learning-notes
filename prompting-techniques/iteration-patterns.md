# Iteration Patterns Deep Dive

## The Feedback Loop

Working with Claude is iterative:

```
Prompt → Response → Evaluate → Adjust → Repeat
```

Knowing when and how to adjust is key to productivity.

---

## Interrupting Claude

### How to Interrupt

Press `Esc` to stop Claude mid-response.

### When to Interrupt

**Interrupt immediately when:**
- Claude is editing the wrong file
- Approach is completely wrong
- You realize you gave bad instructions
- You can provide crucial missing context

**Let Claude continue when:**
- Approach seems reasonable even if not perfect
- Claude is in verification phase (running tests)
- Minor issues you can correct after
- Task is exploratory (let it explore)

### How to Redirect After Interrupt

**Be specific about the error:**
```
Stop - that's the wrong file. The auth logic is in src/Auth/TokenService.cs, not src/Services/AuthService.cs
```

**Provide the missing context:**
```
Wait - I should have mentioned we use JWT tokens, not session cookies. The token refresh is handled by...
```

**Steer the approach:**
```
Hold on. Instead of adding a new method, modify the existing ValidateToken method to handle this case.
```

---

## Correcting Claude

### Light Corrections

Small adjustments, Claude is mostly right:

```
Good, but use ILogger instead of Console.WriteLine
```

```
Close - the method should be async
```

```
That's right, but also add null check for the user parameter
```

### Significant Corrections

Approach is off but salvageable:

```
The logic is correct but it's in the wrong layer.
Move the validation from the controller to the service.
Keep the same validation rules.
```

```
Good structure, but we need to handle the async case.
Wrap the entire method in Task.Run and await the result.
```

### The Two-Correction Rule

If you've corrected Claude twice on the same issue and it's still wrong:

**Stop correcting. Start fresh.**

```
/clear
```

Then write a better prompt that addresses the confusion upfront.

**Why this works:**
- Context is polluted with failed attempts
- Claude may be pattern-matching on wrong examples
- Fresh start with better prompt is faster

---

## When to Clear and Restart

### Clear For New Tasks

```
/clear
```

Use when:
- Switching to unrelated work
- Previous task is complete
- Context is cluttered

### Clear After Failed Attempts

```
/clear
Let me try again with more context.

[Better prompt with clearer constraints]
```

### Clear When Confused

Signs Claude is confused:
- Repeating same mistake
- Contradicting earlier statements
- Asking questions you already answered
- Referencing files that don't exist

### Don't Clear When

- Mid-task and making progress
- Just need a small correction
- Building on previous work
- Exploring and learning together

---

## Compacting vs Clearing

### Use /compact When

- Same task, running low on context
- Want to preserve key decisions
- Long session with good progress

```
/compact focus on the authentication changes
```

### Use /clear When

- Task complete, starting new work
- Context polluted with wrong approaches
- Need fresh perspective
- Switching topics entirely

---

## Iteration Strategies

### The Incremental Approach

Break large tasks into steps:

```
Step 1: Add the IOrderRepository interface
```
*Verify, then:*
```
Step 2: Implement OrderRepository with basic CRUD
```
*Verify, then:*
```
Step 3: Add the query methods
```

**Advantages:**
- Easier to correct at each step
- Less wasted work if something's wrong
- Builds shared understanding

### The Checkpoint Approach

```
Add the validation logic. Stop before running tests - I want to review first.
```

*Review the changes, then:*
```
Looks good. Now run the tests.
```

**Advantages:**
- Catch issues before they compound
- Maintain control over risky operations
- Learn Claude's patterns

### The Trust-Then-Verify Approach

```
Implement the entire feature.
Write tests.
Run all tests and show me results.
If any fail, fix them.
```

**Advantages:**
- Faster for well-defined tasks
- Claude can self-correct
- Less back-and-forth

**Use when:**
- Task is well-scoped
- Good test coverage exists
- Pattern is established

---

## Handling Wrong Approaches

### Redirect Immediately

If Claude starts down wrong path:

```
[Esc]
Stop - let's think about this differently.
Instead of adding a new service, extend the existing OrderService.
```

### Explain Why

Help Claude understand the constraint:

```
Don't create a new table. We can't modify the schema because
other teams depend on it. Use the existing Orders table with
a new Status column instead.
```

### Show the Pattern

```
That's not our pattern. Look at how UserRepository does it:
@src/Repositories/UserRepository.cs
Follow that same approach.
```

### Reset Expectations

```
Let's step back. The goal isn't to refactor everything -
just fix this one null reference. Minimal change.
```

---

## Session Continuation

### Resuming Work

When returning to a session:

```
claude -c
```

Then reorient:
```
Where did we leave off with the payment refactor?
```

Or if context is stale:
```
/clear
Continue the payment refactor. We completed:
- IPaymentStrategy interface
- CreditCardStrategy implementation
Next: DebitCardStrategy following same pattern
```

### The Handoff Pattern

End of session:
```
Summarize what we completed and what's left
```

Save that summary. Next session:
```
Continue payment refactor. Completed: [paste summary]
```

---

## Debugging Communication Issues

### Claude Keeps Making Same Mistake

**Problem:** You've corrected it, but it keeps doing the same thing.

**Solution:**
1. `/clear` to reset
2. State the constraint upfront, prominently
3. Explain why (Claude learns from reasoning)

```
IMPORTANT: Do not use DateTime.Now directly.
We use IDateTimeProvider for testability.
Inject it via constructor and use _dateTimeProvider.Now.
```

### Claude Doesn't Understand Requirements

**Problem:** Output is technically correct but misses the point.

**Solution:** Provide concrete examples.

```
The validation should work like this:
- "user@example.com" → valid
- "user@" → invalid (no domain)
- "@example.com" → invalid (no local part)
- "user@example" → invalid (no TLD)
```

### Claude Is Too Literal

**Problem:** Follows instructions exactly but misses obvious implications.

**Solution:** State the intent, not just the action.

**Too literal:**
```
Add logging to the CreateOrder method
```
*Claude adds a single log statement*

**Better:**
```
Add comprehensive logging to CreateOrder so we can debug
order creation issues. Log: input validation, database calls,
external service calls, success/failure outcomes.
```

### Claude Is Too Creative

**Problem:** Over-engineers or adds unrequested features.

**Solution:** Constrain explicitly.

```
Just fix the bug. Do not:
- Refactor surrounding code
- Add new features
- Change method signatures
- Update unrelated tests

Minimal change that fixes the issue.
```

---

## Notes

<!-- Add observations about iteration patterns as you practice -->

