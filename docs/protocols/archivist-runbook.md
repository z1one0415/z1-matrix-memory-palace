# Archivist Runbook (Palace 2.0)

The Archivist is the distillation pipeline that turns completed task chains into compact telegraphs and, when justified, promotes them to reusable reflection assets.

## Responsibilities (Palace 2.0)
1. **Read completed corridor cards** – each card represents a finished task.
2. **Inspect linked project rooms** – load the project room's index, chain map, and reflection backfill.
3. **Distill long chains into telegraphs** – produce 150–400 token summaries following the four‑part template:
   - Background
   - Failed attempts and why they failed
   - Final working solution
   - Generalizable principle
4. **Store telegraphs** – save to `reflection_wing/telegrams/` with front‑matter linking to the original corridor card.
5. **Promote cross‑task logic** – if a telegraph carries reusable value, move it to the appropriate reflection subdirectory:
   - `principles/` – general rules for decision‑making.
   - `prompt_kernels/` – reusable prompt fragments.
   - `failure_patterns/` – common failure modes and recoveries.
   - `thinking_paths/` – reasoning structures.
   - `constitution_candidates/` – proposed system amendments.
6. **Remove narrative foam** – delete intermediate chatter, polite acknowledgments, and low‑increment trial‑and‑error descriptions.

## Restrictions
- **Do not participate in frontline task execution** – Archivist works only on completed work.
- **Do not use chat transcripts as primary evidence** – prefer file artifacts (logs, outputs, written summaries).
- **Do not promote local tricks into global law without repeated proof** – require multiple successful applications before elevating a heuristic to a principle.
- **Do not delete original raw logs** – cold‑storage decisions are handled separately by the Archive Basement.

## Integration with Watchdog
- Telegraphs and promoted reflection assets are **indexed by Watchdog** as hot‑zone summaries.
- This creates a virtuous cycle: completed work → telegraphs → reflection assets → better retrieval for future queries.

## Context‑Budget Alignment
Archivist's compression directly supports the context‑budget constitution: shorter telegraphs mean less history injected into agent contexts, keeping token consumption under control.
