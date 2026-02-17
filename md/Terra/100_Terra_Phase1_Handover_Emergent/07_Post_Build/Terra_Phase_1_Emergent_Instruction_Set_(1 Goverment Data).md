# Terra Phase 1 -- Emergent Instruction Set

Document Version: 1.0

Generated: 2026-02-17 04:50:18

Scope: 4.2 km radius, grid-first biodiversity scoring engine

Objective: Deliver a scientifically defensible biodiversity leverage
platform MVP

## 1. Mission Definition

Build Terra Phase 1 as a grid-first biodiversity scoring engine that:

• Uses 1 km grid cells as the base unit

• Aggregates landscape metrics within a 4.2 km radius

• Produces a 0--100 Biodiversity Leverage Score (TBLS)

• Is fully explainable and source-attributed

• Is grounded in Queensland BPA (BAMM), AquaBAMM, RE mapping, and
WildNet datasets

Constraints:

• Do not design property-first

• Do not design 10 km logic

• Do not implement acoustic or NDVI modelling in Phase 1

## 2. Core Architectural Principles

1\. Grid-first (1 km resolution)

2\. 4.2 km aggregation radius

3\. All scores decomposable into sub-scores

4\. All layers source-attributed

5\. All spatial operations precomputed where possible

6\. No runtime heavy spatial intersections

## 3. MVP Essential Data Layers

### A. BPA (Biodiversity Planning Assessment)

• State / Regional / Local significance polygons

• Criterion H (Core habitat for priority taxa)

• Biodiversity status of Regional Ecosystems

### B. AquaBAMM (Riverine Only -- Phase 1)

• Riverine subsections

• Overall conservation significance

• Criteria ratings (connectivity, priority species, naturalness)

### C. Regional Ecosystems (RE)

• RE polygons

• Biodiversity status

• Remnant vegetation mapping

### D. Threatened + Priority Species

• Status (CR, E, V, NT)

• Occurrence geometry or grid-level presence

### E. WildNet Weeds + Pests

• Weed taxa

• Pest taxa

• Last record date

• Record count

## 4. Data Pipeline Instructions

### Step 1: Ingest

• Import GIS layers into PostGIS

• Reproject to GDA2020

• Validate geometries

• Store metadata: source, version, generated_at, precision, confidence

### Step 2: Generate 1 km Grid

Create table: grid_cells

• grid_id

• geom (1 km polygon)

• centroid

• cluster_id

### Step 3: Precompute Grid Fact Tables

Create:

• grid_bpa_facts

• grid_aquabamm_facts

• grid_re_facts

• grid_species_facts

• grid_weeds_pests_facts

All spatial joins executed offline (scheduled jobs).

### Step 4: Scoring Model

TBLS =

(0.30 × Protection Value)

\+ (0.25 × Hydrological Value)

\+ (0.20 × Species Gravity)

\+ (0.15 × Connectivity)

\+ (0.10 × Restoration Opportunity)

All scores must be 0--100 scaled and versioned.

### Step 5: 4.2 km Landscape Aggregation

Create function: get_landscape_metrics(grid_id, radius = 4200m)

Returns: avg_tbls, pct_cells_over_70, total_threatened_weight,
riverine_cover_pct, remnant_cover_pct, top drivers, mission
recommendation.

Cache results in landscape_scores_cache.

## 5. Interface Logic

### Default View

• Vector tile map of grid_scores

• Colour ramp 0--100

### On Cell Select

Display:

• TBLS score

• Radar chart of sub-scores

• Top 3 score drivers

• Evidence tabs: BPA, Water, Species, RE, Pressure

### Landscape Panel (4.2 km)

Display:

• Landscape average score

• % high leverage cells

• Riverine coverage

• Threatened density

• Recommended mission

## 6. Explainability Requirements

Every score must return clear drivers with dataset references. No
black-box logic.

## 7. Confidence Index

For each grid compute:

• Data freshness

• Spatial precision

• Layer completeness

Return High / Medium / Low confidence badge.

## 8. Non-Goals (Phase 1 Exclusions)

• Acoustic biodiversity modelling

• NDVI temporal modelling

• Soil carbon modelling

• Economic biodiversity credit valuation

• AI predictive habitat modelling

## 9. Performance Constraints

• All grid scoring precomputed

• Vector tiles for map rendering

• Landscape aggregation cached

• \<200ms response for grid select

## 10. Completion Criteria

Phase 1 is complete when:

1\. Entire SEQ cluster scored at 1 km resolution

2\. Any grid cell returns TBLS, sub-scores, evidence, and 4.2 km
landscape metrics

3\. Scores are reproducible and versioned

4\. Every value traceable to source
