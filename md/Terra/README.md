# Terra (Markdown Knowledge Base)

This folder is the **working, version-controlled documentation** for the Terra program.

## Project intent

Terra is a structured documentation set covering:
- Strategy and vision for the Terra initiative
- Governance, standards, and participation frameworks
- Product requirements and UX concepts for the Terra application
- Data/algorithm specifications and supporting technical artefacts
- Technical architecture and engineering guidance

The goal is to keep the program’s “source of truth” in Markdown so changes are reviewable (diff-friendly) and easy to collaborate on.

## How this content is organised

Each numbered folder is a domain area. In general, read from low numbers to high numbers (strategy → governance → product → data → architecture → implementation).

### 00_Strategy_&_Vision
High-level narrative and decision framing:
- Executive summary
- Corridor-first deployment framing
- Founder/board strategic summary
- Performance envelope + cost model
- Structured risk review

### 01_Governance_&_Standards
Operating model, participation, and policy:
- Constitution + governance pack
- Pillar definitions and structural addenda
- NRM alignment addendum
- Security and governance specification
- Governance/participation framework for the Founding 50 observatories

### 02_Product_Terra_App
Product documentation for the Terra app.

Subfolders:
- `02.1 Product Requirements/` — Requirements and functional specs (including older versions).
- `02.2 UX & Concepts/` — UX concept documents and related artifacts.
- `02.3 Roadmap/` — Intended for roadmap materials (may be empty and include `.gitkeep`).

### 03_Data_&_Algorithms
Data processing and algorithm specifications:
- Algorithm and data processing spec (v1.0)
- Addendum(s)
- Consolidated v1.1 spec

Subfolders exist for:
- `03.1 Algorithm & Processing/`
- `03.2 Data Model/`
- `03.3 External Data Integrations/` (including geospatial integration)

### 04_Technical_Architecture
System-level design and engineering guidance:
- Technical architecture
- Engineering pack
- Strategic architecture evolution framework

### 05_Field_Implementation
Intended for field deployment and implementation docs.
- May be empty; `.gitkeep` is used to preserve structure in Git.

### 06_Commercial_&_Partnerships
Intended for commercial strategy and partnership materials.
- May be empty; `.gitkeep` is used to preserve structure in Git.

### 07_Research_&_Scientific_Backbone
Intended for research positioning and scientific backbone materials.

### 08_Brand_&_Media
Intended for brand guidelines and media assets (including images).

### 100_Terra_Phase1_Handover_Emergent
Phase 1 handover documentation organised by topic (context, product, build, post-build, etc.).

### 99_Original Files
A catch-all for migrated/legacy Markdown conversions that may not yet be fully curated into the numbered structure.

## Editing guidelines

- Prefer updating documents in the appropriate numbered folder rather than `99_Original Files`.
- Keep filenames stable when possible (helps with links and Git history).
- When adding a new section/folder that should exist in Git even before content is written, add a `.gitkeep`.
