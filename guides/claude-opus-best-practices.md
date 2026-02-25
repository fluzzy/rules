# Claude Opus 4.6 Best Practices

> **Source**: [Tutorial](https://claude.com/resources/tutorials/get-the-most-from-claude-opus-4-6) | [API Docs](https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) | [Migration](https://platform.claude.com/docs/en/docs/about-claude/models/migrating-to-claude-4)
> **Archive Date**: 2026-02-10

Comprehensive guide covering Claude Opus 4.6 behavioral traits, prompting best practices, agentic coding patterns, and migration from previous models. Compiled from the official tutorial, API documentation, and migration guide.

## Opus 4.6 Key Behavioral Traits

### 1. Follows Instructions More Precisely

- State requirements once — instructions carry through longer sessions without drifting
- Generalizes from fewer examples
- Explain **intent** behind instructions, not just the rule — enables broader application
- Skip reinforcement reminders like "And remember to..."

### 2. Gathers Context Before Acting

- Reads across large files, datasets, and codebases before responding
- Sessions may start slower as it builds understanding first
- Front-load context: share relevant files, link documents, describe the broader system
- Skip "act as an expert" role-setting — Opus 4.6 infers expertise from context
- For simple tasks, narrow scope explicitly: "Just look at this file"
- Ask it to show understanding first: "Walk me through how this is structured, then update X"

### 3. Stays Persistent on Difficult Tasks

- Multi-step tasks more likely to succeed on the first attempt
- May try multiple approaches before checking in
- Can go beyond what was requested — set explicit constraints to keep output focused
- Set check-in points: "Check with me after each major step"
- Recognize looping — step in with a specific alternative if Claude cycles the same approach

### 4. Weighs In More Readily

- May act rather than ask for confirmation — set expectations if you want review first
- Less susceptible to leading questions
- Especially valuable on architectural decisions and strategy
- Ask to explore: "What are three ways we could approach this?"
- Stress-test: "What's wrong with this plan?" or "What am I missing?"

### 5. Delivers Stronger Writing

- Higher baseline writing quality
- Can match styles from provided samples
- Lead with examples, name what to avoid
- Trust Claude with longer creative projects

## Prompting Best Practices

### General Principles

**Be explicit.** If you want above-and-beyond behavior, request it directly:

```
# Less effective
Create an analytics dashboard

# More effective
Create an analytics dashboard. Include as many relevant features
and interactions as possible. Go beyond the basics.
```

**Add context/motivation** behind instructions for better generalization:

```
# Less effective
NEVER use ellipses

# More effective
Your response will be read aloud by a text-to-speech engine,
so never use ellipses since it won't know how to pronounce them.
```

**Be vigilant with examples.** Claude pays close attention — ensure examples align with desired behavior.

### Balancing Autonomy and Safety

```
Consider the reversibility and potential impact of your actions.
Take local, reversible actions freely (editing files, running tests).
For hard-to-reverse, shared, or destructive actions, ask before proceeding.

Examples requiring confirmation:
- Destructive: deleting files/branches, dropping tables, rm -rf
- Hard to reverse: git push --force, git reset --hard
- Visible to others: pushing code, commenting on PRs, sending messages

Do not use destructive actions as shortcuts (e.g., --no-verify).
```

### Reducing Overthinking

Opus 4.6 does significantly more upfront exploration than previous models:

- Replace blanket defaults ("Default to using [tool]") with targeted guidance
- Remove over-prompting — tools that undertriggered before will trigger appropriately now
- "If in doubt, use [tool]" causes overtriggering in Opus 4.6

```
When deciding how to approach a problem, choose an approach and commit.
Avoid revisiting decisions unless new information directly contradicts
your reasoning. Pick one approach and see it through.
```

### Tool Usage Patterns

Claude may suggest rather than implement when asked "can you suggest...":

```
# Will only suggest
Can you suggest some changes to improve this function?

# Will implement
Change this function to improve its performance.
```

**Proactive action prompt:**

```xml
<default_to_action>
By default, implement changes rather than only suggesting them.
If intent is unclear, infer the most useful action and proceed,
using tools to discover missing details instead of guessing.
</default_to_action>
```

**Conservative action prompt:**

```xml
<do_not_act_before_instructions>
Do not jump into implementation unless clearly instructed.
Default to providing information, research, and recommendations.
Only proceed with edits when explicitly requested.
</do_not_act_before_instructions>
```

### Subagent Orchestration

Opus 4.6 has a strong tendency to spawn subagents — may overuse them:

```
Use subagents when tasks can run in parallel, require isolated context,
or involve independent workstreams. For simple tasks, sequential operations,
single-file edits, or tasks needing shared state, work directly.
```

## Long-Horizon & Multi-Window Workflows

### Context Management

```
Your context window will be automatically compacted as it approaches
its limit. Do not stop tasks early due to token budget concerns.
Save progress and state to memory before context refresh.
Be as persistent and autonomous as possible.
```

### Multi-Context Window Strategy

1. **First window**: Set up framework (write tests, create setup scripts)
2. **Subsequent windows**: Iterate on todo-list
3. **Tests in structured format**: `tests.json` with status tracking — "Never remove or edit tests"
4. **Setup scripts**: `init.sh` to start servers, run tests, linters
5. **Fresh start over compaction**: Prescribe how to resume:
   - "Call pwd; only read/write files in this directory"
   - "Review progress.txt, tests.json, and git logs"
   - "Run integration test before implementing new features"

### State Management

```json
// tests.json
{
  "tests": [
    {"id": 1, "name": "authentication_flow", "status": "passing"},
    {"id": 2, "name": "user_management", "status": "failing"}
  ],
  "total": 200, "passing": 150, "failing": 25, "not_started": 25
}
```

```
// progress.txt
Session 3 progress:
- Fixed authentication token validation
- Updated user model to handle edge cases
- Next: investigate user_management test failures
- Note: Do not remove tests
```

## Agentic Coding Patterns

### Avoid Hard-Coding

```
Write a general-purpose solution using standard tools.
Do not create helper scripts or workarounds.
Implement a solution that works for all valid inputs, not just test cases.
If tests are incorrect, inform me rather than working around them.
```

### Minimize Hallucinations

```xml
<investigate_before_answering>
Never speculate about code you have not opened. Read the file before
answering. Investigate relevant files BEFORE answering questions.
Never make claims about code before investigating.
</investigate_before_answering>
```

### Reduce File Creation

```
If you create temporary files, scripts, or helpers for iteration,
clean up by removing them at the end of the task.
```

### Avoid Overengineering

```
Only make changes directly requested or clearly necessary.
- Don't add features, refactor, or "improve" beyond what was asked
- Don't add docstrings/comments to code you didn't change
- Don't add error handling for scenarios that can't happen
- Don't create abstractions for one-time operations
```

### Parallel Tool Calling

```xml
<use_parallel_tool_calls>
If you intend to call multiple tools with no dependencies between them,
make all independent calls in parallel. If some calls depend on previous
results, call them sequentially. Never use placeholders or guess
missing parameters.
</use_parallel_tool_calls>
```

## Frontend Design

```xml
<frontend_aesthetics>
Avoid generic "AI slop" aesthetic. Make creative, distinctive frontends.

Focus on:
- Typography: Beautiful, unique fonts. Avoid Arial, Inter.
- Color: Cohesive aesthetic with CSS variables. Bold accents.
- Motion: Animations and micro-interactions. CSS-only preferred.
- Backgrounds: Atmosphere and depth, not solid colors.

Avoid:
- Overused fonts (Inter, Roboto, Arial, system fonts)
- Cliched color schemes (purple gradients on white)
- Predictable layouts and component patterns
</frontend_aesthetics>
```

## Thinking & Effort

### Adaptive Thinking (Opus 4.6)

```python
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # max, high, medium, low
    messages=[{"role": "user", "content": "..."}],
)
```

### Guide Thinking

```
After receiving tool results, carefully reflect on quality and determine
optimal next steps before proceeding. Use thinking to plan and iterate,
then take the best next action.
```

### Reduce Excessive Thinking

```
Extended thinking adds latency and should only be used when it will
meaningfully improve answer quality — typically for multi-step reasoning.
When in doubt, respond directly.
```

## Formatting Control

### Minimize Markdown

```xml
<avoid_excessive_markdown_and_bullet_points>
Write in clear, flowing prose using complete paragraphs.
Reserve markdown for inline code, code blocks, and simple headings.
Avoid bold, italics, and bullet points unless truly needed.
Incorporate items naturally into sentences instead of listing them.
</avoid_excessive_markdown_and_bullet_points>
```

### LaTeX Control

```
Format your response in plain text only. Do not use LaTeX, MathJax,
or markup notation such as \( \), $, or \frac{}{}.
Write all math expressions using standard text characters.
```

## Migration from Previous Models

### Breaking Changes

- **Prefill removal**: Prefilling assistant messages returns a 400 error
- **Tool parameter quoting**: Slightly different JSON string escaping

### Migration Checklist

1. Update model ID to `claude-opus-4-6`
2. Remove assistant message prefills (BREAKING)
3. Migrate to adaptive thinking (`thinking: {type: "adaptive"}`)
4. Remove beta headers (effort, fine-grained-tool-streaming, interleaved-thinking)
5. Migrate `output_format` to `output_config.format`
6. Verify tool call JSON parsing uses standard JSON parser
7. Review and update prompts — dial back aggressive thoroughness prompts

### Key Prompt Adjustments

- Replace "think" with "consider", "believe", "evaluate" (thinking sensitivity)
- Reduce anti-laziness prompting — Opus 4.6 is naturally more thorough
- Simplify tool triggering language — "CRITICAL: MUST use" → "Use this tool when..."
- Remove role-setting ("act as expert") — no longer needed
