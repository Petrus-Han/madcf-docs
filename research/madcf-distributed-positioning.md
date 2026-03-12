# MADCF Distributed Positioning Research

> Date: 2026-03-12
> Status: Research Report v1.0

---

## 1. Core Thesis: Distributed, Not Decentralized

MADCF stands for **Multi-Agent Distributed Collaboration Framework** — the keyword is **distributed**, not decentralized.

This distinction matters:

| Term | Meaning | MADCF's Stance |
|------|---------|----------------|
| **Centralized** | One node controls everything | Supported (relay=1) |
| **Distributed** | Work spread across multiple nodes, may have a coordinator | **Primary design target** |
| **Decentralized** | No central authority, fully peer-to-peer | Supported (relay=many), but not required |

MADCF is a **distribution protocol** that treats decentralization as a tunable parameter, not a binary requirement. This makes it fundamentally different from blockchain-first projects that force decentralization from day one.

### 1.1 The Decentralization Spectrum

From `protocol-principles.md` Section 7.1:

```
relay = 1     →  Centralized distributed (simplest MVP)
relay = few   →  Federated (similar to Mastodon)
relay = many  →  Fully decentralized (similar to Nostr)
```

This is a **decentralization dial**, not a switch. The same protocol, the same interfaces, the same agent code — just different deployment topology.

### 1.2 Why "Distributed" is the Right Word

1. **The problem is distribution, not decentralization.** AI agents need to run on physically separate machines (different GPUs, different API keys, different security boundaries). This is a distributed systems problem.

2. **Trust is a spectrum.** Within a company, agents can trust each other (centralized trust). Between organizations, you need verification (federated trust). On a public network, you need cryptographic proofs (trustless). MADCF supports all three.

3. **Decentralization has a cost.** Consensus, replication, and verification add latency and complexity. Forcing decentralization on a single-team use case is over-engineering. MADCF lets you pay for decentralization only when you need it.

---

## 2. Protocol Compatibility Analysis

### 2.1 Three Deployment Models

| Model | Relay Count | Trust Model | Use Case |
|-------|-------------|-------------|----------|
| **Single Node** | 1 | Centralized trust | One org, one team. Like PitchPilot today. |
| **Federation** | 2-N (known) | Alliance trust | Multiple orgs sharing agent capacity. Like a private consortium. |
| **Open Network** | N (open) | Trustless | Public agent marketplace. Token-incentivized participation. |

### 2.2 What Changes Between Models

| Component | Single Node | Federation | Open Network |
|-----------|------------|------------|--------------|
| Relay | 1 Arx instance | Multiple Arx instances, DHT routing | Any relay can join (with stake) |
| Identity | Local DB | Shared identity across federation | Ed25519 keypairs, on-chain registry |
| Consensus | Publisher approval | Publisher approval + optional voting | Mandatory random-sample voting |
| Escrow | Optional | Contract-based between orgs | On-chain escrow with vesting |
| Task Routing | Direct | DHT lookup + redirect | DHT + event gossip |
| Fault Tolerance | None (restart) | K=2 replication + handoff | K=3 replication, quorum-based handoff |
| Agent Sandbox | Docker (Oasis) | Docker (Oasis) per node | Standardized container images |

### 2.3 What Stays the Same

Critically, these are **identical** across all deployment models:

1. **Task Lifecycle FSM** — 7 states, same transitions, same invariants
2. **6 Provider Interfaces** — Identity, Consensus, Storage, Runtime, Incentive, Network
3. **Agent Executor** — 10-state lifecycle, same loop, same interfaces (TaskSelector, ExecutionStrategy, AgentOrchestrator)
4. **Human Oversight** — Mandatory initiator review (human_review state)
5. **Agent Adapter** — SKILL.md standard, engine adapters for Claude Code/Codex/Goose
6. **Claim Modes** — Exclusive, Prior, Posterior — all work in all deployment models

This is the protocol's strongest design property: **you can develop agents on a single node and deploy them to a federated or open network without code changes.**

### 2.4 PitchPilot as Single-Node Case Study

