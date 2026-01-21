# OpenAI Agent Design Guide

> **Source**: [OpenAI - A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) (2025)

Core principles and best practices for designing and building agents.

---

## Core Goals

1. **Execute workflows autonomously**, selecting and combining tools as needed
2. Systematically organize **three components (Model, Tools, Instructions)** + **orchestration and guardrails**
3. Ensure safety and UX through **layered guardrails** and **human-in-the-loop** paths

---

## 1. Defining Agents

Unlike simple chatbots or single-turn models, **LLMs drive workflow execution**, autonomously deciding tool calls and exit conditions.

- **LLM-based execution control**: Decides completion, stops/retries on failure, returns control to user when necessary
- **Diverse tool usage**: Selectively calls tools for context gathering/action execution

## 2. When to Choose Agents

- **Complex decision-making**: When exceptions and context-dependent judgment make rule engines fragile
- **Increasing rule maintenance**: When large if-else/rule sets degrade quality
- **Unstructured data dependence**: When natural language/document/conversation-based processing is core

If conditions are unclear, **traditional deterministic automation** may be more suitable.

## 3. Core Components and Run Loop

| Component        | Description                                       |
| ---------------- | ------------------------------------------------- |
| **Model**        | Brain for reasoning and decision-making           |
| **Tools**        | Functions/APIs/agents for interacting with external systems |
| **Instructions** | Specify role, tone, tool usage rules, exit conditions |

**Run loop (while-loop)**: Repeatedly execute model responses until exit condition

- Exit examples: Final output tool call, model response without tool call, error/turn limit exceeded

Recommended flow: Create baseline with **strongest model + minimum necessary tools**, then optimize incrementally

## 4. Tool Design

**Three tool types:**

| Type              | Description                  | Examples                                    |
| ----------------- | ---------------------------- | ------------------------------------------- |
| **Data**          | Context gathering            | DB/CRM queries, document reading, web search |
| **Action**        | External state changes       | Send messages, update records, ticket handoff |
| **Orchestration** | Expose other agents as tools | Research/Refund/Writer agents               |

**Risk levels**: Assign **Low/Med/High** per tool

- Based on read/write, reversibility, permission/financial impact
- Higher levels require additional confirmation/guardrails/human approval

## 5. Instructions Best Practices

- **Leverage existing operational docs**: Convert SOPs/support scripts/policy documents into LLM-friendly steps
- **Break into small pieces**: Separate complex instructions into numbered steps
- **Define clear actions**: Specify "what to ask/verify/execute" at each step
- **Include edge cases**: Add branching instructions for missing information/unexpected requests

## 6. Orchestration Patterns

**Start with single agent**: Add tools incrementally, minimize complexity

**When to switch to multi-agent:**

- When complex branching/conditions make prompt expansion difficult
- When tool overload/overlap causes model to select wrong tools

**Representative patterns:**

- **Manager (agents-as-tools)**: Central manager orchestrates specialist agents via tool calls
- **Decentralized handoff**: Peer agents transfer control via handoff tools as needed

## 7. Guardrails (Layered Defense)

Apply guardrails hierarchically at **input/output/tool call** stages:

| Type                | Description                                     |
| ------------------- | ----------------------------------------------- |
| **Relevance**       | Detect out-of-scope queries                     |
| **Safety**          | Detect jailbreak/prompt injection               |
| **PII filter**      | Prevent unnecessary personal info exposure      |
| **Moderation**      | Block harmful content                           |
| **Tool safeguards** | Approval/block/human review triggers based on tool risk |
| **Rules-based**     | Input length, blacklists, regex filters         |

**Optimistic execution + tripwires**: Main agent proceeds while guardrail functions/agents check in parallel; exceptions raised on violation

## 8. Human-in-the-Loop

- **Failure threshold exceeded**: Request user/operator approval or return control when retry/action count exceeded
- **High-risk actions**: Cancellations/refunds/payments/sensitive data writes always require human approval initially
- **Smooth transitions**: When agent can't complete, gracefully transfer context to human and exit

---

## Practical Checklist

1. **Suitability check**: Review need for complex judgment, rule fatigue, unstructured data
2. **Model baseline**: Establish evaluation baseline with strongest model, then test downscaling
3. **Tool catalog**: Classify Data/Action/Orchestration, assign risk levels
4. **Instructions quality**: Convert existing docs to numbered step instructions, include edge cases
5. **Orchestration**: Start with single agent run loop, expand to Manager/handoff patterns as needed
6. **Guardrails**: Relevance/Safety/PII/Moderation + Tool safeguards + rule-based filters
7. **Human-in-the-loop**: Set up approval/escalation paths for failure thresholds and high-risk actions
