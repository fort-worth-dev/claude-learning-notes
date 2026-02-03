# Core Tools Deep Dive

## Tool Categories

Claude has access to several categories of tools:

| Category | Tools | Purpose |
|----------|-------|---------|
| File Operations | Read, Write, Edit, Glob | Work with files |
| Search | Grep, Glob | Find code and files |
| Execution | Bash | Run commands |
| Web | WebSearch, WebFetch | Access internet |
| Tasks | Task, TaskOutput | Background operations |

---

## File Operations

### Read

Reads file contents into context.

**Usage:**
```
Read src/Services/AuthService.cs
```

**Capabilities:**
- Text files of any type
- Images (PNG, JPG) - Claude sees them visually
- PDFs - extracts text and images
- Jupyter notebooks - shows cells with outputs

**Best for:**
- Viewing specific files you know about
- Examining code before editing
- Reading configuration files

**Context impact:** Full file contents added to context.

### Write

Creates a new file or overwrites existing.

**Usage:**
```
Write src/Models/User.cs with the User class
```

**Best for:**
- Creating new files
- Replacing entire file contents
- Generating boilerplate

**Caution:** Overwrites without confirmation if file exists.

### Edit

Modifies specific parts of an existing file.

**Usage:**
```
Edit src/Services/AuthService.cs to add null check in Login method
```

**Best for:**
- Targeted changes to existing code
- Adding methods to classes
- Fixing specific lines

**How it works:**
- Claude reads the file first
- Makes surgical changes
- Preserves rest of file

### Glob

Finds files matching a pattern.

**Patterns:**
| Pattern | Matches |
|---------|---------|
| `*.cs` | All .cs files in current dir |
| `**/*.cs` | All .cs files recursively |
| `src/**/*.ts` | All .ts files under src/ |
| `**/test*.js` | Files starting with "test" |
| `{*.cs,*.fs}` | .cs and .fs files |

**Usage:**
```
Find all controller files: Glob("**/Controllers/*.cs")
```

**Best for:**
- Discovering file structure
- Finding files by naming convention
- Scoping searches

---

## Search Tools

### Grep

Searches file contents using regex.

**Basic usage:**
```
Search for "async Task" in all .cs files
```

**Regex patterns:**
| Pattern | Matches |
|---------|---------|
| `GetUser` | Literal string |
| `Get.*User` | Get followed by anything then User |
| `public\s+async` | public, whitespace, async |
| `TODO\|FIXME` | TODO or FIXME |

**With file filtering:**
```
Grep for "connectionString" in *.json files
```

**Best for:**
- Finding where something is used
- Searching for patterns
- Locating string literals

### Grep vs Glob

| Task | Use |
|------|-----|
| Find files named *Controller* | Glob |
| Find files containing "Controller" | Grep |
| Find .cs files | Glob |
| Find files with "async" keyword | Grep |

### Combining Glob and Grep

```
First Glob for **/Services/*.cs, then Grep those for "HttpClient"
```

Narrows search efficiently.

---

## Execution

### Bash

Runs shell commands.

**Common uses:**
```bash
dotnet build
dotnet test
git status
npm install
```

**Chaining commands:**
```bash
dotnet build && dotnet test
```

**Best for:**
- Build and test commands
- Git operations
- Package management
- Any CLI tool

**What Bash can't do:**
- Interactive commands (requires input)
- GUI applications
- Long-running servers (use background)

### Command Patterns

**Sequential (dependent):**
```bash
dotnet build && dotnet test && dotnet publish
```
Each runs only if previous succeeds.

**Parallel (independent):**
Ask Claude to run multiple Bash commands in parallel:
```
Run "dotnet test" and "npm run lint" in parallel
```

**Ignore failures:**
```bash
dotnet test; echo "Tests complete"
```
Semicolon runs next command regardless.

---

## Web Tools

### WebSearch

Searches the internet.

**Usage:**
```
Search for "EF Core 8 new features"
```

**Best for:**
- Current documentation
- Recent releases and changes
- Best practices research
- Error message lookups

**Returns:** Search results with links and snippets.

### WebFetch

Retrieves and analyzes web page content.

**Usage:**
```
Fetch https://docs.microsoft.com/... and summarize the key points
```

**Best for:**
- Reading documentation pages
- Analyzing specific URLs
- Extracting information from web content

**Limitations:**
- Can't access authenticated pages
- Some sites block automated access
- Large pages may be truncated

---

## Task Management

### Task Tool

Spawns subagents for complex work.

**Usage:**
```
Use Explore agent to investigate the authentication system
```

**When to use:**
- Broad codebase exploration
- Complex multi-step operations
- Keeping main context clean

### Background Tasks

Run commands without blocking:

**Start background task:**
```
Run the full test suite in the background
```

**Check status:**
```
/tasks
```

**Get output:**
```
Check the background test results
```

**Keyboard shortcut:**
`Ctrl+B` while command is running.

---

## Tool Selection Guide

### Finding Code

| Goal | Tool | Example |
|------|------|---------|
| Find files by name | Glob | `**/Auth*.cs` |
| Find code containing X | Grep | Search for "HttpClient" |
| Understand a file | Read | Read the specific file |
| Explore broadly | Task (Explore) | Use Explore agent |

### Modifying Code

| Goal | Tool | Example |
|------|------|---------|
| Create new file | Write | Write new service class |
| Change existing file | Edit | Add method to class |
| Replace entire file | Write | Regenerate config |
| Rename/move file | Bash | `mv old.cs new.cs` |

### Building and Testing

| Goal | Tool | Example |
|------|------|---------|
| Compile | Bash | `dotnet build` |
| Run tests | Bash | `dotnet test` |
| Start server | Bash (background) | `dotnet run` in background |
| Check git status | Bash | `git status` |

### Research

| Goal | Tool | Example |
|------|------|---------|
| Current docs | WebSearch | Search for topic |
| Specific page | WebFetch | Fetch URL |
| Codebase patterns | Task (Explore) | Use Explore |

---

## Efficiency Tips

### Minimize Context Usage

**Don't:** Read 10 files to find something
**Do:** Grep first, then read only relevant files

### Be Specific

**Don't:** "Find the authentication code"
**Do:** "Grep for 'ValidateToken' in src/Auth/"

### Chain When Possible

**Don't:** Multiple separate Bash calls
**Do:** `dotnet build && dotnet test`

### Use Right Tool for Job

**Don't:** Bash with `cat` to read files
**Do:** Use Read tool directly

**Don't:** Bash with `grep` to search
**Do:** Use Grep tool directly

### Parallel When Independent

```
Run these in parallel:
- Grep for "TODO" in src/
- Glob for **/*.md
- Bash: git log --oneline -10
```

---

## Tool Permissions

### Pre-Approving Tools

In `settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet build)",
      "Bash(dotnet test)",
      "Read(src/**)"
    ]
  }
}
```

### Blocking Tools

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Bash(rm -rf*)"
    ]
  }
}
```

### Permission Modes

| Mode | Read | Write | Bash |
|------|------|-------|------|
| plan | Yes | No | No |
| default | Ask | Ask | Ask |
| full-auto | Yes | Yes | Yes |

---

## Notes

<!-- Add observations about tools as you use them -->

