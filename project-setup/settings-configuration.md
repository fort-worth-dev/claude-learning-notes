# Settings Configuration Deep Dive

## Configuration Hierarchy

### Priority Order (highest first)

1. **Managed** - Organization-wide (IT-deployed)
2. **Local** - `.claude/settings.local.json`
3. **Project** - `.claude/settings.json`
4. **User** - `~/.claude/settings.json`

Higher priority wins when settings conflict.

---

## File Locations

### User Settings

`~/.claude/settings.json`

Applies to all your projects. Personal defaults.

```json
{
  "model": "sonnet",
  "permissions": {
    "deny": [
      "Bash(git push --force*)",
      "Bash(rm -rf *)",
      "Read(**/.env*)"
    ]
  }
}
```

### Project Settings

`.claude/settings.json`

Shared with team via git.

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Bash(git commit *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(appsettings.*.json)"
    ]
  }
}
```

### Local Settings

`.claude/settings.local.json`

Personal overrides for this project. Gitignored.

```json
{
  "model": "opus",
  "permissions": {
    "allow": [
      "Bash(dotnet test --filter Integration)"
    ]
  }
}
```

---

## Permissions

### Allow Rules

Pre-approve operations:

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Bash(npm run *)",
      "Read(src/**)",
      "Edit(src/**)",
      "Write(src/**)"
    ]
  }
}
```

Claude won't ask permission for these.

### Deny Rules

Block operations completely:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/*.pem)",
      "Read(**/secrets/**)",
      "Bash(rm -rf *)",
      "Bash(git push --force*)",
      "Bash(DROP TABLE*)"
    ]
  }
}
```

Even explicit requests are blocked.

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `Bash(npm test)` | Exact command |
| `Bash(npm *)` | Commands starting with "npm " |
| `Bash(*test*)` | Commands containing "test" |
| `Read(.env)` | Specific file |
| `Read(.env*)` | Files starting with ".env" |
| `Read(**/*.pem)` | All .pem files anywhere |
| `Write(src/**)` | Any file under src/ |

### Tool Types

- `Bash(pattern)` - Shell commands
- `Read(pattern)` - File reading
- `Write(pattern)` - File creation
- `Edit(pattern)` - File modification

---

## Permission Strategies

### Conservative Default (User Settings)

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)",
      "Bash(DROP *)",
      "Bash(DELETE FROM *)",
      "Read(**/.env*)",
      "Read(**/*.pem)",
      "Read(**/*secret*)",
      "Read(**/*credential*)"
    ]
  }
}
```

Safety net for all projects.

### Productive Project Settings

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet build *)",
      "Bash(dotnet test)",
      "Bash(dotnet test *)",
      "Bash(dotnet run *)",
      "Bash(dotnet watch *)",
      "Bash(dotnet ef *)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add *)",
      "Bash(git commit *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(appsettings.Development.json)",
      "Read(appsettings.Production.json)"
    ]
  }
}
```

Smooth workflow for .NET projects.

### CI/CD Settings

For automation:
```json
{
  "permissions": {
    "allow": [
      "Read(src/**)",
      "Grep(*)",
      "Glob(*)"
    ],
    "deny": [
      "Edit(*)",
      "Write(*)",
      "Bash(*)"
    ]
  }
}
```

Read-only analysis.

---

## Model Configuration

### In Settings

```json
{
  "model": "sonnet"
}
```

### Options

- `"sonnet"` - Default, balanced
- `"opus"` - Most capable
- `"haiku"` - Fast, lightweight

### Override Order

1. CLI flag: `claude --model opus`
2. Local settings
3. Project settings
4. User settings
5. Default (sonnet)

---

## Environment Variables

### In Settings

```json
{
  "env": {
    "DATABASE_URL": "${DB_CONNECTION_STRING}",
    "API_KEY": "${MY_API_KEY}"
  }
}
```

Variables with `${...}` expand from shell environment.

### Common Variables

```bash
# In ~/.bashrc or ~/.zshrc
export ANTHROPIC_MODEL=sonnet
export ANTHROPIC_API_KEY=sk-ant-...
export MAX_THINKING_TOKENS=10000
```

---

## Settings Merging

### How Merging Works

Settings merge across levels:
- Arrays concatenate
- Objects merge recursively
- Primitives override

### Example

**User (~/.claude/settings.json):**
```json
{
  "model": "sonnet",
  "permissions": {
    "deny": ["Read(**/.env*)"]
  }
}
```

**Project (.claude/settings.json):**
```json
{
  "permissions": {
    "allow": ["Bash(npm test)"],
    "deny": ["Read(secrets/*)"]
  }
}
```

**Effective:**
```json
{
  "model": "sonnet",
  "permissions": {
    "allow": ["Bash(npm test)"],
    "deny": ["Read(**/.env*)", "Read(secrets/*)"]
  }
}
```

---

## Hooks in Settings

### Basic Hook Configuration

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

See the [Hooks deep dive](hooks.md) for details.

---

## Validating Settings

### Check Syntax

```bash
cat .claude/settings.json | jq .
```

Validates JSON syntax.

### View Effective Settings

```
/config
```

Shows merged configuration.

### Diagnose Issues

```
/doctor
```

Checks for configuration problems.

---

## Common Issues

### Permission Denied Unexpectedly

Check all levels:
1. User settings deny rules
2. Project settings deny rules
3. Managed policy (if enterprise)

Deny rules from any level block the action.

### Allow Not Working

- Check pattern matches exactly
- Wildcards: `*` single segment, `**` recursive
- Test pattern manually

### Wrong Model

Check override order:
1. `echo $ANTHROPIC_MODEL`
2. Local settings
3. Project settings
4. User settings

### Settings Not Loading

- Verify JSON syntax
- Check file location
- Restart Claude session

---

## .gitignore Setup

```gitignore
# Claude Code local settings
.claude/settings.local.json
CLAUDE.local.md
```

Never commit local overrides.

---

## Notes

<!-- Add observations about settings -->

