# Hooks Deep Dive

## What Are Hooks?

Shell commands that execute at specific lifecycle points:
- **Unlike CLAUDE.md** (advisory), hooks **always run**
- Can modify behavior, block actions, or trigger automation
- Configured in settings.json

---

## Hook Events

### Available Events

| Event | When | Can Block? | Use Case |
|-------|------|------------|----------|
| `SessionStart` | Session begins | No | Load context, set up environment |
| `PreToolUse` | Before any tool | Yes | Block dangerous commands |
| `PostToolUse` | After tool succeeds | No | Auto-format, notifications |
| `Notification` | Status updates | No | Desktop alerts |
| `Stop` | Claude finishes | No | Force continuation, cleanup |

### Event Details

**SessionStart:**
- Runs once when session starts
- Good for loading additional context
- Cannot block session

**PreToolUse:**
- Runs before each tool execution
- Can block the action (exit code 2)
- Receives tool info via stdin

**PostToolUse:**
- Runs after tool completes successfully
- Cannot block (already done)
- Good for formatting, logging

**Notification:**
- Runs on status changes
- Good for desktop alerts
- Claude continues working

**Stop:**
- Runs when Claude stops
- Can prompt for continuation
- Useful for workflows

---

## Configuration

### Basic Structure

In `.claude/settings.json`:

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "pattern",
        "hooks": [
          {
            "type": "command",
            "command": "your command"
          }
        ]
      }
    ]
  }
}
```

### Matchers

Filter which tools trigger the hook:

| Matcher | Matches |
|---------|---------|
| `""` | All tools |
| `"Edit"` | Edit tool only |
| `"Edit\|Write"` | Edit or Write |
| `"Bash"` | Bash tool only |

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success, action proceeds |
| 2 | Block the action |
| Other | Error (logged, continues) |

### Exit Code 2 (Blocking)

Only works with `PreToolUse`:
```bash
#!/bin/bash
# Block if on protected branch
BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [[ "$BRANCH" == "main" ]]; then
  echo "Cannot edit on main branch" >&2
  exit 2
fi
exit 0
```

---

## Common Hook Patterns

### Auto-Format After Edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $FILE"
          }
        ]
      }
    ]
  }
}
```

### Protect Branches

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/protect-branches.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/protect-branches.sh`:
```bash
#!/bin/bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
PROTECTED=("main" "master" "production")

for protected in "${PROTECTED[@]}"; do
  if [[ "$BRANCH" == "$protected" ]]; then
    echo "Cannot modify files on $BRANCH branch" >&2
    exit 2
  fi
done
exit 0
```

### Desktop Notifications

**Linux (notify-send):**
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Needs attention'"
          }
        ]
      }
    ]
  }
}
```

**macOS:**
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Needs attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Run Tests After Edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --findRelatedTests $FILE"
          }
        ]
      }
    ]
  }
}
```

### Lint Check Before Commit

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/pre-commit-check.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/pre-commit-check.sh`:
```bash
#!/bin/bash
# Read the command from stdin
read -r TOOL_INPUT

# Check if it's a commit command
if echo "$TOOL_INPUT" | grep -q "git commit"; then
  # Run lint check
  npm run lint
  if [ $? -ne 0 ]; then
    echo "Lint check failed. Fix errors before committing." >&2
    exit 2
  fi
fi
exit 0
```

---

## Hook Input

### Tool Information

Hooks receive JSON via stdin:

```json
{
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "old_string": "...",
    "new_string": "..."
  }
}
```

### Extracting Information

```bash
#!/bin/bash
# Read JSON from stdin
read -r INPUT

# Extract file path with jq
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Use the file path
echo "Processing: $FILE"
```

### Environment Variables

Some info available as env vars:
- `$FILE` - File being edited (when applicable)
- `$TOOL` - Tool name

---

## Hook Scripts

### Organizing Hooks

```
.claude/
└── hooks/
    ├── protect-branches.sh
    ├── auto-format.sh
    ├── pre-commit.sh
    └── notify.sh
```

### Make Executable

```bash
chmod +x .claude/hooks/*.sh
```

### Shebang Line

Always include:
```bash
#!/bin/bash
```

---

## Debugging Hooks

### Test Manually

```bash
echo '{"tool_name":"Edit","tool_input":{"file_path":"test.ts"}}' | .claude/hooks/my-hook.sh
echo $?  # Check exit code
```

### Add Logging

```bash
#!/bin/bash
echo "[$(date)] Hook started" >> /tmp/claude-hooks.log
# ... hook logic ...
echo "[$(date)] Hook finished with code $?" >> /tmp/claude-hooks.log
```

### Check Hook Status

```
/hooks
```

Interactive hook management.

---

## Advanced Patterns

### Conditional Hooks

```bash
#!/bin/bash
read -r INPUT
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Only format TypeScript files
if [[ "$FILE" == *.ts ]]; then
  npx prettier --write "$FILE"
fi
exit 0
```

### Chain Multiple Actions

```bash
#!/bin/bash
read -r INPUT
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Format
npx prettier --write "$FILE"

# Lint
npm run lint -- "$FILE"

# Type check
npx tsc --noEmit "$FILE"

exit 0
```

### Async Notifications

```bash
#!/bin/bash
# Don't block - run notification in background
(
  sleep 1
  notify-send "Claude Code" "Task completed"
) &
exit 0
```

---

## Best Practices

### Keep Hooks Fast

Slow hooks degrade experience:
- Avoid network calls in PreToolUse
- Use background processes for notifications
- Cache when possible

### Handle Errors Gracefully

```bash
#!/bin/bash
set -e  # Exit on error

# Trap errors
trap 'echo "Hook error: $?" >&2; exit 1' ERR

# Your logic here
```

### Don't Block Unnecessarily

Only use exit code 2 when truly needed:
- Security violations
- Protected branches
- Required checks

### Test Hooks Thoroughly

Before committing:
1. Test with various inputs
2. Verify exit codes
3. Check edge cases

---

## Notes

<!-- Add observations about hooks -->

