---
name: create-skill
description: Generates production-ready Claude Code skill definitions (SKILL.md) with proper frontmatter, body structure, and supporting files. This skill should be used when the user asks to "create a skill", "make a new skill", "write a SKILL.md", "add a slash command", or "build a skill". Handles requirements gathering, template selection, and quality verification.
argument-hint: "[skill-name]"
---

# Create Skill

Generate a production-ready Claude Code skill (`SKILL.md`) following official best practices from the Anthropic Complete Guide and Skills documentation.

## Workflow

### Phase 1: Gather Requirements (MANDATORY — DO NOT SKIP)

Before writing any file, ask clarifying questions until the skill's purpose, triggers, and behavior are **fully understood**. Keep asking until there is zero ambiguity. If unsure about any aspect, ask another question rather than assume.

**Minimum information needed before proceeding:**

1. **Purpose** — What does the skill do? Which category?
   - Category 1: Document & Asset Creation (consistent output generation)
   - Category 2: Workflow Automation (multi-step processes)
   - Category 3: MCP Enhancement (workflow guidance for MCP tools)
2. **Use cases** — 2-3 concrete scenarios the skill should handle
3. **Triggers** — Exact phrases or situations that should activate it
4. **Arguments** — Does it accept arguments? What kind?
5. **Invocation control** — Should Claude auto-invoke, or manual-only (`/slash`)?
6. **Scope** — Project (`.claude/skills/`) or personal (`~/.claude/skills/`)?
7. **Tooling** — Specific tools needed (Bash, Read, Edit, MCP tools, etc.)?
8. **Isolation** — Main context or forked subagent (`context: fork`)?

Use `AskUserQuestion` repeatedly until all ambiguity is resolved. If the user's description is vague (e.g., "make a deploy skill"), ask follow-ups such as:
- "What deployment target? (Vercel, AWS, Docker, etc.)"
- "Should Claude auto-trigger this when you mention deploying, or manual-only?"
- "What commands or steps should the skill execute?"
- "What does success look like? How will you know the skill is working?"

Do NOT proceed to Phase 2 until confident about the skill's intent and behavior. It is better to ask one more question than to build the wrong thing.

### Phase 2: Create the Skill

1. Create the directory at the chosen scope
2. Write `SKILL.md` with proper frontmatter and body
3. Create supporting files if needed (templates, scripts, references)
4. Report the created file path to the user

### Phase 3: Verify

1. Confirm the file was created at the correct path
2. Show the user the generated frontmatter for review
3. Run through the Quality Checklist
4. Suggest testing: "Try `/<skill-name>` and also ask Claude a natural language query that should trigger it"
5. Suggest debugging: "Ask Claude: 'When would you use the [skill-name] skill?' to verify trigger matching"

### Phase 4: Iterate

After initial deployment, improve the skill based on real usage.

**Pro tip:** The most effective approach is to iterate on a single challenging task until Claude succeeds, then extract the winning approach into the skill.

1. Use the skill in real tasks — note where Claude misinterprets, over-triggers, or misses steps
2. Update `description` triggers, add negative triggers, refine body instructions
3. Move bloated sections to `references/` if SKILL.md grows too large
4. Re-test with both `/skill-name` and natural language invocation

## Core Principles

### 1. Concise is Key

Context window is a shared resource — every token in SKILL.md competes with the user's conversation. **Only include what Claude doesn't already know.** Do not teach Claude general programming, tool usage, or common patterns. Focus exclusively on domain-specific knowledge, exact workflows, and project-specific conventions.

Use `/context` to check how much of the context window a skill consumes. Aim for under 2% per skill.

### 2. Degrees of Freedom

Match instruction specificity to task fragility:

- **High freedom** (guidelines only) — Creative tasks, architecture decisions. Give goals and constraints, let Claude choose the approach.
- **Medium freedom** (structured steps) — Most skills. Provide step-by-step workflow with decision points.
- **Low freedom** (exact reproduction) — Compliance docs, legal templates, API contracts. Specify exact output format, wording, structure.

### 3. Progressive Disclosure (3-Level Loading)

Skills use a three-level loading system to minimize token usage:

- **Level 1 (Frontmatter)**: Always in system prompt. Provides just enough for Claude to know *when* to use the skill.
- **Level 2 (SKILL.md body)**: Loaded when skill is triggered. Contains full instructions.
- **Level 3 (Linked files)**: Files in the skill directory that Claude reads only as needed.

**Three patterns for Level 3 content:**
- **High-level guide** — SKILL.md has the workflow; `references/` has detailed specs
- **Domain-specific** — SKILL.md has instructions; `references/` has domain knowledge (API docs, schemas)
- **Conditional details** — SKILL.md has common paths; `references/` has edge cases loaded only when needed

### 4. All Triggers in Description, NOT in Body

The `description` field is the ONLY thing Claude sees before deciding to load a skill. Any "When to Use" or trigger information placed in the body is invisible during skill selection. Put ALL activation signals in the description.

## Directory Structure

