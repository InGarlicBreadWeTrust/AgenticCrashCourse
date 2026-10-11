# Contributing

Thanks for improving **Agentic AI — From Zero to Shipping Agents**. This is a
hands-on learning path, and the bar for a good contribution is: *a reader could
follow it, and it matches the repo's philosophy.*

## The philosophy (mirror this in every PR)

- **Build, don't just read.** Every module ends with a working artifact.
- **Start from primitives.** Build the tiny thing first, then reach for
  LangGraph / CrewAI / AutoGen / MCP — don't jump straight to frameworks.
- **Production-minded.** Evals, observability, and guardrails are first-class,
  not an afterthought.
- **Model-agnostic.** Examples use OpenAI-compatible APIs so a reader can swap
  in Qwen, Llama, GPT, Claude, or a local Ollama model without changing logic.

## What makes a good contribution

1. **Bug fixes** — a broken example, a dead link, a command that no longer
   works. Include *what command failed* and *what you expected*.
2. **New module or example** — follow the `NN-topic/README.md` structure:
   concept → code → exercises → definition of done. Add the module to the
   "The Path" diagram in the root `README.md`.
3. **Refresh dated content** — the agent field moves fast. If a claim is wrong
   for the current 2025/2026 state (MCP/A2A, guardrails, evals), update it *and
   cite the source* in the module's References.
4. **Security & guardrails** — these are the most valuable contributions. New
   defenses should map to a specific OWASP risk (LLM0x or an Agentic AI Threat).

## Style

- `requirements.txt` is the tool of record — keep deps pinned and minimal.
- Prefer `litellm` / OpenAI-compatible calls over provider-specific SDKs.
- Code in this repo is plain, readable Python 3.10+; no premature abstraction.
- Keep diagrams as the ASCII art already in use; don't introduce image deps.

## Local dev

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# run lint / type-check
ruff check . && mypy .
# run any tests you add
pytest
```

Every change that touches code should pass `ruff check` and `mypy` locally.

## License

Code and examples are released under [MIT](LICENSE). By contributing you agree
your contributions are released under the same license.

## Thanks

Every merged PR is a working agent in the wild. Ship it.
