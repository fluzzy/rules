# Stop Using /init for AGENTS.md

> **Source**: [Addy Osmani - Stop Using /init for AGENTS.md](https://addyosmani.com/blog/agents-md/)
> **Archive Date**: 2026-02-25

Auto-generated AGENTS.md files often backfire. Research shows human-written files improve performance, while LLM-generated ones reduce task success and increase costs.

---

## Key Research Findings

### Lulla et al. (ICSE JAWs 2026)

- AGENTS.md reduced median runtime by **28.64%**
- Output token consumption reduced by **16.58%**

### ETH Zurich Study

- **LLM-generated** context files: task success **-2~3%**, costs **+20%**
- **Human-written** context files: task success **+4%**, costs up to **+19%**
- When documentation stripped from repos, auto-generated files improved performance by **2.7%** — proving agents could discover the same information independently

**Critical insight**: Auto-generated content is largely redundant with what agents can already find by reading code.

---

## What Belongs in AGENTS.md

**Filter**: "If agents can discover this by reading code, delete it."

**Keep:**
- Tool-specific instructions (e.g., use `uv` not `pip`)
- Non-obvious conventions and gotchas
- Custom patterns agents won't guess (authentication middleware specifics, etc.)
- Known landmines to avoid

**Remove:**
- Anything discoverable by reading the codebase
- Deprecated tech references (causes anchoring bias)
- Redundant style/convention descriptions

---

## Problems with Current Approaches

### Anchoring Effect
Mentioning deprecated technologies biases agents toward outdated patterns for every subsequent task.

### Lost in the Middle
Longer contexts degrade performance even when content is relevant.

### Static File Limitation
A monolithic AGENTS.md applies identical instructions to different task types. Documentation changes don't need full test suites.

---

## Recommended Architecture

Instead of a single root file:

1. **Protocol Layer**: Routing document describing personas, skills, and MCP connections — only non-discoverable facts
2. **Focused Persona Files**: Context loaded selectively based on task type
3. **Maintenance Subagent**: Keeps protocol accurate as codebases evolve

---

## Practical Takeaways

1. **Abandon `/init`** for auto-generated files
2. Treat AGENTS.md as **diagnostics for codebase friction**, not permanent configuration
3. Document friction points, then **fix the underlying issues**
4. Hold intuitions about agent needs loosely — empirical evidence shows gaps between assumptions and requirements
5. Start with **minimal content**, add only when agents repeatedly struggle

---

## Caveats

- Neither study measured **correctness** — only efficiency
- Real-world AGENTS.md files suffer from **decay** as codebases evolve
- Fresh files in studies may perform better than stale real-world files

---

## References

- [Addy Osmani: Stop Using /init for AGENTS.md](https://addyosmani.com/blog/agents-md/)
