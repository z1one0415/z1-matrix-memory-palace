# Quick Start

This guide walks you through setting up a minimal Palace 2.0 system with Watchdog vector search and Archivist distillation.

## Prerequisites
- Python 3.9+
- Git
- Basic familiarity with command line and Python virtual environments

## Step 1: Clone the Repository
```bash
git clone https://github.com/your‑org/z1‑matrix‑memory‑palace.git
cd z1‑matrix‑memory‑palace
```

## Step 2: Set Up Python Environment
```bash
# Create virtual environment
python -m venv venv

# Activate it
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install dependencies
pip install sentence‑transformers chromadb
```

## Step 3: Create the Palace Directory Structure
```bash
# Create the core palace folders
mkdir -p memory/palace/{grand_hall,chambers,project_rooms,dispatch_corridor,reflection_wing,archive_basement}
mkdir -p memory/palace/reflection_wing/{telegrams,principles,prompt_kernels,failure_patterns,thinking_paths}

# Add example control panel
cat > memory/palace/grand_hall/control_panel_v1.md << 'EOF'
# Control Panel v1
- **Active projects:** 0
- **Completed today:** 0
- **Reflection backlog:** 0

## System Status
- Watchdog: not indexed
- Archivist: idle
- Context budget: enabled
EOF

# Add an example project room
mkdir -p memory/palace/project_rooms/example_project
cat > memory/palace/project_rooms/example_project/index.md << 'EOF'
# Example Project
- **Goal:** Demonstrate Palace 2.0 workflow.
- **Status:** completed
- **Tags:** demo, quick‑start

## Chain Map
1. Created control panel.
2. Created example project room.
3. Will add a corridor card after finishing quick‑start guide.

## Key Outputs
- `control_panel_v1.md`
- `example_project/index.md`
EOF
```

## Step 4: Index Hot‑Zone Summaries with Watchdog
Create `scripts/watchdog_index.py`:
```python
import chromadb
from sentence_transformers import SentenceTransformer
import os

# Initialize model
model = SentenceTransformer('BAAI/bge-m3')

# Create ChromaDB collection
client = chromadb.PersistentClient(path="./memory/palace/watchdog_index")
collection = client.get_or_create_collection(name="summary_index")

# Walk palace and index hot‑zone summaries
hot_zone_patterns = [
    "grand_hall/*.md",
    "project_rooms/*/index.md",
    "reflection_wing/telegrams/*.md",
    "reflection_wing/principles/*.md"
]

summary_paths = []
summary_texts = []

for root, dirs, files in os.walk("memory/palace"):
    for f in files:
        if not f.endswith(".md"):
            continue
        full_path = os.path.join(root, f)
        # Simple pattern matching (for demo, expand with glob for production)
        if "grand_hall" in full_path or "index.md" in full_path:
            with open(full_path, "r", encoding="utf-8") as fp:
                text = fp.read()
            summary_paths.append(full_path)
            summary_texts.append(text[:1000])

# Batch encode
if summary_texts:
    embeddings = model.encode(summary_texts, normalize_embeddings=True)
    # Add to collection
    collection.add(
        embeddings=embeddings.tolist(),
        documents=summary_texts,
        metadatas=[{"path": p} for p in summary_paths],
        ids=summary_paths
    )
    print(f"Indexed {len(summary_paths)} summaries.")
else:
    print("No hot‑zone summaries found.")
```

Run the indexer:
```bash
python scripts/watchdog_index.py
```

## Step 5: Query the Palace
Create `scripts/query_demo.py`:
```python
import chromadb
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-m3')
client = chromadb.PersistentClient(path="./memory/palace/watchdog_index")
collection = client.get_collection("summary_index")

def query_memory_palace(query_text, k=3):
    query_emb = model.encode([query_text], normalize_embeddings=True)[0]
    results = collection.query(
        query_embeddings=[query_emb.tolist()],
        n_results=k,
        include=["metadatas", "documents"]
    )
    for i, (meta, doc) in enumerate(zip(results["metadatas"][0], results["documents"][0])):
        print(f"{i+1}. {meta['path']}")
        print(f"   {doc[:200]}...")
        print()

# Example query
query_memory_palace("What is the system status?")
```

Run the query:
```bash
python scripts/query_demo.py
```

Expected output (something like):
```
1. memory/palace/grand_hall/control_panel_v1.md
   # Control Panel v1
- **Active projects:** 0
- **Completed today:** 0
- **Reflection backlog:** 0

## System Status
- Watchdog: not indexed
- Archivist: idle
- Context budget: enabled...
```

