# Watchdog Protocol

Watchdog is the vector‑gatekeeper of the Memory Palace. It does not generate text, summarize, or arbitrate. Its only job is to route an agent’s query to the most relevant 1–3 high‑density summary pages.

## Principles
- **Watchdog is a router, not a thinker.** It calculates distances, not judgments.
- **Watchdog is the context‑budget entrance.** It ensures historical memory never overwhelms the current task.
- **Watchdog indexes only hot‑zone summaries.** Raw logs, lengthy chat transcripts, and unfinished work are excluded.

## Responsibilities
### What Watchdog Does
1. Convert queries and summary pages into embeddings.
2. Maintain a lightweight local vector index (e.g., SQLite, ChromaDB, or a JSONL‑based ANN).
3. Return top‑k candidate paths with optional room/type labels.
4. Enforce the context‑budget entrance: total injected history never exceeds 1200 tokens.

### What Watchdog Does Not Do
- Generate natural‑language answers.
- Compress logs.
- Extract principles, prompt kernels, failure patterns, or thinking paths.
- Make constitution‑upgrade judgments.
- Overwrite or edit original memories.

## Technology Stack (Alpha)
| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Embedding model** | `BAAI/bge‑m3` | State‑of‑the‑art multilingual embeddings; Apache‑2.0 license allows unrestricted open‑source use. |
| **Vector store** | Local ChromaDB / SQLite / JSONL | Lightweight, file‑based, no external service required. |
| **Similarity metric** | Cosine similarity | Standard for normalized embeddings. |
| **Index update** | On‑write incremental update | Whenever a new high‑density summary is written, its embedding is added to the index. |

## Index Scope
**Indexed (hot‑zone summaries):**
- `grand_hall/control_panel_v1.md`
- `grand_hall/active_overview.md`
- `grand_hall/completed_overview.md`
- `grand_hall/reflection_overview.md`
- `project_rooms/*/index.md`
- `project_rooms/*/reflection_backfill.md`
- `chambers/*/legacy_index.md`
- High‑value pages in the Reflection Wing (principles, kernels, patterns, thinking paths).

**Excluded (alpha phase):**
- Raw task logs.
- Full‑length chat histories.
- Uncompressed work files.
- Archive Basement content (cold storage).

## Query Flow
### Front‑line Agent Perspective
1. Agent encounters missing context (project history, old decisions, matrix state).
2. Agent calls `query_memory_palace("query text")`.
3. Watchdog returns a list of 1–3 candidate paths, each with a relevance score and optional room/type label.
4. Agent reads the returned summaries (total ≤500 tokens).
5. If summaries are insufficient, agent calls `read_drawer_file(exact_path, line_start, line_end)` for precise deep‑reading.

### Watchdog Internal Steps
```python
def query_memory_palace(query_text):
    # 1. Embed the query
    query_embedding = embedder.encode(query_text, normalize=True)
    
    # 2. Search the index (cosine similarity, top‑k)
    candidates = vector_index.search(query_embedding, k=3)
    
    # 3. Format response
    response = []
    for path, score in candidates:
        # Load summary snippet (first ~200 chars)
        snippet = load_snippet(path, max_chars=200)
        response.append({
            "path": path,
            "score": float(score),
            "snippet": snippet,
            "room": infer_room_from_path(path)
        })
    
    # 4. Enforce token budget: total snippets ≤500 tokens
    trimmed = trim_to_token_limit(response, limit=500)
    return trimmed
```

## Output Rules
Watchdog’s response must be concise:
- **Paths:** 1–3 candidate summary‑page paths.
- **Snippets:** First ~200 characters of each summary (enough for the agent to decide whether to read the whole file).
- **Labels:** Optional room/type tags (e.g., `grand_hall`, `project_room`, `principle`).
- **No explanations:** Watchdog does not inject explanatory text, just the references.

## Context‑Budget Enforcement
- Watchdog is the primary gatekeeper for historical memory injection.
- **Hard limit:** The total token count of all returned snippets must not exceed 500 tokens (configurable).
- **Overall budget:** When combined with other context sources (system prompt, task description), the total historical memory injected into an agent’s context must stay below 1200 tokens.

## Integration with `read_drawer_file`
- `read_drawer_file` is the precision deep‑read tool that follows Watchdog’s retrieval.
- Watchdog’s returned paths are the intended inputs for `read_drawer_file`.
- **Rule:** Agents should never call `read_drawer_file` on a file that hasn’t first been vetted by `query_memory_palace` (except for explicitly named, well‑known control files).

## Deployment Example
### Installation
```bash
pip install sentence‑transformers chromadb
```

### Index Creation Script
```python
import chromadb
from sentence_transformers import SentenceTransformer
import os

# Initialize embedding model
model = SentenceTransformer('BAAI/bge-m3')

# Create ChromaDB collection
client = chromadb.PersistentClient(path="./memory/palace/watchdog_index")
collection = client.create_collection(name="summary_index")

# Walk the palace and index hot‑zone summaries
for root, dirs, files in os.walk("memory/palace"):
    for f in files:
        if f.endswith(".md") and is_hot_zone_summary(os.path.join(root, f)):
            with open(os.path.join(root, f), "r", encoding="utf-8") as fp:
                text = fp.read()
            # Use first 1000 chars as indexable content
            content = text[:1000]
            embedding = model.encode(content, normalize_embeddings=True)
            collection.add(
                embeddings=[embedding.tolist()],
                documents=[content],
                metadatas=[{"path": os.path.join(root, f)}],
                ids=[os.path.join(root, f)]
            )
```

### Query Script
```python
import chromadb
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-m3')
client = chromadb.PersistentClient(path="./memory/palace/watchdog_index")
collection = client.get_collection("summary_index")

query = "How do we handle large‑file read limits?"
query_embedding = model.encode(query, normalize_embeddings=True)

results = collection.query(
    query_embeddings=[query_embedding.tolist()],
    n_results=3
)

for i, (path, content, score) in enumerate(zip(results['metadatas'][0], results['documents'][0], results['distances'][0])):
    print(f"{i+1}. {path['path']} (score: {1-score:.3f})")
    print(f"   {content[:200]}...")
```

## Alpha‑Phase Limitations
1. **Index size:** Only hot‑zone summaries are indexed; scaling to thousands of files will require a more efficient ANN library (e.g., FAISS).
2. **Update latency:** Index updates are synchronous; a production system may need a background update queue.
3. **Embedding cost:** bge‑m3 is free to run locally, but encoding large batches can be CPU/GPU intensive.
4. **Multi‑modal:** Text‑only; images, audio, and other media are not supported.

## Roadmap
1. **Stable hot‑zone definitions** – finalize which file patterns are indexed.
2. **Incremental indexing** – watch for new summary files and update the index automatically.
3. **Typed edges** – enrich the index with relation types (causal, contradictory, reinforces).
4. **Cross‑language search** – leverage bge‑m3’s multilingual capability to query in Chinese and retrieve English summaries (and vice‑versa).

## One‑Sentence Summary
**Watchdog is the router that lets front‑line agents find the right summary before they drown in raw files.**
