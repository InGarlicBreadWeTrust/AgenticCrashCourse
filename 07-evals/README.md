# Module 07 — Evals for Agents

> If you can't measure it, you can't improve it. Evals are how you know your agent is getting better (or worse) with each change.

## Why Evals Are Harder for Agents Than for Models

| Traditional ML | Agent Eval |
|---|---|
| Single input → single output | Multi-step trajectory |
| Ground truth is a label | Ground truth is a *sequence* of actions |
| Accuracy, F1 | Task success rate, step efficiency, cost |
| Test once | Must eval across *many* scenarios |

**The key insight:** you're not just evaluating the final answer. You're evaluating the *path* the agent took to get there.

## Types of Evals

### 1. Task-Level Evals
Did the agent accomplish the goal?

```python
@dataclass
class TaskEval:
    task_id: str
    input: str
    expected_output: str  # or a set of criteria
    agent_output: str
    success: bool
    steps_taken: int
    tokens_used: int
    cost_usd: float
    latency_ms: int
```

**Metrics:**
- **Success rate** — % of tasks completed correctly
- **Partial credit** — % of sub-goals met
- **Step efficiency** — steps taken vs. optimal
- **Cost** — $ per successful task

### 2. Step-Level Evals
Was each individual step correct?

```python
@dataclass
class StepEval:
    step: int
    thought: str
    action: str
    args: dict
    observation: str
    was_action_correct: bool
    were_args_correct: bool
    was_interpretation_correct: bool
```

**This is where you diagnose *why* a task failed.**

### 3. LLM-as-Judge
Use a (usually stronger) LLM to evaluate the agent's output.

```python
JUDGE_PROMPT = """
You are an impartial judge. Evaluate the agent's response to the user's query.

## User Query
{query}

## Agent Response
{response}

## Evaluation Criteria
1. **Correctness** (0-10): Is the factual content accurate?
2. **Completeness** (0-10): Does it address all parts of the query?
3. **Clarity** (0-10): Is it well-organized and easy to understand?
4. **Safety** (0-10): Does it avoid harmful, biased, or inappropriate content?

## Scoring
Return JSON: {{"correctness": int, "completeness": int, "clarity": int, "safety": int, "reasoning": str}}
"""
```

**Caveats:**
- LLM judges are biased (prefer their own style, longer answers, etc.)
- Use a *different* model as judge than the agent uses
- Calibrate: run the judge on known-good and known-bad outputs first

### 4. Regression Evals
Run your eval suite on every change. If a metric drops >2%, block the deploy.

```yaml
# .github/workflows/eval.yml
name: Agent Evals
on: [pull_request]
jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.11" }
      - run: pip install -r requirements.txt
      - run: python -m pytest evals/ -v --tb=short
      - name: Check regression
        run: |
          if python evals/check_regression.py --threshold 0.02; then
            echo "✅ No regression"
          else
            echo "❌ Regression detected"
            exit 1
          fi
```

### 5. Online Metrics (Production)
What users actually experience:

| Metric | How to Measure |
|---|---|
| **User satisfaction** | Thumbs up/down, NPS survey |
| **Task completion rate** | % of sessions that end with a "done" state |
| **Follow-up rate** | % of responses that get a follow-up question (low = good) |
| **Escalation rate** | % of sessions that need human handoff |
| **Latency (p50, p95, p99)** | From tracing |
| **Cost per session** | From billing logs |
| **Error rate** | % of sessions with unhandled errors |

## Building an Eval Suite

### Structure
```
evals/
├── tasks/
│   ├── basic_qa.yaml          # 50 simple Q&A tasks
│   ├── multi_step.yaml        # 30 multi-step tasks
│   ├── edge_cases.yaml        # 20 adversarial / edge cases
│   └── safety.yaml            # 30 safety-critical tasks
├── judges/
│   ├── correctness.py         # LLM-as-judge for correctness
│   ├── completeness.py        # LLM-as-judge for completeness
│   └── safety.py              # Rule-based + LLM safety check
├── runners/
│   ├── run_task.py            # Execute a single task
│   └── run_suite.py           # Run a full suite, aggregate results
├── baselines/
│   └── baseline_2024-12-01.json  # Saved results for comparison
└── check_regression.py        # Compare current vs. baseline
```

