# How to Write a Good Spec for AI Agents

> **Source**: [Addy Osmani - How to Write a Good Spec for AI Agents](https://addyosmani.com/blog/good-spec/)

How to write effective specifications for AI coding agents. Larger specs don't automatically yield better results — strategic specification design does.

---

## Key Principle (TL;DR)

Create clear specs covering sufficient nuance (structure, style, testing, boundaries) to guide AI without overload. Break large tasks into smaller ones rather than monolithic prompts. Plan first in read-only mode, then execute and iterate continuously.

---

## Five Core Principles

### 1. Start with High-Level Vision, Let AI Draft Details

Rather than over-engineering upfront, provide clear goals and core requirements. Leverage AI's strength in elaboration.

**Implementation approach:**
- Prompt the agent to draft a detailed specification covering objectives, features, constraints, and step-by-step plans
- Use Plan Mode (read-only) to explore and refine before code generation
- Save the spec as a persistent document (e.g., `SPEC.md`)
- Keep specs goal-oriented, focusing on "what" and "why" over implementation details

**The spec should answer:** Who uses this? What problem does it solve? What does success look like?

### 2. Structure Like a Professional PRD/SRS

AI specs should be structured documents with clear sections, not loose notes.

**Six core areas every spec should cover:**

1. **Commands** - Full executable commands with flags: `npm test`, `pytest -v`
2. **Testing** - Test framework, where tests live, coverage expectations
3. **Project structure** - Explicit directory organization with examples
4. **Code style** - One real code snippet beats three paragraphs of description
5. **Git workflow** - Branch naming, commit formats, PR requirements
6. **Boundaries** - What the agent should never touch (secrets, vendors, production configs)

**Sample structure:**
```markdown
# Project Spec: Task Management App

## Objective
Build web app for teams to manage tasks...

## Tech Stack
React 18+, TypeScript, Vite, Tailwind CSS
Node.js/Express backend, PostgreSQL, Prisma ORM

## Commands
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint --fix`

## Project Structure
- `src/` – Application source
- `tests/` – Tests
- `docs/` – Documentation

## Boundaries
- Always: Run tests before commits
- Ask first: Database schema changes
- Never: Commit secrets
```

**Three-tier boundary system:**
- **Always do:** Actions without asking
- **Ask first:** High-impact changes requiring approval
- **Never do:** Hard stops

### 3. Break Tasks Into Modular Prompts, Not Monolithic Ones

Give the AI one focused task at a time.

**The curse of too much context:** Research shows that as instructions pile up, models struggle to follow each one well. Even GPT-4 and Claude performance drops significantly with excessive simultaneous requirements.

**Strategies:**

- **Extended TOC/Summaries:** Create a condensed outline where each section references full details. Example: "Security: use HTTPS, protect API keys, validate input (see §4.2)"

- **Sub-agents for domain expertise:** Different agents for different spec sections — Database Designer, API Coder, etc.

- **Parallel agents:** Run non-overlapping work simultaneously — one codes features while another writes tests

| Aspect | Single Agent | Parallel/Multi-Agent |
|--------|--------------|---------------------|
| **Strengths** | Simpler setup; easier debugging | Higher throughput; specialist roles |
| **Challenges** | Context overload on big projects | Coordination overhead; conflicts |
| **Best for** | Isolated modules; small projects | Large codebases; independent features |

**Practical approach:** Divide specs into parts (`SPEC_backend.md`, `SPEC_frontend.md`) and feed only relevant sections per task.

### 4. Build In Self-Checks, Constraints, and Human Expertise

Make specs guide quality control, not just instruction lists.

**Self-verification patterns:**
- Instruct agents to compare outputs against spec requirements
- Use "LLM-as-a-Judge" for subjective criteria (code style, readability)
- Implement conformance testing derived from specs

**Inject domain knowledge:** Your spec should reflect expert insights — common pitfalls, tricky libraries, architectural patterns. Example: "If using library X, watch out for memory leak in version Y (apply workaround Z)"

**Leverage testing:** Incorporate test plans and expected outcomes in specs. Robust test suites give agents validation superpowers — they iterate quickly when tests fail.

### 5. Test, Iterate, and Evolve

Treat spec-writing and agent-building as continuous cycles of testing, feedback, refinement.

**Continuous testing:** After major milestones, run tests or manual checks. If failures occur, update the spec before proceeding.

**Iterating the spec itself:** When discovering incomplete specs or misunderstandings, update the document and re-sync the agent.

**Context management tools:** Use RAG or MCP to automatically surface relevant spec sections based on current tasks.

**Version control:** Commit specs to git alongside code. This preserves history and allows agents to query changes via git diff.

**Cost optimization:** Use cheaper models for initial drafts; reserve top-tier models for critical reasoning and final outputs.

---

## Common Pitfalls to Avoid

**Most agent files fail because they're too vague.** "Build something cool" provides no anchor.

**Key mistakes:**
- Vague specifications without concrete detail
- Massive contexts without hierarchical summarization
- Skipping human review of critical code
- Confusing rapid prototyping ("vibe coding") with production engineering
- Ignoring the "lethal trifecta": speed (faster than you can review), non-determinism (same input, different outputs), cost (encourages corner-cutting on verification)
- Missing the six core areas (commands, testing, structure, style, git workflow, boundaries)

---

## Conclusion

Effective AI agent specs combine solid software engineering with LLM-specific adaptations.

1. **Start with clarity**, let agents expand details
2. **Structure professionally** with six core areas
3. **Keep agent focus tight** via modular contexts
4. **Anticipate failures** through constraints and self-checks
5. **Iterate continuously** using tests and feedback loops

These practices transform AI agents from unpredictable tools into genuinely productive collaborators.

---

## References

- [Addy Osmani - How to Write a Good Spec for AI Agents](https://addyosmani.com/blog/good-spec/)
