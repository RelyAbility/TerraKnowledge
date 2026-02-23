# P2 – Stripe Integration + DCDB Real Data Specification

**From:** Brad  
**To:** Emergent Team  
**Date:** 26 February 2026  
**Status:** P1 Complete ✅ → P2 Active 🟠  

---

## Overview

P1 (NDVI frontend) is complete. Now move to P2: monetization (Stripe) + real cadastral data (DCDB).

**Phase 1 "ready for soft launch" when both P2 items are complete.**

---

## P2a – Stripe Payment Integration

### Requirements

**Trial Period:**
- 30-day free trial (no card required upfront)
- Day 1: User claims property, monitoring_state = `trial_active`
- Day 30: Automatic pause (monitoring_state = `trial_active` → `paused`)
- User must add payment to continue

**Subscription Model:**
- $20 AUD/month (after trial ends)
- Recurring charge (subscription auto-renews)
- Cancel anytime (monitoring_state = `cancelled`)

**Stripe Configuration:**
- Product: "Terra Land Monitoring"
- Price: $20/month recurring
- Trial period: 30 days
- Free trial → auto pause (no graceful degradation)

---

### Acceptance Criteria

1. **Trial Flow Working:**
   - ✅ User claims property → monitoring_state = `trial_active`
   - ✅ Database tracks Day 1 (claim_created_at)
   - ✅ Day 30 automation triggers pause (monitoring_state → `paused`)
   - ✅ User sees "Trial ending soon" message on Day 25–30

2. **Stripe Checkout Integration:**
   - ✅ "Add payment" button on paused claim
   - ✅ Opens Stripe Checkout (redirect or modal)
   - ✅ User adds card, completes subscription
   - ✅ On success: monitoring_state = `subscribed`

3. **Webhook Handling:**
   - ✅ `customer.subscription.created` → set monitoring_state = `subscribed`
   - ✅ `customer.subscription.updated` (payment failed) → set monitoring_state = `past_due`
   - ✅ `customer.subscription.deleted` → set monitoring_state = `cancelled`
   - ✅ All webhook events logged for debugging

4. **Error States:**
   - ✅ Payment failed → show error + "Retry" button
   - ✅ Card declined → clear message + retry link
   - ✅ No network → graceful queue + retry

5. **E2E Test Flow:**
   - ✅ Create claim (trial_active)
   - ✅ Wait 30 seconds (simulate Day 30)
   - ✅ Verify monitoring_state = `paused`
   - ✅ Add payment → monitoring_state = `subscribed`
   - ✅ Cancel subscription → monitoring_state = `cancelled`

---

### Implementation Notes

**Environment Variables:**
```
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

**Database Schema (property_claims additions):**
```sql
ALTER TABLE property_claims
  ADD COLUMN IF NOT EXISTS stripe_customer_id VARCHAR,
  ADD COLUMN IF NOT EXISTS stripe_subscription_id VARCHAR,
  ADD COLUMN IF NOT EXISTS subscription_created_at TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS trial_expires_at TIMESTAMPTZ;
```

**Webhook Endpoint:**
- Path: `/api/webhooks/stripe`
- Method: POST
- Signature verification required: `stripe.webhooks.constructEvent()`

**Trial Expiration Check:**
```python
# In background job or API endpoint
from datetime import datetime, timedelta

def check_trial_expiration(property_id):
    claim = db.query(PropertyClaim).filter_by(id=property_id).first()
    if claim.monitoring_state != 'trial_active':
        return
    
    if datetime.now() >= claim.trial_expires_at:
        claim.monitoring_state = 'paused'
        db.commit()
