**Pack 4 --- Mission Engine**

**Terra Ecological Operating System v1**

(Grid-first, state-driven, score-linked, audit-ready)

**1) Purpose of the Mission Engine**

The Mission Engine converts:

- Grid scores → into actions

- Ecological potential → into behaviour

- Network context → into coordinated intervention

- Time → into measurable uplift

Without this engine, the OS is static.

**2) Mission Design Principles**

1.  **Grid-anchored** --- missions belong to grids, not properties.

2.  **Eligibility-driven** --- missions appear only when ecological
    logic allows.

3.  **Score-linked** --- every mission must have a defined delta
    pathway.

4.  **Network-aware** --- some missions require neighbour participation.

5.  **Verifiable** --- no mission advances grid state without evidence.

6.  **Conservative uplift** --- score increases only after verification
    logic passes.

**3) Core Entities (Database Additions)**

**A) Mission Library (static definitions)**

missions

- mission_id (uuid, pk)

- name

- category (PROTECTION / CONNECTIVITY / UPLIFT)

- description

- ecological_mechanism (text)

- effort_band (LOW / MED / HIGH)

- default_score_delta (numeric, conservative cap)

- verification_type (PHOTO / NDVI_PROXY / TIME_BASED / HYBRID)

- created_at

**B) Mission Eligibility Rules (logic table)**

mission_rules

- mission_id

- dimension_target (EV / CONN / SENS / UPL)

- min_priority

- max_priority

- requires_waterway (bool)

- requires_remnant_pct_min

- requires_conn_below

- requires_neighbour_tier

- json_logic (flexible future logic)

This allows dynamic eligibility per grid.

**C) Grid Mission Instance**

grid_missions

- grid_mission_id (uuid)

- grid_id

- mission_id

- status (AVAILABLE / ACTIVE / COMPLETED / VERIFIED)

- started_by_user_id

- started_at

- completed_at

- verified_at

- verification_confidence

- score_delta_applied

- created_at

**D) Mission Evidence**

mission_evidence

- evidence_id

- grid_mission_id

- evidence_type

- file_url

- metadata (jsonb)

- submitted_at

- review_status

**4) Mission Categories (v1 Library Structure)**

You do NOT need 100 missions.

Start with 12--20 high-leverage ones.

**4.1 Protection Missions**

Goal: prevent degradation.

Examples:

- Remnant buffer fencing

- Fire regime compliance plan

- Weed containment zone

- Edge stabilisation

Primary dimension impact:

- Ecological Value

- Conservation Sensitivity

Score delta logic:

- Small but high confidence

- +3 to +8 max

**4.2 Connectivity Missions**

Goal: increase corridor strength.

Examples:

- Riparian native planting

- Fence removal for fauna passage

- Native understory infill

- Shelterbelt alignment with neighbour

Primary dimension impact:

- Connectivity

Score delta logic:

- +5 to +15 depending on adjacency participation

Network multiplier may apply (see below).

**4.3 Uplift Missions**

Goal: convert degraded land to functional habitat.

Examples:

- Pasture conversion to native grass

- Assisted natural regeneration

- Structural diversity planting

- Gully repair

Primary dimension impact:

- Uplift Potential

- Secondary impact on EV

Score delta logic:

- Larger delta possible (+10 to +20)

- Time-delayed verification required

**5) Mission Eligibility Engine (Grid-Driven)**

When user taps a grid:

System evaluates:

1.  Current grid tier

2.  Dimension weaknesses

3.  NDVI deviation

4.  Waterway presence

5.  Edge-only neighbour states

6.  Property overlap

Then returns:

- "Top 3 High-Leverage Missions"

- "1 Corridor Mission (if eligible)"

This must be deterministic --- same inputs → same recommendations.

**6) Score Delta Model (Controlled & Auditable)**

Missions do NOT directly change the Priority Index.

Instead:

1.  Mission marked COMPLETED

2.  Verification event occurs

3.  Engine recalculates dimension component

4.  Delta applied at next Score_Run

5.  Grid state transitions if threshold crossed

Example:

Grid:

- Priority 58 (High Uplift)

Mission:

- Riparian native planting

Verification:

- NDVI improvement \> X threshold after 6 months

Next Score_Run:

- Connectivity +8

- Priority now 66

- Tier shifts to Strategic Protection

Event logged:\
GRID_TIER_TRANSITION

No manual bumping.

**7) Network Multiplier Logic**

Connectivity missions can receive multiplier only if:

- At least 2 adjacent (edge-only) grids are ACTIVE

- Or corridor segment condition is met

Example rule:

If:

- Grid A + Grid B + Grid C (contiguous) complete connectivity missions

Then:

- Additional +5 shared corridor bonus applied to all three at next run.

This must be transparent in Explain payload.

**8) Grid State Transitions (Connected to Missions)**

State logic:

BASELINE\
→ PRIORITISED (score exists)\
→ MISSIONABLE (eligible missions exist)\
→ ACTIVE (≥1 mission started)\
→ VERIFIED_UPLIFT (verified mission delta applied)

No skipping states.

**9) Property-Level Progress View (Secondary Layer)**

Although grid-first, users need identity.

Property dashboard shows:

- Grids influenced

- Active missions

- Collective corridor progress

- Tier shifts achieved

- "Steward" threshold progress

This builds behavioural retention.

**10) Anti-Gaming Safeguards**

- No unlimited stacking of same mission

- Cap cumulative delta per dimension per year

- Require time delay for large uplift missions

- Flag anomalous rapid NDVI spikes

- Manual admin override requires event log entry

This protects credibility.

**11) What Emergent Must Deliver After Pack 4**

✔ Mission library seeded (minimum 12 missions)\
✔ Eligibility logic implemented\
✔ Grid mission lifecycle functioning\
✔ Score delta integration pathway defined\
✔ Network multiplier logic coded\
✔ Verification pathways implemented\
✔ Event log integration