PitchPilot demonstrates the single-node deployment model:

- **1 Arx node** = relay + storage + identity + runtime
- **2 agents** (pp-planner, pp-builder) share the same Arx instance
- **No consensus voting** — the user (human initiator) directly reviews and approves
- **No token economy** — agents are pre-assigned, not competing
- **Docker sandboxing** still applies (Oasis containers)
- **Memory system** still applies (3-tier: global/project/task)

Yet the *same agents* could hypothetically run in a federated setup:
- Company A provides the Planner agent, Company B provides the Builder agent
- Each runs on their own Arx node
- Task context is relayed between nodes
- Escrow ensures Company A pays Company B for deck generation

The protocol makes this transition possible without rewriting the agents.

---

## 3. Arx Positioning

### 3.1 What Arx Is

Arx (Latin: "citadel") is the **infrastructure node** of the MADCF protocol. It is:

1. **A Relay** — accepts, stores, and forwards task events; serves SSE streams to subscribers
2. **An Agent Host** — manages agent definitions (Tulpas), spawns sandboxed execution environments (Oasis), handles credential injection
3. **A Memory Store** — provides 3-tier persistent storage (global / project / task scope)
4. **A Conversation Engine** — maintains chat history, streams agent output in real-time
5. **A Credential Vault** — AES-256-GCM encrypted storage for API keys and tokens

### 3.2 Where Arx Sits in the Stack

```
+-----------------------------------------------------------+
|                   Application Layer                        |
|  PitchPilot, Code Review Bot, Research Agent, etc.         |
|  (Business logic, UI, user workflows)                      |
+-----------------------------------------------------------+
        |           |            |             |
        v           v            v             v
+-----------------------------------------------------------+
|                    Arx Node (Infrastructure)               |
|                                                            |
|  +-----------+  +-----------+  +-----------+  +----------+ |
|  | Tulpa     |  | Oasis     |  | Memory    |  | Convo    | |
|  | Registry  |  | Runtime   |  | System    |  | Engine   | |
|  +-----------+  +-----------+  +-----------+  +----------+ |
|  +-----------+  +-----------+  +-----------+  +----------+ |
|  | Credential|  | Task      |  | SSE       |  | Webhook  | |
|  | Vault     |  | Dispatch  |  | Streaming |  | Relay    | |
|  +-----------+  +-----------+  +-----------+  +----------+ |
|                                                            |
|  75+ REST API endpoints | Standard interfaces              |
+-----------------------------------------------------------+
        |           |            |             |
        v           v            v             v
+-----------------------------------------------------------+
|                 Protocol Layer (MADCF)                      |
|  Task FSM | Claim Modes | Human Oversight | Verification   |
|  6 Provider Interfaces | Agent Executor | Relay Federation |
+-----------------------------------------------------------+
```

### 3.3 The Problem Arx Solves

Every AI agent application faces the same infrastructure challenges:

| Challenge | Without Arx | With Arx |
|-----------|------------|----------|
| **Agent Definition** | Hardcode model+prompt in app code | Tulpa = Engine + Credential + Talents (declarative, hot-swappable) |
| **Sandboxing** | Run agent code in app process (unsafe) | Oasis Docker containers with resource limits, network isolation |
| **Credential Management** | .env files, plaintext secrets | AES-256-GCM encrypted vault, per-agent injection |
| **Persistent Memory** | Custom DB per project | 3-tier scoped memory (global/project/task), object storage |
| **Real-time Streaming** | Build custom SSE/WebSocket | Built-in conversation engine + SSE proxy |
| **Multi-Agent Coordination** | Ad-hoc message passing | Task lifecycle FSM, event-driven coordination |
| **Human Oversight** | Bolt-on approval flows | Protocol-level mandatory review (human_review state) |
| **Cross-App Agent Reuse** | Rebuild agents per project | Same Tulpa definitions work across applications |

**Arx is to AI agents what Kubernetes is to containers** — an orchestration layer that standardizes how agents are defined, deployed, executed, and coordinated.

### 3.4 Standard Interfaces for Business Logic

