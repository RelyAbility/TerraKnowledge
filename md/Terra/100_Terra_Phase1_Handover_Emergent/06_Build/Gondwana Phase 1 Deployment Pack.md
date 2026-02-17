**Gondwana Phase 1 Deployment Pack**

**Ecological Boundary Pilot -- 4.2 km Radius (Centred on 81 Boscombe
Road, Brookfield)**

**Purpose (what Emergent is building)**

Implement the **Biodiversity Performance & Prioritisation Framework
(BPPF)** as a repeatable, spatially explicit decision engine to:

- define a defensible baseline,

- prioritise protection and restoration,

- quantify ecological risk,

- and measure uplift over time.

> Biodiversity Performance & Prio...

**Scope**

- **Boundary**: 4.2 km radius ecological study area (not 10 km)

- **Primary signal**: landscape + vegetation (satellite + official
  datasets)

- **Analytical unit**: **1 km × 1 km grid cells** ("Performance Units")

> Biodiversity Performance & Prio...

**Pack A --- Boundary & GIS Definition**

**A1. Boundary definition (must be unambiguous)**

- Centre point: **81 Boscombe Road, Brookfield**

- Radius: **4.2 km**

- Projection: **GDA2020**

- Outputs:

  - Boundary polygon (GeoJSON + Shapefile)

  - Basemap PDF (for human reference)

**A2. Property layer (optional but recommended)**

- Parcel boundaries + RP numbers included where available (as an
  operational layer, not the analytical unit)

**A3. Required ecological layers (dataset versions logged)**

- Queensland Regional Ecosystem (RE) mapping (and REDD remnant
  definition)

- Waterway / drainage layers

- Protected areas layer

- Elevation (DEM)

- Any state land-use layer available

**Important:** No RE codes are to be assumed in advance. The RE list
must be *extracted* inside the 4.2 km boundary and recorded.

**Pack B --- BPPF Grid Implementation (Core)**

**B1. Create the grid**

- Overlay **1 km × 1 km grid** across the 4.2 km boundary

- Clip grid to boundary

- Assign unique Grid_ID to each cell

> Biodiversity Performance & Prio...

**B2. Baseline definition (audit-ready)**

- Baseline is defined using:

  - RE mapping + REDD remnant definition

  - Conservation status classes (Endangered / Of Concern / Least
    Concern)

  - Baseline date + dataset versions recorded

> Biodiversity Performance & Prio...

**Pack C --- Scoring Engine (Priority Index 0--100)**

Use the BPPF weighting structure exactly (Version 1.0):

- Ecological Value --- 40%

- Connectivity --- 20%

- Conservation Sensitivity --- 20%

- Uplift Potential --- 20%

> Biodiversity Performance & Prio...

Priority Score formula:\
Priority Score = (EV × 0.4) + (CON × 0.2) + (SENS × 0.2) + (UP × 0.2)

Biodiversity Performance & Prio...

**C1. Classification bands (must match framework)**

- 80--100 Critical Biodiversity Core

- 65--79 Strategic Protection Zone

- 50--64 High Uplift Zone

- 35--49 Connectivity Support

- \<35 Low Ecological Leverage

> Biodiversity Performance & Prio...

**C2. Output requirements**

- Grid table export (CSV): Grid_ID + dimension scores + total + category

- GIS heatmap layer (GeoJSON/Shapefile)

- Area breakdown by category

**Pack D --- Vegetation Baseline (Satellite-First)**

**D1. NDVI and vegetation condition layers**

- Produce NDVI baseline and trend layers suitable for overlay against
  the grid

- Log:

  - sensor/source (e.g., Sentinel-2 if used)

  - cloud masking approach

  - baseline window (e.g., trailing 12 months)

**D2. What NDVI is used for in Phase 1**

- As a repeatable vegetation vigour proxy to support:

  - uplift potential scoring,

  - degradation identification,

  - restoration opportunity detection.

(We can be explicit that NDVI is a proxy and not a direct biodiversity
measure --- that's part of defensibility.)

**Pack E --- Connectivity & Corridor Logic (Ecological Boundary
Justification)**

This is where we justify **why 4.2 km is ecological**, not arbitrary.

**Insert this exact text into the pack (ready to use)**

**Ecological Boundary Rationale (4.2 km):**\
"The Phase 1 boundary is defined as a 4.2 km radius to capture the
functional ecological system surrounding the Gondwana anchor site,
including local hydrological influence, remnant patch interactions, and
short-range connectivity dynamics. This radius is intended to include
the immediate corridor and edge-context that drives habitat integrity,
fragmentation pressure, and restoration leverage, while remaining small
enough to enable high-resolution auditability and repeatable monitoring
using a fixed 1 km² performance grid."

Biodiversity Performance & Prio...

Regional Biodiversity Priority ...

(That reads like a methods section, which is what you want.)

**Pack F --- Deliverables to Hand Back (What Emergent must deliver)**

1.  Boundary package (4.2 km)

2.  1 km² grid package (Performance Units)

3.  Dataset register (source + version + date)

4.  BPPF scores per grid cell + category

5.  Priority heatmap GIS layer + PDF visual

6.  Ranked engagement list (grid clusters → properties)

7.  Short "Methods + Assumptions" appendix (audit-ready)

All consistent with the BPPF and heatmap methodology structure.

Biodiversity Performance & Prio...

Regional Biodiversity Priority ...

**What I'm correcting from the earlier draft**

- **No acoustics** in Phase 1 (unless later added as an overlay in
  monitoring)

- **No pre-listed RE codes** until extracted from the 4.2 km boundary

- **No 10 km references** anywhere
