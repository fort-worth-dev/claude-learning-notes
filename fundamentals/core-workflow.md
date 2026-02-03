# Core Workflow Deep Dive

## The Four Phases

```
Explore → Plan → Implement → Verify
```

This is the recommended workflow for non-trivial tasks. Simple fixes can skip straight to implement.

---

## Phase 1: Explore

### Purpose

Understand the codebase before changing it. Prevents:
- Breaking existing functionality
- Duplicating existing code
- Fighting against established patterns

### How to Explore

**Start in plan mode:**
```bash
claude --permission-mode plan
```

Read-only. Safe to poke around.

**Use the Explore agent:**
```
"Use the Explore agent to find where user authentication happens"
```

The agent searches thoroughly without cluttering your context.

**Ask targeted questions:**
```
"How does the order processing pipeline work?"
"Where are database connections configured?"
"What pattern does this codebase use for validation?"
```

### Exploration Questions to Ask

- Where is X implemented?
- How does Y flow through the system?
- What pattern is used for Z?
- Are there existing examples of similar functionality?
- What tests cover this area?

### When to Stop Exploring

You understand:
- Where to make changes
- What patterns to follow
- What might break
- How to verify your changes work

---

## Phase 2: Plan

### Purpose

Design the approach before writing code. Catches:
- Overlooked edge cases
- Better alternatives
- Scope creep
- Missing dependencies

### Entering Plan Mode

If you started in default mode:
```
Shift+Tab
```

Toggles to Plan mode. Claude will research and propose, not implement.

Or start in plan mode:
```bash
claude --permission-mode plan
```

### What a Good Plan Includes

1. **Goal statement** - What we're trying to achieve
2. **Files to modify** - Specific paths
3. **Approach** - How we'll make the changes
4. **Risks** - What could go wrong
5. **Verification** - How we'll know it works

### Example Planning Prompt

```
"Plan how to add email validation to the user registration flow.
Consider:
- Where validation should happen
- What validation rules to apply
- How to handle validation errors
- What tests to add"
```

### Using the Plan Agent

For complex planning:
```
"Use the Plan agent to design the implementation for adding rate limiting to the API"
```

The agent will:
- Research the codebase
- Identify affected components
- Consider trade-offs
- Produce a detailed plan

### Reviewing the Plan

Before implementing, verify:
- Does the plan match your expectations?
- Are there simpler alternatives?
- Is the scope appropriate?
- Can it be broken into smaller steps?

---

## Phase 3: Implement

### Switching to Implement Mode

```
Shift+Tab
```

Toggles from Plan to Act mode. Now Claude can write code.

### Implementation Best Practices

**Be specific:**
```
"Add email validation to UserService.Register()
following the pattern in AddressValidator"
```

**Provide verification:**
```
"Add the method and write a unit test for it"
```

**Work incrementally:**
```
"First, add the validation method. We'll integrate it next."
```

### Let Claude Work

Once you've given clear instructions:
- Let Claude complete the task
- Don't interrupt mid-edit
- Review the complete change

### Interrupting When Needed

If Claude is heading wrong direction:
- Press `Escape` to stop
- Provide correction
- Be specific about what's wrong

**Rule of thumb:** If you've corrected twice and Claude isn't getting it, `/clear` and restart with a clearer prompt.

---

## Phase 4: Verify

### Why Verification Matters

Code that compiles isn't necessarily correct. Verify:
- Functionality works as expected
- Existing tests still pass
- New functionality is tested
- No regressions introduced

### Verification Methods

**Run existing tests:**
```
"Run dotnet test to verify nothing is broken"
```

**Write new tests:**
```
"Add unit tests for the new validation logic"
```

**Manual verification:**
```
"Build and run the app so I can test the change"
```

**Code review:**
```
"Review the changes we made for potential issues"
```

### Automated Verification Pattern

Ask Claude to verify its own work:
```
"After making the change, run the tests and show me the results"
```

Or even better:
```
"Add email validation to the registration flow.
Write tests for it.
Run all tests and confirm they pass."
```

---

## Putting It Together

### Example: Adding a Feature

**1. Explore:**
```bash
claude --permission-mode plan
```
```
"I need to add a 'forgot password' feature.
Where is the current authentication flow implemented?
Are there any existing password reset mechanisms?"
```

**2. Plan:**
```
"Plan the implementation for forgot password:
- Send reset email with token
- Token expires after 1 hour
- User can set new password with valid token"
```

Review Claude's plan. Adjust if needed.

**3. Implement:**
```
Shift+Tab  # Switch to Act mode
```
```
"Implement the plan. Start with the token generation and storage."
```

Work through the plan incrementally.

**4. Verify:**
```
"Run the tests to verify nothing is broken"
"Add tests for the new password reset flow"
"Build and run so I can test manually"
```

---

## Workflow Variations

### Quick Fix

Skip explore/plan for obvious fixes:
```
"Fix the null reference in OrderService.GetById -
return null when order not found instead of throwing"
```

### Bug Investigation

Heavy on explore, light on plan:
```bash
claude --permission-mode plan
```
```
"The checkout process sometimes fails with error X.
Help me trace through the code to find the cause."
```

Once found:
```
Shift+Tab  # Switch to Act mode
"Fix the bug by..."
```

### Large Refactor

Full workflow, chunked:
```
"Let's refactor the data layer to use repository pattern.
We'll do this in phases. First, just plan the overall approach."

# Review plan, then:
"Good. Let's start with UserRepository only."

# Complete and verify, then:
/clear
"Now add OrderRepository following the same pattern."
```

---

## When to Use Each Mode

### Plan Mode (Shift+Tab or --permission-mode plan)

- Exploring unfamiliar code
- Designing complex changes
- Reviewing approaches before committing
- When you're not sure what you want yet

### Act Mode (Default)

- Implementing planned changes
- Quick fixes and small edits
- When you know exactly what you want
- Writing and running tests

### The Toggle

`Shift+Tab` toggles between modes. Use it:
- Explore → Plan → Ready to code? Toggle to Act
- Implementing → Realize you need more research? Toggle to Plan
- Uncertain about approach? Stay in Plan until confident

---

## Anti-Patterns to Avoid

### Jumping Straight to Code

**Don't:**
```
"Add caching to the API"
```

**Do:**
```
"I want to add caching to the API.
First, show me how data fetching currently works.
What caching patterns are already in use?"
```

### Vague Implementation Requests

**Don't:**
```
"Make it better"
```

**Do:**
```
"Refactor GetOrdersByCustomer to use async/await
and add pagination with a default page size of 20"
```

### Skipping Verification

**Don't:**
```
# Make change
"Looks good, thanks!"
```

**Do:**
```
# Make change
"Run the tests to confirm nothing broke"
```

### Fighting Claude

**Don't:**
```
"No, that's wrong"
"That's still not right"
"You're not understanding"
```

**Do:**
```
/clear
"Let me be more specific about what I need..."
```

---

## Workflow Commands Reference

| Action | How |
|--------|-----|
| Start read-only | `claude --permission-mode plan` |
| Toggle modes | `Shift+Tab` |
| Clear context | `/clear` |
| Compact context | `/compact [focus]` |
| Check context usage | `/context` |
| Undo last file change | `/undo` |
| Name session | `/rename "name"` |

---

## Notes

<!-- Add observations about the workflow as you practice -->

