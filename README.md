# Claude Code Notes

Notes and quick references for using Claude Code (CLI, IDE, and workflows). Each topic has a root-level summary with links to detailed subfolder docs.

---

## Contents

- **[Fundamentals](fundamentals.md)** — CLI commands, session lifecycle, context management (CLAUDE.md hierarchy, limits), session persistence (resume, fork, compaction), configuration (settings, permissions, env vars), and the core workflow: Explore → Plan → Implement → Verify.

- **[Agents and Tools](agents-and-tools.md)** — Subagents (Explore, Plan, General-purpose, Bash), when to delegate, skills and custom slash commands, core tools (Read, Edit, Grep, Bash, etc.), and background tasks.

- **[Code Workflows](code-workflows.md)** — Plan → Implement → Test cycle, code review (direct and PR), refactoring (safe patterns, TDD, fan-out migrations), and debugging (investigation, test-first, root cause).

- **[Efficiency Tips](efficiency-tips.md)** — Token and context management, parallel work (sessions, worktrees, subagents), model selection (Sonnet, Opus, Haiku), and keyboard shortcuts and CLI flags.

- **[Prompting Techniques](prompting-techniques.md)** — Prompt structure (goal, scope, constraints, verification), iteration (correcting, interrupting, when to clear), anti-patterns, and verification-first prompting.

- **[Integration Patterns](integration-patterns.md)** — Git (commits, PRs, worktrees, hooks), CI/CD (headless mode, GitHub Actions), IDE usage, and MCP servers and extensions.

- **[Project Setup](project-setup.md)** — CLAUDE.md structure and conventions, per-project vs global settings, modular rules (`.claude/rules/`), hooks, and team/shared configuration.
