# Prompt Structure Deep Dive

## The Four Elements

Every effective prompt has:

1. **Goal** - What you want to accomplish
2. **Scope** - Which files, modules, or areas
3. **Constraints** - What NOT to do, patterns to follow
4. **Verification** - How to know when it's done

You don't always need all four, but consider each.

---

## Element 1: Goal

### Be Outcome-Focused

**Weak:**
```
Look at the authentication code
```

**Strong:**
```
Find why users are logged out after 5 minutes of inactivity
```

The strong version tells Claude what success looks like.

### Action Verbs That Work

| Verb | When to Use |
|------|-------------|
| Fix | Known bug with symptoms |
| Add | New functionality |
| Refactor | Same behavior, different structure |
| Extract | Pull code into new function/class |
| Replace | Swap implementation |
| Remove | Delete functionality |
| Investigate | Unknown cause, need research |

### Examples

```
Fix the null reference in OrderService.GetById
Add email validation to the registration form
Refactor UserService to use async/await throughout
Extract the date formatting logic into a DateHelper class
Replace the custom logger with Serilog
Remove the deprecated PaymentV1 endpoint
Investigate why tests fail intermittently on CI
```

---

## Element 2: Scope

### Why Scope Matters

Claude can read your entire codebase. Without scope:
- May change wrong files
- Could miss the right location
- Might over-engineer solution

### Scoping Techniques

**By directory:**
```
The issue is in src/services/, specifically UserService.cs
```

**By file:**
```
Update src/Models/Order.cs to add the Status property
```

**By function:**
```
Fix the CalculateTotal method in OrderService
```

**By pattern:**
```
Update all repository classes to use IAsyncRepository
```

### Using @ References

Include files directly in prompt:
```
Explain the flow in @src/auth/login.cs
Follow the pattern in @src/services/BaseService.cs
```

The `@` syntax tells Claude to read that file immediately.

### Explicit Exclusions

```
Refactor the data layer.
Do NOT touch the migration files.
Do NOT change the public API signatures.
```

---

## Element 3: Constraints

### When Constraints Help

- Team conventions differ from defaults
- Security requirements exist
- Backwards compatibility needed
- Specific patterns required

### Types of Constraints

**Style constraints:**
```
Use async/await, not .Result or .Wait()
Use var only when type is obvious
No regions in new code
```

**Architecture constraints:**
```
Keep business logic out of controllers
Use repository pattern for data access
No direct SQL - use EF Core only
```

**Safety constraints:**
```
Don't modify the database schema
Don't change public API signatures
Don't delete existing tests
```

**Pattern constraints:**
```
Follow the pattern in UserRepository for the new OrderRepository
Match the error handling style in existing services
```

### Constraint Examples

```
Add caching to the product lookup.
Constraints:
- Use IMemoryCache, not Redis
- Cache for 5 minutes max
- Don't cache null results
- Add cache invalidation when products update
```

```
Refactor to async/await.
Constraints:
- Keep synchronous wrappers for backwards compatibility
- Don't change method signatures on public interfaces
- Update tests to use async patterns
```

---

## Element 4: Verification

### Why Verification Matters

Without verification, Claude:
- May produce plausible but broken code
- Can't self-correct
- Doesn't know when it's done

With verification, Claude:
- Runs tests to check work
- Fixes issues before reporting done
- Provides confidence in the result

### Verification Types

**Run existing tests:**
```
After making changes, run dotnet test and ensure all pass
```

**Write new tests:**
```
Write unit tests for the new validation logic, then run them
```

**Specific test cases:**
```
Test with: empty string → false, valid email → true, missing @ → false
```

**Build check:**
```
Verify the solution builds without warnings
```

**Manual verification:**
```
Show me the diff before committing
```

### Verification Examples

```
Add input sanitization to the search endpoint.
Verification:
- Write tests for XSS attempts: <script>, onclick=, javascript:
- Run existing API tests
- Show me the sanitization function
```

```
Refactor OrderService to async.
Verification:
- All existing tests must pass
- No synchronous database calls should remain
- Run dotnet build with TreatWarningsAsErrors
```

---

## Putting It Together

### Template

```
[GOAL]: What to accomplish

[SCOPE]: Where to look/change
- Files: ...
- Do NOT touch: ...

[CONSTRAINTS]:
- Pattern to follow: ...
- Things to avoid: ...

[VERIFICATION]:
- Tests to run: ...
- How to confirm: ...
```

### Full Example: Bug Fix

```
Fix the session timeout bug where users are logged out too early.

Scope:
- Check src/Auth/SessionManager.cs
- Check src/Auth/TokenService.cs
- Do NOT modify the User model

Constraints:
- Keep token refresh mechanism, just fix the timing
- Use existing IDateTimeProvider, don't call DateTime.Now directly

Verification:
- Write a test that sessions last at least 30 minutes of inactivity
- Run all auth tests: dotnet test --filter Category=Auth
- Show me the key changes before finishing
```

### Full Example: New Feature

```
Add email notification when orders ship.

Scope:
- Add notification in src/Services/OrderService.cs
- Create email template in src/Templates/
- Follow pattern in src/Services/NotificationService.cs

Constraints:
- Use existing IEmailSender interface
- Queue emails, don't send synchronously
- Include order number and tracking link in email

Verification:
- Write unit test that verifies email is queued on status change
- Run: dotnet test --filter "OrderService"
- Show me the email template content
```

### Full Example: Refactor

```
Refactor PaymentService to use the Strategy pattern for payment methods.

Scope:
- src/Services/PaymentService.cs (main changes)
- Create src/Strategies/IPaymentStrategy.cs
- Create implementations for Credit, Debit, PayPal

Constraints:
- Don't change PaymentService public API
- Keep existing payment method enum for backwards compatibility
- Use DI to register strategies

Verification:
- All existing payment tests must pass unchanged
- Add tests for strategy selection logic
- Run full test suite before finishing
```

---

## Scaling Complexity

### Simple Task (Brief Prompt)

```
Fix the typo in ErrorMessages.cs - "Authenication" should be "Authentication"
```

Goal is clear, scope is obvious, no constraints needed, verification is trivial.

### Medium Task (Structured Prompt)

```
Add pagination to the /api/products endpoint.
Use offset/limit pattern matching other endpoints.
Add tests for page boundaries.
Run API tests when done.
```

All four elements, concise.

### Complex Task (Detailed Prompt)

Full template with explicit sections. Worth the extra words for:
- Multi-file changes
- Unfamiliar code
- Critical functionality
- Team handoff

---

## Prompt Length Guidelines

### Too Short

```
Fix authentication
```

Missing: what's broken, where to look, how to verify.

### Too Long

```
I need you to help me fix an authentication issue. We've been having
problems where users are sometimes logged out unexpectedly. This has
been happening for about a week now, and our users are complaining...
[500 more words of backstory]
```

Background noise drowns out the actual task.

### Just Right

```
Users report unexpected logouts after ~5 minutes.
Check token refresh in src/Auth/TokenService.cs.
Write a test that verifies 30-minute session persistence.
```

Clear goal, specific scope, verification included.

---

## Notes

<!-- Add observations about prompt structure as you practice -->

