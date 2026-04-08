# Z1 Matrix Memory Palace 2.0

**Chinese name: Z1矩阵记忆宫殿 2.0**

A file‑driven memory architecture for multi‑agent systems with built‑in vector search, memory distillation, and strict context‑budget discipline.

## Overview
Z1 Matrix Memory Palace is an open‑source blueprint for building long‑lived, high‑density memory systems for autonomous agent workflows.

Instead of treating memory as a temporary retrieval layer, this project treats memory as a structured operating layer:
- raw outputs live in files
- tasks move through explicit routes
- project results are metabolized into reusable principles
- a continuously compiled wiki‑like layer serves as the single source of truth

**Palace 2.0** introduces three core upgrades:
1. **Watchdog** – a lightweight vector router that answers `query_memory_palace` with top‑k relevant summaries.
2. **Archivist** – a distillation pipeline that turns completed task chains into compact telegraphs and reusable logic.
3. **Context‑Budget Constitution** – a hard limit on historical context injection to prevent token waste and lost‑in‑the‑middle drift.

## Inspirations
This project is explicitly inspired by:
- **Andrej Karpathy's llm‑wiki idea** – knowledge should be continuously compiled into a persistent, structured middle layer.
- **Jeff Pierce's memory‑palace approach** – long‑term memory should live outside the model as a durable, connected system layer.
- **BAAI's bge‑m3 embedding model** – a state‑of‑the‑art multilingual embedding model that enables efficient local semantic search.

## Core Synthesis
Palace 2.0 combines five ideas into one runtime architecture:
- **Memory Palace as the spatial backbone**
- **LLM Wiki as the compilation soul**
- **Watchdog as the vector router**
- **Archivist as the memory distiller**
- **A file‑driven constitution as the operating discipline**

In short:
- the Palace defines where memory lives and how work routes across rooms
- the Wiki defines how artifacts become stable knowledge
- the Watchdog ensures agents find the right summary before digging into raw files
- the Archivist continuously dehydrates long task chains into reusable assets
- the Constitution keeps token consumption under control

## Architecture
The system is organized into four layers:

1. **Raw Layer** – work artifacts, logs, task outputs.
2. **Palace Layer** – rooms, routes, project containers, dispatch records.
3. **Watchdog Layer** – vector index of high‑density summaries, serving `query_memory_palace`.
4. **Reflection Layer** – principles, prompt kernels, failure patterns, thinking paths, constitution candidates.

### Palace Topology
Suggested core spaces:
- Grand Hall (control panels, overviews)
- Agent Chambers (agent‑specific indices)
- Project Rooms (per‑project index, chain maps, backfills)
- Dispatch Corridor (completed task cards)
- Reflection Wing (telegrams, principles, kernels, patterns)
- Archive Basement (cold storage, rarely accessed raw logs)

Each room is not just a folder metaphor. It represents a semantic domain, routing rule, maintenance responsibility, and lifecycle rule.

## Why file‑driven?
This project adopts a file‑driven constitution:
- file‑first over chat‑heavy coordination
- single main execution chain by default
- report after results, not during every intermediate state
- workspace as a state panel
- nothing counts as done until it is written down

## Palace 2.0 Components

### 1. Watchdog – The Vector Router
Watchdog is the gatekeeper of the memory palace. It does not generate text, summarize, or arbitrate. Its only job is to route an agent’s query to the most relevant 1–3 high‑density summary pages.

**Responsibilities:**
- Convert queries and summary pages into embeddings.
- Maintain a lightweight local vector index (e.g., SQLite, ChromaDB, or a simple JSONL‑based ANN).
- Return top‑k candidate paths with optional room/type labels.
- Enforce the context‑budget entrance: total injected history never exceeds 1200 tokens.

**Technology stack (alpha):**
- Embedding model: `BAAI/bge‑m3` (multilingual, 1024‑dimension, Apache‑2.0 licensed).
- Index: local vector store with cosine similarity.
- Index scope: only “hot‑zone” summary pages (Grand Hall control panels, project room indices, reflection backfills, etc.).

**Default query flow:**
```python
# Pseudocode
results = watchdog.query("How did we handle large‑file reading limits?")
# returns [(path, score), ...]
```

### 2. Archivist – The Memory Distiller
Archivist is a background compiler that turns completed task chains into compact telegraphs and, when justified, promotes them to reusable reflection assets.

**Responsibilities:**
- Read completed corridor cards and corresponding project room materials.
- Distill long chains into 150–400 token telegraphs (stored in `reflection_wing/telegrams/`).
- Decide whether a telegraph carries cross‑task value; if yes, promote to principles, prompt kernels, failure patterns, thinking paths, or constitution candidates.
- Remove narrative foam: intermediate chatter, polite acknowledgments, low‑increment trial‑and‑error descriptions.

**Distillation template (keeps four parts):**
1. Background
2. Failed attempts and why they failed
3. Final working solution
4. Generalizable principle

### 3. Context‑Budget Constitution
A hard rule set that prevents historical memory from squeezing out the current task.

**Default budget:**
- System/persona/constitution: 500–800 tokens.
- Current task description: 800–1500 tokens.
- Watchdog‑injected history: ≤1200 tokens.
- Deep‑reading of raw files: by explicit path, not bulk ingestion.

**Front‑line agent reading discipline:**
1. Read Grand Hall control panel.
2. Read relevant project room / chamber index.
3. Read reflection pages.
4. If still insufficient, call `read_drawer_file(exact_path, line_start, line_end)`.

