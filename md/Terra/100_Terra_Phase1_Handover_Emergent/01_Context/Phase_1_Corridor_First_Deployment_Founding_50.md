**Phase 1 Acoustic Sensor Deployment\
Corridor-First Model (Founding 50)**

4.2 km radius centred on 81 Boscombe Rd, Brookfield (Gondwana cluster)\
Restoration mode \| 1 km x 1 km targeting grid \| 50 sensor cap

# 1. Executive Summary

Phase 1 establishes a 50-node acoustic observatory using a
corridor-first deployment strategy. The network is anchored by two
existing seed nodes: Gondwana (mapped watercourse on-property) and
Andrew Grace (Gold Creek frontage). Deployment prioritises hydrological
connectivity (headwaters → tributary → river) and restoration leverage,
enabling defensible measurement of biodiversity response while remaining
practical to implement and scale.

Figure 1. Conceptual corridor structure (schematic).

![](media/image1.png){width="6.2in" height="6.352834645669291in"}

# 2. Corridor-First Deployment Bands

  ---------------------------------------------------------------------------
  Band              Purpose           Sensors           Why it matters (Phase
                                                        1)
  ----------------- ----------------- ----------------- ---------------------
  Band 1            Gold Creek spine  18                Highest leverage
                    (primary                            corridor for
                    tributary                           movement, riparian
                    corridor)                           specialists, and HLW
                                                        alignment.

  Band 2            Gondwana feeder   10                Headwater integrity
                    micro-catchment                     and
                    (mapped                             feeder-to-tributary
                    watercourse +                       continuity.
                    adjacent feeders)                   

  Band 3            Riparian buffer   8                 Captures edge effects
                    halo (edge and                      and uplift response
                    transition zones)                   outward from
                                                        corridors.

  Band 4            Ridge & upland    8                 Captures
                    connectors                          cross-corridor
                    (stepping stones                    movement and
                    between                             fragmentation risk.
                    corridors)                          

  Band 5            Interior controls 6                 Baseline/control
                    & mosaic tiles                      sites and high-uplift
                                                        conversion targets.
  ---------------------------------------------------------------------------

Figure 2. 1 km tiles showing band assignment and initial sensor
allocation (numbers indicate sensors per tile).

![](media/image2.png){width="6.3in" height="6.43785542432196in"}

# 3. 1 km x 1 km Tile Targeting and Allocation

The 4.2 km radius footprint is tessellated into 1 km x 1 km tiles. Tiles
are prioritised by proximity to (a) the Gold Creek corridor line, (b)
the Gondwana feeder watercourse line, and (c) connector geography
between the two. This creates a repeatable, scalable framework: replace
the corridor lines with mapped waterway centreline(s) and tributaries
for any new region, then score tiles by corridor distance and
connectivity.

Table 2. Top priority tiles (illustrative IDs are relative offsets in km
from the centre; Band and sensors indicate Phase 1 placement).

  --------------------------------------------------------------------------
  Tile ID        Band           Sensors        Distance to    Distance to
                                               Gold Creek     Gondwana
                                               line (km)      feeder line
                                                              (km)
  -------------- -------------- -------------- -------------- --------------
  T(+0,+0)       Band 1         5              0.25           0.00

  T(+2,-3)       Band 1         5              0.00           1.81

  T(+1,-1)       Band 1         4              0.12           0.93

  T(-1,+2)       Band 1         2              0.13           2.24

  T(+0,+1)       Band 1         2              0.24           1.00

  T(+0,-1)       Band 2         5              0.75           0.07

  T(+0,-2)       Band 2         3              1.24           0.14

  T(+0,-3)       Band 2         2              1.74           0.28

  T(+1,-2)       Band 3         2              0.37           0.85

  T(+2,-2)       Band 3         1              0.50           1.85

  T(-1,+3)       Band 3         1              0.59           3.16

  T(+1,+0)       Band 3         1              0.62           1.00

  T(-1,+1)       Band 3         1              0.63           1.41

  T(+0,+2)       Band 3         1              0.74           2.00

  T(+1,-3)       Band 3         1              0.87           0.82

  T(+2,-1)       Band 3         0              0.99           1.92

  T(-1,+0)       Band 3         0              1.12           1.00

  T(+2,-4)       Band 3         0              1.00           2.16

  T(+3,-3)       Band 3         0              1.00           2.81

  T(-2,+2)       Band 4         1              1.00           2.83

  T(-2,+3)       Band 4         1              1.09           3.61

  T(+0,+3)       Band 4         1              1.23           3.00

  T(-1,+4)       Band 4         1              1.59           4.12

  T(+1,+2)       Band 4         1              1.61           2.24

  T(-2,+4)       Band 4         1              1.83           4.47
  --------------------------------------------------------------------------

