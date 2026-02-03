# Project Setup

## CLAUDE.md Structure and Conventions

### What CLAUDE.md Is

A memory file Claude reads at session start. Provides persistent context Claude cannot infer from code alone.

### File Locations and Hierarchy

| Location | Scope | Shared |
|----------|-------|--------|
| `~/.claude/CLAUDE.md` | All projects | No |
| `./CLAUDE.md` | Project root | Yes (git) |
| `./.claude/CLAUDE.md` | Project | Yes (git) |
| `./CLAUDE.local.md` | Project personal | No (gitignored) |
| `./.claude/rules/*.md` | Modular rules | Yes (git) |

### What to INCLUDE

- Build commands: `dotnet build`, `npm install`
- Test commands: `dotnet test`, `npm test`
- Code style rules that differ from defaults
- Repository etiquette (branch naming, PR format)
- Architectural decisions
- Environment quirks and gotchas

### What to EXCLUDE

- Standard conventions Claude already knows
- Things Claude can infer from code
- Detailed API docs (link instead)
- Long tutorials or explanations
- File-by-file codebase descriptions

### Example CLAUDE.md

```markdown
# Build and Test
- Build: `dotnet build`
- Test: `dotnet test`
- Run: `dotnet run --project ./src/WebApp`

# Code Style
- Use modern C#: null-conditional ?., pattern matching
- Prefer async/await over .Result, .Wait()
- Use var when type obvious from RHS
- Prefer DI over static methods
- Always dispose IDisposable (using statements)

# Architecture
- Blazor Server for internal apps
- Use EventCallback over Action for component events
- Use .AsNoTracking() for read-only EF queries

# Azure
- Blob Storage for files; SAS tokens on-demand
- Never commit connection strings—use Key Vault

# Git
- Commit messages: explain "why" not "what"
- Don't commit unless explicitly asked
```

### Keep It Concise

For each line, ask: *"Would removing this cause Claude to make mistakes?"*

If not, cut it. Long files cause Claude to ignore rules.

---

## Per-Project vs. Global Settings

### Configuration Hierarchy (highest priority first)

1. **Managed** - Organization-wide (IT-deployed, cannot override)
2. **Local** - `.claude/settings.local.json` (personal project overrides)
3. **Project** - `.claude/settings.json` (team-shared)
4. **User** - `~/.claude/settings.json` (personal defaults)

### Configuration Locations

#### User-Level: `~/.claude/`

Applies to all your projects:
- `settings.json` - Personal preferences
- `CLAUDE.md` - Personal instructions
- `rules/` - Personal modular rules
- `agents/` - Personal subagents
- `skills/` - Personal skills

#### Project-Level: `.claude/`

Team-shared (commit to git):
- `settings.json` - Permissions, hooks
- `CLAUDE.md` - Project instructions
- `rules/` - Modular rules
- `agents/` - Custom subagents
- `skills/` - Reusable skills

#### Local-Level: `.claude/settings.local.json`

Personal project overrides (gitignored automatically).

### Example settings.json

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet test)",
      "Bash(dotnet build)",
      "Bash(git commit *)",
      "Edit(src/**)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

### Importing Files in CLAUDE.md

```markdown
See @README.md for overview and @package.json for commands.
Git workflow: @docs/git-instructions.md
```

---

## Modular Rules with .claude/rules/

### Directory Structure

```
.claude/
├── rules/
│   ├── code-style.md
│   ├── testing.md
│   ├── security.md
│   └── api-design.md
```

### Path-Specific Rules

Target rules to specific files:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Rules
- All endpoints must include input validation
- Use standard error response format
- Include OpenAPI documentation
```

### Supported Glob Patterns

- `**/*.ts` - All TypeScript files
- `src/**/*` - All files under src/
- `src/**/*.{ts,tsx}` - Multiple extensions

---

## Hooks and Automation

Hooks are shell commands that execute at specific lifecycle points. Unlike CLAUDE.md (advisory), hooks **always run**.

### Available Hook Events

| Event | When | Use Case |
|-------|------|----------|
| `SessionStart` | Session begins | Load context |
| `PreToolUse` | Before tool (can block) | Block dangerous commands |
| `PostToolUse` | After tool succeeds | Auto-format code |
| `Notification` | Status updates | Desktop alerts |
| `Stop` | Claude finishes | Force continuation |

### Hook Configuration

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
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### Desktop Notifications

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

### Hook Exit Codes

- **0**: Action proceeds
- **2**: Blocking error (action prevented, message to Claude)
- **Other**: Non-blocking error

### Managing Hooks

```
/hooks  # Interactive hook management
```

---

## Team and Shared Configurations

### What to Commit (Share with Team)

```
.claude/
├── settings.json       # Permissions, hooks
├── CLAUDE.md           # Project instructions
├── rules/              # Modular rules
├── agents/             # Custom subagents
└── skills/             # Reusable workflows
```

### What NOT to Commit

- `.claude/settings.local.json` - Personal overrides
- `CLAUDE.local.md` - Personal preferences
- `.env` files with secrets

### Team Settings Example

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run build)",
      "Bash(git commit *)",
      "Edit(src/**)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### CI/CD Permissions

```bash
claude -p "Fix lint errors" --allowedTools "Edit,Bash(npm run lint)"
```

---

## Project Initialization

### Using /init

```bash
cd /path/to/project
claude
> /init
```

The `/init` command:
1. Analyzes codebase (build systems, test frameworks)
2. Generates starter CLAUDE.md
3. Prints content for review

### Post-Init Steps

1. Review generated CLAUDE.md
2. Keep useful content, delete obvious stuff
3. Add project-specific context
4. Commit to git

### Recommended Directory Structure

```
your-project/
├── .claude/
│   ├── settings.json           # Team permissions
│   ├── settings.local.json     # Personal (gitignored)
│   ├── rules/
│   │   ├── code-style.md
│   │   └── testing.md
│   ├── agents/
│   └── skills/
├── CLAUDE.md                   # Main instructions
├── CLAUDE.local.md             # Personal (gitignored)
├── .gitignore
└── src/
```

### Minimal vs. Full Setup

**Minimal** (small projects):
- `CLAUDE.md`
- `.claude/settings.json`

**Full** (larger teams):
- `CLAUDE.md`
- `.claude/settings.json`
- `.claude/rules/*.md`
- `.claude/agents/`
- `.claude/skills/`
- `.claude/hooks/`

---

## Checklists

### CLAUDE.md Quality

- [ ] Build and test commands documented
- [ ] Code style differences from defaults listed
- [ ] Architectural decisions noted
- [ ] Concise (under 100 lines ideal)
- [ ] Each rule passes "would removing this break Claude?" test
- [ ] Committed to git

### Settings Configuration

- [ ] `.claude/settings.json` in git (team-shared)
- [ ] `.claude/settings.local.json` in .gitignore
- [ ] Permissions explicit (allow safe, deny dangerous)
- [ ] Hooks automate repetitive tasks

### .gitignore Additions

```gitignore
# Claude Code local settings
.claude/settings.local.json
CLAUDE.local.md
```

---

## Notes

<!-- Personal observations as you use Claude Code -->
