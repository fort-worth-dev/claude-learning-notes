# IDE Integration Deep Dive

## VS Code Extension

### Installation

```bash
code --install-extension anthropic.claude-code
```

Or search "Claude Code" in VS Code Extensions.

### Key Features

| Feature | Description |
|---------|-------------|
| Inline diffs | See changes before accepting |
| Plan Mode | Review approach before implementation |
| Multi-tab | Multiple conversation threads |
| @-mentions | Reference files with line ranges |
| Context awareness | Understands open files |

### Plan Mode in VS Code

```
Ctrl+Shift+Tab
```

Toggles Plan Mode - Claude researches without making changes.

### @-Mentions

Reference files directly:
```
@src/auth/login.cs explain the token validation
@src/api/users.ts:45-60 what does this function do?
```

### Inline Diffs

When Claude suggests changes:
1. See diff highlighting
2. Accept/reject per-change
3. Edit before accepting

---

## Extension vs CLI

### When to Use Extension

| Extension Best For |
|-------------------|
| Interactive development |
| Reviewing diffs visually |
| Quick inline edits |
| File-focused work |
| Plan Mode review |

### When to Use CLI

| CLI Best For |
|--------------|
| Automation and scripting |
| Batch operations |
| CI/CD pipelines |
| Background tasks |
| Multi-directory work |

### Complementary Workflow

**Extension for review:**
```
Review the auth structure and plan changes
```

**CLI for batch execution:**
```bash
claude -p "Fix all eslint errors" --allowedTools "Read,Edit"
```

---

## Shared Sessions

### Continue Between Extension and CLI

Start in extension, continue in CLI:
```bash
claude --continue
```

Or start in CLI, continue in extension.

### Named Sessions

Name sessions for easy switching:
```
/rename "feature-auth"
```

Then in either tool:
```bash
claude -r "feature-auth"
```

---

## VS Code Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+Shift+Tab` | Toggle Plan Mode |
| `Ctrl+Enter` | Submit prompt |
| `Escape` | Cancel operation |

### Customizing Shortcuts

VS Code: `File > Preferences > Keyboard Shortcuts`

Search for "Claude" to see available commands.

---

## Multi-Tab Conversations

### Why Multiple Tabs?

- Separate contexts for different tasks
- Compare approaches
- Keep reference conversation open

### Managing Tabs

- New tab: Fresh conversation
- Tabs persist across VS Code sessions
- Each tab has independent context

---

## Context from Editor

### Automatic Context

Extension sees:
- Currently open file
- Selected text
- Visible code
- Project structure

### Explicit Context

Use @-mentions for specific files:
```
@package.json what testing framework?
@src/models/ explain the data models
```

### Selection-Based Prompts

1. Select code in editor
2. Open Claude extension
3. Ask about selection:
```
Explain this code
Refactor this function
Add tests for this
```

---

## Extension Settings

### Accessing Settings

VS Code: `File > Preferences > Settings`
Search for "Claude"

### Common Settings

| Setting | Purpose |
|---------|---------|
| Default model | Sonnet, Opus, or Haiku |
| Auto-accept mode | Skip confirmation for edits |
| Theme | Match VS Code theme |

---

## Workflow Patterns

### Feature Development

1. **Plan in extension:**
   - `Ctrl+Shift+Tab` for Plan Mode
   - Ask Claude to analyze existing code
   - Review the plan

2. **Implement in extension:**
   - `Ctrl+Shift+Tab` to exit Plan Mode
   - Implement with inline diffs
   - Accept/reject changes

3. **Test via CLI:**
   ```bash
   claude -p "Run tests and fix failures" --allowedTools "Bash(npm test),Edit"
   ```

### Code Review

1. **Open PR files in VS Code**
2. **Ask Claude in extension:**
   ```
   Review @src/auth/login.ts for security issues
   ```
3. **See inline annotations**

### Debugging

1. **Select error in terminal**
2. **In extension:**
   ```
   Help me debug this error: [paste]
   ```
3. **Claude references open files for context**

---

## Troubleshooting

### Extension Not Working

1. Check extension is installed and enabled
2. Verify API key in settings
3. Check VS Code output panel for errors

### Session Not Syncing

Sessions are directory-based:
- CLI and extension must be in same directory
- Use `--cwd` flag if needed

### Context Not Loading

- Ensure files are saved
- Check file isn't too large
- Try explicit @-mention

---

## Notes

<!-- Add observations about IDE integration -->

