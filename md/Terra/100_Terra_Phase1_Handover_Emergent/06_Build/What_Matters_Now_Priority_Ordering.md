# What Matters Now: Priority Ordering (Feb 22 – Mar 1)

**Status:** Claims persistence ✅ Infrastructure solid.  
**Next:** Three parallel workstreams with **clear ordering within each**.

---

## 1️⃣ Stripe Integration (This Week – Feb 22-24)

### The Goal
- User claims property → Trial starts immediately
- Stripe knows about it (customer created, subscription linked)
- Day 30 → Automatically paused (no card charged unless user resumes)
- Webhook updates `monitoring_state` in Supabase

### Dead Simple Requirements

#### What Must Happen
```sql
-- In property_claims table (already exists)
stripe_customer_id    → Store Stripe customer
stripe_subscription_id → Store subscription
monitoring_state      → trial_active | subscribed | paused | inactive
```

#### Stripe Flow
```
User claims property
  ↓
POST /api/claims
  ↓
Create Stripe customer (if new user)
  ↓
Create Stripe subscription (immediate 30-day trial)
  ↓
Store stripe_customer_id + stripe_subscription_id in property_claims
  ↓
monitoring_state = trial_active
  ↓
Webhook fires (daily or on Day 25)
  ↓
monitoring_state → subscribed (if paid) OR paused (if 30 days passed)
```

#### Stripe Config (Test Keys)
```
STRIPE_API_KEY=sk_test_...
STRIPE_PUBLIC_KEY=pk_test_...

Product: "Terra Satellite Monitoring"
Price:   $20/mo
Trial:   30 days
Card:    None (not required upfront)
```

#### Day 0-30 Logic
```
Day 0:  monitoring_state = trial_active     (badge on UI: "Trial active")
Day 1-24: monitoring_state = trial_active   (no change)
Day 25-29: monitoring_state = trial_active  (send reminder email if card missing)
Day 30: monitoring_state = paused           (trial expired, awaiting card)
        Stripe automatically paused subscription
        Icon fades on UI
Day 31+: monitoring_state = paused (unless user adds card → subscribed)
```

#### What NOT to Do
- ❌ Multiple pricing tiers
- ❌ Promo codes
- ❌ Custom billing cycles
- ❌ Auto-charge on Day 30 (pause instead, wait for user action)
- ❌ Complex webhook logic
- ❌ Payment failure retries

#### What MUST Work
- ✅ Stripe customer created on trial claim
- ✅ Subscription ID stored in Supabase
- ✅ Webhook updates monitoring_state correctly
- ✅ Day 30 pause works (simulate with test data)
- ✅ Test claim → Trial → Pause → Resume flow end-to-end

### Deliverable (By Thu 24 Feb EOD)
```
[ ] POST /api/claims creates Stripe customer + subscription
[ ] stripe_customer_id + stripe_subscription_id stored in DB
[ ] Webhook handler updates monitoring_state on subscription event
[ ] Day 30 pause automation (cron job or PostgreSQL trigger)
[ ] Test flow: Claim → Day 0 (trial_active) → Day 30 (paused) → Resume (subscribed)
[ ] 5 test claims with different monitoring_states active in DB
```

**Success:** You can manually test the full 30-day cycle with test Stripe keys + Supabase data.

---

## 2️⃣ Satellite Icon State (Start After Stripe Skeleton – Feb 25+)

### The Goal
Icon visibility = monitoring_state only. Not NDVI readiness. Not data freshness. Pure state.

### Icon States
```
inactive        → No icon (not claimed)
trial_active    → Blue satellite icon (32x32, opacity 1.0)
subscribed      → Green satellite icon (32x32, opacity 1.0)
paused          → Gray satellite icon (32x32, opacity 0.5)
```

### Frontend Logic
```typescript
// In PropertyClaimFlow or MapLayer component

function getIconState(monitoring_state: string) {
  switch(monitoring_state) {
    case 'inactive':
      return { visible: false };
    case 'trial_active':
      return { visible: true, color: '#3B82F6', opacity: 1.0 }; // Blue
    case 'subscribed':
      return { visible: true, color: '#10B981', opacity: 1.0 }; // Green
    case 'paused':
      return { visible: true, color: '#9CA3AF', opacity: 0.5 }; // Gray
  }
}
```

### Where Icon Appears
- Map layer (Leaflet, Mapbox, or Deck.gl)
- One icon per claimed property
- Positioned at parcel centroid
- Clickable → Shows parcel info + NDVI value (if available)

### What Does NOT Control Icon
- ❌ NDVI calculation status ("Refining data" state)
- ❌ Satellite data freshness (5 days old OK)
- ❌ Historical trend (baseline only)
- ❌ User preferences or filters

