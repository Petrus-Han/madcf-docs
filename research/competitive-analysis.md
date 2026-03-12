# Multi-Agent AI Framework Landscape Analysis (March 2026)

A comprehensive comparison of major multi-agent AI frameworks and protocols, with positioning analysis for MADCF (Multi-Agent Distributed Collaboration Framework).

---

## 1. Framework-by-Framework Analysis

### 1.1 CrewAI

**What it does:** CrewAI is a Python framework for orchestrating role-playing, autonomous AI agents that work together as "crews." It assigns roles (Researcher, Writer, Analyst, etc.) to agents who collaborate on tasks using a dual-model architecture: Crews (autonomous agent teams) and Flows (event-driven workflow orchestration). It emphasizes ease of use with minimal abstractions and ships with 100+ built-in tool integrations.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized (single-machine orchestrator) |
| Cross-network collaboration | No — agents run within a single process/crew |
| Trust/verification layer | No — all agents are trusted within the crew |
| Economic incentives | No |
| Sandboxed execution | No — agents run in-process (Python) |
| Persistent memory | Yes — shared short-term, long-term, entity, and contextual memory |
| Open source / License | Yes — MIT License |
| GitHub stars | ~45k-54k |
| Adoption | 450M+ monthly workflows, Fortune 500 companies, 100k+ certified developers |

### 1.2 Microsoft AutoGen / Microsoft Agent Framework

**What it does:** AutoGen pioneered multi-agent conversation patterns where LLM agents, tool-using agents, and humans participate in group chats to solve tasks collaboratively. In October 2025, Microsoft merged AutoGen with Semantic Kernel into the unified "Microsoft Agent Framework" (GA targeted Q1 2026). The new framework provides event-driven, distributed, cross-language runtimes with orchestration patterns including sequential, concurrent, group chat, handoff, and Magentic (manager-led dynamic task ledger).

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized to federated (v0.4 supports distributed event-driven runtimes) |
| Cross-network collaboration | Limited — distributed runtimes possible but within trusted infrastructure |
| Trust/verification layer | No formal trust layer |
| Economic incentives | No |
| Sandboxed execution | Optional — supports Docker code execution containers |
| Persistent memory | Yes — conversation history and shared state |
| Open source / License | Yes — MIT License |
| GitHub stars | ~40k (AutoGen repo) |
| Adoption | Large enterprise adoption via Microsoft ecosystem |

### 1.3 LangGraph (LangChain)

**What it does:** LangGraph is a low-level orchestration framework for building stateful, long-running agents as directed graphs. Inspired by Pregel and Apache Beam, it models agent workflows as nodes (actions) and edges (transitions) with durable execution, human-in-the-loop breakpoints, and comprehensive memory (short-term working + long-term cross-session). LangGraph Platform adds managed deployment with auto-scaling task queues.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized (graph execution on single runtime; Platform adds managed infra) |
| Cross-network collaboration | No — agents are nodes in a single graph |
| Trust/verification layer | No |
| Economic incentives | No |
| Sandboxed execution | No — in-process execution |
| Persistent memory | Yes — stateful graph with checkpointing, short-term + long-term memory |
| Open source / License | Yes — MIT License (core); Platform is commercial |
| GitHub stars | ~24.6k (LangGraph); LangChain overall ~100k+ |
| Adoption | Used by Uber, LinkedIn, GitLab, Klarna, Replit |

### 1.4 MetaGPT

**What it does:** MetaGPT simulates an AI software company by assigning SOP-driven roles (Product Manager, Architect, Engineer, QA) to agents that collaboratively produce software from a one-line requirement. It generates professional artifacts (PRDs, system designs, code) through structured role-based pipelines. Their commercial product MGX (MetaGPT X) launched in February 2025 as "the world's first AI agent development team."

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized (SOP pipeline with shared memory pool) |
| Cross-network collaboration | No |
| Trust/verification layer | No — all agents are trusted roles within the company simulation |
| Economic incentives | No |
| Sandboxed execution | No — in-process Python execution |
| Persistent memory | Yes — shared memory pool for agent synchronization |
| Open source / License | Yes — MIT License |
| GitHub stars | ~60k |
| Adoption | ICLR 2024 oral (top 1.2%), ICLR 2025 oral (top 1.8%), strong academic influence |

