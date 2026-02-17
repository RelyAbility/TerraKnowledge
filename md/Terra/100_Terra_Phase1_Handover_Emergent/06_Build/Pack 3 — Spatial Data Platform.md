**Pack 3 --- Spatial Data Platform**

**Supabase + PostGIS (Grid-first, edge-only adjacency, audit-ready)**

**1) Core Design Rules**

- **PostGIS is the source of truth** for geometry.

- **Never overwrite history**: scores, missions, events are append-only
  with latest views.

- **Grid-first queries must be fast**: map needs to load dozens of grids
  with tiers in \<1s.

- **Edge-only adjacency** is generated once per boundary (or regenerated
  if grid changes).

**2) Database Schema (Minimum Viable)**

**A) Boundary**

boundaries

- boundary_id (uuid, pk)

- name (text)

- centre_geom (geometry(Point, 7844)) *(GDA2020 EPSG:7844)*

- radius_m (int) *(= 4200)*

- geom (geometry(Polygon, 7844))

- crs_epsg (int, default 7844)

- created_at

**Indexes**

- GIST on geom

**B) Grid (1 km² performance units)**

grids

- grid_id (text, pk) *(deterministic)*

- boundary_id (uuid, fk)

- geom (geometry(Polygon, 7844))

- centroid (geometry(Point, 7844))

- area_m2 (numeric)

- is_clipped (bool)

- created_at

**Indexes**

- GIST on geom

- BTREE on boundary_id

**C) Grid adjacency (edge-only)**

grid_adjacency

- boundary_id (uuid)

- grid_id (text)

- neighbour_grid_id (text)

- adjacency_type (text) default \'EDGE\'

**Constraints**

- Unique(boundary_id, grid_id, neighbour_grid_id)

- grid_id != neighbour_grid_id

**Indexes**

- BTREE on (boundary_id, grid_id)

- BTREE on (boundary_id, neighbour_grid_id)

**D) Properties (tenure overlay)**

properties

- property_id (uuid, pk)

- rp_number (text, nullable)

- geom (geometry(Polygon, 7844))

- boundary_id (uuid, fk, nullable) *(optional: link if inside boundary)*

- boundary_confidence (text) *('RP', 'DRAWN', 'PARTNER_VERIFIED')*

- created_at

**Indexes**

- GIST on geom

- BTREE on rp_number

**E) Property ↔ Grid mapping**

grid_properties

- grid_id (text, fk)

- property_id (uuid, fk)

- area_overlap_pct (numeric)

- area_overlap_m2 (numeric)

**Constraints**

- Unique(grid_id, property_id)

**Indexes**

- BTREE on grid_id

- BTREE on property_id

**F) Score runs and snapshots (append-only)**

score_runs

- score_run_id (uuid, pk)

- boundary_id (uuid, fk)

- model_version (text) *(e.g. BPPF_v1.0)*

- dataset_versions (jsonb)

- run_at (timestamptz)

- notes (text, nullable)

grid_scores

- grid_id (text, fk)

- score_run_id (uuid, fk)

- priority_index (numeric)

- tier (text)

- ev (numeric)

- conn (numeric)

- sens (numeric)

- upl (numeric)

- confidence (numeric)

- explain (jsonb)

- created_at (timestamptz)

**Indexes**

- BTREE on (score_run_id, grid_id)

- BTREE on (grid_id, created_at)

- BTREE on tier

**G) Events (audit log)**

event_log

- event_id (uuid, pk)

- event_type (text)

- actor_id (uuid, nullable) *(user or system)*

- entity_type (text) *(GRID / PROPERTY / MISSION / SCORE_RUN / CLAIM)*

- entity_id (text) *(store as text for flexibility)*

- payload (jsonb)

- created_at (timestamptz)

**Indexes**

- BTREE on (entity_type, entity_id)

- BTREE on event_type

- BTREE on created_at

**3) Views for "Latest" (So the app is simple)**

**Latest score per grid (per boundary)**

v_grid_latest_score

- For each grid_id, join to the most recent grid_scores by
  score_run.run_at.

This enables:

- GET /boundary/:id/grids → grids + v_grid_latest_score.

**4) PostGIS Functions Emergent Must Implement**

**F1) Build boundary polygon**

fn_make_boundary(centre_point, radius_m) -\> polygon

Uses ST_Buffer in metres. (If working in EPSG:7844, convert to a metric
projection for buffering, then back.)

**F2) Generate 1 km grid for boundary**

fn_generate_grids(boundary_id)

Steps:

- Create a 1 km fishnet covering boundary bbox

- Clip cells to boundary polygon

- Compute centroid, area, is_clipped

- Create deterministic grid_id

**Deterministic Grid_ID rule**

- Use centroid coordinates rounded to a fixed precision (or integerised
  grid indices)

- Example: BND-{boundary_id_short}-X{col}-Y{row} (preferred)

- Avoid random UUIDs for grids (breaks audit repeatability)

**F3) Generate edge-only adjacency**

fn_generate_grid_adjacency(boundary_id)

Rule:

- Neighbours if **shared boundary line length \> 0**

- Implementation: ST_Touches(a.geom, b.geom) **AND**
  ST_Dimension(ST_Intersection(a.geom, b.geom)) = 1

- Exclude corner-only touch.

**F4) Property ↔ Grid intersection mapper**

fn_map_property_to_grids(property_id)

- Intersect property geom with grids

- Store overlap area and % in grid_properties

**5) Triggers & Jobs (Operational Automation)**

**T1) On property insert/update → remap to grids**

Trigger:

- After insert/update on properties

- Run fn_map_property_to_grids(property_id)

- Emit event_log entry PROPERTY_MAPPED_TO_GRIDS

**T2) Monthly NDVI refresh → new score run**

This can be a scheduled job (Supabase cron / external worker) that:

- Creates new score_runs row

- Computes grid_scores batch

- Emits event_log entry SCORE_RUN_COMPLETED

(Scoring compute can run outside DB, but results are stored here.)

**6) Security Model (RLS) --- Minimal but Safe**

**Publicly readable**

- boundaries, grids, v_grid_latest_score\
  (These are the "map layer" --- no private landholder data.)

**Restricted**

- properties, claims, any user-linked tables\
  Landholder polygons should not be publicly visible by default.

**Recommended privacy approach**

- App shows **grid performance publicly**

- Property geometry only visible to the owner (and authorised
  partner/admin)

**7) Performance Requirements (Non-negotiable)**

- Boundary grid map request should return:

  - grid_id, geometry (simplified), tier, priority_index, confidence

- Geometry for mobile should be **simplified**:

  - Create geom_simplified column or view using
    ST_SimplifyPreserveTopology

- Use GIST indexes everywhere geometry is joined.

**8) What Emergent Hands Back After Pack 3**

- Working Supabase project with PostGIS enabled

- Boundary loaded (4.2 km)

- Grids generated + deterministic IDs

- Edge-only adjacency generated

- Views returning "latest scores" (empty until Pack 2 service runs)

- Property claim pipeline creates properties + grid intersections +
  event logs