### Icon Wire-Up Checklist
```
[ ] Icon SVG component created (4 states: inactive, trial, subscribed, paused)
[ ] Map layer renders icons for all claimed properties
[ ] Icon state bound to monitoring_state from Supabase
[ ] Icon persists on page refresh (fetches claim state from DB)
[ ] Icon color changes when monitoring_state changes (e.g., Day 30 pause)
[ ] Icon clickable → Shows parcel details + satellite info
[ ] Icon positioning correct (on parcel boundary center)
[ ] Test: Claim property → Icon appears as blue → Day 30 passes → Icon turns gray ✓
```

**Success:** Open map → See blue/green icons for active claims, gray for paused, nothing for unclaimed.

---

## 3️⃣ Real NDVI Baseline (Parallel Track – Start Now, Complete by Feb 28)

### The Question You Must Answer First

**Do you currently have a live NDVI computation pipeline?**

**If YES:** Wire it to Supabase immediately. Store in property_claims:
```sql
ndvi_baseline       DECIMAL(5,2)           -- Single snapshot value
ndvi_last_updated   TIMESTAMP              -- When baseline was captured
ndvi_source         VARCHAR (e.g., "Sentinel-2 L2A")
ndvi_3year_delta    DECIMAL(5,2) NULLABLE  -- If 3-year history available
```

Then show on UI:
```
"NDVI Baseline: 0.62 (Sep 21, 2025)"
"3-Year Trend: +0.08"
"Source: Sentinel-2"
```

**If NO:** Here's the fastest realistic approach:

### NDVI Baseline Pipeline (If Starting Now)

