# Brad's Response: P0 NDVI E2E Testing + P1 Frontend Design — Feb 24 Evening

**From:** Brad  
**To:** Emergent Team  
**Date:** 23-24 February 2026  
**Status:** RQ code written, needs testing — PROCEED WITH P0  

---

## Priority Summary

**P0 (Now):** Test NDVI pipeline end-to-end  
**P1 (After P0 ✅):** Frontend NDVI with accessible design  
**P2 (After P1 ✅):** Stripe integration  
**P3 (Backlog):** DCDB replacement, animation refinement  

---

## P0 – NDVI Pipeline E2E Testing (PRIORITY NOW)

### Test Sequence

**1. Start RQ Worker**
```bash
# Terminal 1: Start the worker
rq worker default
```

**2. Create Test Claim via API**
```bash
# Terminal 2: Create claim
curl -X POST http://localhost:8000/api/claims \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "test-user-123",
    "parcel_id": "test-parcel-456",
    "latitude": -36.7465,
    "longitude": 142.8862,
    "monitoring_state": "trial_active"
  }'
```

**Expected response:**
```json
{
  "id": "claim-uuid",
  "parcel_id": "test-parcel-456",
  "ndvi_status": "pending",
  "created_at": "2026-02-24T..."
}
```

**3. Monitor Worker Logs**

Watch Terminal 1 for output like:
```
Job started: compute_ndvi_job
  - property_id: <uuid>
  - latitude: -36.7465
  - longitude: 142.8862
Job completed: NDVI result returned
```

**Expect wait time:** 5-15 seconds (GEE query + Sentinel-2 processing)

### 4. Verify Status Transitions

Check database for status flow:

```sql
SELECT id, ndvi_status, ndvi_delta, ndvi_trend, ndvi_last_updated, ndvi_error_message
FROM property_claims
WHERE parcel_id = 'test-parcel-456'
ORDER BY created_at DESC
LIMIT 1;
```

**Expected output sequence:**
```
1st query (immediately after create):  ndvi_status = 'pending'
2nd query (after 5-15s):                ndvi_status = 'ready'
                                         ndvi_delta = 0.033 (e.g.)
                                         ndvi_trend = 'improving'
                                         ndvi_baseline = 0.840
                                         ndvi_current = 0.872
                                         date_baseline_start = '2023-02-21'
                                         date_baseline_end = '2023-05-21'
                                         date_current_start = '2025-11-21'
                                         date_current_end = '2026-02-21'
                                         ndvi_last_updated = '2026-02-24T...'
```

### 5. Error Handling Test

Simulate error condition to verify error handling:

```bash
# Create claim with invalid coordinates (ocean/no data)
curl -X POST http://localhost:8000/api/claims \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "test-user-error",
    "parcel_id": "test-parcel-error",
    "latitude": -90.0,
    "longitude": 0.0
  }'
```

**Expected:**
```sql
-- Query after 15-20s
SELECT ndvi_status, ndvi_error_message
FROM property_claims
WHERE parcel_id = 'test-parcel-error';

-- Should show:
ndvi_status = 'error'
ndvi_error_message = 'No Sentinel-2 imagery available for coordinates...' (or similar)
```

### 6. Idempotency & Retry Verification

**Verify job can safely re-run:**

```bash
# Manually re-queue a job (simulate retry)
# Your RQ client code or admin panel should allow this

# Expected: Status changes back to 'pending', then 'ready' with updated values
```

**Confirm:** No duplicate records, no broken state.

---

## Success Criteria (P0 Must-Have)

- ✅ RQ worker starts without error
- ✅ Test claim created successfully
- ✅ `ndvi_status` transitions: `pending` → `ready` (within 20s)
- ✅ All NDVI fields populated (baseline, current, delta, trend, date windows, timestamp)
- ✅ Error handling works (invalid coordinates → ndvi_status='error' + message)
- ✅ Job can safely retry (idempotency)
- ✅ No database corruption on re-runs
- ✅ Worker logs show clear job execution flow

**Once all ✅, P0 is complete. Ready to move to P1 Frontend.**

---

## P1 – Frontend NDVI Display (Design Guidance)

### Target Audience & Accessibility

**Demographics:** 60+ year-old landholders  
**Priorities:** Clear, simple, trustworthy, accessible  

### Design Principles

- **Light theme** (white/cream background)
- **Large fonts:** 16px minimum for body text, 24px+ for headers
- **Generous spacing:** 20-30px gaps between sections
- **High contrast:** Dark text on light (WCAG AA minimum)
- **No jargon:** Replace technical terms with plain language
- **Slow animations:** 800-1500ms (large audience may need time to track movement)

### Component Layout

**Card Title:**
```
🛰️ Vegetation Health (from Satellite)
```
*(Not "NDVI score" or "biomass metric")*

**While Pending (`ndvi_status = 'pending'` or `'processing'`):**

