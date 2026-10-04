# Module 08 — Observability & Tracing

> You can't debug what you can't see. Observability is how you understand what your agent is actually doing in production.

## The Three Pillars (Applied to Agents)

| Pillar | Traditional | Agent-Specific |
|---|---|---|
| **Metrics** | CPU, memory, latency | Tokens, cost, step count, tool call distribution |
| **Logs** | Structured logs | Thought/action/observation traces |
| **Traces** | Request → response | Multi-step trajectory with all LLM calls, tool calls, decisions |

## What to Trace

Every agent run should produce a **trace** that captures:

```
Trace: agent_run_abc123
├── Span: user_input
│   └── "Find the cheapest flight to Tokyo"
├── Span: llm_call_1 (thinking)
│   ├── Model: gpt-4o
│   ├── Tokens in: 850
│   ├── Tokens out: 120
│   ├── Latency: 1.2s
│   ├── Cost: $0.003
│   └── Output: "I need to search for flights. Let me use search_flights..."
├── Span: tool_call_1 (search_flights)
│   ├── Args: {origin: "SFO", destination: "HND", date: "2025-03-15"}
│   ├── Latency: 0.3s
│   ├── Status: success
│   └── Result: 5 flights found
├── Span: llm_call_2 (synthesis)
│   ├── Model: gpt-4o
│   ├── Tokens in: 1200
│   ├── Tokens out: 250
│   ├── Latency: 1.8s
│   ├── Cost: $0.005
│   └── Output: "The cheapest flight is Delta, $487, departing 10:30 AM..."
└── Span: final_response
    └── "The cheapest flight to Tokyo on March 15 is Delta at $487..."

Total: 3 LLM calls, 1 tool call, 2.3s latency, $0.008 cost, 2170 tokens
```

## Tracing with OpenTelemetry

OpenTelemetry (OTel) is the vendor-neutral standard. Use it to emit traces, then send them to any backend.

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

# Setup
provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4318")))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("my_agent")

# Usage
with tracer.start_as_current_span("llm_call") as span:
    span.set_attribute("llm.model", "gpt-4o")
    span.set_attribute("llm.tokens_in", 850)
    span.set_attribute("llm.tokens_out", 120)
    span.set_attribute("llm.cost_usd", 0.003)
    result = llm.chat(messages)
    span.set_attribute("llm.output_preview", result[:200])
```

## Tracing Backends

| Backend | Type | Strength |
|---|---|---|
| **LangSmith** | Commercial | Best for LangChain/LangGraph, great UI |
| **Langfuse** | OSS + Cloud | OpenTelemetry-native, self-hostable |
| **Arize Phoenix** | OSS | Great for ML teams, OpenTelemetry-based |
| **Honeycomb** | Commercial | Best for high-cardinality debugging |
| **Jaeger / Tempo** | OSS | General-purpose, great for infra |
| **Grafana + Tempo** | OSS | Metrics + traces in one UI |

**Recommendation:** Langfuse (free tier, OSS, OTel-compatible) for teams. Honeycomb if you're already using it.

## Key Agent Metrics

### Per-Run
```python
@dataclass
class RunMetrics:
    run_id: str
    timestamp: datetime
    user_id: str
    task: str
    success: bool
    steps: int
    llm_calls: int
    tool_calls: int
    tokens_in: int
    tokens_out: int
    total_tokens: int
    cost_usd: float
    latency_ms: int
    error: str | None
