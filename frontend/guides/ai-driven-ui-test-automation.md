# AI-Driven UI Test Automation: Toss Tax Refund QA Case Study

> **Source**: [Toss Tech Blog](https://toss.tech/article/ai-driven-ui-test-automation)
> **Author**: Jeong Su-ho (QA Manager, Toss Income)
> **Archive Date**: 2026-02-27

A five-month experiment (July–November 2025) automating UI testing for Toss's complex tax refund service using AI agents as primary development partners, achieving 35 stable E2E test scenarios with one person and three Claude agents.

---

## The Challenge

Toss's tax refund platform handles four interconnected services — year-end settlement, cash receipts, tax assistant, and hidden refund finder — with dozens of deduction categories and variable user flows.

**Manual QA pain points:**
- 160–300 hours needed to build 40 E2E test scenarios
- UI and policy changes required constant maintenance
- Single person managing all responsibilities

**Reframed question:** Could AI handle 99% of implementation while humans focused on problem definition and quality validation?

---

## AI Team Composition

Three specialized Claude agents:

| Agent | Role |
| --- | --- |
| **SDET Agent** | Test architecture and design |
| **Documentation Specialist** | Automated reporting and knowledge management |
| **Git Master** | Commit messages and version control |

**Supporting tools:** Cursor IDE for real-time error resolution; supplementary code analysis capabilities.

---

## Key Technical Solutions

### React Interaction Readiness Strategy

DOM rendering and event handler binding occur asynchronously in React. Standard click detection failed ~30% of the time because elements appeared clickable before their handlers loaded.

**Three-tier stability checking:**
1. DOM content load confirmation
2. Element visibility verification
3. Event handler binding validation

Result: Success rates improved from ~70% to 100%.

### Service-Agnostic Consent Management

Instead of hardcoding consent flows per service, the system detects:
- Service type via URL patterns
- Consent structure via checkbox counts
- User cohort classification

A single `clickInitialConsent()` function automatically accommodated subsequent agreement policy changes across all services.

### Page Object Model (POM) Implementation

Centralizing selectors eliminated maintenance overhead. When a button text changed from "수정하기" to "정보 수정," only one location required updates rather than scattered test files.

---

## Workflow Integration

Testing results fed directly into internal messaging with:
- Pass/fail status and execution timing
- Test-specific user IDs for reproducibility
- Screenshots and logs for failures
- Automated discussion threads for debugging

This created a **human-AI feedback loop**: developers reviewed AI-generated reports, requested specific reanalysis, and incorporated recommendations.

---

## Development Timeline

| Month | Milestone |
| --- | --- |
| **July** | 5 core tests established; POM architecture adopted |
| **August** | React timing issues resolved; documentation framework built |
| **September** | 2,147-line file refactored into modular structure; userNo collision prevention |
| **October** | Consent system redesigned for multi-service compatibility |
| **November** | 35 tests operational; transitioned to maintenance mode (v3.3.0) |

---

## Key Metrics

- **Code generation**: AI wrote 90%+ of implementation
- **Human effort**: Problem definition, review, validation (~10%)
- **Team capacity**: 1 person + 3 AI agents ≈ 4–5 person team output
- **Test coverage**: 35 stable scenarios across complex workflows
- **Maintenance**: Automated documentation eliminated 30–40 minute daily journaling

---

## Key Takeaways

- AI amplifies QA professionals rather than replacing them
- The critical skill shift: mastery of **problem articulation** and **quality judgment** over manual coding proficiency
- Three-tier readiness checks are essential for reliable React UI testing
- Page Object Model centralizes maintenance burden for evolving UIs
- Service-agnostic patterns (URL detection, dynamic consent) reduce per-service duplication

> "QA becomes the person who determines not just *how fast* automation runs, but *where* it should focus."

---

## References

- [Toss Tech — AI-Driven UI Test Automation](https://toss.tech/article/ai-driven-ui-test-automation)
