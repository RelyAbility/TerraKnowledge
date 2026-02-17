**Gondwana BioReserve**

**Phase 1 Execution Layer Specification**

**(Non-Ecological -- Operational Sequencing Layer)**

Version 1.0 -- Brookfield Pilot (4.2 km Radius)

**1. Purpose**

The Phase 1 Execution Layer is an operational overlay designed to
prioritise landholder engagement and restoration sequencing within a
defined pilot boundary.

It does **not** modify ecological scoring.

It operates in parallel to the Biodiversity Priority Index (BPI) and
exists purely to:

- Identify actionable grid cells

- Sequence engagement efficiently

- Distinguish public vs private leverage

- Support Phase 1 delivery

**2. Separation of Concerns**

  ------------------------------------------------------
  **Layer**     **Function**        **Affects Ecological
                                    Score?**
  ------------- ------------------- --------------------
  BPPF / BPI    Biodiversity        Yes
                significance        

  Execution     Delivery            No
  Layer         feasibility         
  ------------------------------------------------------

Ecological integrity remains untouched.

Biodiversity Performance & Prio...

**3. Spatial Scope**

Pilot Boundary:

- Centre: 81 Boscombe Road, Brookfield

- Radius: 4.2 km

- Coordinate system: GDA2020

- Grid: 1 km × 1 km clipped to boundary

Each grid cell retains:

- Biodiversity Priority Index (0--100)

- Ecological classification tier

**4. Implementation Leverage Index (ILI)**

Each grid cell receives a **separate operational score (0--100)** based
on cadastral and land control characteristics.

**4.1 Components**

**A. Private Land Leverage (0--50 points)**

Score proportional to % of private land within grid cell.

100% private → 50 points\
0% private → 0 points

Rationale: Private land drives Phase 1 participation potential.

**B. Parcel Fragmentation (0--30 points)**

Based on number of distinct RPs intersecting cell.

1 RP → 30\
2 RPs → 22\
3--4 RPs → 12\
5+ RPs → 0

Rationale: Fewer landholders = lower coordination friction.

**C. Parcel Dominance (0--20 points)**

Based on largest RP share of grid cell.

≥80% → 20\
60--79% → 14\
40--59% → 8\
\<40% → 0

Rationale: Strong single-parcel control increases execution certainty.

**4.2 Formula**

ILI = Private Score + Fragmentation Score + Dominance Score

Range: 0--100

**5. Phase 1 Execution Categories**

Each grid cell is assigned one of four execution categories.

**Category A --- Engagement Cells**

Criteria:

- ILI ≥ 60

- BPI ≥ 50

Primary Phase 1 landholder onboarding focus.

**Category B --- Public Core Cells**

Criteria:

- Public land ≥ 70%

- BPI ≥ 65

Ecologically critical.\
Engagement via council/state partnerships, not private onboarding.

**Category C --- Mixed Strategy Cells**

Criteria:

- BPI ≥ 50

- ILI between 40--59

Require targeted engagement and corridor stitching.

**Category D --- Deferred**

Criteria:

- BPI \< 50 or ILI \< 40

Not priority for Phase 1.

**6. Data Architecture Alignment**

This layer integrates directly with:

Terra -- Cadastral (RP) Integration Addendum v1.0

Terra_Cadastral_RP_Integration\_...

Specifically:

- cadastral_parcels table

- plots.rp_number

- Sub-RP overlay support (Model C)

- Canonical RP geometry linkage

This ensures:

- Engagement traceability by parcel

- Participation density metrics

- Future compliance alignment readiness

**7. Outputs**

Phase 1 implementation produces:

1.  Ecological Heatmap (BPI)

2.  Execution Heatmap (ILI)

3.  Engagement Cell List (Grid_ID + RP list)

4.  Public Core Map

5.  Top 20 RPs ranked by:

    - Overlap with Engagement Cells

    - Cumulative ecological score

**8. Governance Safeguards**

- ILI never alters BPI.

- Ecological ranking remains standards-aligned.

- Cadastral linkage used only for sequencing, not compliance claims.

- Version logging required for:

  - Boundary changes

  - Dataset versions

  - Scoring thresholds

**9. Strategic Outcome**

This execution layer transforms Gondwana Phase 1 from:

"A biodiversity mapping exercise"

into

"A coordinated ecological engagement programme across legally defined
land units."

It enables:

- Faster early wins

- Visible corridor stitching

- Concentrated sensor deployment

- Credible reporting

- Reduced execution risk
