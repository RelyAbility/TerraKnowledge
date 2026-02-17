**Pack 2 --- Biodiversity Scoring Engine**

**BPPF v1.0 (Grid-first, audit-ready, edge-only adjacency)**

**1) Purpose of the Engine**

For every Grid_ID inside the 4.2 km boundary, compute:

- Four dimension scores (0--100 each or 0--25 each --- see below)

- **Priority Index (0--100)**

- **Tier category**

- **Confidence (0--1)**

- **Explain My Score payload** (structured, not prose)

This is recalculated whenever we run a Score_Run.

**2) Inputs (Version-logged)**

Minimum datasets (Queensland pilot):

- **Regional Ecosystem (RE) mapping** + **REDD remnant definition**

- **RE conservation status** (Endangered / Of Concern / Least Concern)

- **Waterways**

- **Protected Areas**

- **NDVI** (Sentinel-2 baseline window, cloud-masked)

Every run must store dataset versions in Score_Run.dataset_versions.

**3) Outputs (Data Contract)**

**A) Per Grid (Grid_Score)**

- priority_index (0--100)

- tier (Critical / Strategic / High Uplift / Corridor Support / Low)

- dimension_scores:

  - ecological_value

  - connectivity

  - conservation_sensitivity

  - uplift_potential

- confidence (0--1)

- explain (JSON)

**B) Explain JSON (required)**

Example shape:

{

\"drivers\": \[

{\"dimension\":\"ecological_value\",\"factor\":\"endangered_RE_present\",\"impact\":\"+8\"},

{\"dimension\":\"connectivity\",\"factor\":\"touches_waterway\",\"impact\":\"+6\"}

\],

\"data_coverage\": {

\"re_layer\": true,

\"waterways\": true,

\"protected_areas\": true,

\"ndvi\": true,

\"cloud_free_pct\": 0.82

},

\"notes\": \[\"Grid clipped at boundary edge\"\]

}

The app uses this to show "why", transparently.

**4) Scoring Structure (BPPF weights)**

Weights are fixed (v1.0):

- Ecological Value **40%**

- Connectivity **20%**

- Conservation Sensitivity **20%**

- Uplift Potential **20%**

Priority Index formula:

Priority = EV\*0.4 + CONN\*0.2 + SENS\*0.2 + UPL\*0.2

All scores normalised to 0--100.

**5) Dimension Definitions (Operational v1.0)**

**5.1 Ecological Value (EV) --- 0--100**

Measures intrinsic biodiversity value of what exists.

**Components (v1.0)**

- **RE conservation status presence**

  - Endangered RE present: +40

  - Of Concern RE present: +25

  - Least Concern RE present: +10

- **Remnant extent (% of grid remnant under REDD)**

  - 70% remnant: +30

  - 30--70% remnant: +20

  - 10--30% remnant: +10

  - \<10% remnant: +0

- **Protected area overlap**

  - 20% overlap: +20

  - 1--20% overlap: +10

  - 0: +0

Cap at 100.

Note: This avoids hardcoding "Tier 1 RE lists" until you explicitly
define them in a later version. It relies on official status classes
instead, which is cleaner and statewide scalable.

**5.2 Connectivity (CONN) --- 0--100**

Measures corridor function using **edge-only adjacency**.

**Components**

- **Remnant adjacency count (edge-touch neighbours only)**

  - Count neighbours where neighbour remnant % \>30%

  - Score:

    - 4 neighbours: +40

    - 3: +30

    - 2: +20

    - 1: +10

    - 0: +0

- **Waterway interaction**

  - Waterway intersects grid: +30

  - Within 100m of waterway: +20

  - Else: +0

- **Connection to large remnant patch**

  - If grid touches (edge) a remnant polygon \>50 ha: +30

Cap at 100.

**5.3 Conservation Sensitivity (SENS) --- 0--100**

Measures ecological/regulatory risk and vulnerability.

**Components**

- **Endangered RE present:** +40

- **High edge exposure (fragmentation proxy)**

  - Edge-to-core ratio \> threshold: +30\
    (computed from remnant polygons within grid)

- **High urban interface pressure (optional v1.0 if data available)**

  - If land-use indicates peri-urban development adjacency: +30

Cap at 100.

**5.4 Uplift Potential (UPL) --- 0--100**

Measures how much ecological gain is possible via intervention.

**Components**

- **Degraded vegetation proxy (NDVI relative to local benchmark)**

  - NDVI below RE-median by \>X: +40

  - Below by moderate margin: +25

  - Near/above: +0

- **Restoration adjacency opportunity**

  - Grid borders (edge-touch) a higher-tier grid: +30\
    (e.g., neighbour tier is Critical/Strategic)

- **Corridor stitching opportunity**

  - Grid has remnant fragments but low connectivity (CONN \< 30) while
    having \>10% remnant: +30

Cap at 100.

**6) Tier Classification (Fixed Bands)**

- 80--100: **Critical Biodiversity Core**

- 65--79: **Strategic Protection**

- 50--64: **High Uplift Zone**

- 35--49: **Connectivity Support**

- \<35: **Low Ecological Leverage**

**7) Confidence Score (0--1)**

Confidence is NOT the priority score. It indicates reliability of the
score.

**Inputs**

- cloud_free_pct for NDVI window

- dataset completeness flags

- boundary clipping proportion (small clipped grids slightly lower
  confidence)

Example simple model:

- Start at 1.0

- If cloud_free_pct \< 0.6: −0.2

- If NDVI missing: −0.3

- If RE missing (should never happen): −0.6

- If grid area \<0.5 km² due to clipping: −0.1\
  Clamp 0--1.

**8) Execution Cadence (Phase 1)**

- **Baseline run** at launch

- **Monthly refresh** for NDVI-driven components

- Full re-run when dataset versions change

Each run produces a new score_run_id and new snapshots.

**9) Engineering Notes for Emergent**

- All calculations must be automatable

- Store intermediate metrics (remnant %, cloud_free_pct, adjacency
  counts) to support "Explain My Score"

- Never overwrite history; always append new Score_Run + Grid_Score
