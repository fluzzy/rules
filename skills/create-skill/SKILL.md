---
name: create-skill
description: "Generates production-ready Claude Code skill definitions (SKILL.md) with proper frontmatter, body structure, and supporting files. This skill should be used when the user asks to \"create a skill\", \"make a new skill\", \"write a SKILL.md\", \"add a slash command\", or \"build a skill\". Handles requirements gathering, template selection, and quality verification."
argument-hint: "[skill-name]"
disable-model-invocation: true
allowed-tools: "Write, Edit, Bash(mkdir:*), Bash(ls:*), Bash(wc:*)"
---

# Create Skill

Generate a production-ready Claude Code skill (`SKILL.md`) following official best practices.

## Critical

- File MUST be named exactly `SKILL.md` (case-sensitive)
- Folder MUST be kebab-case matching `name` field
- Keep SKILL.md under **500 lines / 5,000 words** — move detailed content to `references/`
- ALL trigger phrases go in `description` field, NOT in body (body is invisible during skill selection)
- Only include knowledge Claude does not already have — do not teach general programming
- Names containing "claude" or "anthropic" are reserved and forbidden
- No XML angle brackets in frontmatter

## Phase 1: Gather Requirements (MANDATORY)

Before writing any file, ask clarifying questions using `AskUserQuestion` until the skill's purpose, triggers, and behavior are **fully understood**. Keep asking until there is zero ambiguity.

**Minimum information needed:**

1. **Purpose** — What does the skill do? Category?
   - Document & Asset Creation (consistent output generation)
   - Workflow Automation (multi-step processes)
   - MCP Enhancement (workflow guidance for MCP tools)
2. **Use cases** — 2-3 concrete scenarios
3. **Triggers** — Exact phrases or situations that activate it
4. **Arguments** — Does it accept arguments? What kind?
5. **Invocation** — Auto-invoke or manual-only (`/slash`)?
6. **Scope** — Project (`.claude/skills/`) or personal (`~/.claude/skills/`)?
7. **Tooling** — Specific tools needed (Bash, Read, Edit, MCP tools)? Should tool access be restricted via `allowed-tools`?
8. **Isolation** — Main context or forked subagent (`context: fork`)?

If description is vague (e.g., "make a deploy skill"), ask follow-ups:
- "What deployment target?"
- "Should Claude auto-trigger or manual-only?"
- "What commands or steps should the skill execute?"
- "What does success look like?"

Do NOT proceed to Phase 2 until confident about intent and behavior.

## Phase 2: Create the Skill

Use `$0` (first argument) as the skill name if provided.

### Directory Structure

```
<skill-name>/
├── SKILL.md            # Main instructions (required)
├── references/         # Domain knowledge, specs (Level 3)
├── scripts/            # Executable scripts for Bash tool
└── assets/             # Templates, static files
```

**When to use each:**
- **references/** — Domain knowledge Claude needs but shouldn't memorize. For large refs (>10k words), add grep patterns in SKILL.md.
- **scripts/** — Shell/Python scripts the skill calls via Bash.
- **assets/** — Templates, configs copied/adapted for output.

Do NOT create: README.md, CHANGELOG.md, or duplicate docs.

### Write SKILL.md

**Frontmatter** — Include at minimum: `name`, `description`, appropriate invocation control. See `references/frontmatter-reference.md` for all fields.

**Description formula:**
```
[What it does] + [When to use it] + [Key capabilities]
```

- Use 3rd person: "This skill should be used when..."
- Include exact trigger phrases users would say, in quotes
- Add negative triggers if needed: "Do NOT use for..."
- Stay under 1024 characters

**Body:**
- Use imperative verb forms: "Create the file", "Run the command"
- Do NOT use second-person ("you")
- Critical instructions at the top with `## Critical` header
- Bullet points and numbered lists for clarity

**Invocation control decision:**

| Configuration | User `/name` | Claude auto | Use case |
|--------------|-------------|------------|----------|
| (default) | Yes | Yes | General-purpose |
| `disable-model-invocation: true` | Yes | No | Side-effect workflows |
| `user-invokable: false` | No | Yes | Background knowledge |

### Progressive Disclosure (3-Level Loading)

- **Level 1 (Frontmatter)**: Always in system prompt. Just enough for trigger matching.
- **Level 2 (Body)**: Loaded when triggered. Full instructions.
- **Level 3 (Linked files)**: Read only as needed from `references/`, `scripts/`, `assets/`.

See `references/templates.md` for starter templates.
See `references/patterns.md` for advanced patterns (dynamic injection, subagent, hooks).

## Phase 3: Verify

1. Confirm file created at correct path
2. Show generated frontmatter for review
3. Run Quality Checklist (below)
4. Suggest testing: "Try `/<skill-name>` and also ask a natural language query that should trigger it"
5. Suggest debugging: "Ask Claude: 'When would you use the [skill-name] skill?'"

## Phase 4: Iterate

After initial deployment, improve based on real usage:

1. **Test triggers** — Run these checks:
   - `/<skill-name>` direct invocation works
   - Paraphrased natural language triggers correctly
   - Unrelated queries do NOT trigger
   - Ask Claude: "When would you use the [skill-name] skill?" to verify matching
2. **Observe real usage** — Note where Claude misinterprets, over-triggers, or misses steps
3. **Refine** — Update description triggers, add negative triggers, tighten instructions
4. **Optimize size** — Move bloated sections to `references/` if SKILL.md grows. Use `/context` to check token budget (aim under 2%).
5. **Re-test** with both `/skill-name` and natural language invocation

### Troubleshooting

| Issue | Fix |
|-------|-----|
| Under-triggering | Add more trigger phrases and keywords to description |
| Over-triggering | Add negative triggers ("Do NOT use for..."), be more specific |
| Instructions not followed | Put critical instructions at top with `## Critical` header |
| Large context issues | Move content to `references/`, keep under 500 lines |

## Quality Checklist

**Structure:**
- [ ] File named exactly `SKILL.md`, folder is kebab-case matching `name`
- [ ] No README.md or unnecessary files
- [ ] Under 500 lines / 5,000 words; detailed content in `references/`
- [ ] All referenced files actually exist

**Frontmatter:**
- [ ] Description follows [What] + [When] + [Capabilities], 3rd person, under 1024 chars
- [ ] ALL trigger phrases in `description`, not buried in body
- [ ] `name` is lowercase/hyphenated, no "claude"/"anthropic"
- [ ] Invocation control set correctly

**Body:**
- [ ] Imperative form, no second-person
- [ ] Critical instructions at the top
- [ ] Only domain-specific knowledge Claude doesn't already have

**Testing:**
- [ ] Works via `/skill-name` invocation
- [ ] Works via natural language invocation
- [ ] Does not trigger on unrelated queries
