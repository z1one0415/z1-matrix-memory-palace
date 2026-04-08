# Palace 2.0 Architecture Overview

Palace 2.0 is the second major iteration of the Z1 Matrix Memory Palace. It introduces three new system components—Watchdog, Archivist, and a Context‑Budget Constitution—while preserving the file‑driven, room‑based spatial metaphor of the original design.

## Design Goals
1. **Reduce token waste** – prevent historical memory from overwhelming the current task context.
2. **Enable fast semantic search** – let agents find relevant summaries before digging into raw files.
3. **Compress long task chains** – turn completed work into dense telegraphs and reusable logic.
4. **Stay file‑driven** – keep the system inspectable, version‑controlled, and portable.

## Layer Diagram
```
┌─────────────────────────────────────────────────────────┐
│                    Front‑line Agent                      │
│  (e.g., 01_Core, 02_Intel, 03_Pub, 04_Code, 05_Media)   │
└─────────────────────────────────────────────────────────┘
                                │
                                │ query_memory_palace()
                                ▼
┌─────────────────────────────────────────────────────────┐
│                     Watchdog Layer                       │
│  • Vector index of high‑density summaries               │
│  • Returns top‑k candidate paths (≤500 tokens total)    │
│  • Enforces context‑budget entrance (≤1200 tokens)      │
└─────────────────────────────────────────────────────────┘
                                │
                                │ read_drawer_file()
                                ▼
┌─────────────────────────────────────────────────────────┐
│                     Palace Layer                         │
│  • Grand Hall (control panels, overviews)               │
│  • Agent Chambers (agent‑specific indices)              │
│  • Project Rooms (per‑project index, chain maps)        │
│  • Dispatch Corridor (completed task cards)             │
│  • Reflection Wing (telegrams, principles, kernels)     │
│  • Archive Basement (cold storage)                      │
└─────────────────────────────────────────────────────────┘
                                │
                                │ periodic distillation
                                ▼
┌─────────────────────────────────────────────────────────┐
│                    Archivist Layer                       │
│  • Reads completed corridor cards & project rooms       │
│  • Distills long chains into 150‑400 token telegraphs   │
│  • Promotes telegraphs to reflection assets when justified │
│  • Removes narrative foam (chatter, low‑increment trials)│
└─────────────────────────────────────────────────────────┘
```

## Component Interactions

### Watchdog ↔ Front‑line Agent
- An agent calls `query_memory_palace` when it lacks project history, old decisions, or matrix context.
- Watchdog returns 1–3 summary‑page paths with optional room/type labels.
- If the returned snippets are insufficient, the agent may call `read_drawer_file(exact_path, line_start, line_end)`.
- **Hard rule:** Watchdog never injects more than 1200 tokens of history.

### Archivist ↔ Palace Layer
- Archivist periodically scans the Dispatch Corridor for completed task cards.
- For each completed task, it reads the corresponding project room index, chain map, and reflection backfill.
- It produces a telegraph (stored in `reflection_wing/telegrams/`) that retains only:
  - Background
  - Failed attempts and why they failed
  - Final working solution
  - Generalizable principle
- If the telegraph carries cross‑task value, Archivist promotes it to the appropriate reflection subdirectory (principles, prompt kernels, failure patterns, thinking paths, or constitution candidates).

### Context‑Budget Constitution
- A set of hard limits that apply to every agent interaction.
- **Default budgets:**
  - System/persona/constitution: 500‑800 tokens.
  - Current task description: 800‑1500 tokens.
  - Watchdog‑injected history: ≤1200 tokens.
  - Deep‑reading of raw files: by explicit path only, not bulk ingestion.
- **Reading discipline:**
  1. Grand Hall control panel.
  2. Relevant project room / chamber index.
  3. Reflection pages.
  4. `read_drawer_file` only if the above are insufficient.

## Key Protocols

### `query_memory_palace`
- **Purpose:** First‑line retrieval when context is missing.
- **Returns:** Top‑3 relevant summaries, total ≤500 tokens.
- **Implementation:** Watchdog’s vector index (bge‑m3 embeddings + local vector store).

### `read_drawer_file`
- **Purpose:** Precision deep‑read after `query_memory_palace`.
- **Parameters:** Exact file path, line‑start, line‑end.
- **Rule:** Never used as a whole‑repository scanner.

### Large‑file read limit
- **Trigger:** File size exceeds a configured threshold.
- **Response:** “File too large, use `query_memory_palace` instead.”
- **Goal:** Force agents to retrieve before they read.

## Vector‑Search Backend
- **Embedding model:** `BAAI/bge‑m3` (multilingual, 1024‑dim, Apache‑2.0).
- **Index:** Lightweight local store (ChromaDB, SQLite, or JSONL‑based ANN).
- **Indexed content:** Only “hot‑zone” summaries—Grand Hall control panels, project room indices, reflection backfills, etc.
- **Update policy:** Index is rebuilt or incrementally updated whenever a new high‑density summary is written.

## Distillation Pipeline
1. **Input:** Completed corridor card + linked project room materials.
2. **Compression:** Remove narrative foam (politeness, intermediate chatter, low‑increment trials).
3. **Telegraph generation:** Produce 150‑400 token summary following the four‑part template.
4. **Promotion decision:** Does the telegraph contain reusable logic?
   - **Yes** → move to appropriate reflection subdirectory.
   - **No** → keep only in `telegrams/` as a project‑specific record.
5. **Output:** Updated reflection wing and, optionally, new principle/kernel/pattern entries.

## Room Responsibilities
| Room | Purpose | Maintained By |
|------|---------|---------------|
| **Grand Hall** | System‑wide control panels, active/completed/reflection overviews. | 01_Core |
| **Agent Chambers** | Agent‑specific legacy indices, skill inventories, persona notes. | Each agent |
| **Project Rooms** | Per‑project index, chain maps, reflection backfills, raw outputs. | Project owner |
| **Dispatch Corridor** | Completed task cards (one file per finished task). | 01_Core |
| **Reflection Wing** | Telegrams, principles, prompt kernels, failure patterns, thinking paths, constitution candidates. | Archivist |
| **Archive Basement** | Cold storage for rarely accessed raw logs, historical chat transcripts. | 01_Core |

## Open‑Source Adaptation
The open‑source version of Palace 2.0 strips away all private paths, identities, API keys, and internal operational traces. It provides:
- A clean room structure with example files.
- Watchdog prototype scripts using the freely licensed `BAAI/bge‑m3` model.
- Archivist distillation templates and promotion logic.
- Example project pages and corridor cards that illustrate the workflow.

## Next Steps
1. **Stabilize protocols** – finalize naming, routing, and reflection conventions.
2. **Implement Watchdog** – produce a ready‑to‑run script with bge‑m3 and ChromaDB.
3. **Implement Archivist** – create a standalone Python module for telegraph generation and promotion.
4. **Integrate with existing agents** – adapt 01_Core, 02_Intel, etc. to use `query_memory_palace` and respect the context budget.
