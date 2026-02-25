# AGENTS.md Writing Guide

> **Source**: [HumanLayer - Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [AGENTS.md](https://agents.md/)

A standardized file that helps AI coding agents work effectively on projects.

---

## Core Principle: LLMs are (Mostly) Stateless

LLMs are stateless functions. Weights are frozen at inference time and don't learn over time. **The only thing the model knows about your codebase is the tokens you give it.**

**Implications:**

1. Coding agents know nothing about your codebase at session start
2. You must tell them everything important at the start of each session
3. `AGENTS.md` is the preferred method for this

---

## Onboarding Agents with AGENTS.md

Since agents know nothing at session start, onboard them with `AGENTS.md`:

- **WHAT**: Tech stack, project structure. Provide a map of the codebase. Especially important in monorepos
- **WHY**: The **purpose** of the project. The purpose and function of each part of the repository
- **HOW**: How to work on the project. Using `bun` instead of `node`? How to test, type-check, compile

---

## Writing Good AGENTS.md

### Fewer Instructions Are Better

Research findings:

1. Frontier reasoning LLMs follow ~150-200 instructions reasonably well
2. Smaller models degrade much faster
3. LLMs are biased toward instructions at prompt boundaries (beginning and end)
4. As instruction count increases, models start uniformly ignoring all instructions

**`AGENTS.md` should contain as few instructions as possible** — ideally only universally applicable ones.

### File Length and Applicability

Since `AGENTS.md` is **included in every session**, content should be as universally applicable as possible.

Example: Don't include how to structure new database schemas — it will distract the model during other unrelated tasks!

Length: **Under 300 lines is best**, shorter is better. HumanLayer's root `AGENTS.md` is **under 60 lines**.

### Progressive Disclosure

A concise `AGENTS.md` may not cover everything. Use the **progressive disclosure** principle:

Keep task-specific instructions in **separate markdown files** somewhere in the project:

```
agent_docs/
  ├── building_the_project.md
  ├── running_tests.md
  ├── code_conventions.md
  └── service_architecture.md
```

Include a list of these files with brief descriptions in `AGENTS.md`. Instruct the agent to determine relevant files and read them before starting work.

**Prefer pointers over copies.** Avoid including code snippets when possible — they become stale quickly. Instead, include `file:line` references.

### Agents Are Not (Expensive) Linters

LLMs are relatively expensive and **much slower** compared to traditional linters/formatters. **Use deterministic tools when possible.**

**LLMs are in-context learners!** If your code follows specific style guidelines, agents armed with codebase search tend to follow existing patterns without being told.

---

## Conclusion

1. `AGENTS.md` is about onboarding agents to the codebase. Define **WHY**, **WHAT**, **HOW**
2. **Fewer instructions are better.** Don't omit necessary instructions, but include as few as possible
3. Keep content **concise and universally applicable**
4. **Progressive disclosure** — don't tell agents everything they need to know, tell them **how to find important information**
5. Agents are not linters. Use linters and formatters
6. **`AGENTS.md` is the highest leverage point** — don't auto-generate it, craft it carefully by hand

---

## References

- [HumanLayer: Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [AGENTS.md Official Site](https://agents.md/)
