**Pack 5 --- Network Effects Engine**

**Terra Ecological Operating System v1**

(Edge-only adjacency, grid-first, landscape-coordination logic)

**1) Purpose**

The Network Engine ensures that:

- A single landholder acting has impact.

- Multiple adjacent landholders acting creates disproportionate impact.

- Ecological performance compounds as density increases.

- Participation increases value for everyone.

This is what turns Terra into infrastructure.

**2) Core Network Primitives (Built on Pack 3)**

You already have:

- grid_adjacency (edge-only)

- grid_scores

- grid_missions

- grid_state

We now build intelligence on top.

**3) Adjacency Graph Model**

Each grid is a node.

Edges exist only where:

- Shared boundary line length \> 0

- (Corner touch excluded --- already locked)

This produces a clean planar corridor graph.

We store nothing new here --- adjacency table already exists.

**4) Corridor Detection Engine**

A **corridor** is defined as:

A contiguous chain of edge-connected grids where:

- At least one of:

  - remnant vegetation \> threshold

  - waterway intersects

  - tier ≥ Connectivity Support

- AND grid_state ≥ ACTIVE (for activation phase)

**4.1 Corridor Cluster Table**

grid_clusters

- cluster_id (uuid)

- boundary_id

- cluster_type (REMNANT_CORE / CONNECTIVITY_CORRIDOR / UPLIFT_CLUSTER)

- grid_count

- aggregate_priority

- activation_state

- created_at

- updated_at

**4.2 Cluster Detection Logic (Run after each score_run)**

Algorithm:

1.  Filter grids by:

    - tier ≥ 35 OR remnant % ≥ X

2.  Build connected components using edge adjacency

3.  Each connected component becomes a cluster

4.  Calculate:

    - average priority_index

    - \% of grids in ACTIVE or VERIFIED_UPLIFT

    - total remnant area

Clusters update automatically.

**5) Cluster Activation States**

Clusters also have state:

1.  INACTIVE

2.  EMERGING

3.  ACTIVE

4.  STRATEGIC_NODE

**Rules**

- INACTIVE → EMERGING:

  - ≥2 adjacent grids in ACTIVE

- EMERGING → ACTIVE:

  - ≥3 contiguous grids VERIFIED_UPLIFT

- ACTIVE → STRATEGIC_NODE:

  - Cluster average priority ≥ 70

  - AND ≥5 contiguous grids

This creates visible landscape milestones.

**6) Corridor Completion Mechanic**

When user views a grid:

System checks:

- Is this grid part of a cluster?

- How many adjacent grids are ACTIVE?

- How many are unclaimed?

UI shows:

"This corridor is 2 of 5 grids activated."

This drives neighbour engagement.

**7) Collective Uplift Index (CUI)**

This is the landscape-level compounding metric.

**Definition:**

CUI =\
(Σ verified delta across cluster)\
÷\
(total cluster area)

Scaled 0--100.

This is separate from Priority Index.

It answers:

"How much real uplift has occurred in this corridor?"

This metric:

- Increases as missions are verified.

- Drives cluster activation.

- Can later support funding allocation.

**8) Participation Density Effect**

Connectivity score recalculates not only on vegetation but also on
participation density.

Add modifier:

If:

- ≥50% of grids in cluster are ACTIVE

Then:

- Connectivity dimension receives +X bonus (conservative, e.g. +3--5)

This reflects corridor stabilisation via coordinated stewardship.

Must be:

- Transparent

- Logged in Explain payload

**9) Reputation & Stewardship Layer (Lightweight in v1)**

Not gamification.

Ecological role signalling.

Property-level roles:

- Participant

- Corridor Contributor

- Cluster Builder

- Steward

Derived from:

- Number of VERIFIED_UPLIFT missions

- Participation in cluster activation

- Sustained engagement over time

These roles are shown on profile.

This increases retention without trivialising the mission.

**10) Landholder Acquisition Loop**

Network effect growth requires acquisition mechanics.

When a grid becomes ACTIVE:

System identifies:

- Adjacent grids with no claimed properties

Send invitation logic:

- "This corridor needs 2 more properties."

- Shareable link tied to specific grid cluster.

This makes growth organic and geographically targeted.

**11) Visibility Rules (Critical)**

Public layer shows:

- Grid tiers

- Cluster activation state

- Collective Uplift Index

Private layer shows:

- Property boundaries

- User identity

No doxxing.\
No exposed land ownership without consent.

**12) Compounding Logic Summary**

As more landholders join:

- Adjacency graph density increases.

- Corridor clusters grow.

- Connectivity bonuses activate.

- Collective Uplift Index rises.

- Reputation roles unlock.

- Network acquisition accelerates.

This is the exponential layer.

**13) What Emergent Must Deliver After Pack 5**

✔ Cluster detection service\
✔ Connected component algorithm\
✔ Cluster state transitions\
✔ Collective Uplift Index calculation\
✔ Participation density modifier\
✔ Cluster visibility API\
✔ Reputation role computation\
✔ Invitation trigger logic