```

---

## P2b – DCDB Real Cadastral Data Integration

### Current State
- Mock: `/api/cadastral/by-point` returns hardcoded parcel data
- Address search works, but no real geometry validation

### Target State
- Real: `/api/cadastral/by-point` queries Queensland cadastral database
- Real parcel boundaries render on map
- User can only claim actual parcels within DCDB

---

### DCDB Integration

**Data Source:** Queensland Spatial Catalogue (cadastral.qld.gov.au)

**Integration Options** (choose one):
1. **WFS (Web Feature Service)** — Real-time queries, live data
2. **Cached GeoJSON** — Download DCDB once, serve from local file
3. **Postgres PostGIS** — Import DCDB shapefile into database

**Recommendation:** Option 2 (Cached GeoJSON) for Phase 1
- Simplest implementation
- Sufficient for Gondwana corridor
- Can upgrade to Option 1 (WFS) post-launch

---

### Acceptance Criteria

1. **Real Address Search:**
   - ✅ User searches "123 Main St, Atherton QLD"
   - ✅ Query matches DCDB data (not mock)
   - ✅ Returns real parcel_id, boundary geometry, owner_name (if public)
   - ✅ Validates coordinates are within actual parcel

2. **Map Display:**
   - ✅ Parcel boundary renders on map (GeoJSON)
   - ✅ Polygon displays correctly (coordinates in lon/lat)
   - ✅ Color indicates state: trial_active=blue, subscribed=green, paused=gray
   - ✅ Zoom to parcel on claim

3. **Claim Validation:**
   - ✅ User can only claim parcels existing in DCDB
   - ✅ Attempt to claim outside DCDB → Error: "Property not found in cadastre"
   - ✅ Prevents arbitrary lat/lon claims (real data only)

4. **E2E Test Flow:**
   - ✅ Search real address in Gondwana corridor
   - ✅ Parcel geometry displays
   - ✅ Create claim → monitoring_state = `trial_active`
   - ✅ Map shows real boundary with correct color

---

### Implementation

**DCDB GeoJSON Source:**
```bash
# Download QLD cadastral data
# From: cadastral.qld.gov.au or spatial.information.qld.gov.au
# Filter to Gondwana corridor (lat -17.2 to -17.4, lon 142.7 to 142.9)
# Convert to GeoJSON
```

**Endpoint Implementation:**
```python
@app.get("/api/cadastral/by-point")
async def get_cadastral_by_point(latitude: float, longitude: float):
    # Load DCDB GeoJSON
    with open('data/dcdb_gondwana.geojson') as f:
        features = geojson.load(f)
    
    # Find parcel containing point
    point = (longitude, latitude)
    for feature in features['features']:
        polygon = shape(feature['geometry'])
        if polygon.contains(Point(point)):
            return {
                'parcel_id': feature['properties']['id'],
                'geometry': feature['geometry'],
                'owner': feature['properties'].get('owner'),
                'address': feature['properties'].get('address')
            }
    
    # No match
    raise HTTPException(status_code=404, detail="Property not found in cadastre")
```

**Frontend (Map Rendering):**
```javascript
// Display real parcel boundary
const parcelGeoJSON = {
  type: 'Feature',
  geometry: claim.cadastral_geometry,  // From DCDB
  properties: { parcel_id: claim.parcel_id }
};

map.addLayer({
  id: 'parcel-boundary',
  type: 'fill',
  source: { type: 'geojson', data: parcelGeoJSON },
  paint: {
    'fill-color': getColorByMonitoringState(claim.monitoring_state),
    'fill-opacity': 0.3
  }
});
```

---

## Testing Checklist (Emergent)

### Stripe Tests
- [ ] Create claim → monitoring_state = `trial_active`
- [ ] Check database: trial_expires_at = 30 days from now
- [ ] Mock time advance to Day 30 → verify pause automation triggers
- [ ] "Add payment" button appears on paused claim
- [ ] Stripe Checkout opens and processes payment
- [ ] On success: monitoring_state = `subscribed`
- [ ] Webhook events logged (check logs for `customer.subscription.created`)
- [ ] Cancel subscription → monitoring_state = `cancelled`

### DCDB Tests
- [ ] Search real address in Gondwana corridor
- [ ] Results match DCDB data (not mock)
- [ ] Parcel boundary displays on map
- [ ] Boundary color matches monitoring_state
- [ ] Create claim → parcel saved to database
- [ ] Attempt to claim outside DCDB → Error message
- [ ] Map zoom-to-parcel works correctly

### E2E Tests
- [ ] Full flow: search → claim → trial → Day 30 → paused → add payment → subscribed
- [ ] Real DCDB data throughout
- [ ] All states (trial/paused/subscribed) showing correct colors/messages

---

## Success Criteria (Phase 1 Ready)

✅ Stripe integration complete (trial + subscription working)  
✅ DCDB real data integrated (address search + map display)  
✅ Trial pause automation (Day 30 → paused state)  
✅ Webhook handling (subscription status changes)  
✅ All E2E flows tested and passing  
✅ Error handling graceful (no crashes on payment fail)  
✅ Ready for soft launch (5 real users Mar 1-3)  

---

## Timeline

- **Feb 26 Morning:** Stripe setup + admin + webhooks (2-3 hours)
- **Feb 26 Afternoon:** Stripe integration complete + testing (2-3 hours)
- **Feb 27 Morning:** DCDB source setup + geocoding (2 hours)
- **Feb 27 Afternoon:** DCDB integration + map display (2 hours)
- **Feb 27 Evening:** E2E testing + final verification (1-2 hours)

---

## Reference

- **P0 Testing:** [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md)
- **P1 Spec:** [Brad_P1_Frontend_Acceptance_Criteria.md](Brad_P1_Frontend_Acceptance_Criteria.md)
- **Progress:** [01_Property_Claim/Week_1_Progress_Tracking.md](01_Property_Claim/Week_1_Progress_Tracking.md)

---

**Status:** Ready for P2 execution (Feb 26)  
**Owner:** Emergent  
**Blockers:** None  
**Next:** Go/No-Go decision (Feb 28) → Soft launch (Mar 1)  

---

**Notes:**

- Keep Stripe test mode for manual testing (don't use real payments)
- DCDB can be cached GeoJSON initially (upgrade to WFS later if needed)
- Trial automation can be a background job or api endpoint check
- ngrok tunnel instability noted but doesn't block core functionality
- Backlog items (animation, vector tiles) deferred post-launch
