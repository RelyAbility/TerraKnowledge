# Terra Phase 1 -- Property Claim System

Revenue-Ready Specification

Version 1.0

## 1. Purpose

Implement a frictionless, GPS-based property claim flow using cadastral
intersection. EPUs remain analytical units. Properties become identity
and billing units.

## 2. Architectural Separation

Identity Layer: Cadastral Property Polygon\
Analytical Layer: 1km EPU Grid\
Landscape Layer: 4.2km Aggregation Radius

## 3. Required Spatial Data

• State cadastral parcel polygons (Lot/Plan, geometry, area)\
• EPU grid table (geometry + precomputed TBLS)\
• Property--EPU intersection cache table

## 4. Claim Flow

Step 1: Capture GPS\
Step 2: Backend point-in-polygon query\
Step 3: Display boundary overlay for confirmation\
Step 4: Account creation (email + password)\
Step 5: Associate property with user\
Step 6: Compute weighted property score\
Step 7: Offer Monitoring Trial

## 5. Subscription Hook

Free: Static score + landscape context\
Monitor (\$10/month): NDVI trend, vegetation change detection, alerts,
quarterly updates

## 6. Network Integration

After claim, show corridor status and invite neighbour functionality.
Referral grants 1 free month if invitee subscribes.

## 7. Performance Requirements

• Parcel detection \<300ms\
• No client-side spatial math\
• All intersections cached server-side\
• No runtime crashes permitted

## 8. Phase 1 Non-Scope

• Legal ownership validation\
• Government registry integration\
• Complex parcel merging logic

## 9. Success Criteria

• Claim completed \<60 seconds\
• Property boundary visible\
• Subscription trial available immediately\
• Stable production build (no crashes)
