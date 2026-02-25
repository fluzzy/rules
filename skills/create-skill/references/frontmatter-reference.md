# Frontmatter Reference

Complete reference for SKILL.md frontmatter fields.

## All Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | No (defaults to dir name) | Lowercase, numbers, hyphens only. Max 64 chars. Becomes `/slash-command`. Must match folder name. |
| `description` | Recommended | What it does + when to use it + key capabilities. Max 1024 chars. No XML angle brackets. |
| `argument-hint` | No | Autocomplete hint. Example: `[issue-number]`, `[filename] [format]`. |
| `disable-model-invocation` | No | `true` = manual `/name` only. Use for side-effect workflows. |
| `user-invokable` | No | `false` = hidden from `/` menu. Use for background knowledge. Note spelling: `invokable`, not `invocable`. |
| `allowed-tools` | No | Restrict available tools. Example: `"Read, Write, Bash(git:*)"`. Comma-separated list with optional glob patterns for Bash. |
| `model` | No | Override model: `sonnet`, `opus`, `haiku`. |
| `context` | No | `fork` = run in isolated subagent without conversation history. |
| `agent` | No | Subagent type when `context: fork`: `Explore`, `Plan`, `general-purpose`, or custom. |
| `hooks` | No | Lifecycle hooks scoped to this skill. Supports `PreToolUse` and `PostToolUse` matchers. |
| `license` | No | For open-source skills. Common: `MIT`, `Apache-2.0`. |
| `compatibility` | No | Environment requirements (1-500 chars). |
| `metadata` | No | Custom key-value pairs: `author`, `version`, `mcp-server`, `category`, `tags`, etc. |

## `allowed-tools` Examples

```yaml
# Allow only read operations
allowed-tools: "Read, Glob, Grep"

# Allow Bash with specific command patterns
allowed-tools: "Read, Write, Bash(git:*), Bash(npm:*)"

# Allow everything except destructive operations
allowed-tools: "Read, Write, Edit, Glob, Grep, Bash(ls:*), Bash(cat:*)"

# Allow MCP tools
allowed-tools: "Read, Write, mcp__server__tool_name"
```

## Security Restrictions

- No XML angle brackets (`<` or `>`) in frontmatter values
- Names with "claude" or "anthropic" are reserved and forbidden

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed to the skill |
| `$ARGUMENTS[N]` | Argument by 0-based index |
| `$N` | Shorthand for `$ARGUMENTS[N]` (`$0` = first arg) |
| `${CLAUDE_SESSION_ID}` | Current session ID |

If `$ARGUMENTS` is not present in content and arguments are provided, Claude Code appends `ARGUMENTS: <value>` to the end.

## Description Writing Guide

### Formula

```
[What it does] + [When to use it] + [Key capabilities]
```

### Rules

1. Use 3rd person: "This skill should be used when..."
2. Include exact trigger phrases users would say, in quotes
3. Be specific to prevent over-triggering or under-triggering
4. Add negative triggers if needed: "Do NOT use for simple X (use Y skill instead)"
5. Mention relevant file types if applicable
6. Stay under 1024 characters, no XML tags

### Good Examples

```yaml
# Specific triggers, 3rd person, clear scope
description: "Analyzes Figma design files and generates developer handoff documentation. Use when user uploads .fig files, asks for \"design specs\", \"component documentation\", or \"design-to-code handoff\"."

# Includes what, when, and negative trigger
description: "Advanced data analysis for CSV files. Use for statistical modeling, regression, clustering. Do NOT use for simple data exploration (use data-viz skill instead)."

# MCP-enhanced with clear value
description: "End-to-end customer onboarding workflow for PayFlow. Handles account creation, payment setup, and subscription management. Use when user says \"onboard new customer\", \"set up subscription\", or \"create PayFlow account\"."
```

### Bad Examples

```yaml
# Too vague, no triggers
description: Helps with projects.

# Missing trigger phrases
description: Creates sophisticated multi-page documentation systems.

# Too technical, no user-facing triggers
description: Implements the Project entity model with hierarchical relationships.
```