#### Step 1: Get Satellite Imagery
**Tool:** Sentinel-2 (free, global, 10m resolution)  
**API:** [Sentinel Hub](https://www.sentinelhub.com/) or [Google Earth Engine](https://earthengine.google.com/)

**Simplest:** Google Earth Engine (Python client)
```python
import ee

def get_ndvi_baseline(latitude, longitude):
    # Point of interest (parcel centroid)
    poi = ee.Geometry.Point([longitude, latitude])
    
    # Latest Sentinel-2 image (within 30 days)
    image = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED") \
        .filterBounds(poi) \
        .filterDate("2025-01-21", "2025-02-21") \
        .sort("CLOUD_COVERAGE_ASSESSMENT") \
        .first()
    
    # NDVI = (NIR - Red) / (NIR + Red)
    ndvi = image.normalizedDifference(["B8", "B4"])
    
    # Get value at point
    value = ndvi.sample(poi, 10).first().get("normalized_difference").getInfo()
    
    return {
        "ndvi": value,
        "date": image.get("system:time_start").getInfo(),
        "source": "Sentinel-2 L2A"
    }
```

#### Step 2: Queue NDVI Jobs Asynchronously
When user claims property:
```
1. Store claim immediately (monitoring_state = trial_active)
2. Queue async job: "compute_ndvi_baseline(parcel_id)"
3. Frontend shows: "Refining data..." spinner
4. Job runs (5-10 seconds)
5. Store result: NDVI value + date in property_claims
6. Frontend updates: Shows NDVI value
```

#### Step 3: Store in Supabase
```sql
ALTER TABLE property_claims ADD COLUMN (
  ndvi_baseline       DECIMAL(5,2),
  ndvi_last_updated   TIMESTAMP,
  ndvi_source         VARCHAR(50) DEFAULT 'Sentinel-2 L2A',
  ndvi_processing     BOOLEAN DEFAULT FALSE
);
```

#### Step 4: Display on UI
```
When monitoring_state = trial_active or subscribed:

If ndvi_processing = TRUE:
  Show spinner: "Acquiring satellite data..."
  
If ndvi_processing = FALSE:
  Show card:
    "NDVI Baseline: 0.62"
    "Date: September 21, 2025"
    "Source: Sentinel-2 L2A"
    
  Optional (if available):
    "3-Year Trend: +0.08 ↑"
```

#### What You Do NOT Need
- ❌ Daily NDVI updates (baseline only)
- ❌ Seasonal smoothing algorithms
- ❌ Trend prediction models
- ❌ Multi-year time series
- ❌ Cloud-free composite (use best available)
- ❌ Sub-meter resolution (10m is fine)

#### What You DO Need
- ✅ Real NDVI value (not mock)
- ✅ Computation date
- ✅ Satellite source name
- ✅ Fast async job (< 10 seconds)
- ✅ Graceful display ("Refining data..." not broken)

### NDVI Pipeline Checklist
```
[ ] Google Earth Engine account + API key (or Sentinel Hub)
[ ] Python script: get_ndvi_baseline(lat, lon) works locally
[ ] Async job queue created (Bull.js, Celery, or PostgreSQL notify)
[ ] Backend endpoint: POST /api/claims/:id/compute-ndvi
[ ] On claim creation: Trigger async job immediately
[ ] Storage: NDVI values persist to property_claims
[ ] Frontend: Show "Refining data..." spinner, then NDVI card
[ ] Test: 5+ test properties → Real NDVI values in DB → Values visible on UI
```

**Success:** Claim property → 5-second wait → "NDVI: 0.62 (Sentinel-2, Sep 21)" appears

---

## 🎥 The March 1 Demo (What This Enables)

### The Journey
```
1. Open app
2. Type "123 Main St, Gondwana"
3. Confirm property boundary on map
4. Click "Start Satellite Monitoring"
5. Icon appears (blue/trial)
6. Below icon: "NDVI Baseline: 0.62"
7. Zoom out → See 3-5 icons for other test properties
8. All have real NDVI values displayed
```

### The Script
> "Here's your property. We're now monitoring it from space.
> 
> **[Point to icon]** That's your satellite icon. It's active because you're on a 30-day trial.
>
> **[Point to NDVI]** This is your baseline: NDVI 0.62, measured by Sentinel-2.
>
> **[Zoom out]** Here are your neighbors doing the same thing. All monitored from space.
>
> What you're looking at is regeneration visible from orbit. And that signal — that data — is what we're building financial markets around."

### Why This Works
- **Real:** Not a mock. Real satellite data. Real coordinates. Real baseline.
- **Immediate:** Icon exists the moment you claim. NDVI appears within 5 seconds.
- **Proof:** 3-5 properties with icons + data = proof of concept. Not 1 demo property.
- **Narrative:** "Regeneration visible from space" = why this matters.

---

## Priority If Everything Slips

**If you run out of time:**

1. **Keep Stripe 100% (no negotiation)**
   - Trial logic MUST work
   - Day 30 pause MUST work
   - Icons MUST follow monitoring_state

2. **Keep Icon State (60% minimum)**
   - trial_active → blue icon (required)
   - subscribed → green icon (required)
   - paused → gray icon (nice to have)
   - inactive → hide (required)

3. **NDVI can ship with mock data (temporarily)**
   - Looks real on UI
   - Says "NDVI: 0.62 (Mock Data, Sentinel-2 Sep 21)"
   - Real pipeline integrated Week 2

**But tell Brad immediately if NDVI slips.** It's the difference between "We're monitoring" and "We're monitoring FROM SPACE."

---

## Success Metrics (Feb 28 Evening)

```
STRIPE:
[ ] 5+ test claims in Supabase with stripe_customer_id
[ ] All have monitoring_state = trial_active
[ ] Webhook fires on subscription events
[ ] Day 30 pause successfully tested (manual time advance)
[ ] Resume from paused → subscribed works

ICON:
[ ] Blue icons render for trial_active claims
[ ] Green icons render for subscribed claims
[ ] Gray icons render for paused claims
[ ] Icons disappear for inactive claims
[ ] Icons persist on page refresh
[ ] Click icon → Shows parcel details

NDVI:
[ ] 5+ properties have real ndvi_baseline values in DB
[ ] Values show on UI with dates + source
[ ] Async job completes in < 10 seconds
[ ] "Refining data..." spinner shows during processing
[ ] If mock (approved fallback): Labeled as mock data
```

**Go/No-Go:** Need all Stripe + Icon checks. NDVI can be 80% real if needed.

---

## Decision Matrix

| If... | Then... |
|-------|---------|
| Stripe works + Icon works + Real NDVI works | ✅ March 1 demo is **production-ready** |
| Stripe works + Icon works + Mock NDVI | ✅ March 1 demo is **credible** — real plumbing visible |
| Stripe works + Icon 50% + NDVI anything | ❌ March 1 demo is **incomplete** — fix icon state first |
| Stripe incomplete | ❌ **STOP** — nothing else matters until Stripe works |

---

## Your March 1 Position

**What Brad will say to Coordinator:**

> "We have 5 real properties monitored. Real Stripe subscriptions running (in test). Real satellite data. Each property has an icon, each icon shows monitoring state, and we're pulling baseline NDVI from Sentinel-2.
>
> This isn't a wireframe. This isn't a prototype. This is working infrastructure.
>
> What can we do with real landholders now?"

**That's the conversation.**

Not architecture.  
Not roadmap.  
Not "coming soon."

Proof.

---

**Owner:** Emergent  
**Timeline:** Stripe (3 days) → Icon (1-2 days) → NDVI (3-4 days, parallel)  
**Risk:** NDVI slips if Sentinel-2 API flaky. Acceptable if Stripe + Icon ship on time.  
**Updated:** 21 Feb 21:45 UTC
