# Module 01 — Harness Engineering

> The harness is everything *around* the model that makes it useful: prompts, context management, output parsing, error handling, the API layer.

## What Is the "Harness"?

Think of the LLM as an engine. The harness is the car:
- The **steering** (system prompt + few-shot examples)
- The **dashboard** (context window management)
- The **transmission** (parsing structured outputs)
- The **safety features** (validation, retries, error recovery)

A great model with a bad harness = garbage. A mediocre model with a great harness = often wins in production. **This is where most of your engineering effort should go.**

## Core Components

### 1. System Prompt Design

The system prompt is your agent's constitution. It should answer:
- **Who** are you? (role, persona)
- **What** can you do? (tools, capabilities)
- **How** should you behave? (style, constraints)
- **What** are the boundaries? (refusals, safety)

**Anti-pattern:** one giant 5000-word prompt that tries to cover everything.
**Better:** modular, with clear sections, and *tested* (you'll build this in Module 07).

```python
SYSTEM_PROMPT = """
You are {role}.

## Capabilities
{tool_descriptions}

## Behavior
- Be concise. Default to <200 words unless asked for detail.
- Always cite which tool returned which fact.
- If you don't know, say "I don't know" — do not guess.

## Boundaries
- Never fabricate URLs, emails, or phone numbers.
- Refuse requests that involve {safety_list}.
"""
```

### 2. Context Window Management

The context window is your **working memory**. It's finite, expensive, and degrades with size.

**Key concepts:**
- **Token budget** — how much context you're willing to spend per turn
- **Relevance ranking** — what to keep, what to drop
- **Compaction** — summarizing older turns when the window fills
- **Sliding window** — keep last N turns, drop older (naive but works)
- **Hierarchical memory** — hot (in-context), warm (RAG), cold (DB)

**Rule of thumb:** every token you put in context is a token you *can't* use for the answer. Spend it wisely.

### 3. Structured Output Parsing

Models are great at JSON, but not *perfect*. Your harness needs to:
- **Constrain the output** (use `response_format`, JSON mode, or function calling)
- **Validate the output** (pydantic, jsonschema)
- **Recover from bad output** (retry with the parse error in the next prompt)

```python
# The "repair loop" — a pattern you'll use everywhere
def call_with_repair(client, schema, messages, max_retries=3):
    for i in range(max_retries):
        resp = client.chat(messages)
        try:
            return schema.model_validate_json(resp.content)
        except ValidationError as e:
            messages.append(user_msg(f"Your output was invalid: {e}. Try again."))
    raise OutputParsingError("Could not get valid output after retries")
```

### 4. Provider Abstraction

Don't hardcode OpenAI. Wrap your LLM calls so you can swap providers:

```python
class LLM:
    def chat(self, messages, tools=None, **kwargs) -> LLMResponse: ...
    def stream(self, messages, **kwargs) -> Iterator[Token]: ...

class OpenAI(LLM): ...
class Ollama(LLM): ...
class DashScope(LLM): ...
```

The `litellm` library does this for you — use it, but understand what it's doing.

## The Harness Quality Checklist

Before you ship an agent, your harness should handle:

- [ ] Model returns malformed JSON → retry with error in context
- [ ] Model times out → retry with backoff
- [ ] Model refuses → detect and surface to user (don't silently fail)
- [ ] Context overflows → compact / truncate gracefully
- [ ] Tool returns an error → model sees the error and adapts
- [ ] API rate limit → exponential backoff
- [ ] Token budget exceeded → stop gracefully, return partial result
- [ ] User interrupts mid-stream → clean up state

## References

- Anthropic: *"Prompt Engineering"* docs — https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- OpenAI: *"Structured Outputs"* — https://platform.openai.com/docs/guides/structured-outputs
- Simon Willison's blog (prompt patterns, context engineering)
- *"The Art of Prompting"* by David Ha

## Exercises

### Exercise 1.1 — Build a Minimal Harness (code)
Build a Python class `Agent` with:
- `chat(user_message: str) -> str`
- System prompt from a template
- Retry logic (3 retries with exponential backoff)
- Token counting (log tokens in/out per call)

**Don't** add tools yet. Just: user → LLM → response, with robust error handling.

*Deliverable: `exercises/ex01/harness.py` + `test_harness.py`*

### Exercise 1.2 — Structured Output with Repair (code)
Extend the harness to return structured output (e.g., a `SearchQuery` pydantic model). Implement the repair loop. Test it by feeding the model a prompt that *forces* it to produce bad JSON, and verify your repair loop catches it.

*Deliverable: `exercises/ex02/structured.py` + tests*

### Exercise 1.3 — Context Compaction (code)
Implement a `ContextManager` that:
- Keeps a list of messages
- When total tokens > budget, summarizes the oldest messages into a single "summary" message
- Uses the LLM itself to do the summarization

*Deliverable: `exercises/ex03/context_manager.py` + tests*

## Definition of Done for Module 01

- [ ] Can write a system prompt from scratch for a given agent
- [ ] Understand token budgeting and can implement compaction
- [ ] Have a working harness with retries, validation, and provider abstraction
