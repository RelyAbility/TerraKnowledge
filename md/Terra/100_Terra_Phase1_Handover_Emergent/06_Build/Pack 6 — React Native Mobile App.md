**Pack 6 --- React Native Mobile App**

**Terra EOS v1 (Landscape-first, grid-first UX)**

**1) Product Goal (Phase 1)**

Deliver a mobile app that:

- Defaults to the **landscape view** (1 km grid heatmap)

- Makes the **Priority Index** understandable ("why this grid matters")

- Converts insight into **missions**

- Shows **corridor/cluster progress** (network effects)

- Onboards landholders into claiming land **without making property the
  centre**

**2) Navigation Structure (simple, OS-first)**

**Bottom tabs (recommended)**

1.  **Landscape** (default landing)

2.  **Missions**

3.  **Network**

4.  **Profile**

Minimal tabs = faster comprehension.

**3) Core Screens (Implementation-ready)**

**A) Landscape Home (default)**

**Purpose:** show the ecological operating layer.

Components:

- Map with:

  - 4.2 km boundary outline

  - 1 km grid overlay

  - Heat tiers (Critical → Low)

- Tap grid opens **Grid Detail Drawer**

Interactions:

- Pan/zoom

- Tap cell → highlight + show summary

Data call:

- GET /boundary/:id/grids (simplified polygons + tier + score +
  confidence)

Performance requirements:

- Use simplified geometry + server-side caching

- Lazy load as tiles or bounding-box query if needed

**B) Grid Detail (drawer or full screen)**

**Purpose:** convert grid score into clarity + action.

Show:

- Tier + Priority Index + Confidence

- Mini breakdown:

  - EV, CONN, SENS, UPL

- "Why this score" bullets (from Explain payload)

- Cluster status if applicable:

  - cluster name/type

  - activation state

  - corridor progress (e.g. 2/5 active)

- **Recommended missions** (Top 3 + 1 corridor mission)

Data calls:

- GET /grid/:grid_id

- GET /grid/:grid_id/missions

- GET /grid/:grid_id/neighbours (summary)

CTA buttons:

- "Start a mission"

- "Claim land in this grid" (secondary)

**C) Missions Tab**

**Purpose:** keep people engaged week-to-week.

Sections:

- **Your Active Missions**

- **Recommended Missions**

  - personalised by grids influenced

- **Corridor Missions**

  - missions that unlock network bonuses

Mission card includes:

- Effort band (Low/Med/High)

- Verification type (photo/time/NDVI)

- Expected outcome + conservative score impact range

Data calls:

- GET /user/me/missions

- GET /user/me/recommended_missions

**D) Mission Detail**

**Purpose:** help user complete the intervention.

Must include:

- Steps checklist

- Upload evidence (photo/notes)

- Timer logic if time-based verification

- "What changes when you complete this" (grid + cluster implications)

Actions:

- Start mission

- Submit completion

- Submit verification evidence

Calls:

- POST /mission/start

- POST /mission/complete

- POST /mission/verify

**E) Network Tab (corridors & clusters)**

**Purpose:** make network effects visible and motivating.

Show:

- Clusters in boundary:

  - Activation state (Emerging/Active/Strategic Node)

  - Collective Uplift Index

  - "Needed to activate" progress

- "Gaps" list:

  - clusters close to activation but missing 1--2 grids

- Optional: invite/share for specific corridor

Calls:

- GET /boundary/:id/clusters

- GET /cluster/:cluster_id

**F) Claim Flow (property is secondary, but essential)**

**Entry points:**

- From a grid: "Claim land in this grid"

- From Profile: "Add land"

Two options:

1.  **Enter RP number**

2.  **Draw boundary on map**

Post-claim:

- Show confirmation: "Your property overlaps 2 grids"

- Immediately show missions available in those grids

Calls:

- POST /property/claim

- GET /user/me/claims

Privacy note:

- Property polygon visible only to claimant (unless consented)

**G) Profile Tab (identity, not vanity)**

Show:

- Role: Participant / Corridor Contributor / Cluster Builder / Steward

- Grids influenced (count + tiers)

- Missions completed + verified

- Contribution to Collective Uplift Index (cluster-level)

Calls:

- GET /user/me/profile

- GET /user/me/impact

**4) Map Rendering Strategy (critical)**

Recommend:

- MapLibre or RN Maps depending on stack; key is **performance**

- Render **grid polygons simplified** for mobile

- Use:

  - bbox queries or tile-like loading if boundary expands later

- Avoid rendering property polygons by default

**5) Behaviour Design (the engagement loop)**

**Primary loop (weekly)**

1.  User opens Landscape view

2.  Taps grid

3.  Understands "why"

4.  Starts mission

5.  Submits evidence

6.  Sees progress in cluster activation + uplift index

7.  Invites neighbour if gap remains

**Trigger moments**

- "Corridor is 1 grid away from activation"

- "Your mission completion unlocked a cluster bonus"

- "A neighbouring grid joined --- corridor progress increased"

**6) Notifications (minimal but powerful)**

Push events:

- Mission verification milestone reached

- Cluster state changes (Emerging → Active)

- "Gap alert" for corridor completion

- Monthly "score refresh" report available

No spam.

**7) Offline + Low-friction UX**

- Cache latest grid layers locally

- Allow mission evidence capture offline, upload when connected

- If connectivity poor: degrade gracefully (no blank maps)

**8) Success Metrics (Phase 1 OS metrics)**

You track:

- Grid taps → mission starts conversion

- Mission start → verified completion rate

- Cluster activation velocity

- Invite conversions in adjacent grids

- DAU/WAU among claimed users

- \% of boundary grids moving into ACTIVE

These are operating system metrics, not "app vanity".

**9) Emergent Deliverables After Pack 6**

✔ React Native app with navigation + screens above\
✔ Grid-first map performant and stable\
✔ Mission lifecycle fully functional\
✔ Cluster/corridor views implemented\
✔ Claim flow (RP + draw) working\
✔ Profile + reputation computed and displayed\
✔ Notification hooks wired