### 1.5 OpenAI Swarm / OpenAI Agents SDK

**What it does:** Swarm was an educational/experimental framework exploring lightweight multi-agent orchestration via simple agent handoffs. It is stateless (built entirely on Chat Completions API) and designed for learning, not production. In March 2025, OpenAI replaced Swarm with the production-ready **Agents SDK**, which adds tracing, guardrails, realtime voice agents, and structured handoffs. AgentKit was announced at DevDay October 2025.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized (client-side orchestration, stateless between calls) |
| Cross-network collaboration | No |
| Trust/verification layer | Guardrails in Agents SDK for input/output validation |
| Economic incentives | No |
| Sandboxed execution | No — client-side execution |
| Persistent memory | No (Swarm is stateless); Agents SDK adds session management |
| Open source / License | Swarm: MIT (educational); Agents SDK: MIT |
| GitHub stars | ~20k (Swarm); Agents SDK growing |
| Adoption | OpenAI ecosystem, large developer community |

### 1.6 CAMEL-AI

**What it does:** CAMEL (Communicative Agents for "Mind" Exploration of Large Language Models) is a research-driven framework exploring the scaling laws of multi-agent systems. Its core innovation is role-playing communication between agents, where agents are assigned roles and collaborate through structured dialogue. It includes a "Workforce" system for multi-agent task solving and supports simulations with up to one million agents (OASIS project).

| Dimension | Detail |
|-----------|--------|
| Architecture model | Centralized (role-playing pairs or workforce orchestration) |
| Cross-network collaboration | No |
| Trust/verification layer | No |
| Economic incentives | No |
| Sandboxed execution | No |
| Persistent memory | Yes — in-context and external memory systems |
| Open source / License | Yes — Apache 2.0 |
| GitHub stars | ~15.2k |
| Adoption | Strong academic community (NeurIPS 2023), research-focused |

### 1.7 Google A2A (Agent-to-Agent Protocol)

