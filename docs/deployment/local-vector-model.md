# Local Vector Model Deployment

Palace 2.0 uses `BAAI/bge‑m3` as its default embedding model for the Watchdog vector router. This document explains how to set up the model, create and update the vector index, and integrate it with the Watchdog service.

## Model Card
| Property | Value |
|----------|-------|
| **Name** | BAAI/bge‑m3 |
| **Dimensions** | 1024 |
| **License** | Apache‑2.0 |
| **Languages** | Multilingual (optimized for Chinese‑English, supports 100+ languages) |
| **MTEB Rank** | State‑of‑the‑art on both MTEB and C‑MTEB benchmarks |
| **Hugging Face** | [https://huggingface.co/BAAI/bge‑m3](https://huggingface.co/BAAI/bge‑m3) |

## Why bge‑m3?
- **Free & open‑source** – Apache‑2.0 license allows unrestricted use in open‑source projects.
- **Multilingual** – works seamlessly across English, Chinese, and many other languages, making it suitable for bilingual or international teams.
- **High performance** – ranks at the top of mainstream embedding benchmarks.
- **Easy local deployment** – runs on CPU (slower) or GPU (faster) with standard Python libraries.

## Installation
### Option 1: Using `sentence‑transformers` (Recommended)
```bash
pip install sentence‑transformers
```

### Option 2: Using `transformers` + `torch`
```bash
pip install transformers torch
```

### Optional: Vector Store Backend
Choose one of the following lightweight vector databases:
```bash
# ChromaDB (simple, persistent)
pip install chromadb

# FAISS (fast, memory‑efficient)
pip install faiss‑cpu  # or faiss‑gpu if you have CUDA

# SQLite‑based (via sqlite‑vec)
pip install sqlite‑vec
```

## Quick Model Test
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-m3')
embeddings = model.encode(
    ["Hello world", "你好世界"],
    normalize_embeddings=True,
    show_progress_bar=False
)
print(embeddings.shape)  # (2, 1024)
```

## Index Creation Workflow
### Step 1: Collect Hot‑Zone Summaries
Watchdog only indexes high‑density summary pages. The following file patterns are considered hot‑zone:
- `grand_hall/*.md` (control panels, overviews)
- `project_rooms/*/index.md`
- `project_rooms/*/reflection_backfill.md`
- `chambers/*/legacy_index.md`
- `reflection_wing/telegrams/*.md`
- `reflection_wing/principles/*.md`
- `reflection_wing/prompt_kernels/*.md`
- `reflection_wing/failure_patterns/*.md`
- `reflection_wing/thinking_paths/*.md`

### Step 2: Generate Embeddings
```python
import os
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-m3')

summary_paths = []
summary_texts = []

for root, dirs, files in os.walk("memory/palace"):
    for f in files:
        if not f.endswith(".md"):
            continue
        full_path = os.path.join(root, f)
        if not is_hot_zone_summary(full_path):  # implement your filter
            continue
        with open(full_path, "r", encoding="utf-8") as fp:
            text = fp.read()
        # Use first 1000 characters as indexable content
        summary_paths.append(full_path)
        summary_texts.append(text[:1000])

# Batch encode (much faster than one‑by‑one)
embeddings = model.encode(summary_texts, normalize_embeddings=True)
```

### Step 3: Store in Vector Database
#### ChromaDB Example
```python
import chromadb
from chromadb.config import Settings

client = chromadb.PersistentClient(
    path="./memory/palace/watchdog_index",
    settings=Settings(anonymized_telemetry=False)
)

collection = client.get_or_create_collection(
    name="summary_index",
    metadata={"hnsw:space": "cosine"}  # cosine similarity
)

# Add embeddings in batches of 100
batch_size = 100
for i in range(0, len(summary_paths), batch_size):
    batch_paths = summary_paths[i:i+batch_size]
    batch_texts = summary_texts[i:i+batch_size]
    batch_embeddings = embeddings[i:i+batch_size]
    
    collection.add(
        embeddings=batch_embeddings.tolist(),
        documents=batch_texts,
        metadatas=[{"path": p} for p in batch_paths],
        ids=batch_paths
    )
```

#### SQLite‑Vec Example (Lightweight Alternative)
```python
import sqlite3
import numpy as np

conn = sqlite3.connect("./memory/palace/watchdog_index.db")
conn.enable_load_extension(True)
conn.load_extension("vec")  # requires sqlite‑vec installed

# Create table with vector column
conn.execute("""
CREATE VIRTUAL TABLE IF NOT EXISTS summary_index USING vec0(
    path TEXT PRIMARY KEY,
    text TEXT,
    embedding FLOAT[1024]
)
""")

# Insert rows
for path, text, emb in zip(summary_paths, summary_texts, embeddings):
    conn.execute(
        "INSERT OR REPLACE INTO summary_index (path, text, embedding) VALUES (?, ?, ?)",
        (path, text, emb.tolist())
    )
conn.commit()
```

## Querying the Index
### Basic Query Function
```python
def query_memory_palace(query_text, k=3):
    # 1. Embed query
    query_emb = model.encode([query_text], normalize_embeddings=True)[0]
    
    # 2. Search (ChromaDB)
    results = collection.query(
        query_embeddings=[query_emb.tolist()],
        n_results=k,
        include=["metadatas", "documents", "distances"]
    )
    
    # 3. Format response
    candidates = []
    for meta, doc, dist in zip(results["metadatas"][0], results["documents"][0], results["distances"][0]):
        candidates.append({
            "path": meta["path"],
            "snippet": doc[:200],  # first 200 chars
            "score": 1 - dist,     # cosine distance → similarity
            "room": infer_room(meta["path"])
        })
    
    # 4. Enforce token budget (total snippets ≤500 tokens)
    return trim_to_token_budget(candidates, limit=500)
```

### Using with `read_drawer_file`
The returned paths are intended to be used with the precision deep‑read tool:
```python
candidates = query_memory_palace("How do we handle context‑budget violations?")
if candidates:
    # Read the top candidate in full (or a line range)
    content = read_drawer_file(candidates[0]["path"], line_start=1, line_end=50)
```

## Incremental Updates
Whenever a new hot‑zone summary is written (e.g., a new telegraph is created by Archivist), update the index:

```python
def update_index(new_summary_path):
    with open(new_summary_path, "r") as f:
        text = f.read()[:1000]
    emb = model.encode([text], normalize_embeddings=True)[0]
    
    # ChromaDB upsert
    collection.upsert(
        embeddings=[emb.tolist()],
        documents=[text],
        metadatas=[{"path": new_summary_path}],
        ids=[new_summary_path]
    )
```

## Performance Considerations
- **Batch encoding:** Always encode texts in batches (e.g., 32–128 at a time) for speed.
- **GPU acceleration:** If you have a CUDA‑capable GPU, install `sentence‑transformers` with `pip install sentence‑transformers[gpu]` and set `model = SentenceTransformer('BAAI/bge‑m3', device='cuda')`.
- **Index size:** With thousands of summaries, consider switching to FAISS or a dedicated vector database for faster approximate nearest neighbor search.
- **Memory footprint:** bge‑m3 requires ~400 MB of RAM when loaded; the vector index size depends on the number of summaries and the store used.

## Troubleshooting
### “OSError: Unable to load model”
- Ensure you have an internet connection for the first download (the model will be cached locally).
- Alternatively, download the model manually from Hugging Face and point to the local folder:
  ```python
  model = SentenceTransformer('/path/to/local/bge‑m3')
  ```

### Slow encoding on CPU
- Reduce batch size (e.g., 16 instead of 128).
- Consider using a smaller model for development (e.g., `BAAI/bge‑small‑en‑v1.5`).

### ChromaDB “collection not found”
- Use `client.get_or_create_collection` instead of `get_collection`.
- Ensure the persistence path is consistent across runs.

## Example Configuration File
Create `config/watchdog.yaml`:
```yaml
embedding:
  model: BAAI/bge‑m3
  device: cpu  # or cuda
  normalize: true

index:
  store: chromadb  # chromadb, faiss, sqlite
  path: memory/palace/watchdog_index
  metric: cosine

hot_zone_patterns:
  - grand_hall/*.md
  - project_rooms/*/index.md
  - project_rooms/*/reflection_backfill.md
  - chambers/*/legacy_index.md
  - reflection_wing/telegrams/*.md
  - reflection_wing/principles/*.md
  - reflection_wing/prompt_kernels/*.md
  - reflection_wing/failure_patterns/*.md
  - reflection_wing/thinking_paths/*.md

query:
  top_k: 3
  max_token_per_response: 500
```

## Attribution
The `BAAI/bge‑m3` model is developed by the Beijing Academy of Artificial Intelligence (BAAI) and is licensed under Apache‑2.0. Please include the following notice in your project’s `NOTICE` file:
```
This product includes software components developed by BAAI.
Embedding model: BAAI/bge‑m3 (Apache‑2.0)
https://huggingface.co/BAAI/bge‑m3
```

## Next Steps
1. **Integrate with Watchdog script** – wrap the above functions into a callable `query_memory_palace` endpoint.
2. **Add incremental indexing** – watch the file system for new hot‑zone summaries and update the index automatically.
3. **Benchmark retrieval speed** – measure latency for queries over growing index sizes.
4. **Explore hybrid search** – combine vector similarity with keyword matching (BM25) for improved recall.
