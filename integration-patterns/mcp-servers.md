# MCP Servers Deep Dive

## What Is MCP?

Model Context Protocol - open standard for AI-tool integrations.

MCP servers expose:
- **Tools** - Functions Claude can call
- **Resources** - Data access
- **Prompts** - Templates

### Why Use MCP?

- Integrate external services (GitHub, databases, etc.)
- Extend Claude's capabilities
- Standardized protocol across tools

---

## Adding MCP Servers

### HTTP Servers (Remote)

```bash
# GitHub
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Sentry
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

### Stdio Servers (Local)

```bash
# PostgreSQL
claude mcp add --transport stdio postgres -- npx -y pg-mcp-server \
  --env DATABASE_URL="postgresql://user:pass@localhost/db"

# Custom server
claude mcp add --transport stdio myserver -- node ./my-mcp-server.js
```

### Project Configuration

Create `.mcp.json` in project root:

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

Environment variables with `${VAR}` are expanded.

---

## Managing Servers

### List Servers

```bash
claude mcp list
```

### Server Details

```bash
claude mcp get github
```

### Remove Server

```bash
claude mcp remove github
```

### In-Session Status

```
/mcp
```

Shows active servers and their context cost.

---

## Common Integrations

### GitHub

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

**Usage:**
```
Review PR #456
Create an issue for this bug
List open PRs assigned to me
```

### Database

```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --env DATABASE_URL="postgresql://..."
```

**Usage:**
```
Show schema for the orders table
Find customers with no orders in 90 days
What indexes exist on users table?
```

### Sentry (Error Monitoring)

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

**Usage:**
```
What are the most common errors today?
Show details for error XYZ
List unresolved issues in production
```

### Jira

```bash
claude mcp add --transport http jira https://your-jira-mcp-endpoint
```

**Usage:**
```
Create a ticket for this bug
Show my assigned issues
Update PROJ-123 status to In Progress
```

---

## MCP vs CLI Tools

### When to Use MCP

| Use Case | Why MCP |
|----------|---------|
| GitHub API operations | Rich integration, OAuth |
| Database queries | Schema awareness, safety |
| Error monitoring | Live data access |
| Third-party services | API abstraction |

### When to Use CLI

| Use Case | Why CLI |
|----------|---------|
| Git operations | `git` is simpler |
| Local builds | `dotnet build`, `npm run` |
| Package management | `npm install`, `dotnet add` |
| Simple file operations | Built-in tools faster |

### Comparison

```
# MCP - for API-heavy operations
"List all open PRs with their review status"
(Uses GitHub MCP for rich API access)

# CLI - for simple operations
"Run git status"
(Uses Bash tool directly)
```

---

## Context Cost

### MCP Overhead

Each MCP server adds to context:
- Tool schemas
- Resource definitions
- Server metadata

Check with `/mcp`:
```
/mcp
GitHub: ~2k tokens
Database: ~3k tokens
Total MCP: ~5k tokens (2.5% of context)
```

### Reducing Overhead

**Disable unused servers:**
```
/mcp
> disable sentry
```

**Tool Search for many servers:**
```bash
ENABLE_TOOL_SEARCH=auto claude
```

Tools load on-demand instead of all at once.

### Tool Search Thresholds

```bash
# Enable when MCP exceeds 5% of context
ENABLE_TOOL_SEARCH=auto:5 claude

# Always enable tool search
ENABLE_TOOL_SEARCH=always claude
```

---

## Security Considerations

### Environment Variables

Never hardcode credentials:
```json
{
  "mcpServers": {
    "database": {
      "env": {
        "DATABASE_URL": "${DB_CONNECTION_STRING}"
      }
    }
  }
}
```

Set `DB_CONNECTION_STRING` in your shell.

### Scoped Access

Some MCP servers support scoped permissions:
```json
{
  "database": {
    "scope": "read-only"
  }
}
```

### Project vs User Config

- Project `.mcp.json` - Shared with team (no secrets)
- User `~/.claude/mcp.json` - Personal servers (can have auth)

---

## Creating Custom MCP Servers

### Basic Structure

MCP servers implement a simple protocol:
1. Declare available tools
2. Handle tool calls
3. Return results

### Example (Node.js)

```javascript
// my-mcp-server.js
const { McpServer } = require('@modelcontextprotocol/sdk');

const server = new McpServer({
  tools: [{
    name: 'get_weather',
    description: 'Get weather for a city',
    parameters: {
      type: 'object',
      properties: {
        city: { type: 'string' }
      }
    }
  }],

  async handleToolCall(name, params) {
    if (name === 'get_weather') {
      // Call weather API
      return { temperature: 72, conditions: 'sunny' };
    }
  }
});

server.start();
```

### Registering Custom Server

```bash
claude mcp add --transport stdio weather -- node ./my-mcp-server.js
```

---

## Troubleshooting

### Server Not Connecting

```bash
# Check server status
claude mcp get servername

# Test manually
npx -y server-package --help
```

### Tools Not Available

Check `/mcp` in session:
- Is server listed?
- Is it enabled?
- Check tool count

### High Context Usage

```
/mcp
```

If MCP using >10% context:
- Disable unused servers
- Enable tool search
- Use CLI alternatives

### Authentication Errors

- Check environment variables
- Verify credentials are valid
- Some servers need re-authentication

---

## Notes

<!-- Add observations about MCP servers -->

