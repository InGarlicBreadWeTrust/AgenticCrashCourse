# Agentic AI — From Zero to Shipping Agents

A hands-on learning path that takes you from "I know what an LLM is" to "I can design, build, eval, and operate production agents."

## Philosophy

- **Build, don't just read.** Every module ends with a working artifact.
- **Start from primitives.** You'll build a tiny agent harness from scratch before reaching for LangGraph / CrewAI / AutoGen.
- **Production-minded.** Evals, observability, and guardrails aren't an afterthought — they're first-class modules.
- **Model-agnostic.** Examples use OpenAI-compatible APIs so you can swap in Qwen, Llama, GPT, Claude, etc.

## Prerequisites

- Comfortable with Python 3.10+
- Basic async / await
- Have access to at least one LLM API (OpenAI, Anthropic, local via Ollama, or Qwen via DashScope)
- `git` for version control

## The Path

```
┌─────────────────────────────────────────────────────────────────┐
│ MODULE 00  Fundamentals          ← Where you are now            │
│   What is an "agent"? Mental models. Taxonomy.                  │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 01  Harness Engineering  ← The chassis of the agent      │
│   Prompt scaffolding, system prompts, context management        │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 02  Loop Engineering     ← The heartbeat                 │
│   ReAct, plan-act-observe, termination, retries, backoff        │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 03  Tool Design          ← The hands                     │
│   Schema design, tool routing, MCP, sandboxing                  │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 04  Memory Architecture  ← The mind                      │
│   Working, episodic, semantic memory. RAG. Summarization        │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 05  Orchestration        ← The choreography              │
│   Single-agent, multi-agent, supervisor, swarm, graphs          │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 06  Guardrails           ← The seatbelt                  │
│   Permissions, sandboxing, PII, content filters, human-in-loop  │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 07  Evals                ← The scorecard                 │
│   Task evals, LLM-as-judge, regression suites, online metrics   │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 08  Observability        ← The dashboard                 │
│   Tracing (OpenTelemetry / LangSmith / Langfuse), token costs,  │
│   latency, failure analysis                                       │
├─────────────────────────────────────────────────────────────────┤
│ MODULE 99  Capstone             ← Ship it                       │
│   Build a production-grade agent with all of the above.         │
└─────────────────────────────────────────────────────────────────┘
```

## How to Use This Repo

1. **Read the module README** — concepts, key terms, references.
2. **Work through the `exercises/` folder** — each one is a small, buildable task.
3. **Run the `demo/` if present** — reference implementation.
4. **Keep a `notes.md`** in each module for your own thinking.
5. **Don't skip to the next module** until you've *shipped* the exercise.

## Time Estimate

- **Fast track** (strong Python, shipped an LLM app): 8–10 weeks @ 4–6 hrs/week
- **Standard track**: 14–16 weeks
- **Deep track**: 20–24 weeks

## Stack Recommendations

| Concern | Default pick | Alternative |
|---|---|---|
| Language | Python 3.11+ | TypeScript |
| LLM client | `openai` (any OpenAI-compat) | `litellm` for multi-provider |
| Local model | Ollama + Qwen / Llama | LM Studio |
| Tool protocol | Function calling (native) | MCP (Model Context Protocol) |
| Vector store | Chroma (local) | pgvector, Qdrant, Pinecone |
| Tracing | OpenTelemetry + Langfuse | LangSmith, Arize Phoenix |
| Evals | `deepeval` or custom pytest | Ragas, Braintrust |
| Multi-agent | Build your own first, then LangGraph | CrewAI, AutoGen, OpenAI Swarm |
| Deployment | FastAPI + Uvicorn | Modal, Fly.io, Vercel |

## Getting Started

```bash
cd ~/Documents/agents
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY=sk-...   # or use Ollama / DashScope / Anthropic
open 00-fundamentals/README.md
```

## What "Done" Looks Like

- ✅ Explain the difference between a chatbot, a workflow, and an agent
- ✅ Build a ReAct-style agent from scratch in <200 lines
- ✅ Design a tool schema an LLM can use reliably
- ✅ Implement short-term + long-term memory with appropriate backends
- ✅ Choose the right orchestration pattern for a given task
- ✅ Add guardrails that catch bad behavior (not just vibes)
- ✅ Write an eval suite that catches regressions before users do
- ✅ Trace a multi-step agent run and explain *why* it failed
- ✅ Ship one production-grade agent end-to-end

That's the bar. Let's get you there.