```

### Aggregate (dashboard)
```
┌─────────────────────────────────────────────────────────┐
│                    AGENT DASHBOARD                        │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Success Rate│  │  Avg Cost   │  │  P95 Latency│     │
│  │   94.2%     │  │  $0.012/run │  │   3.2s      │     │
│  │  (+0.3%)    │  │  (-8%)      │  │  (-12%)     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                         │
│  Tool Call Distribution                                  │
│  search_flights  ████████████████████  45%              │
│  get_weather     ████████████          22%              │
│  calculator      ██████                12%              │
│  other           ████████              21%              │
│                                                         │
│  Failure Breakdown                                       │
│  Wrong tool      ████████          35%                  │
│  Wrong args      ██████            25%                  │
│  Bad observation ████████          30%                  │
│  Termination     ███               10%                  │
│                                                         │
│  Cost Trend (7 days)                                     │
│  $0.015 ┤                                              │
│  $0.012 ┤──────●──────●──────●──────●                  │
│  $0.009 ┤                                              │
│  $0.006 ┤                                              │
│         └──┬──┬──┬──┬──┬──┬──┬──                       │
│           Mon Tue Wed Thu Fri Sat Sun                    │
└─────────────────────────────────────────────────────────┘
```

## Cost Tracking

```python
class CostTracker:
    PRICING = {
        "gpt-4o": {"in": 2.50, "out": 10.00},       # per 1M tokens
        "gpt-4o-mini": {"in": 0.15, "out": 0.60},
        "claude-3-5-sonnet": {"in": 3.00, "out": 15.00},
        "qwen3-8b": {"in": 0.00, "out": 0.00},       # local = free
    }

    def cost(self, model: str, tokens_in: int, tokens_out: int) -> float:
        p = self.PRICING.get(model, {"in": 1.0, "out": 3.0})
        return (tokens_in / 1_000_000) * p["in"] + (tokens_out / 1_000_000) * p["out"]

    def track(self, model: str, tokens_in: int, tokens_out: int):
        c = self.cost(model, tokens_in, tokens_out)
        self.total_cost += c
        self.calls.append({"model": model, "tokens": tokens_in + tokens_out, "cost": c})
        if self.total_cost > self.budget:
            raise BudgetExceeded(f"Cost ${self.total_cost:.4f} exceeds budget ${self.budget}")
```

## Failure Analysis

When a run fails, the trace should let you answer:
1. **Where** did it fail? (which step)
2. **What** was the model thinking? (the thought at that step)
3. **What** did it try to do? (the action)
4. **What** came back? (the observation)
5. **Why** was that wrong? (your analysis)

This is the "5 whys" of agent debugging. Without traces, you're guessing.

## Alerting

Set up alerts for:
- **Success rate** drops below 90% (5-min window)
- **P95 latency** exceeds 10s
- **Cost per run** exceeds 3x the 7-day average
- **Error rate** exceeds 5%
- **Tool call failure rate** exceeds 10% for any specific tool

## Structured Logging (Complement to Tracing)

```python
import structlog

log = structlog.get_logger()

log.info("agent.run_start",
    run_id=run_id,
    user_id=user_id,
    task=task[:100],
    model=model,
)

log.info("agent.llm_call",
    run_id=run_id,
    step=step,
    model=model,
    tokens_in=tokens_in,
    tokens_out=tokens_out,
    latency_ms=latency,
    cost_usd=cost,
)

log.warning("agent.tool_error",
    run_id=run_id,
    step=step,
    tool=tool_name,
    error=str(e),
)

log.info("agent.run_end",
    run_id=run_id,
    success=success,
    total_steps=steps,
    total_cost=total_cost,
    total_latency_ms=total_latency,
)
```

## References

- OpenTelemetry docs: https://opentelemetry.io
- Langfuse docs: https://langfuse.com/docs
- LangSmith docs: https://docs.smith.langchain.com
- Arize Phoenix docs: https://docs.arize.com/phoenix
- Honeycomb: *"The Art of SRE"* (great for thinking about observability)

## Exercises

### Exercise 8.1 — Instrument Your Agent (code)
Add OpenTelemetry tracing to your agent from Module 02. Every LLM call and tool call should produce a span with:
- Model name
- Tokens in/out
- Latency
- Cost
- Output preview (first 200 chars)

Export to Jaeger (Docker) or Langfuse (cloud). View the trace in the UI.

*Deliverable: `exercises/ex01/traced_agent.py` + screenshot of trace*

### Exercise 8.2 — Metrics Dashboard (code)
Build a simple dashboard (FastAPI + a basic HTML/JS frontend, or Grafana) that shows:
- Success rate (last 24h, 7d, 30d)
- Avg cost per run
- P50/P95/P99 latency
- Tool call distribution
- Failure breakdown

Feed it data from your trace store.

*Deliverable: `exercises/ex02/dashboard/` + screenshot*

### Exercise 8.3 — Cost Analyzer (code)
Build a script that:
- Reads all traces from the last 7 days
- Groups by model, tool, task type
- Computes cost per category
- Identifies the top 5 most expensive runs
- Suggests optimizations (e.g., "30% of runs use gpt-4o for simple tasks — switch to gpt-4o-mini")

*Deliverable: `exercises/ex03/cost_analysis.py` + sample output*

## Definition of Done for Module 08

- [ ] Every agent run produces a complete trace
- [ ] Can view a trace in a UI and understand what happened step by step
- [ ] Have a dashboard with the key metrics
- [ ] Can identify the most expensive runs and suggest optimizations
- [ ] Have alerts set up for critical metrics
