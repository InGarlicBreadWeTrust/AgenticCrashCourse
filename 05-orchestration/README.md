# Module 05 — Orchestration

> Orchestration is how multiple agents (or agent steps) coordinate to accomplish a goal.

## When Do You Need Orchestration?

**Single agent is enough when:**
- The task fits in one context window
- One set of tools covers the job
- The reasoning is linear

**You need orchestration when:**
- The task requires specialized expertise (research + coding + design)
- The context is too large for one agent
- You need parallelism (research 10 sources simultaneously)
- You need separation of concerns (planner vs. executor vs. critic)

## Orchestration Patterns

### 1. Sequential Pipeline
```
Agent A → Agent B → Agent C → Result
```
Simple, predictable. Each agent's output is the next agent's input.

**Use when:** steps have a clear dependency chain.
**Example:** Research → Draft → Edit → Final

### 2. Parallel Fan-out / Fan-in
```
         ┌── Agent A ──┐
Input ───┼── Agent B ──┼──► Synthesizer ──► Result
         └── Agent C ──┘
```
**Use when:** subtasks are independent.
**Example:** Research 5 topics in parallel → synthesize into one report

### 3. Supervisor / Router
```
         ┌── Worker A ──┐
         ├── Worker B ──┤
Supervisor ┼──────────────┼──► Result
         ├── Worker C ──┤
         └── Worker D ──┘
```
A "manager" agent decides which worker to call, in what order.

**Use when:** the routing decision is complex and context-dependent.
**Example:** Customer support → route to billing agent, tech agent, or escalation

### 4. Hierarchical
```
        CEO Agent
       /      |      \
   Team A   Team B   Team C
   / \      / \      / \
  W  W     W  W     W  W
```
**Use when:** the problem decomposes into sub-problems, each of which decomposes further.
**Example:** "Build a web app" → Plan → (Design, Backend, Frontend, Testing)

### 5. Swarm / Decentralized
Agents talk to each other peer-to-peer, no central coordinator.

**Use when:** you want emergent behavior (rare in production).
**Example:** OpenAI Swarm, early AutoGen

### 6. Graph (State Machine)
Define agents as nodes, transitions as edges. Explicit control flow.

**Use when:** you need complex, conditional routing with loops and branches.
**Example:** LangGraph, Temporal workflows

## Choosing a Pattern

| Question | If yes → |
|---|---|
| Is the task a fixed sequence? | Sequential pipeline |
| Are subtasks independent? | Parallel fan-out |
| Does routing depend on context? | Supervisor |
| Does it decompose recursively? | Hierarchical |
| Do you need loops and complex state? | Graph |
| Is it simple enough for one agent? | **Just use one agent** |

**The #1 mistake:** using multi-agent when a single agent with good tools would work. Multi-agent adds cost, latency, and debugging complexity. Don't do it by default.

## Communication Between Agents

How do agents pass information?

| Method | When |
|---|---|
| **Structured JSON** | Most common. Define a shared schema. |
| **Natural language** | Flexible but lossy. Good for summarization. |
| **Shared memory / blackboard** | Agents read/write to a shared state. Good for parallel workers. |
| **Messages** | Explicit send/receive. Good for supervisor patterns. |

**Rule:** use structured data for machine-to-machine, natural language for human-facing summaries.

## Cost & Latency Considerations

Multi-agent is **expensive**:
- Each agent call = at least 1 LLM call
- A 3-agent sequential pipeline = 3x the base cost
- Parallel fan-out with 5 workers = 5x the base cost (but same latency)

**Optimization strategies:**
- Cache agent outputs (same input → same output)
- Use cheaper models for workers, expensive model for supervisor
- Batch parallel calls
- Short-circuit: if the supervisor is confident, skip the worker

## Frameworks (know them, but build your own first)

| Framework | Strength | Weakness |
|---|---|---|
| **LangGraph** | Graph-based, stateful, production-ready | Steep learning curve |
| **CrewAI** | Role-based, easy to define teams | Less control over flow |
| **AutoGen** | Research-oriented, flexible | Hard to make production-stable |
| **OpenAI Swarm** | Simple, educational | Not production-ready |
| **Your own** | Full control, no abstraction tax | More code to write |

**Recommendation:** build your own supervisor pattern in <200 lines before touching a framework. You'll understand the abstractions better.

## References

- Anthropic: *"Building effective agents"* (patterns section)
- LangGraph docs
- Cognition: *"Don't build multi-agents"* (great counter-argument, read it)
- *"Patterns for Multi-Agent Systems"* (various blog posts, 2024-2025)

## Exercises

### Exercise 5.1 — Sequential Pipeline (code)
Build a 3-agent pipeline:
1. **Researcher** — gathers facts on a topic
2. **Writer** — drafts a report from the facts
3. **Editor** — reviews and improves the draft

Each agent is a separate LLM call with its own system prompt. Pass structured data between them.

*Deliverable: `exercises/ex01/pipeline.py`*

### Exercise 5.2 — Parallel Fan-out (code)
Refactor to parallel:
- 3 researchers work on different subtopics simultaneously (use `asyncio.gather`)
- A synthesizer combines their outputs

Measure: sequential vs. parallel latency.

*Deliverable: `exercises/ex02/parallel.py` + `notes/latency.md`*

### Exercise 5.3 — Supervisor Pattern (code)
Build a supervisor that:
- Receives a user request
- Decides which worker to call (research, code, math, general)
- Passes the request to the right worker
- Returns the worker's response

The supervisor uses an LLM call to make the routing decision.

*Deliverable: `exercises/ex03/supervisor.py`*

## Definition of Done for Module 05

- [ ] Can name the 6 orchestration patterns and when to use each
- [ ] Have built a sequential pipeline and a parallel fan-out
- [ ] Understand the cost implications of multi-agent
- [ ] Can argue for *or against* multi-agent for a given task
