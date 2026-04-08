# Archivist Protocol

Archivist is the background compiler that turns completed task chains into compact telegraphs and, when justified, promotes them to reusable reflection assets.

## Principles
- **Archivist distills, does not delete.** Its goal is to remove narrative foam while preserving the essential logic.
- **Archivist works on completed work only.** It does not participate in frontline task execution.
- **Archivist produces two‑layer outputs:** telegraphs (project‑specific summaries) and reflection assets (cross‑task reusable logic).

## Responsibilities
### What Archivist Does
1. Read completed corridor cards and their linked project room materials.
2. Distill long task chains into 150–400 token telegraphs, stored in `reflection_wing/telegrams/`.
3. Decide whether a telegraph carries cross‑task value; if yes, promote it to the appropriate reflection subdirectory (principles, prompt kernels, failure patterns, thinking paths, or constitution candidates).
4. Remove narrative foam: intermediate chatter, polite acknowledgments, low‑increment trial‑and‑error descriptions.

### What Archivist Does Not Do
- Participate in frontline task execution.
- Use chat transcripts as primary evidence (prefers file artifacts).
- Promote local tricks into global law without repeated proof.
- Automatically delete original raw logs (cold storage is handled separately).

## Input Scope
Archivist reads:
1. **Completed corridor cards** – each card represents a finished task.
2. **Corresponding project room index** – the `index.md` of the project room referenced by the corridor card.
3. **Chain maps and reflection backfills** – if present, these provide the detailed step‑by‑step narrative.
4. **Key raw artifacts** – only when the above are insufficient (e.g., a critical output file that isn’t summarized elsewhere).

**Archivist does not read:**
- Raw chat logs (unless no file artifacts exist).
- Unfinished or abandoned task chains.
- Live, in‑progress work.

## Distillation Pipeline
### Step 1: Gather Materials
- Locate the corridor card (`dispatch_corridor/YYYY‑MM‑DD‑task‑title.md`).
- Extract the project room reference from the card’s metadata.
- Load the project room’s `index.md`, `chain_map.md`, and `reflection_backfill.md` (if they exist).
- If necessary, load a small number of key output files mentioned in the chain map.

### Step 2: Remove Narrative Foam
Delete from the narrative:
- Polite acknowledgments (“Thanks!”, “Got it.”).
- Intermediate status reports (“I’m starting now.”, “I’ve completed part 1.”).
- Repeated trial‑and‑error loops that did not produce new insight.
- Editorializing or motivational asides.
- Redundant descriptions of the same event.

### Step 3: Apply the Four‑Part Template
Every telegraph must retain these four sections:
1. **Background** – what problem was being solved, under what constraints?
2. **Failed attempts and why they failed** – what was tried, what went wrong, what was learned?
3. **Final working solution** – what actually worked, in concrete terms?
4. **Generalizable principle** – what rule, pattern, or heuristic can be extracted from this experience?

### Step 4: Compress to Target Length
- Target telegraph length: 150–400 tokens.
- If the draft exceeds 400 tokens, further remove illustrative examples, minor details, or tangential observations.
- **Compression ratio goal:** 5000‑token raw chain → 200‑500‑token telegraph.

### Step 5: Write Telegraph
- Save to `reflection_wing/telegrams/YYYY‑MM‑DD‑project‑slug.md`.
- Include front‑matter linking back to the original corridor card and project room.

### Step 6: Promotion Decision
Ask: **Does this telegraph contain logic that could be reused across different tasks?**
- **Yes** → promote to the appropriate reflection subdirectory.
- **No** → leave it only in `telegrams/` as a project‑specific record.

