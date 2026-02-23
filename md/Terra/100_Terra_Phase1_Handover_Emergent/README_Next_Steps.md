# README – Next Steps (Quick Reference)

**Last Updated:** 24 February 2026  
**Status:** P0 Complete ✅ → P1 Starting (Feb 24)  
**No Blockers** ✅

---

## Current State

✅ **GEE Setup:** Pod environment configured  
✅ **Python NDVI:** Tested with 5 real coordinates (real Sentinel-2 data)  
✅ **Database Schema:** 11 NDVI columns deployed to Supabase (with ndvi_trend)  
✅ **RQ Code:** Tested and working end-to-end  
✅ **P0 NDVI Pipeline:** COMPLETE — trend persisted to database ✅  
✅ **Frontend Spec:** Detailed acceptance criteria ready  

---

## Emergent's Next Action (Feb 24 Afternoon)

**P1 – Build NDVICard Frontend Component**

1. Read: [Brad_P1_Frontend_Acceptance_Criteria.md](06_Build/Brad_P1_Frontend_Acceptance_Criteria.md) ← **START HERE**

2. Build:
   - **NDVICard component** (display vegetation health)
   - **useClaimNDVI hook** (Supabase real-time subscription)
   - **Satellite icon animation** (pending & ready states)
   - **Satellite map icon** (visibility controls)

3. P1 Acceptance Checklist (10 items):
   - ✅ NDVICard displays for trial_active/subscribed claims
   - ✅ Shows all fields (trend + current + source + timestamp)
   - ✅ Pending state: calm satellite pulse + message
   - ✅ Ready state: data visible + brief animation
   - ✅ Error state: message + "Try again" button
   - ✅ Satellite icon on map (visible/faded/hidden rules)
   - ✅ 16px+ fonts, light theme, accessible
   - ✅ Real-time updates via Supabase subscription
   - ✅ Mobile responsive
   - ✅ E2E: create claim → spinner → data displays  

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
| Feb 24 (Now) | P0 NDVI E2E Testing | ✅ **COMPLETE** | — |
| **Feb 24 Afternoon** | **P1 Frontend NDVICard** | **🔴 START NOW** | **4-6 hours** |
| **Feb 25** | **P1 Testing + Polish** | **🔴 NEXT** | **2-4 hours** |
| Feb 26 | P1 Complete + P2 Stripe prep | Parallel | 4-6 hours |
| Feb 27 | P2 Stripe integration | After P1 ✅ | 4-6 hours |
| Feb 27+ | P2 DCDB replacement | After P1 ✅ | 3-4 hours |
| **Feb 28 09:00** | **Go/No-Go Decision** | **Checkpoint** | **30 min** |
| Mar 1-3 | Soft launch (5 real users) | If Feb 28 ✅ | — |

---

## Key Documents (Read in This Order)

### P1 Frontend (New — P0 Complete)
1. **[Brad_P1_Frontend_Acceptance_Criteria.md](06_Build/Brad_P1_Frontend_Acceptance_Criteria.md)** ← **START HERE** (detailed specs + testing checklist)
2. [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](06_Build/Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md) (additional design guidance)

### P0 Reference (Already Complete)
- [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](06_Build/Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md) (P0 sections for reference)
- [Brad_Feb24_Instructions_RQ_Integration.md](06_Build/Brad_Feb24_Instructions_RQ_Integration.md) (RQ job code reference)

### NDVI Technical Reference
- [02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md](06_Build/02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md) — Python code
- [02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md](06_Build/02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md) — SQL schema (with ndvi_trend)

### Daily Tracking
- [01_Property_Claim/Week_1_Progress_Tracking.md](06_Build/01_Property_Claim/Week_1_Progress_Tracking.md) — Update daily with P1 progress

---

## Status Update

**P0 (NDVI Backend Pipeline):** ✅ COMPLETE
- GEE credentials working
- Python NDVI functions tested with real Sentinel-2 data (5/5 pass)
- Supabase schema deployed (11 columns with ndvi_trend)
- RQ job tested end-to-end, trend persisting
- No blockers

**P1 (Frontend NDVICard):** 🔴 NOW — Start immediately
- Detailed acceptance criteria locked ✅
- Design specifications finalized ✅
- Real NDVI data available in Supabase ✅
- Ready to build

---

## Success Criteria (Feb 28 Go/No-Go)

**For Feb 28 09:00 decision, MUST have:**

P0 ✓
- ✅ Claims persistent
- ✅ NDVI pipeline working (RQ job complete)
- ✅ Trend persisting to database

P1 ✓
- ✅ NDVICard component displaying (trial_active claims)
- ✅ Pending animation working (calm satellite pulse)
- ✅ Ready state showing data + animation
- ✅ Error handling + "Try again" working
- ✅ Real-time updates (Supabase subscription)
- ✅ Satellite icon on map (visibility rules)
- ✅ Accessible (16px+ fonts, light theme, contrast)

P2 (Optional, not blocking)
- Stripe integration (if time permits)
- DCDB real data (if time permits)

---

## Next: P1 Frontend

Start with: [Brad_P1_Frontend_Acceptance_Criteria.md](06_Build/Brad_P1_Frontend_Acceptance_Criteria.md)

Open files, build components, test against checklist.

**Good luck!**
