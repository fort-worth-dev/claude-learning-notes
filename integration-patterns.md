# Integration Patterns

This is a quick reference. For detailed notes, see the [integration-patterns/](integration-patterns/) subfolder:

- [Git Workflows](integration-patterns/git-workflows.md) - Commits, PRs, worktrees, hooks
- [CI/CD Integration](integration-patterns/ci-cd-integration.md) - Headless mode, GitHub Actions, output formats
- [IDE Integration](integration-patterns/ide-integration.md) - VS Code extension, complementary workflows
- [MCP Servers](integration-patterns/mcp-servers.md) - Setup, configuration, common integrations

---

## Git Workflows

### Commit Workflow

```
Look at my staged changes and create an appropriate commit
```

**Pre-approve git commands in `.claude/settings.json`:**
```json
{
  "permissions": {
    "allow": [
      "Bash(git commit *)",
      "Bash(git log *)",
      "Bash(git diff *)",
      "Bash(git status *)"
    ]
  }
}
```

### PR Creation

```
/pr-summary           # Summarize PR changes
Create a PR with a detailed description
```

**Resume from PR:**
```bash
claude --from-pr 123
```

### Branch Management with Worktrees

```bash
# Create isolated worktrees for parallel work
git worktree add ../project-feature-a -b feature-a
cd ../project-feature-a
claude

# Another terminal
git worktree add ../project-bugfix -b bugfix-123
cd ../project-bugfix
claude

# Manage
git worktree list
git worktree remove ../project-feature-a
```

### Protect Branches with Hooks

`.claude/hooks/protect-branches.sh`:
```bash
#!/bin/bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
PROTECTED=("main" "production")

for protected in "${PROTECTED[@]}"; do
  if [[ "$BRANCH" == "$protected" ]]; then
    echo "Cannot commit to protected branch: $BRANCH" >&2
    exit 2
  fi
done
exit 0
```

---

## CI/CD Integration

### Headless Mode

```bash
claude -p "Your instructions" [flags]
```

**Key flags:**
```bash
--allowedTools "Read,Edit,Bash(npm test)"  # Control permissions
--output-format json                        # Structured output
--max-turns 5                               # Limit iterations
--model sonnet                              # Specify model
```

### GitHub Actions

**Quick setup:**
```bash
claude /install-github-app
```

**Workflow example:**
```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "/review"
          claude_args: "--max-turns 5"
```

**Mention-based (responds to @claude):**
```yaml
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Output Formats

**JSON for structured processing:**
```bash
claude -p "List API endpoints" --output-format json | jq '.result'
```

**Streaming for real-time:**
```bash
claude -p "Analyze logs" --output-format stream-json
```

**JSON Schema for typed output:**
```bash
claude -p "Extract endpoints" --output-format json \
  --json-schema '{"type":"object","properties":{"endpoints":{"type":"array"}}}'
```

---

## IDE Usage Alongside CLI

### VS Code Extension

```bash
code --install-extension anthropic.claude-code
```

**Features:**
- Inline diffs before accepting
- Plan Mode review (`Ctrl+Shift+Tab`)
- Multi-tab conversations
- `@`-mentions for files with line ranges

### Complementary Workflows

| Extension | CLI |
|-----------|-----|
| Interactive iteration | Headless automation |
| Inline diffs | Batch operations |
| Plan Mode review | Background tasks |

**Pattern: Extension for review, CLI for batch:**
```bash
# Extension: Review structure
# Terminal: Batch fix
claude -p "Fix all eslint errors" --allowedTools "Read,Edit"
```

**Continue session between extension and CLI:**
```bash
claude --continue
```

---

## MCP Servers and Extensions

### What MCP Is

Model Context Protocol - open standard for AI-tool integrations. Servers expose:
- **Tools** (functions Claude can call)
- **Resources** (data access)
- **Prompts** (templates)

### Adding MCP Servers

**HTTP server:**
```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

**Stdio server (local):**
```bash
claude mcp add --transport stdio postgres -- npx -y pg-mcp-server \
  --env DATABASE_URL="postgresql://user:pass@localhost/db"
```

### Project MCP Config (`.mcp.json`)

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "database": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub"],
      "env": {
        "DATABASE_URL": "${DB_CONNECTION_STRING}"
      }
    }
  }
}
```

### Managing Servers

```bash
claude mcp list      # List configured
claude mcp get github  # Server details
claude mcp remove github  # Remove
/mcp                 # Check status in session
```

### Common Integrations

**GitHub:**
```
Review PR #456 and suggest improvements
Create an issue for this bug
```

**Database:**
```
Find customers who haven't purchased in 90 days
Show schema for orders table
```

**Sentry:**
```
What are the most common errors in the last 24 hours?
```

### MCP vs CLI Tools

| Use MCP | Use CLI |
|---------|---------|
| External services (GitHub, Jira) | Local builds (npm, dotnet) |
| Database queries | Git operations |
| Error monitoring (Sentry) | Package management |
| Real-time API data | Custom scripts |

### Tool Search for Many Servers

When MCP tools exceed 10% of context, enable Tool Search:
```bash
ENABLE_TOOL_SEARCH=auto claude
ENABLE_TOOL_SEARCH=auto:5 claude  # 5% threshold
```

---

## Example Workflows

### Feature Implementation

```bash
# 1. Explore (Plan Mode)
claude --permission-mode plan
> Analyze auth system, suggest OAuth2 approach

# 2. Plan
> Create detailed implementation plan
# Ctrl+G to edit plan

# 3. Implement (Normal Mode)
# Shift+Tab to switch
> Implement OAuth2 from plan, write tests

# 4. PR
> Create PR with descriptive message
```

### CI/CD Code Review

```yaml
steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
      prompt: |
        Review for:
        1. Security issues
        2. Performance concerns
        3. Missing error handling
      claude_args: "--max-turns 5"
```

### Batch Migration

```bash
#!/bin/bash
FILES=$(find . -name "*.ts" -not -path "./node_modules/*")

for file in $FILES; do
  claude -p "Add strict mode and fix errors in $file" \
    --allowedTools "Read,Edit" \
    --max-turns 3
done
```

---

## Notes

<!-- Personal observations as you use Claude Code -->