```
┌─────────────────────────────────────────┐
│ 🛰️ Vegetation Health (from Satellite)   │
├─────────────────────────────────────────┤
│                                         │
│   ⚪ (slow pulsing satellite icon)      │
│                                         │
│   "Analysing satellite imagery for    │
│    your site... (takes a minute)"     │
│                                         │
│   [Progress bar - optional, 0-100%]   │
│                                         │
└─────────────────────────────────────────┘
```

**Satellite Animation (Pending):**
- Icon: Glow/pulse effect (not spinning, not flashing)
- Duration: 1200-1500ms cycle
- Opacity: 100% → 40% → 100%
- Calm, "listening for signal" vibe

**When Ready (`ndvi_status = 'ready'`):**

```
┌──────────────────────────────────────────────┐
│ 🛰️  Vegetation Health (from Satellite)        │
├──────────────────────────────────────────────┤
│                                              │
│  Status:   ✅ Improving                      │
│            (or "Stable", "Declining")        │
│                                              │
│  Change:   +0.071  (3-year comparison)       │
│            ▲ 7.1% increase in vegetation     │
│                                              │
│  Details:                                    │
│  • Current: 0.733                            │
│  • 3 years ago: 0.662                        │
│  • Source: Sentinel-2 satellite              │
│  • Last checked: 24 Feb 2026, 14:32 UTC     │
│                                              │
│  Date range:                                 │
│  • Current reading: Nov 21 - Feb 21, 2026   │
│  • 3-year baseline: Dec 18, 2022 - Mar 18, │
│    2022                                      │
│                                              │
└──────────────────────────────────────────────┘
```

**Satellite Icon Animation (Status Ready):**
- Brief pulse/glow when status changes from pending → ready
- Duration: 600-800ms (one pulse)
- Effect: Subtle highlight, draws attention without overwhelming
- Then: Return to normal (no continuous animation)

**Trend Label & Color Coding (Optional for Feb 24, Nice-to-Have):**
| Trend | Label | Emoji | Color | Meaning |
|-------|-------|-------|-------|---------|
| Improving | ✅ Improving | ▲ | Green (#2E7D32) | Vegetation recovering (good for Conservation story) |
| Stable | ➖ Stable | — | Gray (#757575) | No significant change |
| Declining | ⚠️ Declining | ▼ | Orange/Amber (#F57C00) | Vegetation loss (alert for action) |

### Keys to Simplicity

1. **Plain Language:**
   - Not: "NDVI delta coefficient 0.071"
   - Say: "7.1% increase in vegetation"

2. **Show Trust:**
   - "Last updated: 24 Feb 2026, 14:32 UTC"
   - "Source: Sentinel-2 satellite (European Space Agency)"
   - Date ranges prove satellite data is real and recent

3. **Avoid Jargon:**
   - "3-year comparison" not "3-year offset baseline"
   - "Vegetation health" not "normalized difference vegetation index"
   - "Satellite imagery" not "multispectral radiometric data"

4. **Generous White Space:**
   - Cards 320px wide minimum on mobile
   - Text 16px+, line-height 1.6+
   - Padding 20-30px inside cards
   - Gap 20px between sections

5. **Loading States:**
   - Spinner: Calm pulsing satellite icon (not spinner wheel)
   - Message: "Analysing satellite imagery for your site... (takes a minute)"
   - Never show: "Wait forever" or vague messages

---

## P1 Implementation Checklist

- ✅ Create `NDVICard` component (displays all fields above)
- ✅ Create `useClaimNDVI` hook (Supabase real-time subscription to ndvi_status)
- ✅ Implement pending spinner (satellite pulse animation + text)
- ✅ Implement ready state (all fields visible, pulse animation on status change)
- ✅ Implement error state (show ndvi_error_message, retry option)
- ✅ Accessible fonts/colors (WCAG AA minimum, test with older devices)
- ✅ Responsive design (works on phone + tablet + desktop)
- ✅ Test with actual NDVI data (feed from P0 test claims)

---

## Next Steps After P1 Complete

Once P1 testing is stable (48-72 hours):

→ **P2 Stripe Integration** (30-day free trial, $20/month, Day 30 pause automation)  
→ **P2 DCDB Replacement** (connect address search to real cadastral data)

---

## File References

- **P0 NDVI Job Code:** [02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md](02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md)
- **RQ Integration:** [Brad_Feb24_Instructions_RQ_Integration.md](Brad_Feb24_Instructions_RQ_Integration.md)
- **Progress Tracking:** [01_Property_Claim/Week_1_Progress_Tracking.md](01_Property_Claim/Week_1_Progress_Tracking.md)

---

**Status:** P0 ready to test (RQ code written)  
**Owner:** Emergent  
**Timeline:** P0 (Feb 24-25), P1 (Feb 25-26), P2 (Feb 27+)  
**Go/No-Go:** Feb 28 09:00 (checkpoint)
