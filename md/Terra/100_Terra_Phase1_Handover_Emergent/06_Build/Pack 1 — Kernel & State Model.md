**Pack 1 --- Kernel & State Model**

**Terra EOS v1 (Landscape-first, 1 km Grid First)**

**1) Core Principles**

- **Primary unit:** Grid_ID (1 km × 1 km "Performance Unit")

- **Secondary unit:** Property_ID (land tenure overlay only)

- **Boundary:** ecological study area (4.2 km radius from 81 Boscombe
  Rd, Brookfield)

- **Everything is versioned:** scoring logic + dataset versions + run
  timestamp

- **No manual overrides** without an audit log entry

**2) Entities and Identifiers**

**A) Grid**

**Entity:** Grid\
**Key:** grid_id (stable, deterministic)

**Fields (minimum)**

- grid_id (string, deterministic)

- geom (polygon)

- centroid (point)

- boundary_id (links to the 4.2 km study boundary)

- area_km2 (should be 1.0 except clipped edges)

- created_at, updated_at

**B) Boundary**

**Entity:** Boundary\
**Key:** boundary_id

**Fields**

- boundary_id

- name ("Brookfield Pilot")

- geom

- centre_point

- radius_km (= 4.2)

- crs (= GDA2020)

- created_at

**C) Property (overlay)**

**Entity:** Property\
**Key:** property_id

**Fields**

- property_id

- geom

- rp_number (nullable)

- confidence_property_boundary (drawn vs RP verified)

- created_at

**D) User**

**Entity:** User\
**Key:** user_id

**Fields**

- user_id

- role (landholder / partner / admin)

- created_at

**E) Claim (User ↔ Property)**

**Entity:** Claim\
**Key:** claim_id

**Fields**

- claim_id

- user_id

- property_id

- claim_status (see states below)

- verification_method (RP / draw / partner verified)

- created_at, updated_at

**F) Grid-Property Link (many-to-many)**

**Entity:** Grid_Property\
**Key:** composite (grid_id, property_id)

**Fields**

- grid_id

- property_id

- area_overlap_pct

**3) State Machines (the "OS behaviour")**

**A) Grid State**

**Enum:** grid_state

1.  BASELINE

2.  PRIORITISED

3.  MISSIONABLE

4.  ACTIVE

5.  VERIFIED_UPLIFT

**Entry / Exit Triggers**

- BASELINE → PRIORITISED: scoring engine run exists for grid (Priority
  Index computed)

- PRIORITISED → MISSIONABLE: at least 1 mission is eligible for this
  grid

- MISSIONABLE → ACTIVE: at least 1 mission started by any claimed
  property overlapping the grid

- ACTIVE → VERIFIED_UPLIFT: verification event completed (satellite
  proxy delta OR evidence + time threshold)

**B) Property Claim State**

**Enum:** claim_status

1.  UNCLAIMED

2.  CLAIMED_UNVERIFIED

3.  CLAIMED_VERIFIED

4.  ACTIVE

5.  STEWARD

**Triggers**

- UNCLAIMED → CLAIMED_UNVERIFIED: user submits RP or draws polygon

- CLAIMED_UNVERIFIED → CLAIMED_VERIFIED: RP match / partner verification
  / admin review

- CLAIMED_VERIFIED → ACTIVE: user starts first mission

- ACTIVE → STEWARD: sustained participation threshold (e.g., 3+ missions
  completed OR 1 corridor mission completed)

**4) Versioning & Auditability (non-negotiable)**

**A) Score Versioning**

**Entity:** Score_Run

Fields

- score_run_id

- model_version (e.g., "BPPF_v1.0")

- dataset_versions (JSON: RE layer, waterways, protected areas, NDVI
  window, etc.)

- run_at

- boundary_id

**B) Grid Score Snapshot**

**Entity:** Grid_Score

Fields

- grid_id

- score_run_id

- priority_index (0--100)

- tier (Critical / Strategic / High Uplift / Corridor Support / Low)

- dimension_scores (JSON: EV, CONN, SENS, UPLIFT)

- confidence (0--1)

- explain (short text / JSON keys for "why this score")

**C) Event Log (everything important)**

**Entity:** Event_Log

Fields

- event_id

- event_type (CLAIM_CREATED, SCORE_RUN, MISSION_STARTED,
  VERIFICATION_COMPLETED, etc.)

- actor_id (user/system)

- entity_type (Grid/Property/Mission)

- entity_id

- payload (JSON)

- timestamp

**5) Required APIs (minimum contract)**

**Grid-first UX endpoints**

- GET /boundary/:id/grids\
  Returns grid polygons + latest priority_index + tier.

- GET /grid/:grid_id\
  Returns full grid details: latest score, dimensions, explanation, NDVI
  trend summary, eligible missions, adjacency IDs.

- GET /grid/:grid_id/neighbours\
  Returns adjacent grid_ids and their tier/state (privacy rules apply).

**Claim / property**

- POST /property/claim (RP or drawn polygon)

- GET /user/me/claims

**Missions**

- GET /grid/:grid_id/missions (eligible + recommended)

- POST /mission/start

- POST /mission/complete

- POST /mission/verify

**6) Adjacency & Corridor Primitives (kernel-level)**

Even though full network effects are Pack 5, the kernel must include:

**Entity:** Grid_Adjacency

- grid_id

- neighbour_grid_id

- type (edge-touch / corner-touch --- pick one and freeze it)

This lets every other engine build consistently.

**7) UX Rule: Grid is the home screen**

Home defaults to:

- Boundary map

- 1 km grid overlay

- Heat tiers

- Tap a grid → "Why", "Missions", "Neighbour context"\
  Property only appears after you zoom/claim.

**What Emergent can start building immediately**

- Supabase schema (tables above)

- Deterministic grid generator + IDs

- Boundary loader (4.2 km circle)

- Event log middleware

- API scaffolding for grid-first app
