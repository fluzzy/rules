# Claude Code Showcase

> **Source**: https://github.com/ChrisWiles/claude-code-showcase
> **Archive Date**: 2026-01-19

A comprehensive configuration example for making Claude Code agent your team's superpower colleague. Build consistent, systematic development workflows through reusable skills, agents, and hooks.

## Directory Structure

```
.claude/
├── settings.json        # Hooks, environment variables, permissions
├── settings.md          # Settings documentation
├── agents/              # Custom AI agents
├── commands/            # Slash commands (/command)
├── hooks/               # Automation scripts
├── skills/              # Domain knowledge documents
└── rules/               # Modular instructions

.mcp.json               # MCP server config (JIRA, GitHub, etc.)
CLAUDE.md               # Project memory
```

## Core Components

### 1. CLAUDE.md - Project Memory

Persistent context that auto-loads at session start.

**Contents:**

- Project stack and architecture
- Test, build, lint commands
- Code style guidelines
- Important directory descriptions

**Location priority:**

1. `.claude/CLAUDE.md`
2. `./CLAUDE.md` (project root)
3. `~/.claude/CLAUDE.md` (user level)

### 2. settings.json - Hooks and Environment

Central configuration for automated quality control.

**Main hook events:**

| Event              | Trigger Time         | Use Case                       |
| ------------------ | -------------------- | ------------------------------ |
| `PreToolUse`       | Before tool execution | Block main branch edits        |
| `PostToolUse`      | After tool completion | Auto-format, run tests         |
| `UserPromptSubmit` | On prompt submission  | Add context, suggest skills    |
| `Stop`             | On agent termination  | Decide whether to continue     |

**Hook response format:**

```json
{
  "block": true,
  "message": "reason",
  "feedback": "info",
  "continue": false
}
```

### 3. MCP Servers - External Integrations

Connect with JIRA, GitHub, Slack, etc. via Model Context Protocol.

**.mcp.json structure:**

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**Key integrations:**

| Category       | Servers          | Capabilities              |
| -------------- | ---------------- | ------------------------- |
| Issue Tracking | JIRA, Linear     | Read/modify tickets       |
| Code/DevOps    | GitHub, Sentry   | PR management, error tracking |
| Communication  | Slack            | Send messages             |
| Database       | PostgreSQL       | Execute queries, check schema |

### 4. Skills - Domain Knowledge

Project-specific guides. YAML header + markdown content format.

```yaml
---
name: testing-patterns
description: Jest testing patterns. Use when writing tests/mocking.
---

# Testing Patterns

## Test Structure
- Group with describe blocks
- AAA pattern: Arrange, Act, Assert
```

### 5. Agents - Specialized Assistants

Auto-execute or user-invocable specialists for specific tasks.

- **Code Review Agent**: TypeScript strict mode, error handling checks
- **Docs Sync**: Monthly commit reading → document alignment check
- **Quality Review**: Weekly random directory review and auto-fix
- **Dependency Audit**: Bi-weekly safe updates

### 6. Commands - Slash Commands

Workflows explicitly invoked by users.

- `/ticket PROJ-123`: Read ticket → implement code → update status → create PR
- `/onboard`: Deep work exploration
- `/pr-review`: PR review workflow

## GitHub Actions Workflows

| Workflow        | Frequency | Function               |
| --------------- | --------- | ---------------------- |
| PR Claude Review | Auto      | Full PR review         |
| Docs Sync       | Monthly   | Docs-code sync         |
| Code Quality    | Weekly    | Random review and fix  |
| Dependency Audit | Bi-weekly | Safe updates           |

## Quick Start

```bash
# 1. Create directories
mkdir -p .claude/{agents,commands,hooks,skills}

# 2. Write CLAUDE.md (project root)
# 3. Configure settings.json (add hooks)
# 4. Write first skill: .claude/skills/testing-patterns/SKILL.md
# 5. Configure MCP (optional): .mcp.json
```

## License

MIT License
