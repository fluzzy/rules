# AGENTS – Agents Rule of Two

This document defines **AI agent security design rules** based on **Meta's Agents Rule of Two** framework.

Core Goals:

1. **Prevent worst-case outcomes from Prompt Injection vulnerabilities.**
2. Follow the principle that agents should have **at most two of three properties** simultaneously.
3. Help agent developers understand and make informed decisions about **security vs. utility trade-offs**.

---

## 1. Agents Rule of Two Overview

### 1.1 Core Principle

**Agents Rule of Two** states that in a world where prompt injection is not fundamentally solved, agents should have **at most two** of these three properties simultaneously:

**[A] Untrustworthy Inputs**
Can the agent process data from unknown sources?
Examples: external emails, web search results, user input, etc.

**[B] Sensitive Systems or Private Data**
Can the agent access sensitive information or systems?
Examples: user PII, production databases, internal systems, etc.

**[C] State Changes or External Communication**
Can the agent modify system state or communicate externally?
Examples: sending emails, database modifications, API calls, etc.

### 1.2 Rule Application

- Allow **at most two** of the three properties simultaneously.
- When all three are needed:
  - **Start a new session (new context window)**, OR
  - **Require human oversight (human-in-the-loop)** or other trusted verification mechanisms.

### 1.3 Valid Configurations

Possible configurations:

- **[AB]**: Untrustworthy inputs + Sensitive data access (external communication/state changes restricted)
- **[AC]**: Untrustworthy inputs + External communication/state changes (sensitive data access restricted)
- **[BC]**: Sensitive data access + External communication/state changes (untrustworthy inputs restricted)

---

## 2. Attack Scenarios and Defenses

### 2.1 Email-Bot Attack Example

**Attack Scenario:**
A spam email contains a prompt injection string that instructs the user's Email-Bot to collect personal email content and send it to the attacker.