# 4. Wave 1-3 Invitation Sequence (Terra → Founding 50)

Sequencing choice: Terra Free launches first to build uptake and social
proof, then Founding 50 invitations are selectively issued to the
highest-priority corridor tiles. This avoids premature scarcity
signalling and increases conversion when scarcity is introduced.

Figure 3. Indicative rollout timeline (weeks).

![](media/image3.png){width="6.3in" height="1.657211286089239in"}

Wave definitions:

> Wave 1: Band 1 tiles (Gold Creek spine) -- first invitations; fastest
> corridor density build; strong HLW narrative.
>
> Wave 2: Band 2-3 tiles (Gondwana feeder + riparian halo) -- extends
> upstream integrity and edge-response capture.
>
> Wave 3: Band 4-5 tiles (connectors + controls) -- completes
> connectivity and embeds controls for defensibility.

# 5. Acoustic Coverage Overlap (Parameterised)

Effective acoustic detection range varies by species, habitat structure,
microphone, weather, and noise. For planning, treat detection radius as
a tunable parameter (r). Coverage footprint per sensor is approximately
πr², but overlap is desirable along corridors to ensure continuity and
resilience to noise/obstruction.

Figure 4. Layout with illustrative footprints (250 m and 500 m) shown
for scale; use real-world calibration once sensors are deployed.

![](media/image4.png){width="6.3in" height="6.455299650043744in"}

Practical corridor spacing guidance (rule-of-thumb):

> Corridor continuity: aim for centre-to-centre spacing of \~0.8r to
> 1.5r along the creek line, depending on vegetation density and ambient
> noise.
>
> Junctions and transition zones: increase density (shorter spacing)
> where tributaries meet or habitat changes sharply.
>
> Controls: place farther apart; controls are for contrast, not corridor
> continuity.

# 6. Scenario Comparison (30 vs 50 vs 70 Sensors)

The table below provides indicative network density metrics for the 4.2
km radius footprint (area ≈ 55.4 km²). Nearest-neighbour spacing is
shown as a baseline expectation for a random distribution;
corridor-first placement will be denser on waterways and sparser in the
interior.

  -------------------------------------------------------------------------
  Scenario          Average area per  Sensors per km²   Expected
                    sensor (km²)                        nearest-neighbour
                                                        distance (km,
                                                        random baseline)
  ----------------- ----------------- ----------------- -------------------
  30                1.85              0.54              0.68

  50                1.11              0.90              0.53

  70                0.79              1.26              0.44
  -------------------------------------------------------------------------

Interpretation for Phase 1:

> 30 sensors: viable for corridor proof-of-concept, but less robust for
> tile-level classification and edge-effect modelling.
>
> 50 sensors (chosen): strong balance of scientific defensibility and
> practical rollout; supports corridor continuity + controls.
>
> 70 sensors: increases redundancy and spatial resolution, but raises
> cost/coordination burden; best reserved for Phase 2 expansion once
> conversions are proven.

# 7. Repeatable Method (How to Replicate in Any Region)

1\. Set centre point and radius (e.g., 4.2 km).

2\. Generate 1 km x 1 km tile grid intersecting the radius.

3\. Import/trace waterway centrelines (primary tributary + key feeders).

4\. Score each tile on: (a) distance-to-waterway, (b) corridor
connectivity (adjacent waterway tiles), (c) vegetation/remnant proxy,
(d) restoration leverage (edge + opportunity).

5\. Assign bands (1-5) and allocate sensors per band (e.g.,
18/10/8/8/6).

6\. Run Terra Free for 2-4 weeks to build uptake, then issue invitations
Wave 1 → 2 → 3 based on band priority.

7\. Calibrate effective acoustic radius in-field and adjust
spacing/overlap rules for the region.
