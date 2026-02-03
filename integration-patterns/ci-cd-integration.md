# CI/CD Integration Deep Dive

## Headless Mode

### What Is Headless Mode?

Running Claude non-interactively with `-p` flag:
```bash
claude -p "Your instructions"
```

No interactive session - runs, outputs, exits.

### Key Flags for CI/CD

| Flag | Purpose | Example |
|------|---------|---------|
| `-p` | Print mode (non-interactive) | `claude -p "Analyze code"` |
| `--allowedTools` | Whitelist tools | `--allowedTools "Read,Edit"` |
| `--output-format` | Output format | `--output-format json` |
| `--max-turns` | Limit iterations | `--max-turns 5` |
| `--max-budget-usd` | Cost limit | `--max-budget-usd 2.00` |
| `--model` | Specify model | `--model sonnet` |

### Basic CI Script

```bash
#!/bin/bash
claude -p "Review the changes for issues" \
  --allowedTools "Read,Grep,Glob" \
  --max-turns 3 \
  --output-format json
```

---

## GitHub Actions

### Quick Setup

```bash
claude /install-github-app
```

### Basic Workflow

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
          prompt: "Review this PR for bugs, security issues, and style"
          claude_args: "--max-turns 5"
```

### Mention-Based Trigger

Respond to @claude mentions in comments:

```yaml
name: Claude Mention Response

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  respond:
    runs-on: ubuntu-latest
    if: contains(github.event.comment.body, '@claude')

    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Custom Review Prompts

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: |
      Review this PR for:
      1. Security vulnerabilities (OWASP Top 10)
      2. Performance issues (N+1 queries, missing async)
      3. Missing error handling
      4. Test coverage gaps

      Format findings as:
      - [SEVERITY] file:line - description
    claude_args: "--max-turns 5 --model sonnet"
```

### Conditional Workflows

```yaml
# Only for specific file types
on:
  pull_request:
    paths:
      - '**.cs'
      - '**.ts'

# Only for certain labels
jobs:
  review:
    if: contains(github.event.pull_request.labels.*.name, 'needs-review')
```

---

## Output Formats

### JSON Output

```bash
claude -p "List all API endpoints" --output-format json
```

Output:
```json
{
  "result": "...",
  "usage": {...}
}
```

Process with jq:
```bash
claude -p "List endpoints" --output-format json | jq '.result'
```

### Streaming JSON

```bash
claude -p "Analyze codebase" --output-format stream-json
```

Real-time output as JSON events.

### JSON Schema (Typed Output)

```bash
claude -p "Extract API endpoints" \
  --output-format json \
  --json-schema '{
    "type": "object",
    "properties": {
      "endpoints": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "path": {"type": "string"},
            "method": {"type": "string"},
            "description": {"type": "string"}
          }
        }
      }
    }
  }'
```

Enforces structured output.

---

## CI/CD Patterns

### PR Review on Every Push

```yaml
on:
  pull_request:
    types: [opened, synchronize]

steps:
  - uses: anthropics/claude-code-action@v1
    with:
      prompt: "/review"
```

### Nightly Code Analysis

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily

jobs:
  analyze:
    steps:
      - uses: actions/checkout@v4

      - name: Security Scan
        run: |
          claude -p "Scan for security vulnerabilities" \
            --allowedTools "Read,Grep,Glob" \
            --output-format json > security-report.json

      - uses: actions/upload-artifact@v4
        with:
          name: security-report
          path: security-report.json
```

### Batch Migration

```yaml
jobs:
  migrate:
    strategy:
      matrix:
        file: [src/auth.ts, src/api.ts, src/db.ts]

    steps:
      - run: |
          claude -p "Migrate ${{ matrix.file }} to new API" \
            --allowedTools "Read,Edit" \
            --max-turns 3
```

### Pre-Merge Checks

```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  checks:
    steps:
      - name: Code Review
        uses: anthropics/claude-code-action@v1
        with:
          prompt: "Review for bugs and security issues"

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

---

## Tool Permissions in CI

### Read-Only Analysis

```bash
claude -p "Analyze code quality" \
  --allowedTools "Read,Grep,Glob"
```

Safe for any PR - can't modify anything.

### Edit with Constraints

```bash
claude -p "Fix linting errors" \
  --allowedTools "Read,Edit,Bash(npm run lint)"
```

Can edit files and run lint, nothing else.

### Full Access (Careful)

```bash
claude -p "Implement the feature" \
  --allowedTools "Read,Edit,Write,Bash(*)"
```

Only in trusted contexts with proper review.

---

## Error Handling

### Check Exit Codes

```bash
#!/bin/bash
result=$(claude -p "Analyze code" --output-format json)
exit_code=$?

if [ $exit_code -ne 0 ]; then
  echo "Claude failed with exit code $exit_code"
  exit 1
fi

# Process result
echo "$result" | jq '.result'
```

### Timeout Handling

```yaml
steps:
  - uses: anthropics/claude-code-action@v1
    timeout-minutes: 10
    with:
      prompt: "Review code"
```

### Budget Protection

```bash
claude -p "Analyze entire codebase" \
  --max-budget-usd 5.00 \
  --max-turns 10
```

Stops if budget or turns exceeded.

---

## Secrets Management

### GitHub Secrets

```yaml
with:
  anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Never hardcode API keys.

### Environment Variables

```yaml
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

steps:
  - run: claude -p "Analyze code"
```

### Protecting Sensitive Files

```bash
claude -p "Review code" \
  --allowedTools "Read(src/**),Grep(src/**)"
```

Restricts access to src/ only.

---

## Notes

<!-- Add observations about CI/CD integration -->

