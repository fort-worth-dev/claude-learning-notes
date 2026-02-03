# Debugging Deep Dive

## Bug Investigation Workflow

### The Process

```
1. Reproduce → 2. Isolate → 3. Understand → 4. Fix → 5. Verify
```

### Step 1: Reproduce

Make the bug happen consistently:
```
I'm seeing "TypeError: Cannot read property 'email' of undefined"
when running npm test in the auth module.

Steps to reproduce:
1. Run npm test
2. Look for failing test "should handle missing user"
3. Error occurs in src/middleware/auth.js around line 45
```

### Step 2: Isolate

Narrow down where the bug is:
```
Use Explore to trace the code path from the test
to where the error occurs. What function calls lead there?
```

### Step 3: Understand

Figure out WHY it's happening:
```
In auth.js line 45, we call user.email but user is undefined.
Why would user be undefined at this point?
Check what sets the user object upstream.
```

### Step 4: Fix

Apply the fix:
```
Add a null check before accessing user.email.
Or fix the upstream code that should be setting user.
```

### Step 5: Verify

Confirm the fix works:
```
Run the failing test - it should pass now.
Run the full test suite - no regressions.
```

---

## Providing Context to Claude

### Good Bug Report

```
Error: "NullReferenceException: Object reference not set"

Where: src/Services/OrderService.cs, GetOrderById method
When: Calling API endpoint /api/orders/123
Frequency: Happens for orders created before Jan 2024

Stack trace:
[paste full stack trace]

What I've tried:
- Checked if order exists (it does)
- Logged the order object (shows null for Customer property)

Suspicion: Orders migrated from old system might be missing Customer data
```

### What to Include

| Element | Why It Helps |
|---------|--------------|
| Exact error message | Identifies the exception type |
| Stack trace | Shows call chain |
| Reproduction steps | Enables Claude to understand flow |
| When it happens | Patterns reveal causes |
| When it started | "Worked before X" narrows scope |
| What you've tried | Avoids duplicate effort |

### What to Avoid

```
BAD: "The login is broken"
GOOD: "Login fails with 401 when using email with + character"

BAD: "It crashes sometimes"
GOOD: "Crashes after ~100 requests, OutOfMemoryException in ProcessQueue"
```

---

## Test-First Debugging

### The Pattern

```
1. Write a test that reproduces the bug
2. Run the test - it should FAIL
3. Fix the code
4. Run the test - it should PASS
5. Commit with test included
```

### Why Write the Test First?

- Proves you understand the bug
- Prevents regression
- Defines "fixed"
- Documents the edge case

### Example

```
The login fails when email contains a + sign.

1. Write a failing test:
   [Fact]
   public async Task Login_WithPlusInEmail_ShouldSucceed()
   {
       var result = await _authService.Login("user+test@example.com", "password");
       Assert.True(result.Success);
   }

2. Run test - confirm it fails

3. Find and fix the bug (probably URL encoding issue)

4. Run test - confirm it passes

5. Run full suite - no regressions
```

---

## Root Cause Analysis

### Don't Just Fix Symptoms

```
BAD: "Add null check to stop the crash"
GOOD: "Find out WHY it's null and fix that"
```

### Asking the Right Questions

```
The API returns 500 errors at 3 PM daily.

Questions to investigate:
1. What runs at 3 PM? (scheduled jobs, backups?)
2. What resources are constrained? (memory, connections, CPU?)
3. What changed recently? (deployments, config?)
4. What's in the logs at that time?
5. Does it correlate with traffic patterns?
```

### Tracing Back

```
The order total is wrong.

Trace the calculation backwards:
1. Where is total displayed? → OrderSummary.cs
2. Where is it calculated? → OrderService.CalculateTotal()
3. What inputs does it use? → line items, tax, discounts
4. Which input is wrong? → Discount is applied twice
5. Why applied twice? → Called from two places

Root cause: Discount logic in wrong layer
Fix: Move discount calculation to single location
```

---

## Using Subagents for Investigation

### Broad Investigation

```
Use Explore to investigate the complete auth flow
and identify where the null reference is being introduced
```

### Targeted Search

```
Use a subagent to find all places where UserContext is set
and check if any path could leave it null
```