## Step 6: Simulate a Completed Task (Corridor Card)
Create a corridor card to represent a finished task:
```bash
cat > memory/palace/dispatch_corridor/2026‑04‑08‑quick‑start‑demo.md << 'EOF'
---
project: example_project
status: completed
date: 2026‑04‑08
---

# Quick Start Demo
- **Goal:** Set up Palace 2.0 with Watchdog indexing.
- **Outcome:** Created control panel, example project room, and indexed 2 summaries.
- **Token cost:** ~1200 tokens for this guide.

## Steps Taken
1. Cloned repository.
2. Installed dependencies.
3. Created palace directory structure.
4. Indexed hot‑zone summaries with bge‑m3.
5. Ran a sample query.

## Reflection
- The indexing script works but needs better hot‑zone pattern matching.
- Query results are returned with cosine similarity scores.
- Next step: integrate with an actual agent front‑end.
EOF
```

## Step 7: Distill with Archivist (Alpha)
Create a simple distillation script `scripts/archivist_demo.py`:
```python
import os
from datetime import datetime

def distill(card_path):
    with open(card_path, "r") as f:
        content = f.read()
    
    # Extremely naive distillation: keep only lines that start with #, -, **
    lines = content.split("\n")
    kept = [l for l in lines if l.startswith("#") or l.startswith("-") or l.startswith("**")]
    telegraph = "\n".join(kept[:20])  # limit to 20 lines
    
    # Save telegraph
    os.makedirs("memory/palace/reflection_wing/telegrams", exist_ok=True)
    out_path = f"memory/palace/reflection_wing/telegrams/{datetime.now().strftime('%Y‑%m‑d')}‑{os.path.basename(card_path)}"
    with open(out_path, "w") as f:
        f.write(telegraph)
    
    print(f"Telegraph saved to {out_path}")
    return out_path

# Distill the corridor card we just created
card = "memory/palace/dispatch_corridor/2026‑04‑08‑quick‑start‑demo.md"
if os.path.exists(card):
    telegraph_path = distill(card)
else:
    print("Corridor card not found, skipping distillation.")
```

Run it:
```bash
python scripts/archivist_demo.py
```

## Step 8: Update Watchdog Index with New Telegraph
Now that a new telegraph exists, we need to add it to the vector index. Modify `watchdog_index.py` to support incremental updates, or simply re‑run the full indexing script:
```bash
python scripts/watchdog_index.py
```

Then query again to see the telegraph appear:
```bash
python scripts/query_demo.py "quick start demo"
```

## Step 9: Implement Context‑Budget Discipline
In your agent code, enforce the following reading order:
1. Call `query_memory_palace` when context is missing.
2. Read the returned summaries (total ≤500 tokens).
3. If still insufficient, call `read_drawer_file` on a specific path.

Example pseudo‑code:
```python
def agent_workflow(task_description):
    # Step 1: Check if we need historical context
    if needs_context(task_description):
        candidates = query_memory_palace(task_description, k=3)
        for cand in candidates:
            # Inject snippet (first 200 chars) into context
            add_to_context(cand["snippet"])
    
    # Step 2: If candidates are not enough, deep‑read the top hit
    if still_unsatisfied():
        content = read_drawer_file(candidates[0]["path"], line_start=1, line_end=100)
        add_to_context(content)
    
    # Step 3: Proceed with task execution
    return execute_task(task_description)
```

## Step 10: Next Steps
Congratulations! You now have a working Palace 2.0 skeleton. To move beyond the demo:

1. **Replace naive scripts** with the full Watchdog and Archivist implementations from the `docs/architecture/` directory.
2. **Integrate with your agents** – modify your agent code to call `query_memory_palace` and respect the context budget.
3. **Expand hot‑zone patterns** – index more summary types (principles, kernels, patterns, thinking paths).
4. **Set up periodic distillation** – run Archivist on a schedule (cron, systemd timer, or background worker).
5. **Add monitoring** – track token usage, retrieval accuracy, and distillation compression ratios.

## Troubleshooting
### “No module named 'chromadb'”
Ensure you installed ChromaDB: `pip install chromadb`

### “Collection not found”
Run the indexing script (`watchdog_index.py`) before querying.

### Query returns unrelated results
- Check that your summaries are being indexed (look at `summary_paths` printed by the indexer).
- Ensure the query embedding is normalized (the script uses `normalize_embeddings=True`).
- Try re‑indexing after adding more content.

### Slow query performance
- For small collections (<1000 items), linear scan is fine. For larger collections, consider switching to FAISS or HNSW‑enabled ChromaDB.
- Use GPU for embedding generation if available.

## Cleaning Up
To start fresh:
```bash
# Remove generated files
rm -rf memory/palace/watchdog_index
rm -rf memory/palace/reflection_wing/telegrams/*.md
# Keep the structure and example files
```

## Need Help?
- Open an issue on GitHub for bugs or questions.
- Check the `docs/architecture/` directory for detailed protocol specifications.
- Refer to the `README.md` for high‑level overview and design philosophy.
