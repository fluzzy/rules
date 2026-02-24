# Skill Templates

Starter templates for common skill types. Copy and adapt as needed.

## Reference Skill Template

For background knowledge and conventions that Claude should apply automatically.

```markdown
---
name: skill-name
description: "This skill should be used when the user asks to \"phrase1\", \"phrase2\", or mentions topic. Do NOT use for out-of-scope-thing."
user-invokable: false
---

# Skill Title

## Overview

Brief description of the conventions or knowledge this skill provides.

## Guidelines

- Guideline 1
- Guideline 2

## Patterns

### Pattern Name

Description and code examples.

## Anti-Patterns

- What to avoid and why.
```

## Task Skill Template

For side-effect workflows that should be manually invoked.

```markdown
---
name: skill-name
description: "This skill should be used when the user asks to \"phrase1\", \"phrase2\", or wants to perform-action."
argument-hint: "[required-arg]"
disable-model-invocation: true
---

# Skill Title

## Critical

Key constraints and rules that must never be violated.

## Task

What this skill accomplishes.

## Steps

1. Step 1 — clear and actionable
2. Step 2
3. Step 3

## Arguments

- `$0` — Description of first argument
- `$1` — Description of second argument (optional)

## Error Handling

- Error: common error
  Cause: why it happens
  Solution: how to fix

## Validation

After completion, verify:
- [ ] Check 1
- [ ] Check 2
```

## MCP Enhancement Skill Template

For guiding workflow around MCP tools.

```markdown
---
name: skill-name
description: "End-to-end workflow for service-name. Use when user says \"trigger phrase 1\", \"trigger phrase 2\", or needs to perform-action. Requires MCP server mcp-server-name."
metadata:
  mcp-server: mcp-server-name
---

# Skill Title

## Overview

Brief description of the MCP-enhanced workflow.

## Prerequisites

- MCP server `mcp-server-name` must be configured and running

## Workflow

1. Step 1 — Use `mcp__server__tool_name` to perform action
2. Step 2 — Process results
3. Step 3 — Present output

## Tool Reference

| Tool | Purpose |
|------|---------|
| `mcp__server__tool1` | Description |
| `mcp__server__tool2` | Description |
```

## Forked Subagent Skill Template

For isolated tasks that do not need conversation history.

```markdown
---
name: skill-name
description: "Performs isolated-task. Use when user asks to \"phrase1\" or \"phrase2\"."
context: fork
agent: general-purpose
---

# Skill Title

## Task

Complete description of what to accomplish. Must be self-contained since
this runs without conversation history.

## Steps

1. Step 1
2. Step 2

## Output

Return format and expected results.
```
