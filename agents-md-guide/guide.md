# AGENTS.md Writing Guide

This document is based on [HumanLayer's "Writing a good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md) and [AGENTS.md](https://agents.md/).

`AGENTS.md` is a standardized file that helps AI coding agents work effectively on your project.

---

## Core Principle: LLMs Are (Mostly) Stateless

LLMs are stateless functions. Weights are frozen at inference time and don't learn over time. **The only thing the model knows about your codebase is the tokens you put into it.**

Similarly, coding agent harnesses must explicitly manage the agent's memory. `AGENTS.md` is **the only file included in every conversation by default**.

**What This Means:**

1. Coding agents know nothing about the codebase at the start of each session.
2. Agents must be told everything important about the codebase at each session start.
3. `AGENTS.md` is the preferred method for doing this.

---

## AGENTS.md Onboards Agents to Your Codebase

Since agents know nothing about the codebase at session start, use `AGENTS.md` to onboard them. At a high level, include:

- **WHAT**: Tech stack, project structure. Provide a map of the codebase. Especially important in monorepos — explain what the apps are, what shared packages exist, what everything is for.
- **WHY**: The **purpose** of the project. What are the purposes and functions of different parts of the repository?
- **HOW**: How to work on the project. Do you use `bun` instead of `node`? Include all information needed to do meaningful work. How can the agent validate changes? How to run tests, type checks, compile steps?

But **how you do this matters**. Don't try to fill `AGENTS.md` with every command the agent could run — you'll get suboptimal results.

---

## Agents Often Ignore AGENTS.md

Regardless of which model you use, you'll find that agents often ignore the contents of `AGENTS.md`.

Agent harnesses inject the `AGENTS.md` file into the agent's user message with a system reminder like:

```
<system-reminder>
  IMPORTANT: this context may or may not be relevant to your tasks.
  You should not respond to this context unless it is highly relevant to your task.
</system-reminder>
```

As a result, agents will ignore `AGENTS.md` content if they deem it irrelevant to the current task. The more information in the file that is **not universally applicable**, the more likely the agent is to ignore the file's instructions.

Therefore, `AGENTS.md` content should be **as universally applicable as possible**.

---

## Writing a Good AGENTS.md File

The following sections provide recommendations for writing a good `AGENTS.md` file that follows context engineering best practices.

_Your mileage may vary._ These rules are not optimal for every setup. As with everything, only break a rule when you:

1. Understand when and why you can break it
2. Have a good reason to do so

### Fewer Instructions Are Better

You may be tempted to fill `AGENTS.md` with every command the agent could run, along with code standards and style guidelines. **This is not recommended.**

Research shows:

1. **Frontier thinking LLMs can follow ~150-200 instructions with reasonable consistency.** Smaller models can pay attention to fewer instructions than larger models, and non-thinking models can pay attention to fewer instructions than thinking models.
2. **Smaller models degrade much faster.** Specifically, smaller models tend to see exponential decreases in instruction-following performance as instruction count increases, while large frontier thinking models see linear decreases.
3. **LLMs are biased toward instructions at the peripheries of prompts**: the very beginning (agent harness system message and `AGENTS.md`) and the very end (most recent user message).
4. **Instruction-following quality degrades uniformly as instruction count increases.** This means that when you give an LLM more instructions, it doesn't simply ignore new ("further down the file") instructions — it **uniformly starts ignoring all instructions**.

Agent harness system prompts contain about **50 individual instructions**. Depending on your model, this is almost 1/3 of the instructions the agent can reliably follow — and that's before rules, plugins, skills, or user messages.

This means your `AGENTS.md` file should contain **as few instructions as possible** — ideally only things that are universally applicable to tasks.

### AGENTS.md File Length and Applicability

All else being equal, **LLMs perform better on tasks when the context window is filled with focused, relevant context including examples, relevant files, tool calls, and tool results, rather than lots of irrelevant context.**

Since `AGENTS.md` is **included in every session**, its contents should be as universally applicable as possible.

For example, don't include instructions on how to structure new database schemas — this is not important and will distract the model when doing other unrelated tasks!

Length-wise, **less is also better**. While there's no official recommendation from Anthropic on how long `AGENTS.md` should be, general consensus is that **under 300 lines is best**, and shorter is better.

HumanLayer's root `AGENTS.md` file is **under 60 lines**.

### Progressive Disclosure

Writing a concise `AGENTS.md` that covers everything the agent needs to know can be challenging, especially for larger projects.

To solve this, leverage the principle of **progressive disclosure** to ensure agents only see task or project-specific instructions when they need them.

Instead of including all other instructions on building the project, running tests, code conventions, or other important context in the `AGENTS.md` file, keep task-specific instructions in **separate markdown files somewhere in the project** with self-explanatory names.

For example:

```
agent_docs/
  |- building_the_project.md
  |- running_tests.md
  |- code_conventions.md
  |- service_architecture.md
  |- database_schema.md
  |- service_communication_patterns.md
```

In your `AGENTS.md` file, include a list of these files with a brief description of each, and instruct the agent to determine which files (if any) are relevant and read them before starting work. Alternatively, ask the agent to present which files it wants to read for approval before reading them.

**Prefer pointers over copies.** Avoid including code snippets in these files when possible — they quickly become stale. Instead, include `file:line` references so the agent can point to authoritative context.

### Agents Are Not (Expensive) Linters

One of the most common things people put in `AGENTS.md` is code style guidelines. **Never send an LLM to do a linter's job.** LLMs are relatively expensive and **massively slower** compared to traditional linters and formatters. **Always use deterministic tools** when possible.

Code style guidelines inevitably add instructions and mostly-irrelevant code snippets to the context window, degrading LLM performance and instruction following while consuming context window.

**LLMs are in-context learners!** If your code follows specific style guidelines or patterns, an agent armed with a few searches (or a good research document!) from your codebase should tend to follow existing code patterns and conventions even without being told.

If you feel strongly about this, consider setting up agent harness stop hooks that run formatters and linters and present errors to the agent to fix. Don't make the agent find formatting issues directly.

**Bonus points**: Use a linter that can auto-fix issues (we like Biome), and carefully tune rules for what can be safely auto-fixed for maximum (safe) coverage.

### Don't Use /init or Auto-Generate AGENTS.md

Both Claude Code and other harnesses have ways to auto-generate `AGENTS.md` files.

Because `AGENTS.md` is **included in every session**, it's **one of the highest leverage points** in the harness — for better or worse.

A bad line of code is one bad line of code. A bad line in an implementation plan has the potential to create **many** bad lines of code. A bad line of research that misunderstands how a system works has the potential to create many bad lines in a plan, which in turn can create **even more** bad lines of code.

But `AGENTS.md` affects **every step of the workflow** and every artifact it produces. As a result, you should spend time thinking very carefully about **every single line** that goes into the file.

---

## Conclusion

1. `AGENTS.md` is about onboarding agents to your codebase. Define the **WHY**, **WHAT**, and **HOW** of the project.
2. **Fewer instructions are better.** You shouldn't omit necessary instructions, but include as few instructions as possible in the file.
3. Keep `AGENTS.md` contents **concise and universally applicable**.
4. Use **progressive disclosure** — don't tell the agent everything it might need to know. Instead, tell it **how to find important information** so it can find and use it as needed without bloating the context window or instruction count.
5. Agents are not linters. Use linters and code formatters, and use features like Hooks and Slash Commands as needed.
6. **`AGENTS.md` is the highest leverage point** in the harness, so don't auto-generate it. Contents should be carefully hand-written for best results.

---

## References

- [HumanLayer: Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [AGENTS.md Official Site](https://agents.md/)
