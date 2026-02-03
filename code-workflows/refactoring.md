# Refactoring Deep Dive

## Safe Refactoring Principles

### The Golden Rule

**Tests must pass before AND after refactoring.**

```
1. Run tests → All pass (baseline)
2. Refactor code
3. Run tests → All pass (verification)
4. Commit
```

If tests fail after refactoring, the refactor broke something.

### Behavior Preservation

Refactoring changes structure, not behavior:
- Same inputs → Same outputs
- Same side effects
- Same error handling
- Same API contracts

---

## Test-Driven Refactoring

### The TDD Refactoring Pattern

```
1. Before refactoring, write comprehensive tests
2. Run tests - establish baseline (all pass)
3. Refactor the code
4. Run tests - verify behavior preserved
5. Commit only when tests pass
```

### Writing Tests First

```
Before refactoring utils.js:

1. Write tests covering ALL existing behavior:
   - Happy paths
   - Edge cases
   - Error conditions
   - Boundary values

2. Include tests for undocumented behavior
   (code might depend on it)

3. Run tests - they should all pass

4. Now you have a safety net for refactoring
```

### Example Prompt

```
I want to refactor OrderService.cs to use async/await.

First, write comprehensive tests covering:
- All public methods
- Edge cases (empty orders, null items)
- Error conditions (invalid IDs, network failures)
- Any behavior that callers might depend on

Run the tests to establish baseline.
```

---

## Small, Incremental Refactors

### Why Incremental?

- Easier to debug when something breaks
- Can commit working states
- Reduces cognitive load
- Reviewable changes

### The Pattern

```
Refactor in small steps:
1. Extract method A → Run tests
2. Rename variables → Run tests
3. Simplify conditionals → Run tests
4. Move to new class → Run tests

Commit after each successful step.
```

### Example: Extracting a Class

```
Refactor UserService.cs - it's too large.

Step 1: Identify responsibilities
- User CRUD (keep in UserService)
- Password management (extract)
- Email notifications (extract)

Step 2: Extract PasswordService
- Move password methods
- Update UserService to use it
- Run tests

Step 3: Extract EmailService
- Move email methods
- Update dependencies
- Run tests

Step 4: Final cleanup
- Remove dead code
- Update DI registration
- Run tests
```

---

## Large-Scale Refactoring

### The Fan-Out Pattern

For massive changes (100+ files):

**1. Generate task list:**
```
List all files that need updating for the Django 2→4 migration.
Save to migration-files.txt
```

**2. Create migration script:**
```bash
for file in $(cat migration-files.txt); do
  claude -p "Migrate $file from Django 2.0 to 4.0. Preserve all behavior." \
    --allowedTools "Edit,Bash(python -m py_compile *)"
done
```

**3. Test on small batch first:**
```
Migrate just src/auth/*.py first.
Run tests for that module.
Fix any issues before scaling up.
```

**4. Scale up gradually:**
```
Migrate src/api/*.py (10 files)
Run tests
Migrate src/services/*.py (15 files)
Run tests
...continue...
```

### Parallel Refactoring

For independent modules:
```
Run these refactors in parallel:
1. Migrate src/auth/ to new pattern
2. Migrate src/orders/ to new pattern
3. Migrate src/users/ to new pattern

They don't depend on each other.
```

---

## Backwards Compatibility

### When You Can't Break Callers

```
Refactor the auth module to use async/await
while maintaining backwards compatibility.

Requirements:
- New internal code uses async
- Old public API still works (sync wrappers)
- Callers don't need to change
- Deprecation warnings for old API
```

### Adapter Pattern

```
Create an adapter layer:

Old API (deprecated):
  public User GetUser(int id)
  {
    return GetUserAsync(id).GetAwaiter().GetResult();
  }

New API:
  public async Task<User> GetUserAsync(int id)
  {
    // New implementation
  }

Mark old methods [Obsolete] with migration guidance.
```

### Strangler Fig Pattern

Gradually replace old with new:
```
Phase 1: New code path exists alongside old
Phase 2: Route some traffic to new
Phase 3: Route all traffic to new
Phase 4: Remove old code
```

---

## Common Refactoring Scenarios

### Async Conversion

```
Convert OrderService to async/await:

1. Find all sync methods that do I/O
2. Add async versions (keep sync for now)
3. Update callers to use async versions
4. Run tests after each method conversion
5. Remove old sync methods when all callers updated
6. Run full test suite
```

**.NET Specific:**
```
When converting to async:
- Use ConfigureAwait(false) in library code
- Don't mix .Result with async (deadlocks)
- Update interface to include async methods
- Consider CancellationToken parameters
```

### Extract Method/Class

```
UserService.cs is 800 lines. Refactor:

1. Identify logical groupings of methods
2. Create new service classes for each group
3. Move methods one group at a time
4. Update DI registration
5. Run tests after each extraction
```

### Rename Refactoring

```
Rename "Manager" classes to "Service" across codebase:

1. List all *Manager.cs files
2. For each file:
   - Rename class
   - Update all references
   - Update DI registration
   - Run tests
3. Update documentation
```

### Remove Dead Code

```
Find and remove unused code:

1. Search for methods with no callers
2. Check for [Obsolete] code past removal date
3. Find commented-out code blocks
4. Remove each piece, run tests
5. Don't remove public API without deprecation period
```

---

## Refactoring Smells

### Signs You Need to Refactor

| Smell | Refactoring |
|-------|-------------|
| Long method (50+ lines) | Extract methods |
| Large class (500+ lines) | Extract classes |
| Duplicate code | Extract common method |
| Deep nesting | Guard clauses, extract method |
| Long parameter list | Parameter object |
| Feature envy | Move method to right class |
| Primitive obsession | Create value objects |

### Identifying Smells

```
Analyze src/Services/ for code smells:
- Methods over 30 lines
- Classes over 300 lines
- Duplicate logic
- Deep nesting (>3 levels)

Suggest refactorings for each smell found.
```

---

## Refactoring with Confidence

### Safety Checklist

Before refactoring:
- [ ] Tests exist and pass
- [ ] Understand what code does
- [ ] Know who calls this code
- [ ] Have a rollback plan

During refactoring:
- [ ] Small, incremental changes
- [ ] Run tests frequently
- [ ] Commit working states
- [ ] Don't change behavior

After refactoring:
- [ ] All tests pass
- [ ] Manual verification if needed
- [ ] Code review
- [ ] Monitor after deployment

### When Tests Don't Exist

```
The code I need to refactor has no tests.

1. First, write characterization tests:
   - Test what the code DOES (not what it should do)
   - Cover all code paths
   - Include edge cases you discover

2. Run tests - establish baseline

3. Now safe to refactor

4. Tests verify behavior preserved
```

---

## Refactoring Prompts

### General Refactoring

```
Refactor [file] to improve [aspect].
Maintain the same external behavior.
Run tests after changes.
```

### Specific Patterns

**Extract:**
```
Extract the email validation logic from UserService
into a dedicated EmailValidator class.
Run tests after.
```

**Rename:**
```
Rename processData to calculateOrderTotal
across the entire codebase.
Update all references.
```

**Simplify:**
```
Simplify the nested conditionals in ProcessOrder method.
Use guard clauses and early returns.
Same behavior, cleaner code.
```

**Modernize:**
```
Update this code to use modern C# features:
- Null-conditional operators
- Pattern matching
- Expression-bodied members
Run tests to verify no behavior change.
```

---

## Notes

<!-- Add observations about refactoring patterns -->

