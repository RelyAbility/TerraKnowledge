**Phase 1 Corridor‑First Deployment Structure (50 Sensors)**

**Centrepoint (Gondwana pin):** -27.490422615161563, 152.91568918292165

**Footprint:** 4.2 km radius circular pilot boundary; planning grid is 1
km × 1 km tiles indexed from the centre tile (0,0).

**Seed nodes locked for Phase 1:**

• Gondwana (PUC-17057): -27.490423, 152.915689 \| offsets ≈ (+0.00 km E,
+0.00 km N) \| tile T_0_0

• Andrew.grace.1981: -27.468685, 152.904706 \| offsets ≈ (-1.08 km E,
+2.42 km N) \| tile T_W1_N2

**Corridor‑first deployment bands (allocation totals)**

  -----------------------------------------------------------------------
  Band                    Intent                  Sensors
  ----------------------- ----------------------- -----------------------
  Band 1 -- Gold Creek    Primary tributary       18
  spine                   corridor backbone       
                          (movement + riparian    
                          response).              

  Band 2 -- Feeder        Headwater/feeder        10
  micro-catchments        corridors including     
                          Gondwana watercourse;   
                          links toward Gold       
                          Creek.                  

  Band 3 -- Riparian      Edge/transition zones   8
  buffer halo             adjacent to corridors;  
                          uplift measurement      
                          belt.                   

  Band 4 -- Ridge &       Ridge/stepping‑stone    8
  upland connectors       connectors between      
                          drainage lines (lateral 
                          connectivity).          

  Band 5 -- Control &     Controls + mixed‑use    6
  interior mosaic         interior to keep        
                          inference defensible.   
  -----------------------------------------------------------------------

**Wave sequence (Terra → Founding 50)**

Stage 1 (2--4 weeks): Terra Free to all parcels in footprint; show
river/tributary context + registered‑properties overlay; no Founding 50
scarcity.

Stage 2: Launch Founding 50 invites only to Wave 1 tiles (Band 1).

Stage 3: Expand to Wave 2 (Bands 2--3), then Wave 3 (Bands 4--5) as
corridor density builds.

**Acoustic overlap planning (parameterised)**

Use this as a tunable planning parameter (R) rather than a fixed
assumption:

• R = effective detection radius (m) for target taxa + habitat + mic
placement.

• Along corridors: target overlap of \~30--50% (spacing ≈ 1.0--1.4 × R).

• In interior/control: target overlap of \~0--20% (spacing ≈ 1.6--2.5 ×
R).

Calibrate R after first 2--3 deployments using empirical detection
density and false‑positive rates.

**30 vs 50 vs 70 sensor scenarios (rule‑of‑thumb)**

• 30 sensors: corridor sampling only; weaker interior inference; best
for proof‑of‑value.

• 50 sensors (selected): corridor + feeder + connectors + controls;
balanced defensibility and practicality.

• 70 sensors: higher corridor granularity + replicated controls;
stronger statistical power for uplift attribution.

**Tile allocation file**

A machine‑readable tile plan (centroids, band, wave, sensors per tile,
seed nodes) is provided as CSV.