### 4. Query Protocols

#### `query_memory_palace`
- First‑line retrieval when an agent lacks project history, old decisions, or matrix context.
- Returns top‑3 relevant summaries, total ≤500 tokens.
- If snippets are insufficient, the agent may call `read_drawer_file`.

#### `read_drawer_file`
- Precision deep‑read after `query_memory_palace`.
- Accepts exact file path and line range.
- Forbidden as a whole‑repository scanning tool.

#### Large‑file read limit
- Files above a threshold trigger a “File too large, use `query_memory_palace` instead” response.
- Forces agents to develop the “retrieve first, read locally later” habit.

## Local Vector Model: bge‑m3 Deployment

### Model Card
- **Name:** BAAI/bge‑m3
- **Dimensions:** 1024
- **License:** Apache‑2.0
- **Languages:** Multilingual (Chinese‑English optimized, works with 100+ languages)
- **Performance:** State‑of‑the‑art on MTEB and C‑MTEB benchmarks.

### Installation
```bash
# Install sentence‑transformers (or your preferred embedding library)
pip install sentence‑transformers

# Optional: install a local vector store (ChromaDB, FAISS, etc.)
pip install chromadb
```

### Quick Start Script
```python
from sentence_transformers import SentenceTransformer
import numpy as np
import json
import os

# Load the model (first download will cache locally)
model = SentenceTransformer('BAAI/bge-m3')

# Encode summaries during indexing
summaries = ["..."]  # list of summary texts
paths = ["..."]       # corresponding file paths
embeddings = model.encode(summaries, normalize_embeddings=True)

# Store embeddings and paths (simplest example: JSONL)
with open("memory/palace/watchdog_index.jsonl", "w") as f:
    for path, emb in zip(paths, embeddings):
        f.write(json.dumps({"path": path, "embedding": emb.tolist()}) + "\n")

# Query
query = "How do we handle context budget violations?"
query_emb = model.encode([query], normalize_embeddings=True)[0]

# Load index and compute cosine similarity (naive linear scan for small collections)
# Replace with ANN library for larger indices.
best_score = -1
best_path = None
with open("memory/palace/watchdog_index.jsonl", "r") as f:
    for line in f:
        item = json.loads(line)
        score = np.dot(query_emb, np.array(item["embedding"]))
        if score > best_score:
            best_score = score
            best_path = item["path"]

print(f"Top match: {best_path} (score: {best_score:.3f})")
```

### Notes
- The example uses a simple JSONL index for transparency. In production you may want ChromaDB, FAISS, or SQlite‑based vector storage.
- Keep the index lightweight; only index “hot‑zone” summaries, not raw logs.
- Embedding generation can be batched and cached; update the index whenever a new high‑density summary is written.

## Repository Structure
```text
z1‑matrix‑memory‑palace/
├─ README.md
├─ README.zh‑CN.md
├─ LICENSE
├─ NOTICE (attribution for third‑party models)
├─ docs/
│  ├─ architecture/
│  │  ├─ palace‑2.0‑overview.md
│  │  ├─ watchdog‑protocol.md
│  │  └─ archivist‑protocol.md
│  ├─ deployment/
│  │  ├─ local‑vector‑model.md
│  │  └─ quick‑start.md
│  └─ references/
├─ memory/
│  └─ palace/
│     ├─ grand_hall/
│     ├─ chambers/
│     ├─ project_rooms/
│     ├─ dispatch_corridor/
│     ├─ reflection_wing/
│     │  ├─ telegrams/
│     │  ├─ principles/
│     │  ├─ prompt_kernels/
│     │  ├─ failure_patterns/
│     │  └─ thinking_paths/
│     └─ archive_basement/
├─ examples/
├─ templates/
└─ scripts/
```

## MVP
The first open‑source version (2.0‑alpha) focuses on:
- Repository skeleton with Palace 2.0 room conventions.
- Watchdog prototype using `BAAI/bge‑m3` and a local vector store.
- Archivist distillation pipeline (telegraph generation and promotion logic).
- Context‑budget constitution and query protocols.
- Example project pages and corridor cards.

## Non‑goals
This project is not, at least in the MVP stage:
- A production‑ready vector database service.
- A complete hosted memory‑as‑a‑service platform.
- A chat‑log dumping utility.
- A universal agent framework.
- A polished SaaS product.

## The Archivist (Alpha)
In the alpha phase, the Archivist role is temporarily assigned to a dedicated agent (e.g., 04_Code) that runs periodically. It does not lead frontline work; it processes what has already been proven in files.

## Open‑source Notes
This repository intentionally avoids private local filesystem paths, private identities, internal‑only operational traces, and any secret keys. It keeps the structure, protocols, and examples, while stripping environment‑specific details.

## Roadmap
1. Stabilize Palace 2.0 room naming, routing, and reflection protocols.
2. Provide a ready‑to‑run Watchdog script with `BAAI/bge‑m3` and ChromaDB.
3. Add semantic retrieval with typed edges (causal, contradictory, reinforces).
4. Explore service/plugin integrations (e.g., Slack, Discord, Feishu) for automatic memory ingestion.

## License
MIT

## Attribution
The vector‑search component relies on the `BAAI/bge‑m3` embedding model, which is licensed under Apache‑2.0. See `NOTICE` for full attribution details.
