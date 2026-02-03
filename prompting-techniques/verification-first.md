# Verification-First Prompting Deep Dive

## The Core Idea

**Include tests and verification so Claude can check its own work.**

This is the highest-leverage prompting technique. It transforms Claude from "code generator" to "self-correcting developer."

---

## Why Verification Matters

### Without Verification

```
Implement validateEmail function
```

Claude produces:
- Plausible-looking code
- May miss edge cases
- No way to confirm correctness
- You discover bugs later

### With Verification

```
Implement validateEmail function.
Test cases:
- "user@example.com" → true
- "invalid" → false
- "user@.com" → false
- "" → false
Run tests after implementing.
```

Claude now:
- Has concrete success criteria
- Can run tests immediately
- Sees failures and fixes them
- Reports verified results

---

## Verification Types

### 1. Run Existing Tests

```
Refactor OrderService to async.
Run 'dotnet test --filter OrderService' after each change.
All tests must pass.
```

Uses existing test coverage to catch regressions.

### 2. Write New Tests

```
Add email validation to registration.
Write tests covering:
- Valid emails
- Missing @ symbol
- Empty string
- Null input
Run tests and fix any failures.
```

Creates new coverage as part of the task.

### 3. Specific Test Cases

```
Implement parseDate function.
Cases:
- "2024-01-15" → Date(2024, 1, 15)
- "01/15/2024" → Date(2024, 1, 15)
- "invalid" → null
- "" → null
Verify each case works.
```

Explicit input/output pairs Claude can verify.

### 4. Build Verification

```
Refactor the auth module.
After changes, run:
- dotnet build (must succeed)
- dotnet build --warnaserror (no warnings)
```

Catches compilation errors and warnings.

### 5. Runtime Verification

```
Add the new API endpoint.
After implementing:
- Start the server: dotnet run
- Call: curl http://localhost:5000/api/health
- Verify 200 response
```

Confirms runtime behavior.

---

## The Self-Checking Pattern

### Basic Form

```
[Task description]
[How to verify]
[What to do if verification fails]
```

### Examples

```
Add null check to GetUser.
Run 'dotnet test --filter GetUser'.
If any test fails, fix and rerun until all pass.
```

```
Refactor to use IOptions<T> for configuration.
Verify:
1. Build succeeds
2. All existing tests pass
3. App starts without errors
Fix any issues before finishing.
```

```
Add caching to ProductService.
Write a test that:
1. Calls GetProduct twice
2. Verifies database is only hit once
Run test. If it fails, fix the caching implementation.
```

---

## Verification Patterns by Task Type

### Bug Fixes

```
Fix: Users can't log in after password reset.

1. Write a test that reproduces the bug:
   - Create user
   - Reset password
   - Attempt login with new password
   - Assert: login succeeds

2. Run test - confirm it fails (proving bug exists)
3. Fix the bug
4. Run test - confirm it passes
5. Run all auth tests - confirm no regressions
```

**Why this works:**
- Test proves bug exists before fix
- Test proves fix works after
- Regression check catches side effects

### New Features

```
Add email notifications for order shipment.

Verification:
1. Write test: when order ships, email is queued
2. Write test: email contains order number and tracking
3. Run both tests - should fail initially
4. Implement feature
5. Run tests - all should pass
6. Run existing notification tests - no regressions
```

**Why this works:**
- Tests define expected behavior
- TDD approach ensures tests are meaningful
- Regression check maintains existing functionality

### Refactoring

```
Refactor PaymentService to Strategy pattern.

Verification:
1. Run all payment tests BEFORE refactoring
2. Note which pass (should be all)
3. Refactor incrementally
4. After each step, run payment tests
5. All tests must pass after refactor
6. No new tests needed - behavior unchanged
```

**Why this works:**
- Existing tests prove current behavior
- Same tests prove refactor is behavior-preserving
- Incremental verification catches issues early

### Performance Improvements

```
Optimize the product search query.

Verification:
1. Write benchmark test:
   - Search 10,000 products
   - Assert: completes in < 100ms
2. Run benchmark - note current time (probably fails)
3. Implement optimization
4. Run benchmark - should pass
5. Run functional tests - results still correct
```

**Why this works:**
- Benchmark quantifies the requirement
- Before/after comparison shows improvement
- Functional tests ensure correctness maintained

---

## Test-Driven Prompting

### The TDD Flow with Claude

```
Step 1: Write failing tests

Add tests for the new discount calculator:
- 10% off orders over $100
- 20% off orders over $500
- No discount under $100
Run tests - confirm they fail (no implementation yet)
```

```
Step 2: Implement to pass

Implement DiscountCalculator to pass all tests.
Run tests after implementation.
Fix any failures.
```

```
Step 3: Refactor if needed

Clean up the implementation if needed.
Run tests after any changes.
All tests must still pass.
```

### Why TDD with Claude Works

1. **Forces clarity** - Writing tests first clarifies requirements
2. **Provides verification** - Claude knows when it's done
3. **Prevents over-engineering** - Just enough code to pass tests
4. **Documents behavior** - Tests serve as specification

---

## Handling Test Failures

### Explicit Instructions

```
If tests fail:
1. Read the failure message
2. Identify the root cause
3. Fix the implementation (not the test)
4. Rerun tests
5. Repeat until all pass
```

### Setting Limits

```
Try to fix failing tests.
If still failing after 3 attempts, stop and show me:
- Which tests fail
- What you tried
- Your best guess at the issue
```

### Distinguishing Test Bugs from Code Bugs

```
If a test fails, determine whether:
- Implementation is wrong → fix implementation
- Test is wrong → tell me, don't fix test without asking
```

---

## Verification in Complex Tasks

### Multi-Step Verification

```
Add user authentication:

Step 1: Add User model
- Run: dotnet build
- Should compile

Step 2: Add UserRepository
- Run: dotnet test --filter Repository
- All repository tests pass

Step 3: Add AuthService
- Write tests for login/logout
- Run tests - should pass

Step 4: Add AuthController
- Write integration tests
- Run all tests
- All pass
```

### Checkpoint Verification

```
Add payment processing.

After each file you modify, run:
- dotnet build

After completing a logical unit, run:
- dotnet test --filter Payment

Before finishing, run:
- dotnet test (all tests)
```

---

## When Verification Isn't Possible

### No Existing Tests

```
This code has no tests.
After making changes:
1. Build and run the application
2. Manually verify [specific behavior]
3. Show me the diff so I can review
```

### UI Changes

```
Update the login form layout.
I'll verify visually - just show me what changed.
```

### External Dependencies

```
Update the Stripe integration.
We can't easily test this automatically.
Write tests with mocked Stripe responses.
Note: real integration needs manual testing.
```

---

## Verification Checklist

Before finishing a prompt, ask:

- [ ] How will Claude know the code works?
- [ ] Are there existing tests to run?
- [ ] Should new tests be written?
- [ ] What specific cases must pass?
- [ ] What should Claude do if verification fails?

---

## Quick Reference

| Task Type | Verification Approach |
|-----------|----------------------|
| Bug fix | Write test that repros, fix, verify passes |
| New feature | Write tests first, implement, verify |
| Refactor | Run existing tests before/after |
| Performance | Benchmark before/after |
| No tests | Build, run, show diff for review |

---

## Notes

<!-- Add observations about verification as you practice -->

