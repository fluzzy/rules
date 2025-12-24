# AGENTS – OpenAI Practical Guide to Building Agents

This document summarizes core principles and best practices for designing and building agents, based on OpenAI's **"A practical guide to building agents"**.

Core Goals:

1. Design agents that can **autonomously execute workflows** and select/combine tools as needed.
2. Systematically configure the **three components (Model, Tools, Instructions)** along with **orchestration and guardrails**.
3. Establish **layered guardrails** and **human-in-the-loop** pathways to ensure both safety and user experience.

---

## 1. Agent Definition and Scope

- Unlike simple chatbots or single-turn models, agents have **LLMs drive workflow execution**, calling tools and determining termination conditions autonomously.
- Key characteristics:
  - **LLM-based execution control**: Decides completion, stops/retries on failure, returns control to user when needed
  - **Diverse tool usage**: Selectively calls tools for context gathering/action execution, operating within clear guardrails

## 2. When to Choose Agents

- **Complex decision-making**: When exceptions and context-dependent judgments make rule engines fragile
- **Increasing rule maintenance difficulty**: When massive if-else/rule sets degrade quality
- **Unstructured data dependency**: When natural language/document/conversation-based processing is core
- If these conditions are unclear, **traditional deterministic automation** may be more suitable.

## 3. Core Components and Run Loop

- **Model**: The brain for reasoning and decision-making
- **Tools**: Functions/APIs/agents that interact with external systems
- **Instructions**: Specify role, tone, tool usage rules, and termination conditions
- **Run loop (while-loop)**: Iteratively executes model responses until termination conditions
  - Termination examples: **Final output tool call**, **Model response without tool calls**, **Error/turn limit exceeded**
- Recommended flow: Create baseline with **strongest model + minimal necessary tools**, then optimize incrementally

## 4. Model Selection Principles

1. Establish accuracy baseline with **evals/tests** first (use strongest model)
2. After meeting baseline, **swap to smaller/faster models** to optimize cost/latency
3. Consider **model mix** based on task complexity (simple classification/search ↔ complex judgment)

## 5. Tool Design

- **Standardized definitions**: Clearly define name, description, parameters, return schema; maximize reusability.
- **Three tool types**:
  - **Data**: Context gathering (DB/CRM queries, document reading, web search)
  - **Action**: External state changes (message sending, record updates, ticket handoff)
  - **Orchestration**: Expose other agents as tools (e.g., Research/Refund/Writer agent)
- **Risk rating**: Assign **Low/Med/High** per tool based on read/write, reversibility, permission/financial impact → Higher ratings require additional confirmation/guardrails/human approval
- **Test & document**: Manage as discoverable catalog, avoid duplicate definitions.

## 6. Instructions Best Practices

- **Leverage existing operational docs**: Convert SOPs/support scripts/policy docs to LLM-friendly steps
- **Break down into small pieces**: Separate complex instructions into numbered steps to eliminate ambiguity
- **Define clear actions**: Each step specifies "what to ask/verify/execute"
- **Include edge cases**: Add branching instructions for insufficient info/unexpected requests
- **Use models for auto-generation**: Use o1, o3-mini etc. to convert policy docs to instructions as drafts, then human review

## 7. Orchestration Patterns

- **Single agent first**: Add tools incrementally, minimize complexity. Inject policy variables into prompt templates for reusability.
- **When to switch to multi-agent**:
  - When **complex branching/conditions** make prompt scaling difficult
  - When **tool overload/overlap** causes model to select wrong tools
- **Representative patterns**:
  - **Manager (agents-as-tools)**: Central manager orchestrates specialist agents via tool calls, integrates results
  - **Decentralized handoff**: Equal agents pass control via **handoff** tools as needed, transferring conversation state together
- **Code over graph declarations**: Express branching/loops in regular code without unnecessary DSLs for flexibility

## 8. Guardrails (Layered Defense)

- Apply guardrails layered at **input/output/tool call** stages
- Key types:
  - **Relevance**: Detect scope drift
  - **Safety**: Detect jailbreak/prompt injection
  - **PII filter**: Prevent unnecessary personal info exposure
  - **Moderation**: Block harmful content
  - **Tool safeguards**: Trigger approval/block/human review based on tool risk
  - **Rules-based**: Input length, blacklists, regex filters
  - **Output validation**: Verify brand/policy compliance
- **Optimistic execution + tripwires**: Main agent proceeds with execution while guardrail functions/agents check in parallel; raise exceptions on violations
- **Application heuristics**: (1) Prioritize data privacy & content safety, (2) Rapidly add edge cases discovered in real usage, (3) Adjust balancing security vs. UX

## 9. Human-in-the-Loop

- **Failure threshold exceeded**: Request user/operator approval or return control when retry/action counts exceeded
- **High-risk actions**: Initially always require human approval before execution for cancellation/refunds/payments/sensitive data writes
- **Smooth transitions**: When agent can't complete, gracefully hand context to human and terminate

## 10. Practical Checklist

1. **Fitness check**: Examine complex judgment, rule fatigue, unstructured data needs
2. **Model baseline**: Establish eval baseline with strongest model, then test downscaling
3. **Tool catalog**: Categorize Data/Action/Orchestration, assign risk ratings
4. **Instruction quality**: Transform existing docs → numbered step instructions, include edge cases
5. **Orchestration**: Start with single agent run loop, expand to Manager/handoff patterns when needed
6. **Guardrails**: Relevance/Safety/PII/Moderation + Tool safeguards + rule-based filters
7. **Human-in-the-loop**: Set approval/escalation paths for failure thresholds and high-risk actions

---

## References

- OpenAI, "A practical guide to building agents" (2025)
  - <https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf>
