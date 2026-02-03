# Configuration Deep Dive

## Configuration File Locations

### Hierarchy Overview

| Scope | Location | Shared via Git? |
|-------|----------|-----------------|
| User | `~/.claude/settings.json` | No |
| Project | `.claude/settings.json` | Yes |
| Project Local | `.claude/settings.local.json` | No (gitignored) |

### Priority Order

Local > Project > User

More specific settings override general ones.

### Finding Your Config

```bash
# User settings
cat ~/.claude/settings.json

# Project settings (if exists)
cat .claude/settings.json
```

Or in session:
```
/config
```

---

## Settings File Structure

### Basic Structure

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [],
    "deny": []
  }
}
```

### Full Example

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Bash(dotnet run)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/secrets/**)",
      "Bash(rm -rf*)",
      "Bash(git push --force*)"
    ]
  }
}
```

---

## Permission Rules

### How Permissions Work

Claude asks permission before:
- Reading certain files
- Writing files
- Running bash commands
- Using certain tools

You can pre-approve or block these with permission rules.

### Allow Rules

Pre-approve specific operations:

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Bash(npm install)",
      "Bash(npm run *)",
      "Read(src/**)",
      "Write(src/**)"
    ]
  }
}
```

### Deny Rules

Block operations entirely:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(**/*.pem)",
      "Read(**/credentials*)",
      "Bash(rm -rf *)",
      "Bash(git push --force*)",
      "Bash(DROP TABLE*)"
    ]
  }
}
```

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `Bash(dotnet build)` | Exact command |
| `Bash(dotnet *)` | Commands starting with "dotnet " |
| `Bash(*test*)` | Commands containing "test" |
| `Read(.env)` | Specific file |
| `Read(.env*)` | Files starting with ".env" |
| `Read(**/*.pem)` | All .pem files in any subdirectory |
| `Write(src/**)` | Any file under src/ |

### Tool Types for Permissions

- `Bash(pattern)` - Shell commands
- `Read(pattern)` - File reading
- `Write(pattern)` - File writing/creating
- `Edit(pattern)` - File editing

---

## User Settings (~/.claude/settings.json)

### Purpose

Personal defaults that apply to all projects.

### Common User Settings

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log*)"
    ],
    "deny": [
      "Bash(git push --force*)",
      "Bash(rm -rf /)",
      "Read(**/.env)",
      "Read(**/*.pem)"
    ]
  }
}
```

### What Belongs Here

- Your default model preference
- Global safety rules (never force push, never read secrets)
- Commands you always want auto-approved

---

## Project Settings (.claude/settings.json)

### Purpose

Team conventions, committed to git, shared with everyone.

### Example for .NET Project

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Bash(dotnet run --project *)",
      "Bash(dotnet ef migrations *)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(appsettings.*.json)",
      "Read(**/secrets/**)"
    ]
  }
}
```

### What Belongs Here

- Build/test commands for the project
- Project-specific blocked files
- Team-agreed safety rules

---

## Local Settings (.claude/settings.local.json)

### Purpose

Personal overrides for this project, not shared.

### Example

```json
{
  "model": "opus",
  "permissions": {
    "allow": [
      "Bash(dotnet test --filter Category=Integration)"
    ]
  }
}
```

### What Belongs Here

- Your personal model preference (if different from team)
- Additional commands you want pre-approved
- Overrides for your local workflow

### Gitignore It

Add to `.gitignore`:
```
.claude/settings.local.json
```

---

## Environment Variables

### Model Selection

```bash
# Default model for all sessions
export ANTHROPIC_MODEL=sonnet

# Specific model versions
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-5-20251101
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-20250514
```

### API Configuration

```bash
# API key (if not using claude login)
export ANTHROPIC_API_KEY=sk-ant-...

# Custom API endpoint (enterprise)
export ANTHROPIC_API_URL=https://api.anthropic.com
```

### Setting Permanently

Add to `~/.bashrc` or `~/.zshrc`:
```bash
export ANTHROPIC_MODEL=sonnet
```

---

## Keybindings

### Location

`~/.claude/keybindings.json`

### Default Bindings

| Action | Default |
|--------|---------|
| Submit | `Ctrl+J` |
| Newline | `Enter` |
| Cancel | `Escape` |
| Toggle Plan/Act | `Shift+Tab` |

### Customizing

```json
{
  "submit": "ctrl+enter",
  "newline": "enter",
  "toggle_mode": "ctrl+p"
}
```

### Getting Help

Use the keybindings-help skill:
```
/keybindings-help
```

---

## Permission Mode Presets

### Available Modes

| Mode | Behavior |
|------|----------|
| `default` | Ask before risky operations |
| `plan` | Read-only, no writes or executes |
| `full-auto` | Trust all operations |

### Setting via CLI

```bash
claude --permission-mode plan
```

### Setting in Config

```json
{
  "permissionMode": "default"
}
```

### When to Use Each

**default** (recommended for most work):
- Normal development
- When you want to review changes

**plan** (read-only exploration):
- Understanding unfamiliar code
- Before starting major changes
- Code review assistance

**full-auto** (use carefully):
- Trusted, well-tested workflows
- CI/CD automation
- When running in controlled environment

---

## Configuration Patterns

### Conservative Default

User settings with strict safety:
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

### Productive .NET Setup

Project settings for smooth workflow:
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
      "Bash(git branch*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(appsettings.Development.json)",
      "Read(appsettings.Production.json)"
    ]
  }
}
```

### Exploration Mode

For investigating unfamiliar repos:
```bash
claude --permission-mode plan
```

No configuration needed - just prevents all writes.

---

## Troubleshooting

### Check Current Config

```
/config
```

### Diagnose Issues

```
/doctor
```

Checks for configuration problems.

### Common Issues

**Permission denied unexpectedly:**
- Check deny rules in all config files
- User settings may block something project allows
- Deny rules take precedence

**Command still asks for permission:**
- Check pattern syntax - must match exactly
- Wildcards: use `*` for single segment, `**` for recursive

**Wrong model being used:**
- Check env vars: `echo $ANTHROPIC_MODEL`
- Check settings files at all levels
- CLI flag overrides everything: `claude --model opus`

---

## Notes

<!-- Add observations about configuration as you learn -->

