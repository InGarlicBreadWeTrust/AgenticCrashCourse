# Module 04 — Memory Architecture

> Memory is what separates a stateless API call from an *agent*. Without memory, every conversation starts from zero.

## The Memory Taxonomy

Borrowed from cognitive science, adapted for agents:

```
┌─────────────────────────────────────────────────────────┐
│                    AGENT MEMORY                          │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  WORKING MEMORY (in-context)                    │    │
│  │  • Current conversation                         │    │
│  │  • Active task state                            │    │
│  │  • Recent observations                          │    │
│  │  • LIFESPAN: one interaction                    │    │
│  │  • BACKEND: context window                      │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  EPISODIC MEMORY (conversation history)         │    │
│  │  • Past conversations                           │    │
│  │  • What was decided, what was rejected          │    │
│  │  • User preferences expressed                   │    │
│  │  • LIFESPAN: days to months                     │    │
│  │  • BACKEND: DB (Postgres, SQLite)               │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  SEMANTIC MEMORY (knowledge base)               │    │
│  │  • Facts, documents, reference material         │    │
│  │  • User profile (name, role, preferences)       │    │
│  │  • Domain knowledge                             │    │
│  │  • LIFESPAN: indefinite                         │    │
│  │  • BACKEND: vector DB + structured DB           │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  PROCEDURAL MEMORY (skills / learned behaviors) │    │
│  │  • "User prefers concise answers"               │    │
│  │  • "Always use British English"                 │    │
│  │  • Learned tool-usage patterns                  │    │
│  │  • LIFESPAN: indefinite                         │    │
│  │  • BACKEND: config / fine-tuning / RAG          │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

## Working Memory (In-Context)

This is what the model *sees* right now. It's your context window.

**Key techniques:**
- **Sliding window** — keep last N messages, drop older
- **Compaction** — summarize older messages into a single summary
- **Selective inclusion** — only include messages relevant to the current task
- **Scratchpad** — a dedicated section in context for the model to "think" in

```python
class WorkingMemory:
    def __init__(self, max_tokens=8000):
        self.messages = []
        self.max_tokens = max_tokens

    def add(self, role, content):
        self.messages.append({"role": role, "content": content})
        self._compaction_check()

    def _compaction_check(self):
        if count_tokens(self.messages) > self.max_tokens * 0.8:
            self._compact()

    def _compact(self):
        # Summarize oldest 50% of messages into one summary
        old = self.messages[:len(self.messages)//2]
        summary = llm_summarize(old)
        self.messages = [{"role": "system", "content": f"Earlier context: {summary}"}] + self.messages[len(self.messages)//2:]
```

## Episodic Memory (Conversation History)

Store past interactions so the agent can reference them later.

**Schema (simplified):**
```sql
CREATE TABLE conversations (
    id UUID PRIMARY KEY,
    user_id UUID,
    started_at TIMESTAMPTZ,
    ended_at TIMESTAMPTZ,
    summary TEXT
);

CREATE TABLE messages (
    id UUID PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id),
    role TEXT,  -- 'user' | 'assistant' | 'tool'
    content TEXT,
    created_at TIMESTAMPTZ
);

CREATE TABLE observations (
    id UUID PRIMARY KEY,
    message_id UUID REFERENCES messages(id),
    tool_name TEXT,
    input JSONB,
    output JSONB,
    duration_ms INT
);
```

**Retrieval pattern:**
```
User: "Can you remind me what we decided about the API design?"
Agent: → search episodic memory for "API design"
      → find: "In conversation 2024-11-15, we decided to use REST for v1, gRPC for v2"
      → respond with that context
```

## Semantic Memory (RAG)

Retrieval-Augmented Generation: pull relevant documents from a knowledge base and inject them into context.

**Pipeline:**
```
Document → Chunk → Embed → Store in Vector DB
                                    ↓
Query → Embed → Similarity Search → Top-K chunks → Inject into context
```

**Key decisions:**
| Decision | Options | Tradeoff |
|---|---|---|
| Chunk size | 100–2000 tokens | Small = precise, Large = more context |
| Embedding model | sentence-transformers, OpenAI, Cohere | Quality vs. cost |
| Vector store | Chroma (local), Qdrant, pgvector, Pinecone | Simplicity vs. scale |
| Retrieval | Cosine, BM25, hybrid | Semantic vs. keyword |
| Re-ranking | Cohere Rerank, cross-encoder | Better precision, higher latency |

**Anti-pattern:** stuffing the entire document into context. Retrieve, don't dump.

## Procedural Memory (Learned Behaviors)

This is the "how does this agent behave" layer. Options:

1. **System prompt** — static, you write it
2. **User profile** — dynamic, learned from interactions ("prefers concise answers")
3. **Fine-tuning** — expensive, but bakes behavior into weights
4. **RAG over past successful trajectories** — "last time a similar task was done, here's what worked"

## Memory Architecture Patterns

### Pattern 1: Stateless (no memory)
Every request is independent. Simplest, least capable.

### Pattern 2: Conversation-only
Keep the current conversation in context. Lose everything between sessions.

### Pattern 3: Conversation + RAG
Keep current conversation + retrieve relevant documents. Most common in production.

### Pattern 4: Full stack
Working + Episodic + Semantic + Procedural. Most capable, most complex.

**Start with Pattern 3. Add Episodic when users start saying "remember when..."**

## References

- Lewis et al. — *"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"* (2020)
- "Memory Bank" papers (2023-2024)
- LangChain memory docs
- LlamaIndex RAG docs

## Exercises

### Exercise 4.1 — Working Memory Manager (code)
Build a `WorkingMemory` class with:
- Token counting
- Auto-compaction when >80% of budget
- `get_context()` method that returns the messages to send to the LLM

*Deliverable: `exercises/ex01/working_memory.py` + tests*

### Exercise 4.2 — Episodic Memory with Postgres (code)
Set up a Postgres (or SQLite for simplicity) schema for conversations + messages. Implement:
- `store_conversation(user_id, messages)`
- `retrieve_recent(user_id, n=5)`
- `search_conversations(user_id, query, k=3)` (full-text search)

*Deliverable: `exercises/ex02/episodic.py` + tests*

### Exercise 4.3 — RAG Pipeline (code)
Build a RAG pipeline:
1. Chunk a sample document (use your `README.md` or any text)
2. Embed chunks (use a free embedding model)
3. Store in Chroma
4. Implement `query(text, k=5)` that returns relevant chunks
5. Wire it into your agent: before responding, retrieve relevant context

*Deliverable: `exercises/ex03/rag.py` + demo script*

## Definition of Done for Module 04

- [ ] Can explain the four memory types and when to use each
- [ ] Have a working working-memory manager with compaction
- [ ] Have a working RAG pipeline (even if simple)
- [ ] Understand the tradeoffs of each memory backend
