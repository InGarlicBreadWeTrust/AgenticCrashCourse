# Module 99 — Capstone: Ship a Production Agent

> This is where everything comes together. Build one agent, end-to-end, with all the modules applied.

## The Assignment

Build a **Research Assistant Agent** that:

1. **Takes a research question** from the user
2. **Plans** the research (what to search, what sources to check)
3. **Executes** the research (web search, document reading, code analysis)
4. **Synthesizes** findings into a structured report
5. **Handles** follow-up questions with context from the original research

## Requirements

### Must-Have (MVP)
- [ ] ReAct or Plan-Execute loop (your choice, justify in `notes/architecture.md`)
- [ ] At least 4 tools (e.g., web_search, read_document, calculator, code_runner)
- [ ] Working memory with compaction
- [ ] Structured output (JSON report with sections: Summary, Findings, Sources, Confidence)
- [ ] Budget management (max tokens, max $, max steps)
- [ ] Error handling (retries, graceful failure)
- [ ] Basic guardrails (PII filter, tool permissions)
- [ ] At least 10 eval tasks with automated scoring
- [ ] Tracing (OpenTelemetry → Langfuse or Jaeger)
- [ ] Cost tracking per run
- [ ] README with setup, usage, and architecture decisions

### Nice-to-Have (Stretch)
- [ ] Multi-agent: Researcher + Writer + Critic
- [ ] Episodic memory (remember past research sessions)
- [ ] RAG over a knowledge base
- [ ] Human-in-the-loop for expensive actions
- [ ] Web UI (FastAPI + simple frontend)
- [ ] Docker deployment
- [ ] 50+ eval tasks
- [ ] Cost optimization (model routing: cheap model for simple steps, expensive for complex)

## Architecture Decisions to Document

In `notes/architecture.md`, answer:

1. **Loop pattern:** Why ReAct vs. Plan-Execute vs. something else?
2. **Tool design:** Why these tools? Why these schemas?
3. **Memory:** What's in context vs. retrieved? Why?
4. **Orchestration:** Single agent or multi-agent? Why?
5. **Guardrails:** What can go wrong? What's your defense?
6. **Evals:** What are your 10 (or 50) tasks? What does "success" look like?
7. **Observability:** What are you tracing? What would you alert on?
8. **Cost:** What's your target cost per run? How do you optimize?

## Submission

Your capstone is complete when:
- [ ] `docker compose up` (or equivalent) starts the agent
- [ ] `curl -X POST /research -d '{"question": "What is the current state of quantum computing error correction?"}'` returns a structured report
- [ ] `pytest evals/` passes with >80% success rate
- [ ] You can view a trace in Langfuse/Jaeger for any run
- [ ] You can show the cost per run in the dashboard
- [ ] `notes/architecture.md` answers all 8 questions

## Timeline Suggestion (4 weeks)

| Week | Focus |
|---|---|
| 1 | Core loop + tools + basic memory |
| 2 | Guardrails + evals (first 10 tasks) |
| 3 | Observability + cost tracking + dashboard |
| 4 | Polish + stretch goals + writeup |

## Rubric (for self-assessment)

| Category | Weight | What "Excellent" Looks Like |
|---|---|---|
| Core loop reliability | 25% | >90% success on evals, graceful error handling |
| Tool design quality | 15% | Clear schemas, good descriptions, proper error messages |
| Memory & context | 15% | Appropriate compaction, no context overflow, RAG if applicable |
| Guardrails | 15% | PII filtered, permissions enforced, injection defended |
| Evals | 15% | 10+ tasks, automated, regression detection |
| Observability | 10% | Full traces, dashboard, cost tracking |
| Documentation | 5% | Clear README, architecture decisions, setup instructions |

**Total: 100%. Aim for >80% across all categories.**

## You've Got This

By the time you reach this module, you've built every piece of the agent stack from scratch. This is just assembly.

Go ship it.
