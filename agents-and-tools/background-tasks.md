# Background Tasks Deep Dive

## Why Background Tasks?

Some operations take time:
- Full test suites
- Large builds
- Dev server startup
- Long-running scripts

Running these in background lets you:
- Continue working while they run
- Avoid blocking the conversation
- Run multiple operations in parallel

---

## Starting Background Tasks

### Method 1: Ask Claude

```
Run the test suite in the background
```

```
Start the dev server in the background
```

Claude uses the Task tool with background flag.

### Method 2: Keyboard Shortcut

While a command is running:
```
Ctrl+B
```

Sends it to background immediately.

### Method 3: Shell Syntax

Using `!` bash mode:
```
! npm run build &
```

The `&` runs command in background.

---

## Checking Task Status

### List All Tasks

```
/tasks
```

Shows:
- Task ID
- Command/description
- Status (running, completed, failed)
- Duration

### Check Specific Task

```
Check the background test results
```

```
What's the status of the build?
```

Claude reads the task output and summarizes.

### Read Output File

Background tasks write to output files. You can:
```
Read the output from task abc123
```

---

## Task Lifecycle

### States

```
pending → running → completed
                 → failed
```

### Timeline

1. **Start** - Task begins, assigned ID
2. **Running** - Output accumulates
3. **Complete** - Exit code 0, output available
4. **Failed** - Non-zero exit, error output available

### Persistence

- Tasks persist across conversation turns
- Output files remain until cleaned up
- Can check results anytime during session

---

## Common Use Cases

### Running Tests

```
Run dotnet test in the background while I review the code
```

Then later:
```
Did the tests pass?
```

### Build While Working

```
Start a release build in the background
```

Continue coding, then:
```
Is the build done?
```

### Dev Server

```
Start the dev server in the background
```

Server runs while you work. Check status:
```
Is the dev server running?
```

### Parallel Operations

```
In the background, run:
1. dotnet test
2. npm run lint
3. docker build .
```

All three run simultaneously.

---

## Monitoring Output

### Real-Time Output

Background tasks accumulate output. Check periodically:
```
Show me the current test output
```

### Waiting for Completion

```
Let me know when the build finishes
```

Claude monitors and reports back.

### Partial Results

```
Show me test output so far, even if not complete
```

Useful for long-running suites.

---

## Stopping Tasks

### Stop a Running Task

```
Stop the background build
```

```
Cancel the test run
```

### Stop All Tasks

```
Stop all background tasks
```

### When to Stop

- Build is failing, no point continuing
- Wrong command started
- Need to free resources
- Want to restart with different options

---

## Background vs Foreground

### When to Use Background

| Scenario | Background? |
|----------|-------------|
| Full test suite (minutes) | Yes |
| Quick unit test | No |
| Dev server (long-running) | Yes |
| Single file compile | No |
| Docker build | Yes |
| Git status | No |

### Rule of Thumb

- **< 10 seconds:** Foreground
- **> 30 seconds:** Background
- **Indefinite (servers):** Background

---

## Parallel Task Patterns

### Independent Tasks

```
Run these in parallel in the background:
1. Frontend tests: npm test
2. Backend tests: dotnet test
3. Lint check: npm run lint
```

Three separate tasks, all running simultaneously.

### Dependent Tasks

Don't background these - they need to be sequential:
```
dotnet build && dotnet test
```

Build must complete before tests can run.

### Mixed Pattern

```
Start the build in foreground (need it first),
then run tests and linting in parallel background tasks
```

---

## Task Output Handling

### Successful Task

```
Tests passed (143 tests, 0 failures)
```

Claude summarizes key results.

### Failed Task

```
The build failed with 3 errors:
1. CS0246: Type 'UserDto' not found
2. ...
```

Claude highlights failures and errors.

### Verbose Output

For tasks with lots of output:
```
Summarize the test results, just show failures
```

```
Show me the full build output
```

---

## Troubleshooting

### Task Won't Start

- Check command syntax
- Verify permissions
- Try running in foreground first

### Task Seems Stuck

```
What's the current status of the build task?
```

Some tasks legitimately take time. Check if:
- Output is still being generated
- Process is actually running
- Resource constraints (CPU, memory)

### Can't Find Task Output

```
/tasks
```

Lists all tasks with their IDs and status.

### Task Failed Silently

Check the full output:
```
Show me the complete output from the failed task
```

Often the error is at the end.

---

## Integration with Workflow

### Explore → Build → Test

```
# 1. Explore (foreground, need the info)
Use Explore to understand the auth module

# 2. Implement (foreground, interactive)
Add the new validation method

# 3. Build and test (background, takes time)
Run build and tests in the background

# 4. Continue working or wait
Review the changes while tests run
```

### Continuous Feedback

```
Start the test watcher in background: dotnet watch test
```

Tests rerun automatically as you make changes.

### Multi-Project Builds

```
Build all projects in parallel:
- API: dotnet build src/Api
- Web: npm run build
- Worker: dotnet build src/Worker
```

Faster than sequential builds.

---

## Best Practices

### Name Long-Running Tasks

```
Start the integration tests in background (call it "integration-tests")
```

Easier to reference later.

### Check Before Starting New

```
/tasks
```

See what's already running before starting more.

### Clean Up Completed

Old completed tasks clutter the list. Periodically:
```
Clear completed background tasks
```

### Don't Over-Parallelize

More tasks = more resource contention. Usually:
- 2-3 parallel tasks: Good
- 5+ parallel tasks: Diminishing returns

### Monitor Resource-Heavy Tasks

```
Check on the Docker build - is it still making progress?
```

Large builds can stall. Check periodically.

---

## Notes

<!-- Add observations about background tasks as you use them -->

