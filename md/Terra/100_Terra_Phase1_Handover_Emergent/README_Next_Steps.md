# README – Next Steps (Quick Reference)

**Last Updated:** 26 February 2026  
**Status:** P0 ✅ Complete | P1 ✅ Complete | P2 🟠 Active  
**No Blockers** ✅

---

## Current State

✅ **P0 NDVI Pipeline:** Complete and tested end-to-end
- GEE + Sentinel-2 data flowing
- RQ job processing asynchronously
- ndvi_trend persisted to Supabase 
- All acceptance criteria met

✅ **P1 Frontend NDVI:** Complete and deployed
- NDVICard component built (Expo web verified)
- 3 states working (pending/ready/error)
- Accessibility requirements met (16px+, light theme)
- Real-time updates via Supabase subscription
- Integrated into CompletionScreen

🟠 **P2 Stripe + DCDB:** Starting Feb 26
- Stripe: 30-day trial → $20/month subscription
- DCDB: Real cadastral data integration
- Timeline: Feb 26-27 execution

---

## Emergent's Next Action (Feb 26 Morning)

**P2 – Build Stripe + DCDB Integration**

1. Read: [Brad_P2_Stripe_and_DCDB_Spec.md](06_Build/Brad_P2_Stripe_and_DCDB_Spec.md) ← **START HERE**

2. Build P2a (Stripe):
   - 30-day free trial configuration (no card upfront)
   - $20/month subscription after trial
   - Webhook integration (subscription status changes)
   - Trial pause automation (Day 30 → paused state)
   - Error handling (payment failures, retries)

3. Build P2b (DCDB):
   - Real QLD cadastral data integration
   - Address search returns real DCDB results
   - Parcel boundaries display on map (GeoJSON)
   - Claim validation against real parcels only

4. P2 Acceptance Checklist (10 items):
   - ✅ Trial period working (30-day countdown)
   - ✅ Stripe checkout integration complete
   - ✅ Subscription status tracking (active/paused/cancelled)
   - ✅ Webhook handling all events
   - ✅ Trial pause automation (Day 30 triggers)
   - ✅ DCDB address search returns real data
   - ✅ Parcel boundaries display with correct colors
   - ✅ Claim validation against real parcels
   - ✅ E2E flow tested (trial → paid → cancelled)
   - ✅ No critical blockers  

---

## Emergent's Next Action (Feb 24 Morning)

**P0 – Test NDVI Pipeline End-to-End**

1. Read: [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](06_Build/Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md)

2. Execute:
   ```bash
   # Terminal 1: Start RQ worker
   rq worker default
   
   # Terminal 2: Create test claim
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

3. Verify (in Supabase):
   ```sql
   SELECT ndvi_status, ndvi_delta, ndvi_trend, ndvi_last_updated
   FROM property_claims
   WHERE parcel_id = 'test-parcel-456'
   ORDER BY created_at DESC LIMIT 1;
   ```
   Expected: Status transitions `pending → ready` within 15-20s

4. Confirm All 8 P0 Checkpoints ✅:
   - ✅ RQ worker starts
   - ✅ Test claim created
   - ✅ Status flow: pending → ready
   - ✅ All NDVI fields populated
   - ✅ Error handling works (test invalid coords)
   - ✅ Job can retry safely (idempotency)
   - ✅ No database corruption
   - ✅ Worker logs show clear execution

---

## Timeline

| Date | Task | Status | Duration |
|------|------|--------|----------|
| Feb 23-24 | P0 NDVI E2E Testing | ✅ **COMPLETE** | — |
| Feb 24-25 | P1 Frontend NDVICard | ✅ **COMPLETE** | — |
| **Feb 26-27** | **P2 Stripe + DCDB** | **🟠 START NOW** | **4-6 hours** |
| Feb 27 Evening | P2 Testing + Verification | 🟠 FINAL | 2-3 hours |
| **Feb 28 09:00** | **Go/No-Go Decision** | **Checkpoint** | **30 min** |
| Mar 1-3 | Soft launch (5 real users) | If Feb 28 ✅ | — |

---

## Key Documents (Read in This Order)

### P2 (Current Phase — Feb 26-27)
1. **[Brad_P2_Stripe_and_DCDB_Spec.md](06_Build/Brad_P2_Stripe_and_DCDB_Spec.md)** ← **START HERE** (detailed specs + testing checklist)

### P0 + P1 Reference (Completed)
- [Brad_P1_Frontend_Acceptance_Criteria.md](06_Build/Brad_P1_Frontend_Acceptance_Criteria.md) (P1 reference)
- [Brad_Feb24_Instructions_RQ_Integration.md](06_Build/Brad_Feb24_Instructions_RQ_Integration.md) (P0 reference)

### NDVI & Backend Technical Reference
- [02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md](06_Build/02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md) — Python code
- [02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md](06_Build/02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md) — SQL schema

### Daily Tracking
- [01_Property_Claim/Week_1_Progress_Tracking.md](06_Build/01_Property_Claim/Week_1_Progress_Tracking.md) — Update daily with P2 progress

---

## Status Update

**P0 (NDVI Backend Pipeline):** ✅ COMPLETE
- Python NDVI functions operational with real Sentinel-2 data
- RQ async job processing (5-15s per claim)
- All 10 NDVI columns + ndvi_trend persisted to Supabase
- All acceptance tests passing

**P1 (Frontend NDVICard):** ✅ COMPLETE  
- NDVICard component built and deployed
- 3 states working (pending/ready/error)
- 5s polling via useClaimNDVI hook
- Accessibility verified (16px+, light theme, contrast)
- Expo web build successful (860 modules)
- Integrated into CompletionScreen workflow

**P2 (Stripe + DCDB):** 🟠 ACTIVE (Feb 26-27)
- Stripe: 30-day trial → $20/month subscription
- DCDB: Real QLD cadastral data + boundary rendering
- Trial automation: Day 30 pause trigger
- Webhook handling: Subscription status changes
- Timeline: 4-6 hours both components (Feb 26-27)

---

## Success Criteria (Feb 28 Go/No-Go)

**For Feb 28 09:00 decision, MUST have:**

P0 ✓
- ✅ Claims persistent (database verified)
- ✅ NDVI pipeline working (RQ job complete, trend persisted)
- ✅ Real Sentinel-2 data flowing

P1 ✓
- ✅ NDVICard displaying (trial_active claims)
- ✅ All 3 states working (pending/ready/error)
- ✅ Accessibility met (16px+, light theme, contrast)
- ✅ Real-time updates via Supabase subscription
- ✅ Build compiles (Expo web verified)

P2 ✓
- ✅ Stripe trial working (30-day countdown)
- ✅ Subscription payment flow complete
- ✅ Trial pause automation (Day 30 → paused)
- ✅ Webhook handling (subscription status changes)
- ✅ DCDB real data integrated
- ✅ Parcel boundaries display on map
- ✅ Claim validation against real parcels

**Ready for soft launch** (5 real users Mar 1-3) if all P0/P1/P2 ✅

---

## Next: P2 Frontend

Start with: [Brad_P2_Stripe_and_DCDB_Spec.md](06_Build/Brad_P2_Stripe_and_DCDB_Spec.md)

**Good luck with P2!**
