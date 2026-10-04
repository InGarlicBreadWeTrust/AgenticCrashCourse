# Module 02 — Loop Engineering

> The loop is where agents live and die. A good loop is tight, observable, and knows when to stop.

## The Basic Loop

The canonical agent loop (ReAct style):

```
while not done:
    thought   = llm(think(goal, history, tools))
    action    = parse_action(thought)
    result    = execute(action)
    history.append(thought, action, result)
    done      = check_termination(goal, history)
```

That's it. Everything else is variations on this.

## Loop Patterns

### 1. ReAct (Reason + Act)
The default. Model interleaves reasoning with action.

```
Thought: I need to look up the weather in Tokyo.
Action: get_weather(city="Tokyo")
Observation: 22°C, partly cloudy
Thought: Good, now I'll recommend...
```

**Pros:** simple, flexible
**Cons:** can be verbose, model may "forget" the goal after several steps

### 2. Plan-Execute
Model first generates a full plan, then executes step by step.

```
Plan:
  1. Search flights to Tokyo
  2. Filter by price < $1500
  3. Pick cheapest
  4. Book (or return options)

Execute step 1: ...
Execute step 2: ...
```

**Pros:** more predictable, easier to debug
**Cons:** plan may be wrong, less adaptive

### 3. Reflexion / Self-Critique
After an action, the model critiques its own work before continuing.

```
Action: wrote the email
Critique: "This email is too long and misses the key deadline. Rewrite."
Action: rewrote the email
```

**Pros:** catches errors early
**Cons:** 2x the LLM calls, can over-correct

### 4. Tree of Thoughts
Model explores multiple paths in parallel, scores them, prunes.

**Pros:** best for hard problems
**Cons:** expensive, complex to implement

## Termination — When Does the Agent Stop?

This is where most agents fail. Common termination conditions:

| Condition | Example |
|---|---|
| **Goal met** | Model says "I'm done, here's the answer" |
| **Max steps** | Hard cap on iterations (e.g., 20) |
| **Budget** | Token or $ cap reached |
| **No progress** | Same action N times in a row |
| **Error** | Unrecoverable error |
| **User cancel** | User clicks stop |
| **Confidence** | Model's self-reported confidence < threshold |

**Always have at least TWO termination conditions.** "Until done" alone is how you end up with infinite loops.

## Retries, Backoff, and Resilience

```python
import random, time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=1, max=60),
    reraise=True,
)
def call_llm_with_resilience(...):
    ...
```

**Key principles:**
- **Idempotent tools** — safe to retry (GET requests, read-only DB queries)
- **Non-idempotent tools** — NOT safe to retry blindly (POST, send email, place order)
- **Distinguish retryable errors** (timeout, 429, 500) from non-retryable (400, 403)

## Step Budgeting

```python
class StepBudget:
    def __init__(self, max_steps=20, max_tokens=50_000, max_usd=2.0):
        self.max_steps = max_steps
        self.max_tokens = max_tokens
        self.max_usd = max_usd
        self.steps = 0
        self.tokens = 0
        self.usd = 0.0

    def consume(self, tokens: int, usd: float):
        self.steps += 1
        self.tokens += tokens
        self.usd += usd

    def exhausted(self) -> bool:
        return (self.steps >= self.max_steps or
                self.tokens >= self.max_tokens or
                self.usd >= self.max_usd)
```

## Observability in the Loop

Every loop iteration should log:
- Step number
- Thought / action / observation (truncated)
- Tokens used
- Time taken
- Budget remaining

This is what makes debugging possible. (Module 08 goes deep on this.)

## References

- Yao et al. — *"ReAct: Synergizing Reasoning and Acting in Language Models"* (2022)
- Shinn et al. — *"Reflexion: Language Agents with Verbal Reinforcement Learning"* (2023)
- Yao et al. — *"Tree of Thoughts: Deliberate Problem Solving with Large Language Models"* (2023)
- LangGraph docs on stateful loops

## Exercises

### Exercise 2.1 — Build a ReAct Loop (code)
Build a working ReAct agent with:
- A `Thought → Action → Observation` loop
- Max 10 steps
- Two simple tools (e.g., `calculator` and `web_search` stub)
- Clear termination on "Final Answer:" marker

*Deliverable: `exercises/ex01/react_agent.py`*

### Exercise 2.2 — Plan-Execute Variant (code)
Refactor to a Plan-Execute pattern:
- Step 1: LLM generates a JSON plan (list of steps)
- Step 2: Execute each step, feeding results back
- Step 3: LLM synthesizes the final answer

Compare the two on the same task. Which is more reliable? Why?

*Deliverable: `exercises/ex02/plan_execute.py` + `notes/comparison.md`*

### Exercise 2.3 — Termination & Budget (code)
Add a `StepBudget` class to your agent. Test that it correctly stops when:
- Max steps reached
- Token budget exceeded
- No progress detected (same action 3x)

*Deliverable: `exercises/ex03/budget.py` + tests*

## Definition of Done for Module 02

- [ ] Can build a ReAct loop from scratch
- [ ] Understand the tradeoffs of ReAct vs. Plan-Execute vs. Reflexion
- [ ] Have working termination conditions (at least 2)
- [ ] Can implement budget-aware stopping