#### Promotion Destinations
| Destination | Criteria |
|-------------|----------|
| **principles/** | A general rule that should influence future decision‑making. |
| **prompt_kernels/** | A reusable prompt fragment that reliably produces desired behavior. |
| **failure_patterns/** | A common failure mode and its recovery steps. |
| **thinking_paths/** | A reasoning structure that can be applied to similar problems. |
| **constitution_candidates/** | A proposed amendment to the system’s operating constitution. |

## Output Examples
### Telegraph (before promotion)
```markdown
---
corridor_card: dispatch_corridor/2026‑04‑08‑large‑file‑read‑limit.md
project_room: project_rooms/context_budget_patch
date: 2026‑04‑08
---

**Background**
Agents were routinely reading entire large files into context, causing token overflow and lost‑in‑the‑middle drift.

**Failed attempts**
1. Warning messages alone did not change behavior.
2. Hard blocking of all large files broke legitimate deep‑read needs.
3. Dynamic token‑counting added too much runtime complexity.

**Final working solution**
Introduce a soft limit: when a file exceeds threshold, return “File too large, use `query_memory_palace` instead.” Agents must first retrieve relevant summaries via Watchdog, then use `read_drawer_file` for precise deep‑reading.

**Generalizable principle**
Force retrieval before deep‑reading; make the easy path (retrieval) the default, not the exception.
```

### Promoted Principle
If the above telegraph is deemed cross‑task valuable, it could be promoted to `principles/retrieve‑before‑deep‑read.md`:
```markdown
---
source: reflection_wing/telegrams/2026‑04‑08‑context‑budget‑patch.md
date: 2026‑04‑08
tags: [context‑budget, retrieval, file‑io]
---

**Principle: Retrieve Before Deep‑Read**
When an agent needs historical information, it must first call `query_memory_palace` to obtain relevant summaries. Only if those summaries are insufficient may it call `read_drawer_file` on a specific, known path.

**Rationale**
- Prevents token waste from bulk file ingestion.
- Encourages reliance on high‑density summaries (which are cheaper to index and faster to read).
- Maintains the “hot‑zone” indexing discipline (Watchdog only indexes summaries, not raw logs).

**Implementation**
- Watchdog returns top‑3 summaries (≤500 tokens total).
- `read_drawer_file` accepts exact path and line range.
- Large‑file trigger responds with “File too large, use `query_memory_palace` instead.”
```

## Integration with Watchdog
- Telegraphs are **indexed by Watchdog** (they are high‑density summaries).
- Promoted reflection assets are **also indexed** and become part of the hot‑zone.
- This creates a virtuous cycle: completed work → telegraphs → reflection assets → better retrieval for future queries.

## Context‑Budget Alignment
- Archivist’s compression directly supports the context‑budget constitution: shorter telegraphs mean less history injected into agent contexts.
- Promotion to reflection assets further increases information density, making Watchdog’s top‑k returns more valuable per token.

## Alpha‑Phase Limitations
1. **Manual promotion decisions:** Alpha‑phase Archivist requires human‑like judgment to decide whether to promote a telegraph. Full automation may need a classifier trained on past promotion decisions.
2. **Single‑agent concurrency:** The alpha Archivist is assumed to be a single agent (e.g., 04_Code) running periodically; simultaneous distillation of multiple tasks may need queue management.
3. **Cross‑project relevance detection:** Determining whether a telegraph is cross‑task valuable is heuristic‑based; false positives/negatives are expected.

## Deployment Example
### Periodic Distillation Script (Cron‑like)
```python
import os
from datetime import datetime
from archivist import Archivist

def run_archivist():
    # 1. Find completed corridor cards from the last 24 hours
    corridor_path = "memory/palace/dispatch_corridor"
    completed_cards = []
    for f in os.listdir(corridor_path):
        if f.endswith(".md") and is_completed(f):
            completed_cards.append(os.path.join(corridor_path, f))
    
    # 2. Distill each card
    archivist = Archivist()
    for card in completed_cards:
        telegraph = archivist.distill(card)
        telegraph_path = f"memory/palace/reflection_wing/telegrams/{datetime.now().strftime('%Y‑%m‑d')}‑{os.path.basename(card)}"
        with open(telegraph_path, "w") as fp:
            fp.write(telegraph)
        
        # 3. Decide promotion
        if archivist.should_promote(telegraph):
            promoted_path = archivist.promote(telegraph)
            print(f"Promoted to {promoted_path}")
        
        # 4. Update Watchdog index (incremental add)
        update_watchdog_index(telegraph_path)

if __name__ == "__main__":
    run_archivist()
```

## Roadmap
1. **Stabilize telegraph template** – collect feedback on the four‑part structure.
2. **Automate promotion heuristics** – define measurable criteria for cross‑task value.
3. **Integrate with Watchdog index updates** – ensure new telegraphs are searchable immediately.
4. **Add multi‑format support** – distill not only markdown but also structured data (JSON, YAML) and code snippets.

## One‑Sentence Summary
**Archivist is the dehydrator that turns long task chains into dense telegraphs and, when they carry cross‑task logic, promotes them to reusable reflection assets.**
