# Z1 Matrix Memory Palace

**中文名：Z1矩阵记忆宫殿**

A file-driven memory architecture for multi-agent systems: use a Memory Palace as the spatial backbone, an LLM Wiki as the compilation layer, and a file-driven constitution as the runtime discipline.

## Overview
Z1 Matrix Memory Palace is an open-source blueprint for building long-lived memory systems for multi-agent workflows.

Instead of treating memory as a temporary retrieval layer, this project treats memory as a structured operating layer:
- raw outputs live in files
- tasks move through explicit routes
- project results are metabolized into reusable principles
- a wiki-like layer is continuously compiled from work artifacts

## Inspirations
This project is explicitly inspired by:
- **Andrej Karpathy's llm-wiki idea** — for the concept that knowledge should be continuously compiled into a persistent, structured middle layer instead of being reconstructed from scratch on every query.
- **Jeff Pierce's memory-palace approach** — for the concept that long-term memory should live outside the model as a durable, connected system layer.

## Core Synthesis
This project combines three ideas into one runtime architecture:
- **Memory Palace as the spatial backbone**
- **LLM Wiki as the compilation soul**
- **A file-driven constitution as the operating discipline**

In short:
- the Palace defines where memory lives and how work routes across rooms
- the Wiki defines how artifacts become stable knowledge
- the Constitution defines how the system runs with low noise, low token waste, and strong physical closure

## Architecture
The system is organized into three layers:
1. **Raw Layer** — work artifacts, logs, task outputs
2. **Palace Layer** — rooms, routes, project containers, dispatch records
3. **Wiki / Reflection Layer** — principles, failure patterns, prompt kernels, thinking paths, constitution candidates

## Palace Topology
Suggested core spaces:
- Grand Hall
- Agent Chambers
- Project Rooms
- Dispatch Corridor
- Reflection Wing
- Archive Basement

Each room is not just a folder metaphor. It represents a semantic domain, routing rule, maintenance responsibility, and lifecycle rule.

## Why file-driven?
This project adopts a file-driven constitution:
- file-first over chat-heavy coordination
- single main execution chain by default
- report after results, not during every intermediate state
- workspace as a state panel
- nothing counts as done until it is written down

## Repository Structure
```text
z1-matrix-memory-palace/
├─ README.md
├─ LICENSE
├─ docs/
│  ├─ architecture/
│  ├─ protocols/
│  └─ references/
├─ memory/
│  └─ palace/
│     ├─ grand_hall/
│     ├─ chambers/
│     ├─ project_rooms/
│     ├─ dispatch_corridor/
│     ├─ reflection_wing/
│     └─ archive_basement/
├─ examples/
├─ templates/
└─ scripts/
```

## MVP
The first open-source version focuses on:
- repository skeleton
- room and page conventions
- markdown templates
- reflection and metabolism rules
- one or two example project pages

## Non-goals
This project is not, at least in the MVP stage:
- a production-ready vector database
- a complete hosted memory service
- a chat-log dumping utility
- a universal agent framework
- a polished SaaS product

## The Archivist
An optional backend role, `Archivist`, compiles completed work into long-term reusable assets:
- principle cards
- prompt kernels
- failure patterns
- thinking paths
- constitution candidates

The Archivist does not lead frontline work. It processes what has already been proven in files.

## Open-source Notes
This repository intentionally avoids private local filesystem paths, private identities, and internal-only operational traces. It keeps the structure, protocols, and examples, while stripping environment-specific details.

## Roadmap
1. Build the file-based Palace structure
2. Stabilize naming, routing, and reflection protocols
3. Add semantic retrieval and typed edges
4. Explore service/plugin integrations later

## License
MIT