```
<skill-name>/
├── SKILL.md            # Main instructions (required, must be exact case)
├── references/         # Domain knowledge, API docs, specs (Level 3)
│   └── api-spec.md
├── scripts/            # Executable scripts Claude can invoke via Bash
│   └── validate.sh
└── assets/             # Templates, static files used in output generation
    └── template.md
```

**When to use each directory:**
- **references/** — Domain knowledge Claude needs but shouldn't memorize (API docs, schemas, style guides). For large references (>10k words), add grep search patterns in SKILL.md so Claude can target-read specific sections.
- **scripts/** — Shell scripts, Python scripts, or other executables the skill calls via `Bash` tool. Use `allowed-tools` in frontmatter to pre-authorize. For critical validations, prefer scripts over language instructions — code is deterministic, language interpretation isn't.
- **assets/** — Templates, config files, or static content that Claude copies/adapts for output. Not for documentation.

**Critical rules:**
- File MUST be named exactly `SKILL.md` (case-sensitive — not `SKILL.MD`, `skill.md`, etc.)
- Folder name MUST be kebab-case (`my-skill`, not `My Skill`, `my_skill`, `MySkill`)
- Keep `SKILL.md` under **500 lines** and **under 5,000 words**. Move detailed content to `references/`
- Skills should be composable — design them to work alongside other skills
- Note: `.claude/commands/` is the legacy path and still works for backward compatibility

**Do NOT include in a skill folder:**
- `README.md`, `INSTALLATION_GUIDE.md`, `QUICK_REFERENCE.md`, `CHANGELOG.md`
- Duplicate information already in SKILL.md
- General programming knowledge Claude already has

## SKILL.md Anatomy

```markdown
---
(YAML frontmatter — metadata, always in system prompt)
---

(Markdown body — instructions Claude follows when skill is invoked)
```

### Body Writing Style

- Use imperative/infinitive verb forms: "Create the file", "Run the command"
- Do NOT use second-person: avoid "You should create the file"
- Put critical instructions at the top with `## Important` or `## Critical` headers
- Use bullet points and numbered lists for clarity

## Frontmatter Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | No (defaults to dir name) | Lowercase, numbers, hyphens only. Max 64 chars. Becomes `/slash-command`. Must match folder name. |
| `description` | Recommended | What it does + when to use it + key capabilities. Max 1024 chars. No XML angle brackets (`<>`). |
| `argument-hint` | No | Autocomplete hint. Example: `[issue-number]`, `[filename] [format]`. |
| `disable-model-invocation` | No | `true` = manual `/name` only. Use for side-effect workflows. |
| `user-invocable` | No | `false` = hidden from `/` menu. Use for background knowledge. |
| `allowed-tools` | No | Tools without permission prompts. Example: `Bash(python:*) WebFetch`. |
| `model` | No | Override model for this skill. |
| `context` | No | `fork` = run in isolated subagent. |
| `agent` | No | Subagent type with `context: fork`. Options: `Explore`, `Plan`, `general-purpose`, or custom. |
| `hooks` | No | Lifecycle hooks scoped to this skill. |
| `license` | No | For open-source skills. Common: `MIT`, `Apache-2.0`. |
| `compatibility` | No | Environment requirements (1-500 chars). E.g., required platform, packages, network access. |
| `metadata` | No | Custom key-value pairs: `author`, `version`, `mcp-server`, `category`, `tags`, etc. |

**Security restrictions:**
- No XML angle brackets (`<` or `>`) in frontmatter
- Names with "claude" or "anthropic" are reserved and forbidden

## Writing Effective Descriptions

The `description` field is the most critical part. It determines when Claude automatically activates the skill — it is the first level of progressive disclosure.

### Formula

```
[What it does] + [When to use it] + [Key capabilities]
```

### Rules

1. **Use 3rd person**: "This skill should be used when..." (not "Use this skill when...")
2. **Include exact trigger phrases** users would say, in quotes
3. **Be specific** to prevent over-triggering or under-triggering
4. **Add negative triggers** if needed: "Do NOT use for simple X (use Y skill instead)"
5. **Mention relevant file types** if applicable
6. **Stay under 1024 characters**, no XML tags
7. **ALL trigger info goes in description** — body is not loaded during skill selection, so "When to Use" sections in the body are invisible to Claude's trigger matching

### Good Examples

```yaml
# Specific triggers, 3rd person, clear scope
description: Analyzes Figma design files and generates developer handoff
  documentation. Use when user uploads .fig files, asks for "design specs",
  "component documentation", or "design-to-code handoff".

# Includes what, when, and negative trigger
description: Advanced data analysis for CSV files. Use for statistical modeling,
  regression, clustering. Do NOT use for simple data exploration
  (use data-viz skill instead).

# MCP-enhanced with clear value
description: End-to-end customer onboarding workflow for PayFlow. Handles
  account creation, payment setup, and subscription management. Use when user
  says "onboard new customer", "set up subscription", or "create PayFlow account".
```

### Anthropic's Official Example

```yaml
# From the official docx skill — note the comprehensive trigger coverage
description: This skill converts markdown files to docx format. Use when the
  user asks to "create a Word document", "convert to docx", "make a .docx file",
  "export as Word", or wants to generate professional documents from markdown.
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

## Invocation Control

| Configuration | User invokes (`/name`) | Claude auto-invokes | Use case |
|--------------|----------------------|--------------------|----|
| (default) | Yes | Yes | General-purpose skills |
| `disable-model-invocation: true` | Yes | No | Side-effect workflows (deploy, commit) |
| `user-invocable: false` | No | Yes | Background knowledge (conventions) |

**Decision guide:**
- Skill has side effects (writes, deploys, sends)? → `disable-model-invocation: true`
- Skill is passive knowledge? → `user-invocable: false`
- Otherwise → use defaults

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed to the skill |
| `$ARGUMENTS[N]` | Argument by 0-based index |
| `$N` | Shorthand for `$ARGUMENTS[N]` (`$0` = first arg) |
| `${CLAUDE_SESSION_ID}` | Current session ID |

If `$ARGUMENTS` is not present in content and arguments are provided, Claude Code appends `ARGUMENTS: <value>` to the end.

## Advanced Patterns

### Dynamic Context Injection

`` !`command` `` runs shell commands before sending content to Claude:

```markdown
- Branch: !`git branch --show-current`
- Status: !`git status --short`
```

### Subagent Execution (`context: fork`)

Run in isolation — no conversation history access. Only use when skill has explicit task instructions.

```yaml
context: fork
agent: Explore
```

### Skill-Scoped Hooks

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
```

### Extended Thinking

Include "ultrathink" in skill content to enable extended thinking mode.

## Workflow Patterns

When building a skill, choose the pattern that best fits:

1. **Sequential Orchestration** — Multi-step process in specific order with validation gates
2. **Multi-Source Coordination** — Workflow spanning multiple services/MCPs with data passing
3. **Iterative Refinement** — Output improves through draft → validate → refine loops
4. **Context-Aware Selection** — Same outcome, different tools depending on context
5. **Domain-Specific Intelligence** — Specialized knowledge beyond tool access (compliance, etc.)

## Templates

### Reference Skill Template

```markdown
---
name: <skill-name>
description: This skill should be used when the user asks to "<phrase1>",
  "<phrase2>", or mentions <topic>. Do NOT use for <out-of-scope>.
user-invocable: false
---

# <Skill Title>

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

### Task Skill Template

```markdown
---
name: <skill-name>
description: This skill should be used when the user asks to "<phrase1>",
  "<phrase2>", or wants to <action>.
argument-hint: "[required-arg]"
disable-model-invocation: true
---

# <Skill Title>

## Task

<What this skill accomplishes>

## Steps

1. <Step 1 — clear and actionable>
2. <Step 2>
3. <Step 3>

## Arguments

- `$0` — <description of first argument>
- `$1` — <description of second argument> (optional)

## Error Handling

- Error: <common error>
  Cause: <why it happens>
  Solution: <how to fix>

## Validation

After completion, verify:
- [ ] <Check 1>
- [ ] <Check 2>
```

## Testing Guide

After creating a skill, suggest the user test in three areas:

### 1. Triggering Tests
- Should trigger on obvious task phrases
- Should trigger on paraphrased requests
- Should NOT trigger on unrelated topics
- Debug with: "When would you use the [skill-name] skill?"

### 2. Functional Tests
- Valid outputs generated
- All tool/API calls succeed
- Error handling works
- Edge cases covered

### 3. Performance Comparison
- Compare with vs. without the skill
- Target benchmarks: 90%+ trigger accuracy, fewer tool calls, 0 failed API calls
- Qualitative: workflows complete without user correction, consistent results across sessions

### Troubleshooting

- **Under-triggering**: Add more trigger phrases and keywords to description
- **Over-triggering**: Add negative triggers ("Do NOT use for..."), be more specific
- **Instructions not followed**: Put critical instructions at top, use `## Critical` headers, keep instructions concise
- **Large context issues**: Move content to `references/`, keep SKILL.md under 500 lines and 5,000 words. Use `/context` to check token budget

## Quality Checklist

Before finalizing a generated skill, verify:

**Structure:**
- [ ] File named exactly `SKILL.md`, folder is kebab-case matching `name`
- [ ] No `README.md` or unnecessary files in skill folder
- [ ] SKILL.md under 500 lines / 5,000 words; detailed content in `references/`
- [ ] All referenced files actually exist

**Frontmatter:**
- [ ] `description` follows formula: [What] + [When] + [Capabilities], 3rd person, under 1024 chars
- [ ] ALL trigger phrases are in `description`, not buried in body
- [ ] `name` is lowercase/hyphenated, no "claude"/"anthropic"
- [ ] Invocation control set correctly (`disable-model-invocation` / `user-invocable`)

**Body:**
- [ ] Imperative form, no second-person ("you")
- [ ] Critical instructions at the top
- [ ] Only includes knowledge Claude doesn't already have (Concise is Key)

**Testing:**
- [ ] Works via both `/skill-name` and natural language invocation
- [ ] Error handling for common failure cases
