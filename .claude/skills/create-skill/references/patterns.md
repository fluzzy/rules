# Advanced Patterns

## Dynamic Context Injection

`` !`command` `` runs shell commands before sending content to Claude:

```markdown
- Branch: !`git branch --show-current`
- Status: !`git status --short`
- Last commit: !`git log --oneline -1`
```

Use this to inject runtime context (current branch, file lists, env info) into skill instructions.

## Subagent Execution (`context: fork`)

Run in isolation — no conversation history access. Only use when skill has explicit, self-contained task instructions.

```yaml
context: fork
agent: Explore  # or: Plan, general-purpose, custom
```

**When to use:**
- Task does not need conversation context
- Want to protect main context window from large outputs
- Running independent research or analysis

**When NOT to use:**
- Skill needs to reference user's previous messages
- Skill modifies files that depend on conversation state

## Skill-Scoped Hooks

Attach lifecycle hooks that only run when this skill is active:

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
  PostToolUse:
    - matcher: "Write"
      hooks:
        - type: command
          command: "./scripts/check-output.sh"
```

## Extended Thinking

Include the word "ultrathink" anywhere in skill content to enable extended thinking mode. Useful for skills that require deep analysis or complex reasoning.

## Workflow Patterns

Choose the pattern that best fits the skill:

### 1. Sequential Orchestration

Multi-step process in specific order with validation gates between steps.

```
Step 1 → Validate → Step 2 → Validate → Step 3 → Final Check
```

Best for: deployment pipelines, document generation, migration scripts.

### 2. Multi-Source Coordination

Workflow spanning multiple services/MCPs with data passing between them.

```
Fetch from Service A → Transform → Send to Service B → Verify
```

Best for: integration workflows, data sync, cross-service operations.

### 3. Iterative Refinement

Output improves through draft-validate-refine loops.

```
Draft → Validate → Refine → Validate → ... → Final Output
```

Best for: content generation, code review, quality-sensitive outputs.

### 4. Context-Aware Selection

Same outcome, different tools depending on context.

```
Detect Context → Branch A (tool X) OR Branch B (tool Y) → Common Output
```

Best for: cross-platform skills, environment-adaptive workflows.

### 5. Domain-Specific Intelligence

Specialized knowledge beyond tool access (compliance, legal, medical).

```
Input → Apply Domain Rules → Validate Compliance → Output
```

Best for: regulated industries, specialized conventions, expert knowledge.

## Progressive Disclosure Strategies

### High-Level Guide Pattern

SKILL.md has the workflow; `references/` has detailed specs.

```
SKILL.md: "For API schema details, read references/api-spec.md"
```

### Domain-Specific Pattern

SKILL.md has instructions; `references/` has domain knowledge.

```
SKILL.md: "For compliance rules, read references/compliance-rules.md"
```

### Conditional Details Pattern

SKILL.md has common paths; `references/` has edge cases loaded only when needed.

```
SKILL.md: "If encountering error X, read references/error-handling.md"
```

## Composability

Design skills to work alongside other skills:

- Avoid overlapping trigger phrases with existing skills
- Use negative triggers to prevent conflicts
- Reference other skills where appropriate: "For API creation, use the `api-creation-scaffold` skill instead"
- Keep scope focused — one skill, one responsibility
