# Module 00 — Fundamentals

> *"An agent is a system that uses a model to pursue a goal, taking actions in an environment over multiple steps, observing the results, and adapting."*

## The Big Mental Model

```
                    ┌──────────────────────────┐
                    │         GOAL             │
                    │  ("find me the cheapest  │
                    │   flight to Tokyo")      │
                    └─────────────┬────────────┘
                                  ▼
        ┌───────────────────────────────────────────────────┐
        │                  AGENT LOOP                       │
        │                                                   │
        │   ┌─────────┐    ┌─────────┐    ┌─────────────┐  │
        │   │ THINK   │───►│ ACT     │───►│ OBSERVE     │  │
        │   │ (LLM)   │    │ (Tool)  │    │ (Result)    │  │
        │   └─────────┘    └─────────┘    └──────┬──────┘  │
        │         ▲                              │          │
        │         │         loop until           │          │
        │         │         goal met or          │          │
        │         └──────────────────────────────┘          │
        └───────────────────────────────────────────────────┘
```

Three core ingredients, every time:
1. **A model** (the brain — decides what to do)
2. **A set of tools/actions** (the hands — things it can do)
3. **A loop** (the heartbeat — repeat until done)

Everything else in this course is variations on those three.

## Taxonomy: Chatbot vs. Workflow vs. Agent

| Type | What it is | Example |
|---|---|---|
| **Chatbot** | Single prompt in → single response out. No memory across turns (or minimal). | "Summarize this text" |
| **Workflow** | Fixed sequence of LLM calls + tool calls, *you* control the order. | RAG pipeline: retrieve → stuff → generate |
| **Agent** | *The model* decides the order of actions, adapts based on observation. | "Plan a 3-day trip to Tokyo under $1500" |

**Key distinction:** In a workflow, *you* are the control loop. In an agent, *the model* is the control loop.

This is why "agent" is a stronger (and riskier) commitment than "chatbot." More autonomy = more power = more failure modes.

## Key Terms to Know

- **System prompt** — the agent's "personality" and operating instructions
- **Tool / function** — a callable the agent can invoke with structured arguments
- **Observation** — the result of a tool call, fed back to the model
- **Trajectory** — the full sequence of (thought, action, observation) steps in a run
- **Termination** — when the agent stops (goal met, max steps, error, budget)
- **Grounding** — tying the model's output to real-world facts (via tools, RAG, etc.)
- **Hallucination** — confident-sounding output that isn't grounded
- **Sycophancy** — the model agreeing with the user even when wrong
- **Reinforcement from environment** — in production agents, "feedback" usually comes from tool results, not reward models

## The Four Failure Modes of Agents

Every agent fails in one of these four ways. Keep them in mind throughout the course:

1. **Wrong tool choice** — picked the right tools, called the wrong one
2. **Wrong arguments** — right tool, wrong params (often: missing a required field, wrong type)
3. **Wrong interpretation of observation** — tool returned data, model misread it
4. **Wrong termination** — stopped too early (gave up) or too late (infinite loop, wasted budget)

**80% of agent debugging is figuring out which of the four happened.**

## References (read before module 1)

- Anthropic: *"Building effective agents"* — https://www.anthropic.com/research/building-effective-agents
- OpenAI: *"A practical guide to building agents"* — https://cdp.openai.com/docs/guides/agents
- Berkeley: *"The Bergey Guide to Building Agents"* (if available)
- LangChain: *"State of AI Report"* (free PDF, has great agent failure-mode data)

## Exercises

See `exercises/` for the tasks. Summary:

### Exercise 0.1 — Taxonomy Practice (no code)
Classify 10 real-world LLM apps (Copilot, Cursor, ChatGPT, Perplexity, Devin, Replit Agent, etc.) as chatbot / workflow / agent. Write 2–3 sentences justifying each. *Deliverable: `notes/classification.md`*

### Exercise 0.2 — Draw an Agent
Pick a task you'd want an agent to do (e.g., "email triage", "code review bot", "research assistant"). On paper or in a tool like Excalidraw, sketch:
- The goal
- The tools the agent would need
- The loop (what it does in step 1, 2, 3)
- How it knows when to stop

*Deliverable: `notes/my_agent_design.md` + diagram*

### Exercise 0.3 — Failure Mode Journal
Read 5 blog posts / tweets / GitHub issues about agents failing. For each, identify which of the four failure modes it was. *Deliverable: `notes/failure_log.md`*

## Definition of Done for Module 00

- [ ] Can explain the difference between workflow and agent in one sentence
- [ ] Have a written design for one agent you'd want to build
- [ ] Can name the four failure modes without looking them up
