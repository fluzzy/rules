# How to Effectively Write Quality Code with AI

> **Source**: [Article](https://heidenstedt.org/posts/2026/how-to-effectively-write-quality-code-with-ai/) | [Discussion](https://news.hada.io/topic?id=26491)
> **Author**: Mia Heidenstedt
> **Archive Date**: 2026-02-10

12 practices for maintaining code quality when working with AI coding agents. Covers vision, documentation, testing, security marking, and complexity management.

## 1. Establish a Clear Vision

Every undocumented decision will be made by the AI. Define upfront:

- Architecture and interfaces
- Data structures and algorithms
- Testing and validation strategies

Humans must make long-term, difficult-to-change decisions. AI handles implementation within those boundaries.

## 2. Maintain Precise Documentation

- Document requirements, specifications, constraints, and architecture in detail
- Record coding standards, best practices, and design patterns
- Use flowcharts, UML diagrams, and visual aids for complex structures
- Write pseudocode for complex algorithms to guide AI understanding
- Include standardized documents in the code repository

Documentation now has direct practical value — AI actually reads and acts on it.

## 3. Build Debug Systems that Aid the AI

Create systems that collect and abstract log information:

- Aggregate distributed system logs into summaries
- Provide abstracted outputs like "Data was sent to all nodes"
- Reduce the need for multiple expensive CLI commands or browser verification

This reduces execution costs and accelerates problem identification.

## 4. Mark Code Review Levels

Not all code requires equal scrutiny. Add markers for review status:

```
//A  — AI-written, unreviewed code
```

Track which code has been reviewed by a human and which has not.

## 5. Write High-Level Specification Tests

> "AIs will cheat and use shortcuts eventually. They will write mocks, stubs, and hard-coded values to make tests succeed while the code itself is not working."

- Write property-based, high-level specification tests that make cheating difficult
- Separate test files so the AI cannot modify them
- Prompt the AI explicitly not to change these tests
- Implement property-based testing directly

## 6. Write Interface Tests in a Separate Context

Let AI write property-based interface tests with minimal context about the implementation. This prevents the implementation context from influencing tests in ways that compromise their effectiveness. Protect these tests from AI modification.

## 7. Use Strict Linting and Formatting Rules

Consistent code style and linting rules are essential for quality maintenance and early error detection. A recommended static analysis pipeline:

```bash
# Unified under a single command with pre-commit hooks
tsc        # Type checking
eslint     # Code linting
sonarjs    # Code smell detection
knip       # Dead code detection
jscpd      # Copy-paste detection
dependency-cruiser  # Dependency analysis
semgrep    # Security patterns
```

Note: Linting passes do not guarantee correct behavior — behavioral accuracy evaluation remains the most difficult part.

## 8. Use Context-Specific Coding Agent Prompts

Use path-specific prompt files (e.g., `CLAUDE.md`) that provide:

- Coding standards
- Best practices
- Design patterns
- Project-specific requirements

These reduce AI onboarding costs and improve code generation quality. Can be generated automatically.

## 9. Find and Mark Security-Critical Functions

Mark functions handling authentication, authorization, and data processing:

```
//HIGH-RISK-UNREVIEWED  — Security-critical, not yet reviewed
//HIGH-RISK-REVIEWED    — Security-critical, reviewed
```

Instruct the AI to automatically reset the review status back to `UNREVIEWED` when any changes occur to marked functions.

## 10. Reduce Code Complexity

> "Each single line of code will eat up your context window and make it harder for the AI and you to keep track of the overall logic."

Every avoidable line costs energy, money, and increases the probability of failed AI tasks. Maintain simple structures for both AI and human understanding.

## 11. Explore with Experiments and Prototypes

Leverage the low cost of AI-generated code:

- Create multiple prototypes with minimal specifications
- Explore multiple solutions through iteration
- Identify optimal approaches before committing to architecture

## 12. Do Not Generate Too Much Complexity at Once

> "If you have lost the overview of the complexity and inner workings of the code, you have lost control over your code and must restart from a state where you were in control."

- Break complex tasks into function/class-level units
- Validate each component against specifications
- Uncontrolled complexity may require a complete project reset

## Key Patterns Summary

| Pattern | Description |
| --- | --- |
| `//A` marker | AI-written, unreviewed code |
| `//HIGH-RISK-UNREVIEWED` | Security-critical, not reviewed |
| `//HIGH-RISK-REVIEWED` | Security-critical, reviewed (auto-resets on change) |
| `CLAUDE.md` | Project-level AI guidance |
| Property-based tests | High-level specs resistant to AI shortcuts |
| Separated test contexts | Tests written without implementation knowledge |
| Debug log aggregation | Abstracted summaries for AI consumption |
| Static analysis pipeline | `tsc + eslint + sonarjs + knip + jscpd + semgrep` |
| Task decomposition | Break into function/class-level units |
| Zero-cost refactoring | Leverage AI for continuous cleanup |