### Task Definition (YAML)
```yaml
- id: flight_search_001
  category: multi_step
  input: "Find the cheapest flight from SFO to Tokyo on March 15, 2025, under $1200"
  expected:
    must_include:
      - "flight"
      - "Tokyo"
      - "SFO"
    must_not_include:
      - "I don't have access"
    tool_calls:
      - name: search_flights
        args:
          origin: "SFO"
          destination: ["HND", "NRT"]
          date: "2025-03-15"
          max_price: 1200
  max_steps: 5
  max_tokens: 5000
  timeout_s: 30
```

### Running the Suite
```python
async def run_suite(tasks: list[Task], agent: Agent) -> EvalReport:
    results = []
    for task in tasks:
        try:
            trajectory = await asyncio.wait_for(
                agent.run(task.input),
                timeout=task.timeout_s
            )
            result = evaluate(task, trajectory)
            results.append(result)
        except asyncio.TimeoutError:
            results.append(EvalResult(task.id, success=False, error="timeout"))
    return EvalReport(results)
```

## Eval Anti-Patterns

| Anti-pattern | Why it's bad |
|---|---|
| **Evaling only happy paths** | You'll miss the edge cases that break in production |
| **Using the same model as judge and agent** | Self-preference bias |
| **Only evaling the final output** | You can't diagnose *why* it failed |
| **Running evals once** | Models drift, prompts change, tools update — run on every PR |
| **100 tasks is enough** | You need 500+ for statistical significance on the metrics that matter |
| **Ignoring cost in evals** | A 95% success rate at 5x the cost might be worse than 90% at 1x |

## Tools & Frameworks

| Tool | Strength |
|---|---|
| **DeepEval** | pytest-style, LLM-as-judge built-in, easy to adopt |
| **Ragas** | RAG-specific metrics (faithfulness, answer relevance) |
| **Braintrust** | Cloud-based, good for team collaboration |
| **LangSmith** | Tracing + evals, integrated with LangChain |
| **Arize Phoenix** | Open-source, OpenTelemetry-based |
| **Custom (pytest + your own)** | Full control, no vendor lock-in |

**Recommendation:** start with custom (pytest + your own judges). You'll understand the mechanics. Then layer in DeepEval or LangSmith for the team workflow.

## References

- OpenAI: *"Evals"* docs
- Anthropic: *"How we evaluate Claude"*
- DeepEval docs
- *"Evals for LLM Applications"* (various blog posts, 2024-2025)
- METR: *"Measuring AI Ability to Complete Long Tasks"*

## Exercises

### Exercise 7.1 — Build a Task Eval Suite (code)
Create 20 tasks for your agent (mix of simple, multi-step, and edge cases). Define them in YAML. Build a runner that:
- Executes each task
- Checks the output against criteria
- Records steps, tokens, cost, latency
- Outputs a JSON report

*Deliverable: `evals/tasks/basic.yaml` + `evals/runners/run_suite.py` + sample report*

### Exercise 7.2 — LLM-as-Judge (code)
Implement an LLM judge that scores:
- Correctness (0-10)
- Completeness (0-10)
- Safety (0-10)

Test it on 10 known-good and 10 known-bad outputs. Calibrate. Document the bias you observe.

*Deliverable: `evals/judges/llm_judge.py` + `notes/calibration.md`*

### Exercise 7.3 — Regression Detection (code)
Build `check_regression.py` that:
- Loads the current eval results
- Loads the baseline
- Compares key metrics (success rate, avg cost, p95 latency)
- Fails if any metric drops >2%
- Generates a human-readable diff report

*Deliverable: `evals/check_regression.py` + sample diff output*

## Definition of Done for Module 07

- [ ] Have 50+ eval tasks covering happy paths and edge cases
- [ ] Can run the full suite in <10 minutes
- [ ] Have an LLM-as-judge that's calibrated (you know its biases)
- [ ] Have regression detection that blocks bad deploys
- [ ] Can explain *why* a task failed by looking at step-level evals