**What it does:** A2A is an open communication protocol (not a framework) that enables AI agents built on different frameworks by different vendors to discover each other, negotiate capabilities, and delegate tasks. Each agent operates as both client and server. Agents publish "Agent Cards" (JSON metadata) for capability discovery. Supports HTTP, SSE streaming, push notifications, and gRPC (v0.3). Now housed under the Linux Foundation with 150+ supporting organizations.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Decentralized protocol (peer-to-peer agent communication) |
| Cross-network collaboration | **Yes** — core design goal; agents from different vendors/frameworks interoperate |
| Trust/verification layer | Partial — Agent Cards with signed security cards (v0.3), opaque agent model |
| Economic incentives | No |
| Sandboxed execution | No (protocol only, execution is agent-implementor's responsibility) |
| Persistent memory | No (protocol only; stateful task management for long-running tasks) |
| Open source / License | Yes — Apache 2.0 (Linux Foundation) |
| GitHub stars | ~21.5k |
| Adoption | 150+ supporting organizations, all major cloud providers |

### 1.8 Anthropic MCP (Model Context Protocol)

**What it does:** MCP is an open standard for connecting AI models/agents to external tools, data sources, and APIs. Inspired by the Language Server Protocol (LSP), it uses JSON-RPC 2.0 over various transports (STDIO, HTTP) to create a standardized two-way channel between AI applications (MCP clients) and capability providers (MCP servers). It solves the "N x M integration problem" by providing a universal adapter layer. Donated to the Agentic AI Foundation (Linux Foundation) in December 2025.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Client-server protocol (hub-and-spoke per AI application) |
| Cross-network collaboration | Indirect — standardizes tool access, not agent-to-agent communication |
| Trust/verification layer | Transport-level security; no built-in agent trust model |
| Economic incentives | No |
| Sandboxed execution | No (protocol only) |
| Persistent memory | No (protocol provides context access, not memory management) |
| Open source / License | Yes — open specification (Linux Foundation / AAIF) |
| GitHub stars | ~79k (MCP Servers repo) |
| Adoption | 97M+ monthly SDK downloads, adopted by OpenAI, Google, Microsoft, AWS |

### 1.9 IBM ACP (Agent Communication Protocol)

**What it does:** ACP is a lightweight, REST/HTTP-native open standard for agent-to-agent communication, designed for seamless interoperability regardless of framework or language. It features self-describing agents and supports both synchronous and asynchronous messaging. As of late 2025, ACP has **merged with A2A** under the Linux Foundation, with the ACP team contributing its technology to the A2A project.

| Dimension | Detail |
|-----------|--------|
| Architecture model | Decentralized protocol (REST-based peer-to-peer) |
| Cross-network collaboration | Yes |
| Trust/verification layer | Minimal |
| Economic incentives | No |
| Sandboxed execution | No |
| Persistent memory | No |
| Open source / License | Yes — Apache 2.0 (merged into A2A) |
| GitHub stars | N/A (merged into A2A) |
| Adoption | Merged into A2A ecosystem |

### 1.10 Fetch.ai / ASI Alliance (uAgents)

**What it does:** The uAgents Framework enables creation of lightweight, decentralized agents that register on the Almanac (a smart contract on the Fetch.ai blockchain). Agents are cryptographically secured, can discover and transact with each other across networks, and participate in dynamic marketplaces. Part of the Artificial Superintelligence (ASI) Alliance (merger of Fetch.ai, SingularityNET, Ocean Protocol, CUDOS). This is the closest existing system to MADCF's vision.

| Dimension | Detail |
|-----------|--------|
| Architecture model | **Decentralized** (blockchain-based agent registry + P2P communication) |
| Cross-network collaboration | **Yes** — agents communicate across networks via Almanac registry |
| Trust/verification layer | **Yes** — cryptographic identity, blockchain-registered agents |
| Economic incentives | **Yes** — FET/ASI token for transactions, staking, marketplace |
| Sandboxed execution | No — agents run as Python processes |
| Persistent memory | Limited — agent state, no structured multi-tier memory |
| Open source / License | Yes — Apache 2.0 |
| GitHub stars | ~1.5k (uAgents repo) |
| Adoption | Part of $4.3B Web3 AI agent sector, ASI Alliance ecosystem |

### 1.11 Agentic AI Foundation (AAIF) — Meta-Standard

**What it does:** Formed in December 2025 under the Linux Foundation, AAIF is a governance body (not a framework) that houses MCP, A2A, goose (Block's agent framework), and AGENTS.md (OpenAI's project guidance standard). Co-founded by Anthropic, OpenAI, and Block, with support from Google, Microsoft, AWS, Bloomberg, and Cloudflare. NIST launched a parallel AI Agent Standards Initiative in February 2026.

---

## 2. Comparison Matrix

| Dimension | CrewAI | AutoGen/MAF | LangGraph | MetaGPT | OpenAI Agents SDK | CAMEL-AI | A2A | MCP | Fetch.ai/ASI | **MADCF** |
|-----------|--------|-------------|-----------|---------|-------------------|----------|-----|-----|-------------|----------|
| **Topology** | Single-machine | Single to distributed | Single-machine | Single-machine | Single-machine | Single-machine | Distributed protocol | Client-server protocol | Decentralized (blockchain) | **Distributed spectrum** |
| **Trust model** | All trusted | All trusted | All trusted | All trusted | Guardrails | All trusted | Agent Cards + signing | Transport security | Crypto identity + blockchain | **On-chain identity + escrow + human veto** |
| **Economic layer** | None | None | None | None | None | None | None | None | Token economy (FET/ASI) | **Token economy + escrow + settlement** |
| **Execution model** | In-process | In-process (optional Docker) | In-process | In-process | In-process | In-process | N/A (protocol) | N/A (protocol) | In-process (Python) | **Containerized (Docker sandbox)** |
| **Protocol std.** | Proprietary | Proprietary | Proprietary | Proprietary | Proprietary | Proprietary | Open (Linux Foundation) | Open (Linux Foundation) | Custom + blockchain | **Open protocol (6 pluggable interfaces)** |
| **Human oversight** | Optional | Optional (human-in-loop) | Optional (breakpoints) | None | Optional (guardrails) | None | None | None | None | **Mandatory (initiator veto)** |
| **Persistent memory** | 4-tier | Conversation | Checkpoint + long-term | Shared pool | Session | In-context + external | N/A | N/A | Agent state | **3-tier structured** |
| **Cross-network** | No | No | No | No | No | No | **Yes** | Indirect | **Yes** | **Yes** |
| **GitHub stars** | ~50k | ~40k | ~25k | ~60k | ~20k | ~15k | ~21k | ~79k | ~1.5k | N/A |

---

## 3. Key Comparison Dimensions — Deep Analysis

### 3.1 Topology: Single-Machine vs Distributed vs Decentralized

The vast majority of production multi-agent frameworks (CrewAI, LangGraph, MetaGPT, AutoGen, CAMEL-AI) operate as **single-machine orchestrators** — all agents run within one process, communicating via function calls or message queues. This is simple and performant but fundamentally limited to trusted, co-located agents.

**A2A** and **MCP** are protocols, not runtimes — they enable communication across network boundaries but don't prescribe execution topology.

**Fetch.ai/ASI** is the only framework with true **decentralized** topology via blockchain-registered agents communicating peer-to-peer.

**MADCF's positioning:** MADCF introduces a unique **"decentralization spectrum"** — the same protocol works with 1 relay (centralized), a few relays (federated/alliance), or many relays (fully decentralized). No other framework offers this flexibility. This is a genuine differentiator: enterprises can start centralized and gradually decentralize as trust and scale increase.

### 3.2 Trust Model: Trusted vs Untrusted Agents

All major frameworks (CrewAI, AutoGen, LangGraph, MetaGPT, CAMEL-AI) assume **fully trusted agents** — there is no mechanism for verifying agent identity, validating behavior, or handling adversarial agents. This is acceptable for single-organization deployments but breaks down for cross-organization collaboration.

**A2A** adds Agent Cards with cryptographic signing (v0.3) — a step toward identity verification but no behavioral trust or economic guarantees.

**Fetch.ai** uses blockchain-registered identity with cryptographic keys — stronger identity guarantees but no escrow or dispute resolution.

**MADCF's positioning:** MADCF combines **on-chain identity, escrow-backed task lifecycle, and mandatory human veto** — the strongest trust model in the landscape. The escrow mechanism creates economic accountability (agents/providers have skin in the game), while human veto provides a safety net that no other framework mandates.

### 3.3 Economic Layer: None vs Centralized Payment vs Token Economy

The vast majority of frameworks have **no economic layer whatsoever**. Agent collaboration is assumed to be free and cooperative.

**Fetch.ai/ASI** has a token economy (FET/ASI) for agent marketplace transactions, but it's tightly coupled to its own blockchain.

The broader Web3 AI agent space ($4.3B market cap, 282+ projects) is exploring various tokenomics models, but most are focused on DeFi trading agents rather than general-purpose multi-agent collaboration.

**MADCF's positioning:** MADCF's **on-chain escrow with task-lifecycle settlement** is more sophisticated than simple token transactions. The escrow pattern (funds locked at task creation, released on completion/verification) creates proper economic incentives for reliable agent behavior. This is closer to Ethereum smart contract patterns than any existing AI agent framework.

### 3.4 Execution Model: In-Process vs Sandboxed vs Containerized

Almost every framework runs agents **in-process** (same Python/Node runtime). This provides zero isolation — a malicious or buggy agent can crash the entire system, access other agents' data, or execute arbitrary code.

**AutoGen** optionally supports Docker containers for code execution, but this is an add-on, not a core architectural principle.

**MADCF's positioning:** MADCF's **Docker containerized sandboxes** as a first-class primitive is a significant differentiator. Combined with untrusted agent support, this makes MADCF one of the only frameworks where running agents from unknown providers is architecturally safe.

### 3.5 Protocol Standardization: Proprietary vs Open Protocol

The framework landscape is bifurcating:
- **Proprietary frameworks** (CrewAI, LangGraph, MetaGPT) — each has its own agent definition, communication, and orchestration patterns
- **Open protocols** (MCP, A2A) — standardize specific layers (tool access, agent-to-agent communication) but don't cover execution, economics, or trust

The **AAIF** (Agentic AI Foundation) is attempting to unify standards under Linux Foundation governance, but as of March 2026, MCP and A2A remain complementary rather than unified.

**MADCF's positioning:** MADCF's **6 pluggable provider interfaces** offer a middle path — open protocol with abstract interfaces that can be implemented by different providers. This is more comprehensive than A2A (which only covers communication) or MCP (which only covers tool access). However, MADCF would benefit from aligning with A2A for agent discovery and MCP for tool integration rather than creating parallel standards.

### 3.6 Human Oversight: None vs Optional vs Mandatory

| Level | Frameworks |
|-------|-----------|
| **None** | MetaGPT, CAMEL-AI, A2A, MCP, Fetch.ai |
| **Optional** | CrewAI, AutoGen (human-in-loop), LangGraph (breakpoints), OpenAI Agents SDK (guardrails) |
| **Mandatory** | **MADCF only** |

Most frameworks treat human oversight as a developer choice — you *can* add human-in-the-loop but the framework doesn't require it. LangGraph's breakpoints and OpenAI's guardrails are the most structured optional implementations.

**MADCF's positioning:** **Mandatory human veto** is architecturally unique. In a world of untrusted, cross-organization agents handling economic transactions, this is arguably essential. No other framework mandates this level of human control. The key question is whether mandatory oversight creates too much friction for high-frequency, low-stakes tasks — MADCF may need configurable oversight levels.

---

## 4. The Protocol Wars: MCP vs A2A vs MADCF

The 2025-2026 "protocol wars" have largely settled into a complementary two-layer model:

| Layer | Protocol | Function |
|-------|----------|----------|
| **Tool/Context access** | MCP | How agents connect to tools, data, and APIs |
| **Agent-to-agent communication** | A2A | How agents discover, delegate, and collaborate |
| **Execution + Economics + Trust** | ??? | How agents are sandboxed, paid, and verified |

**MADCF fills the missing third layer.** Neither MCP nor A2A address:
- Economic incentives and settlement
- Sandboxed execution guarantees
- Mandatory human oversight
- Decentralization spectrum

MADCF could position itself as the **execution and trust layer** that complements MCP (tool access) and A2A (agent communication), rather than competing with them.

---

## 5. MADCF Differentiated Positioning Summary

### What makes MADCF unique (features no other framework has):

1. **Decentralization spectrum** — Single relay (centralized) to many relays (fully decentralized) with the same protocol. No other framework offers this topology flexibility.

2. **On-chain task lifecycle with escrow** — Economic accountability through locked funds, completion verification, and settlement. Only Fetch.ai/ASI has token economics, but without escrow-based task lifecycle.

3. **Mandatory human oversight** — Initiator veto power is architecturally enforced, not optional. Unique in the entire landscape.

4. **Containerized sandbox as first-class primitive** — Docker isolation for untrusted agents. Only AutoGen has optional Docker support; no other framework makes it core architecture.

5. **6 pluggable provider interfaces** — Abstract interfaces for relay, identity, execution, memory, payment, and trust. More comprehensive than A2A or MCP's single-concern protocols.

6. **3-tier persistent memory** — Structured memory across task, agent, and global tiers. Most frameworks have simpler memory models.

### Potential gaps and recommendations:

| Gap | Recommendation |
|-----|----------------|
| No alignment with MCP/A2A standards | Implement MCP compatibility for tool access; implement A2A Agent Cards for discovery |
| Novel protocol may face adoption resistance | Consider building MADCF as a layer on top of A2A rather than a parallel protocol |
| Mandatory human oversight may be too rigid | Add configurable oversight levels (none/optional/mandatory) based on task value/risk |
| No AAIF membership | Engage with the Agentic AI Foundation for standardization alignment |
| Blockchain dependency may deter enterprises | Ensure the centralized relay mode works without any blockchain requirement |

### Competitive landscape positioning:

```
                        Trust / Economic Guarantees
                                    ^
                                    |
                              MADCF |
                                *   |
                                    |
                     Fetch.ai/ASI   |
                          *         |
                                    |
          A2A *                     |
              |                     |
    ----------|---------------------|---> Decentralization
              |                     |
        MCP * |                     |
              |                     |
    CrewAI *  |  LangGraph *        |
   AutoGen *  | MetaGPT *          |
   OpenAI  *  | CAMEL *            |
              |                     |
```

MADCF occupies a unique position in the upper-right quadrant: high trust/economic guarantees AND high decentralization. The nearest competitor is Fetch.ai/ASI, which has decentralization and token economics but lacks escrow, mandatory human oversight, and containerized sandboxing.

---

## 6. Market Context

- The multi-agent framework market is maturing rapidly, with consolidation happening at both the framework level (Microsoft merging AutoGen + Semantic Kernel) and the protocol level (ACP merging into A2A, MCP + A2A + goose under AAIF).
- The Web3 AI agent sector represents $4.3B in market cap with 282+ funded projects, indicating strong investor interest in decentralized agent economies.
- NIST launched the AI Agent Standards Initiative in February 2026, signaling regulatory attention to agent interoperability and security.
- The x402 protocol (2025) is emerging as a decentralized payment standard for autonomous AI agents, adopted by Google Cloud, AWS, and Anthropic.
- Enterprise adoption favors centralized frameworks (CrewAI, LangGraph) today, but the trend is toward open protocols (MCP, A2A) that enable cross-vendor interoperability.

---

## Sources

- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [CrewAI Official Site](https://crewai.com/)
- [Microsoft AutoGen GitHub](https://github.com/microsoft/autogen)
- [Microsoft Agent Framework Announcement](https://devblogs.microsoft.com/foundry/introducing-microsoft-agent-framework-the-open-source-engine-for-agentic-ai-apps/)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- [LangGraph Official Site](https://www.langchain.com/langgraph)
- [MetaGPT GitHub](https://github.com/FoundationAgents/MetaGPT)
- [MetaGPT — IBM Overview](https://www.ibm.com/think/topics/metagpt)
- [OpenAI Swarm GitHub](https://github.com/openai/swarm)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
- [CAMEL-AI GitHub](https://github.com/camel-ai/camel)
- [Google A2A Protocol](https://a2a-protocol.org/latest/)
- [A2A GitHub](https://github.com/a2aproject/A2A)
- [A2A Announcement — Google Developers Blog](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [MCP Specification](https://modelcontextprotocol.io/specification/2025-11-25)
- [MCP Servers GitHub](https://github.com/modelcontextprotocol/servers)
- [Anthropic MCP Announcement](https://www.anthropic.com/news/model-context-protocol)
- [Anthropic — Donating MCP to AAIF](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation)
- [IBM ACP](https://research.ibm.com/blog/agent-communication-protocol-ai)
- [Fetch.ai uAgents GitHub](https://github.com/fetchai/uAgents)
- [AAIF — Linux Foundation Announcement](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
- [OpenAI — AAIF Co-founding](https://openai.com/index/agentic-ai-foundation/)
- [MCP vs A2A — DigitalOcean](https://www.digitalocean.com/community/tutorials/a2a-vs-mcp-ai-agent-protocols)
- [AI Agent Protocol Wars 2026](https://www.hungyichen.com/en/insights/ai-agent-protocol-wars)
- [Web3 AI Agent Sector Analysis](https://blockeden.xyz/blog/2026/02/07/web3-ai-agent-sector-analysis/)
- [CrewAI 2025 Review — Latenode](https://latenode.com/blog/ai-frameworks-technical-infrastructure/crewai-framework/crewai-framework-2025-complete-review-of-the-open-source-multi-agent-ai-platform)
- [AI Agent Frameworks Compared 2026 — Arsum](https://arsum.com/blog/posts/ai-agent-frameworks/)
- [Top 10 AI Agent Frameworks GitHub 2026 — Medium](https://techwithibrahim.medium.com/top-10-most-starred-ai-agent-frameworks-on-github-2026-df6e760a950b)
- [MCP Year in Review — Pento](https://www.pento.ai/blog/a-year-of-mcp-2025-review)
- [A2A Protocol Upgrade — Google Cloud Blog](https://cloud.google.com/blog/products/ai-machine-learning/agent2agent-protocol-is-getting-an-upgrade)