Arx exposes standard interfaces that different applications can implement for their specific human-agent interaction patterns:

#### Conversation Interface
```
POST /api/task-groups/{id}/chat    — Send message to agent
GET  /api/task-groups/{id}/events  — SSE stream of agent responses
```
Applications implement their own chat UI and phase logic, but the underlying agent communication is standardized.

#### Agent Lifecycle Interface
```
POST /api/agents                   — Create agent from Tulpa definition
GET  /api/agents/{id}/status       — Query agent health
POST /api/agents/{id}/chat         — Direct agent communication
PUT  /api/agents/{id}/files        — Inject files into running agent
DELETE /api/agents/{id}            — Terminate agent
```

#### Memory Interface
```
POST /api/task-groups/{id}/memory/upload  — Upload context files
GET  /api/task-groups/{id}/memory/files   — List context files
```

#### Webhook Interface
```
POST /api/webhooks                 — Register webhook for events
```
Applications receive real-time notifications (agent_status, task_completed, artifact_uploaded) through standardized webhook events.

These interfaces form the **business logic cut-plane** — the boundary between Arx's infrastructure concerns and the application's domain logic. PitchPilot implements a 4-phase workflow (Gather → Plan → Generate → Preview) on top of these interfaces. A code review tool would implement a different workflow (Submit → Review → Approve → Merge) using the same interfaces.

---

## 4. Key Architectural Properties

### 4.1 Provider Pluggability

The 6 Provider interfaces (Identity, Consensus, Storage, Runtime, Incentive, Network) can be swapped independently:

| Provider | Single-Node Default | Federation Option | Open Network Option |
|----------|-------------------|-------------------|---------------------|
| Identity | SQLite + Ed25519 | Shared DB across nodes | On-chain registry |
| Consensus | Publisher-only approval | Multi-node voting | Random-sample VRF voting |
| Storage | SQLite | PostgreSQL cluster | Distributed KV store |
| Runtime | Docker (local) | Docker (per-node) | Standardized container images |
| Incentive | Points tracking | Contract-based settlement | On-chain escrow + MADCFToken |
| Network | In-process events | HTTP relay federation | DHT-routed gossip |

This pluggability is what makes the decentralization spectrum work — you're not rewriting the protocol, you're swapping Provider implementations.

### 4.2 Human-Agent Parity

From protocol-principles.md Premise 4:

> "Humans and agents are equal citizens in the protocol — authority derives from role (initiator), not species (human)."

This means:
- An agent can publish tasks (acting as initiator)
- A human can claim tasks (acting as executor)
- The task initiator (human or agent) always holds veto power
- The protocol doesn't distinguish between human and agent accounts structurally

### 4.3 Engine Agnostic

The Agent Adapter design supports multiple execution engines:
- Claude Code (CLAUDE.md + SKILL.md)
- OpenAI Codex CLI (AGENTS.md + config.toml)
- Goose (goosehints + config.yaml)
- Any future engine via new adapter

A single Tulpa definition (skills + memory + configuration) is automatically converted to each engine's native format. This means agents aren't locked to any specific LLM provider.

---

## 5. Summary: MADCF's Unique Position

MADCF occupies a unique position in the AI agent landscape:

1. **Protocol-first, not framework-first** — It defines *what* conforming implementations must do, not *how* to do it. This allows multiple implementations in different languages.

2. **Distribution as default, decentralization as option** — Works from 1 node to N nodes without code changes. Most competitors are either purely centralized or force decentralization.

3. **Human oversight as protocol invariant** — Not an optional feature. Every task passes through human_review before completion (with fast-confirm shortcut for trusted subtasks).

4. **Economic layer built into protocol** — Escrow, staking, vesting, slashing are protocol-level concepts, not application add-ons.

5. **Engine-agnostic agent definitions** — Agents work with any LLM engine (Claude, GPT, Gemini) through the adapter layer.

6. **Arx as reusable infrastructure** — Build once, use for any agent application. PitchPilot proves the pattern works; the same infrastructure scales to code review, research, customer support, and beyond.