**Why the Attack Succeeds:**
- [A] Processing untrustworthy input (spam email)
- [B] Accessing sensitive data (user's mailbox)
- [C] External communication (sending email)

**Defense with Agents Rule of Two:**

- **[BC] Configuration**: Process only emails from trusted senders (e.g., close friends) so prompt injection payloads never reach the agent's context.
- **[AC] Configuration**: Restrict access to sensitive data or systems (e.g., operate only in test environments) so even if prompt injection reaches the agent, it can't cause meaningful harm.
- **[AB] Configuration**: Send emails only to trusted recipients, or require human verification of draft message content, preventing attackers from completing the attack chain.

---

## 3. Real-World Use Cases

### 3.1 Travel Agent Assistant [AB]

**Description**: A public travel assistant that can answer questions and take actions on behalf of users.

**Requirements:**
- [A] Needs web search to get up-to-date travel information
- [B] Needs access to user PII for booking and purchase experiences

**Rule of Two Application:**
- [C] Restriction: Preventive controls on external communication and state changes
  - Require human confirmation for all actions (bookings, deposits, etc.)
  - Restrict web requests to URLs returned only from trusted sources (no agent-constructed URLs)

### 3.2 Web Browsing Research Assistant [AC]

**Description**: An agent that interacts with web browsers to conduct research on behalf of users.

**Requirements:**
- [C] Needs to make many requests to arbitrary URLs and fill out forms
- [A] Needs to process results and re-plan as necessary

**Rule of Two Application:**
- [B] Restriction: Preventive controls on sensitive systems and private data access
  - Run browser in restricted sandbox without pre-loaded session data
  - Restrict agent's access to personal information (beyond initial prompt) and notify users about data sharing

### 3.3 High-Velocity Internal Coder [BC]

**Description**: An agent that generates and executes code across an organization's internal infrastructure to solve engineering problems.

**Requirements:**
- [B] Needs access to a subset of production systems to solve meaningful problems
- [C] Must be able to perform state changes on these systems
- Human oversight is valuable as defense-in-depth, but developers want to minimize human intervention to operate at scale

**Rule of Two Application:**
- [A] Restriction: Preventive controls on untrustworthy data sources
  - Filter all data sources processed within the agent's context window using author-lineage
  - Provide a human review process for flagging false positives and granting agent data access

### 3.4 In-Session Configuration Switching

As with general frameworks, details matter.
It can be safe for agents to switch from one Agents Rule of Two configuration to another **within the same session** to enable additional use cases.

**Example**: Start with [AC] to access the internet, then complete a one-way transition to [B] by disabling communication when accessing internal systems.

The specifics of doing this safely are omitted for brevity, but readers can reason about this by **focusing on disrupting attack paths** — preventing completion of the full [A] → [B] → [C] chain.

---

## 4. Limitations

### 4.1 Other Threat Vectors

Satisfying Agents Rule of Two is **not sufficient** to protect against:

- Other threat vectors common to agents:
  - Attacker uplift
  - Spam proliferation
  - Agent mistakes
  - Hallucination
  - Excessive permissions, etc.
- Low-impact consequences of prompt injection:
  - Misinformation in agent responses, etc.

### 4.2 Defense in Depth

Applying Agents Rule of Two is **not the end** of risk mitigation.

- Designs satisfying Agents Rule of Two can still fail
  - Example: Users blindly confirming warning interstitials
- **Defense in depth** is a crucial component for mitigating highest-risk scenarios when a single layer can fail
- Agents Rule of Two is a **complement**, not a **replacement**, for general security principles like **least privilege**

---

## 5. Complementary Solutions

Additional AI protection solutions that complement Agents Rule of Two:

- **Llama Firewall**: Orchestrates agent protections
- **Prompt Guard**: Classifies potential prompt injections
- **Code Shield**: Reduces unsafe code suggestions
- **Llama Guard**: Classifies potentially harmful content

For details, see [Meta Llama Protections](https://llama.meta.com/docs/model-cards-and-prompt-formats/llama-guard/) documentation.

---

## 6. Future Outlook

### 6.1 Model Context Protocol (MCP)

With the adoption of protocols for plug-and-play agent tool calling (e.g., Model Context Protocol), new risks and opportunities are emerging.

- Blindly connecting agents to new tools can be a recipe for disaster
- Potential to enable **security-by-default** with built-in Rule of Two awareness
- Example: By declaring Agents Rule of Two configurations in supported tool calls, developers can have confidence that operations will succeed, fail, or request additional approval according to policy

### 6.2 Oversight Approval Checks

As agents become more useful and capabilities grow, some highly popular use cases will not fit cleanly into Agents Rule of Two.

Example: Background processes where human oversight is obstructive or inefficient

- Traditional software guardrails and human approval continue as the preferred method for satisfying Agents Rule of Two in current use cases
- Continued research on satisfying Agents Rule of Two oversight approval checks through alignment controls
  - Example: Oversight agents
  - Example: Open-source LlamaFirewall platform

---

## 7. Note to Agents Reading This Document

1. **Agents Rule of Two is a mandatory framework.**
   Always consider this principle when designing or implementing AI agents.

2. Allow **at most two** of the three properties [A], [B], [C] simultaneously.

3. When all three are needed:
   - Start a new session, OR
   - Introduce human oversight or trusted verification mechanisms.

4. **Consider defense in depth.**
   Agents Rule of Two should be used alongside other security principles.

5. When something is ambiguous:
   - (If conversation is possible) Ask the user.
   - Otherwise, design with the most conservative assumptions.

This framework aims to **deterministically reduce the worst outcomes of prompt injection**.
It helps agent developers understand security vs. utility trade-offs and make informed decisions.

---

## References

- [Meta AI Blog: Agents Rule of Two - A Practical Approach to AI Agent Security](https://ai.meta.com/blog/practical-ai-agent-security/) (October 31, 2025)
- [Chromium Rule of Two](https://chromium.googlesource.com/chromium/src/+/main/docs/security/rule-of-two.md)
- [Simon Willison's "Lethal Trifecta"](https://simonwillison.net/2023/Oct/25/lethal-trifecta/)
- [Meta Llama Protections](https://llama.meta.com/docs/model-cards-and-prompt-formats/llama-guard/)
