# Testing Agent Skills Systematically with Evals

> **Source**: [OpenAI Developer Blog](https://developers.openai.com/blog/eval-skills/)
> **Authors**: Dominik Kundel, Gabriel Chua
> **Published**: 2026-01-22
> **Archive Date**: 2026-02-26

A practical 8-step framework for building systematic, data-driven evaluations (evals) for AI agent skills, using OpenAI Codex as the reference platform. The core message: stop relying on "vibes" and start measuring agent behavior with automated, repeatable tests.

---

## Core Concept

An eval is: **prompt -> captured run (trace + artifacts) -> small set of checks -> score you can compare over time**. It mirrors lightweight end-to-end testing for AI systems.

---

## Step 1: Define Success Before Writing the Skill

Categorize success into measurable dimensions:

- **Outcome goals**: Did the task complete? Does it work?
- **Process goals**: Did the agent use the intended tools and steps?
- **Style goals**: Does the output follow requested conventions?
- **Efficiency goals**: Did it finish without unnecessary repetition?

Keep the list small—focus on critical behaviors, not every preference.

## Step 2: Create the Skill

A Codex skill is a directory containing `SKILL.md` with YAML front matter (`name`, `description`) and Markdown instructions. Name and description are critical signals that determine whether the skill gets invoked.

### Sample Skill: `setup-demo-app`

```yaml
---
name: setup-demo-app
description: Scaffold a Vite + React + Tailwind demo app with a small, consistent project structure.
---

## When to use this
Use when you need a fresh demo app for quick UI experiments or reproductions.

## What to build
Create a Vite React TypeScript app and configure Tailwind. Keep it minimal.

Project structure after setup:
- src/
  - main.tsx (entry)
  - App.tsx (root UI)
  - components/
    - Header.tsx
    - Card.tsx
  - index.css (Tailwind import)
- index.html
- package.json

Style requirements:
- TypeScript components
- Functional components only
- Tailwind classes for styling (no CSS modules)
- No extra UI libraries

## Steps
1. Scaffold with Vite: `npm create vite@latest demo-app -- --template react-ts`
2. Install dependencies: `cd demo-app && npm install`
3. Install and configure Tailwind via Vite plugin
4. Implement minimal UI: Header, Card, App components

## Definition of done
- npm run dev starts successfully
- package.json exists
- src/components/Header.tsx and src/components/Card.tsx exist
```

Skills can be stored at `.codex/skills/[skill-name]/SKILL.md` (repo-scoped) or `~/.codex/skills/[skill-name]/SKILL.md` (user-scoped).

## Step 3: Manually Trigger the Skill

Before automating, manually test to surface hidden assumptions:

- **Triggering**: Skill should activate but doesn't, or activates too eagerly
- **Environment**: Expectations about directory state or tool availability
- **Execution**: Missing steps or incorrect ordering

```bash
codex exec --full-auto \
  'Use the $setup-demo-app skill to create the project in this directory.'
```

## Step 4: Build a Small, Targeted Prompt Set

Create a CSV with 10-20 test cases covering different invocation patterns:

```csv
id,should_trigger,prompt
test-01,true,"Create a demo app named `devday-demo` using the $setup-demo-app skill"
test-02,true,"Set up a minimal React demo app with Tailwind for quick UI experiments"
test-03,true,"Create a small demo app to showcase the Responses API"
test-04,false,"Add Tailwind styling to my existing React app"
```

Four categories of test cases:

| Type | Purpose |
|------|---------|
| **Explicit** | Direct skill name invocation |
| **Implicit** | Describes the scenario without naming the skill |
| **Contextual** | Realistic, slightly noisy prompts |
| **Negative** | Should NOT trigger the skill |

## Step 5: Deterministic Graders

Use `codex exec --json` for structured JSONL event output, enabling automated scoring.

```javascript
// evals/run-setup-demo-app-evals.mjs
import { spawnSync } from "node:child_process";
import { readFileSync, writeFileSync, existsSync, mkdirSync } from "node:fs";
import path from "node:path";

function runCodex(prompt, outJsonlPath) {
  const res = spawnSync(
    "codex",
    ["exec", "--json", "--full-auto", prompt],
    { encoding: "utf8" }
  );
  mkdirSync(path.dirname(outJsonlPath), { recursive: true });
  writeFileSync(outJsonlPath, res.stdout, "utf8");
  return { exitCode: res.status ?? 1, stderr: res.stderr };
}

function parseJsonl(jsonlText) {
  return jsonlText.split("\n").filter(Boolean).map((line) => JSON.parse(line));
}

// Deterministic check: did the agent run `npm install`?
function checkRanNpmInstall(events) {
  return events.some(
    (e) =>
      (e.type === "item.started" || e.type === "item.completed") &&
      e.item?.type === "command_execution" &&
      typeof e.item?.command === "string" &&
      e.item.command.includes("npm install")
  );
}

// Deterministic check: did `package.json` get created?
function checkPackageJsonExists(projectDir) {
  return existsSync(path.join(projectDir, "package.json"));
}

const projectDir = process.cwd();
const tracePath = path.join(projectDir, "evals", "artifacts", "test-01.jsonl");
const prompt = "Create a demo app named demo-app using the $setup-demo-app skill";

runCodex(prompt, tracePath);
const events = parseJsonl(readFileSync(tracePath, "utf8"));

console.log({
  ranNpmInstall: checkRanNpmInstall(events),
  hasPackageJson: checkPackageJsonExists(path.join(projectDir, "demo-app")),
});
```

## Step 6: Model-Based Rubric Grading

For qualitative checks (code style, component structure), use a two-stage approach:

1. Run the skill (writes code to disk)
2. Run a read-only evaluation against the output
3. Require structured response via `--output-schema`

### Style Rubric Schema

```json
{
  "type": "object",
  "properties": {
    "overall_pass": { "type": "boolean" },
    "score": { "type": "integer", "minimum": 0, "maximum": 100 },
    "checks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "pass": { "type": "boolean" },
          "notes": { "type": "string" }
        },
        "required": ["id", "pass", "notes"],
        "additionalProperties": false
      }
    }
  },
  "required": ["overall_pass", "score", "checks"],
  "additionalProperties": false
}
```

### Eval Command

```bash
codex exec \
  "Evaluate the demo-app repository against these requirements:
   - Vite + React + TypeScript project exists
   - Tailwind is configured via @tailwindcss/vite and CSS imports tailwindcss
   - src/components contains Header.tsx and Card.tsx
   - Components are functional and styled with Tailwind utility classes
   Return a rubric result as JSON with check ids: vite, tailwind, structure, style." \
  --output-schema ./evals/style-rubric.schema.json \
  -o ./evals/artifacts/test-01.style.json
```

## Step 7: Extend Evals as the Skill Matures

Add progressive evaluation dimensions:

| Check | What it catches |
|-------|----------------|
| **Command count / thrashing** | Looping or retrying the same command |
| **Token budget** | Inefficient token usage (from `turn.completed` events) |
| **Build checks** | `npm run build` catches config issues |
| **Runtime smoke tests** | Start dev server, test with `curl` or Playwright |
| **Repo cleanliness** | No unwanted files, clean `git status` |
| **Sandbox regressions** | Skill works without permission escalation |

Pattern: start with fast behavioral checks, add slower checks only when they reduce risk.

## Step 8: Key Takeaways

1. **Measure what matters** — evals make regressions clear and failures explainable
2. **Ground success in verifiable outcomes** before implementation
3. **Capture JSONL** with `codex exec --json` and check `command_execution` events
4. **Use model-based grading** where deterministic rules fall short, with structured output
5. **Convert manual failures into automated tests** to prevent regression

---

## References

- [Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills/) — OpenAI Developer Blog
- [Codex Skills Documentation](https://developers.openai.com/codex/skills)
- [Agent Evals Guide](https://developers.openai.com/api/docs/guides/agent-evals)
- [Structured Output Documentation](https://developers.openai.com/api/docs/guides/structured-outputs)