### Parallel Investigation

```
Investigate in parallel:
1. Search for where the error is thrown
2. Find all callers of that method
3. Check recent changes to related files
```

---

## Debugging Patterns

### Binary Search Debugging

For "it worked before":
```
1. Find a commit where it works
2. Find a commit where it's broken
3. Binary search: test middle commit
4. Narrow down to the breaking commit
5. Examine what changed
```

Git bisect:
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.2.0
# Git checks out middle, you test, mark good/bad
```

### Divide and Conquer

For complex flows:
```
The checkout process fails somewhere.

Add logging at each step:
1. Cart validated? ✓
2. Inventory checked? ✓
3. Payment processed? ✗ ← Failed here
4. Order created?
5. Email sent?

Now focus on payment processing.
```

### Rubber Duck Debugging

Explain the code to Claude:
```
Let me walk through what this code does:
1. First it gets the user from the database
2. Then it checks permissions
3. Then... wait, it checks permissions BEFORE validating the token?
   That's the bug - it should validate first.
```

Sometimes explaining reveals the problem.

---

## Debugging Prompts

### Initial Investigation

```
I'm seeing this error: [error]
It happens when: [context]

Help me investigate:
1. What could cause this error?
2. What should I check first?
3. What logs or data would help?
```

### With Stack Trace

```
Here's the full error:
[paste stack trace]

Trace through the call stack and identify:
1. Where the error originates
2. What state is unexpected
3. What could cause that state
```

### Intermittent Bugs

```
This error happens randomly, maybe 1 in 100 requests.

Possible causes of intermittent failures:
- Race conditions
- Resource exhaustion
- External service flakiness
- Timing/timeout issues

Help me add logging to identify which.
```

### Performance Issues

```
The API is slow (5s response time).

Help me profile:
1. Add timing logs to each step
2. Identify the slow operation
3. Suggest optimizations
```

---

## Common Bug Categories

### Null Reference

```
NullReferenceException in GetUser

Check:
- Is the ID valid?
- Does the record exist?
- Is there a race condition?
- Is initialization order correct?
```

### Off-by-One

```
Array index out of bounds

Check:
- Loop boundaries (< vs <=)
- Zero-based vs one-based indexing
- Empty collection handling
```

### Race Condition

```
Works sometimes, fails sometimes

Check:
- Shared mutable state
- Async operations without await
- Missing locks
- Order of operations
```

### Resource Leak

```
Memory grows over time, eventually crashes

Check:
- IDisposable not disposed
- Event handlers not unsubscribed
- Cache without eviction
- Connection pools exhausted
```

### State Corruption

```
Data is wrong after certain operations

Check:
- Concurrent modifications
- Missing validation
- Incorrect initialization
- Cache staleness
```

---

## Debugging .NET Specific

### Async Issues

```
Deadlock or task never completes

Check:
- .Result or .Wait() on UI/ASP.NET thread
- Missing await
- ConfigureAwait issues
- SynchronizationContext problems
```

### EF Core Issues

```
Unexpected database behavior

Check:
- Tracking vs no-tracking queries
- Lazy loading surprises
- Missing Include()
- Transaction boundaries
- Connection lifetime
```

### DI Issues

```
Wrong instance or null service

Check:
- Registration lifetime (Singleton vs Scoped vs Transient)
- Circular dependencies
- Missing registration
- Scope validation
```

---

## After Fixing

### Verify Thoroughly

```
The fix is in. Now verify:
1. Original bug is fixed (failing test passes)
2. No regressions (full test suite passes)
3. Edge cases handled
4. Similar bugs elsewhere? (search for pattern)
```

### Document the Fix

Good commit message:
```
Fix NullReferenceException in OrderService.GetById

Root cause: Orders migrated from legacy system had null
Customer property because migration script didn't populate it.

Fix: Add null check with appropriate error message.
Also: Added validation to migration script for future runs.

Fixes #123
```

### Prevent Recurrence

```
How do we prevent this bug category?
- Add validation at system boundaries
- Improve test coverage for edge cases
- Add logging for better debugging
- Update documentation
```

---

## Notes

<!-- Add observations about debugging patterns -->

