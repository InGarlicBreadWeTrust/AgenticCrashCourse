# Module 03 — Tool Design

> Tools are your agent's hands. Bad tool design is the #1 cause of agent failure.

## The Tool Schema

Every tool needs a **name**, a **description**, and a **parameter schema**. The LLM reads all three to decide whether and how to call the tool.

```json
{
  "name": "search_flights",
  "description": "Search for available flights between two airports on a given date. Returns a list of flights with price, duration, and stops. Use this when the user asks about flight availability or pricing.",
  "parameters": {
    "type": "object",
    "properties": {
      "origin": {
        "type": "string",
        "description": "IATA airport code for the departure airport (e.g., 'SFO', 'JFK')"
      },
      "destination": {
        "type": "string",
        "description": "IATA airport code for the arrival airport (e.g., 'HND', 'NRT')"
      },
      "date": {
        "type": "string",
        "format": "date",
        "description": "Departure date in YYYY-MM-DD format"
      },
      "max_price": {
        "type": "number",
        "description": "Maximum price in USD. Optional. If omitted, no price filter is applied."
      }
    },
    "required": ["origin", "destination", "date"]
  }
}
```

### What Makes a Good Tool Description

**Good:**
- Says *what* the tool does
- Says *when* to use it
- Says *what it returns*
- Explains non-obvious parameters
- Gives examples of valid values

**Bad:**
- `description: "Search flights"` ← too vague, model won't know when to use it
- `description: "A function"` ← useless
- Missing `required` field ← model guesses, breaks things

## Tool Design Principles

### 1. **One tool, one job**
Don't make a `do_everything()` tool. Make `search_flights()`, `book_flight()`, `cancel_flight()`.

### 2. **Name it like a verb**
`get_weather`, `search_docs`, `run_query` — not `weather`, `docs`, `query`.

### 3. **Make the obvious obvious**
If a parameter is `origin`, the description should say "IATA code" not "the first place."

### 4. **Return structured, predictable data**
```python
# Good
return {"flights": [{"price": 450, "duration": "10h 20m", "stops": 1, ...}], "count": 5}

# Bad
return "Found 5 flights. The cheapest is $450 on Delta with 1 stop."
```

The model can *read* structured data reliably. Parsing natural language is where errors happen.

### 5. **Fail loudly, with useful errors**
```python
# Good
raise ToolError("Airport code 'XYZ' is not valid. Valid codes include: SFO, JFK, LAX, ...")

# Bad
raise Exception("error")
```

The model *reads* the error and adapts. A useless error wastes a turn.

### 6. **Idempotent where possible**
`get_weather()` is safe to call twice. `send_email()` is not. Mark non-idempotent tools clearly and make your loop treat them differently.

## Tool Routing — How Does the Model Pick the Right Tool?

The model sees all tool descriptions in its context and picks based on:
1. How well the description matches the user's intent
2. How well the user's message matches the parameter schema
3. Its "gut feeling" (trained on similar patterns)

**Tips to improve routing:**
- Be specific in descriptions ("Use this when the user asks about X" vs. "Does X")
- Avoid overlapping tools (if two tools could both apply, the model will guess)
- Group related tools with a shared prefix (`flights_search`, `flights_book`, `flights_cancel`)
- If you have >10 tools, consider a **tool router** — a separate LLM call that picks the top-3 relevant tools

## MCP (Model Context Protocol)

MCP is an emerging standard for exposing tools to LLMs in a provider-agnostic way. Think of it as "USB for tools."

**Why it matters:** instead of every agent framework re-implementing tool loading, MCP gives you a standard way to:
- Define a tool server (exposes tools)
- Connect it to any MCP-compatible client (Claude Desktop, Cursor, your agent)

You don't need MCP to start, but it's worth knowing it exists.

## Sandboxing

Tools that execute code, hit the network, or touch the filesystem need sandboxing:

| Risk | Mitigation |
|---|---|
| Code execution | Docker / Firecracker / gVisor |
| Network access | Allowlist domains, proxy |
| Filesystem | Mount only what's needed, read-only where possible |
| DB access | Read-only connection for most tools |
| Secrets | Never in tool params, inject at execution time |

**Rule:** a tool should have the *minimum* permissions needed to do its job.

## References

- OpenAI: *"Function Calling"* guide
- Anthropic: *"Tool use"* docs
- MCP spec: https://modelcontextprotocol.io
- Simon Willison: *"Tool use patterns"*

## Exercises

### Exercise 3.1 — Design a Tool Suite (no code)
Pick a domain (e.g., "travel assistant", "code review bot", "research assistant"). Design 5–8 tools for it. For each tool, write:
- Name
- Description (follow the checklist above)
- Parameter schema (JSON)
- Example input → output
- Is it idempotent?

*Deliverable: `exercises/ex01/tool_design.md`*

### Exercise 3.2 — Implement the Tools (code)
Pick 3 of your designed tools and implement them as Python functions with proper error handling. Write a test for each.

*Deliverable: `exercises/ex02/tools.py` + `test_tools.py`*

### Exercise 3.3 — Tool Routing Challenge (code)
Create 10 tools with overlapping descriptions. Feed the agent a task and observe which tool it picks. Then **rewrite the descriptions** to fix mis-routing. Log before/after.

*Deliverable: `exercises/ex03/routing.md` (before/after log)*

## Definition of Done for Module 03

- [ ] Can write a tool schema that an LLM will use correctly >90% of the time
- [ ] Understand idempotency and why it matters for retries
- [ ] Know what MCP is and why it's emerging
- [ ] Can sandbox a tool appropriately
