# Terra Phase 1 -- Ecological Performance Unit (EPU)

Definition & Location-Based User Experience

Instruction Set for Emergent

## 1. Definition -- Ecological Performance Unit (EPU)

An Ecological Performance Unit (EPU) is a fixed 1 km x 1 km grid cell
that aggregates ecological state and restoration opportunity metrics.\
\
An EPU is:\
• Spatially fixed\
• Uniform in size\
• Independent of property boundaries\
• Terra's core analytical unit\
\
An EPU is NOT:\
• A property boundary\
• A cadastral boundary\
• A regulatory zone\
• A user-defined area\
\
The EPU is Terra's foundational unit of ecological measurement.

## 2. Why 1 km?

• Ecologically meaningful at landscape scale\
• Large enough to model connectivity\
• Small enough to feel locally relevant\
• Tiles cleanly across regions\
• Scales internationally\
\
Each EPU must have a unique identifier: EPU_ID.

## 3. Data Stored Within Each EPU (Phase 1 Minimum)

Structural Ecology:\
• Dominant Regional Ecosystem (RE)\
• Secondary REs (\>20% area)\
• Percentage remnant vegetation\
• Total remnant hectares\
\
Landscape Context (4.2 km radius):\
• Neighbourhood remnant percentage\
• Connectivity differential\
\
Derived Metric:\
• Nature Opportunity Rating (0--100)\
\
Optional:\
• Biodiversity overlay flag (0/1)

## 4. Nature Opportunity Rating (Phase 1 Model)

Score =\
(1 - local_remnant_percent) \* 0.4\
+ (neighbourhood_remnant_percent - local_remnant_percent) \* 0.4\
+ biodiversity_overlay_flag \* 0.2\
\
Normalise to 0--100.\
\
Weights are hardcoded for Phase 1 but must be modular for future
adjustment.

## 5. Location-Based Mobile User Experience

On App Open:\
1. Request location permission.\
2. Retrieve GPS coordinates.\
3. Drop a subtle marker at location.\
4. Determine containing EPU and parcel (if available).\
5. Highlight EPU and display bottom summary panel.\
\
Bottom Panel Must Display:\
• EPU_ID\
• Dominant Ecosystem\
• Remnant Percentage\
• Nature Opportunity Rating\
\
If permission denied:\
• Show overview map\
• Prompt user to enter address, draw boundary, or enter Lot/Plan number.

## 6. Property vs EPU Logic

If user inputs property boundary:\
• Intersect property with EPUs.\
• Calculate weighted Nature Opportunity Rating.\
• Display how many EPUs the property spans.\
\
This reinforces landscape logic while remaining property-friendly.

## 7. Non-Scope (Phase 1)

Do NOT build:\
• Variable-sized EPUs\
• Species modelling\
• AI prediction layers\
• Carbon calculations\
• Time-series evolution\
• Compliance scoring\
\
Phase 1 EPU is a fixed, precomputed 1 km grid cell.

## 8. Performance Requirements

• All EPU metrics precomputed server-side.\
• Stored in database.\
• Frontend queries by EPU_ID only.\
• No heavy client-side spatial computation.\
• Map interaction must feel fast and responsive.

## 9. Success Criteria

Phase 1 is successful if:\
• User sees themselves inside an EPU immediately.\
• They understand their remnant percentage within 10 seconds.\
• They ask how to improve their Nature Opportunity Rating.\
\
Feedback from real users is the primary validation mechanism.
