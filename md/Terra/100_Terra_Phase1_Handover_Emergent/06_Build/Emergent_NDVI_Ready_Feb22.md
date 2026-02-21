# Emergent: NDVI Implementation - Ready to Start Feb 22

**Status:** All clarifications resolved. Architecture confirmed. Ready to begin.

---

## Your Response from Brad

Brad's feedback on your clarification questions:

### ✅ Architecture Decision
**Brad:** "Let's stay in Python/FastAPI — use a Python-native queue (Celery+Redis or RQ+Redis). **Recommendation:** Pick whichever is fastest/simplest for you."

**Your choice:** Go with **RQ + Redis** for Phase 1 (simpler than Celery, easier to debug)

---

### ✅ GEE Credentials
**Brad:** "I don't have the service account set up yet. Send me setup instructions."

**You:** Complete GEE setup guide is ready → [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md)

**8-step process:**
1. Create GCP project
2. Enable Earth Engine API
3. Create service account
4. Generate JSON key
5. Grant Editor role in GEE
6. Extract credentials (3 values)
7. Set environment variables
8. Test Python connection

---

### ✅ Scope Confirmed
Brad confirmed all 5 items:
1. ✅ Add 8 NDVI columns to `property_claims` table
2. ✅ Python function to query Sentinel-2 via GEE
3. ✅ Async job queue triggered on claim creation
4. ✅ Frontend hook + NDVICard component with real-time updates
5. ✅ Integration with PropertyClaimFlow

---

### ✅ Logic Correction: 3-Year Baseline
**Brad's correction (important!):**
> "Current = median NDVI over the last ~90 days"
> "Baseline = median NDVI over the same window 3 years ago"
> "Delta = current − baseline"
> "Store the date windows used so we can display 'Sentinel-2 (window vs window)'"

**What this means:**
```
3 years ago (Dec 18, 2021 - Mar 18, 2022):  NDVI median = 0.625
Now (Nov 21, 2025 - Feb 21, 2026):         NDVI median = 0.642
Change: +0.017 (2.7% improvement) ↑
```

**Display to user:**
```
"Your regeneration from space"

Current NDVI: 0.642 (Nov 21, 2025 - Feb 21, 2026)
3 years ago: 0.625 (Dec 18, 2021 - Mar 18, 2022)

Result: +0.017 improvement ↑

Source: Sentinel-2 satellite imagery
```

---

## Feb 22 Action Items (Start Today)

### Morning (2-3 hours)

**Task 1: Set Up Google Earth Engine**

Follow [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md), Steps 1-8:
- Create GCP project
- Enable Earth Engine API
- Create service account
- Generate JSON key
- Grant permissions
- Extract credentials
- Set env vars
- Test Python connection

**Confirm when done:** 
```
✅ GEE account created
✅ Service account with Editor role in GEE
✅ JSON key downloaded (locally only, never commit)
✅ Python test script returns real NDVI values
✅ Env vars set: GEE_PROJECT_ID, GEE_SERVICE_ACCOUNT_EMAIL, GEE_PRIVATE_KEY
```

---

### Afternoon (2-3 hours)

**Task 2: Update Supabase Schema**

Add 9 columns to `property_claims` table:

```sql
ALTER TABLE property_claims ADD COLUMN (
  ndvi_baseline          DECIMAL(5,3),
  ndvi_current           DECIMAL(5,3),
  ndvi_delta             DECIMAL(5,3),
  baseline_window_start  DATE,
  baseline_window_end    DATE,
  current_window_start   DATE,
  current_window_end     DATE,
  ndvi_status            VARCHAR(20) DEFAULT 'pending',
  ndvi_last_updated      TIMESTAMP,
  ndvi_error_message     TEXT
);

CREATE INDEX idx_property_claims_ndvi_status 
  ON property_claims(ndvi_status) 
  WHERE ndvi_status = 'pending';
```

**Confirm when done:**
```
✅ 9 new columns added to property_claims
✅ Index created for ndvi_status lookups
✅ Old test data cleaned if needed
```

---

**Task 3: Test Python NDVI Query Script**

Create `backend/services/ndvi_service.py` with the `get_ndvi_baseline()` function from [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md).

Test locally with 5 Australian coordinates:

```python
from services.ndvi_service import get_ndvi_baseline

test_coords = [
    (-40.1234, 149.5678),   # NSW example
    (-33.8688, 151.2093),   # Sydney area
    (-37.8136, 144.9631),   # Melbourne area
    (-41.1448, 145.1099),   # Croajingolong, VIC
    (-32.5732, 152.6077)    # Coffs Harbour, NSW
]

for lat, lon in test_coords:
    result = get_ndvi_baseline(lat, lon)
    baseline = result['ndvi_baseline']
    current = result['ndvi_current']
    delta = result['ndvi_delta']
    print(f"{lat}, {lon}: Baseline={baseline} → Current={current} (Δ={delta})")
```

**Expected output:**
```
-40.1234, 149.5678: Baseline=0.612 → Current=0.638 (Δ=0.026)
-33.8688, 151.2093: Baseline=0.548 → Current=0.621 (Δ=0.073)
... (5 total)
```

**Confirm when done:**
```
✅ Python script installed (python -c "import ee; print(ee.__version__)")
✅ get_ndvi_baseline() function tested locally
✅ 5+ test coordinates returning real NDVI values
✅ Date windows calculated correctly (3-year offset)
✅ All values in valid NDVI range (0.0 to 1.0)
```

---

### End of Day (Feb 22) Checklist

```
[ ] GEE credentials set up + tested
[ ] Supabase schema updated (9 new columns + index)
[ ] Python NDVI script tested (5+ real values)
[ ] Confirm with Brad: "Feb 22 tasks complete"
```

---

## Next Steps (Feb 23+)

- **Feb 23:** RQ queue setup + FastAPI endpoint wiring + job triggering
- **Feb 24:** Test async NDVI job (claim → job queued → result stored)
- **Feb 25:** Frontend hook + NDVICard component
- **Feb 26:** Integration with PropertyClaimFlow + real-time updates
- **Feb 27-28:** End-to-end testing + error handling + demo prep

---

## Documentation Map

**Core Implementation:**
1. [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md) — GEE setup + Feb 22 start
2. [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md) — Full implementation spec (updated for Python + 3-year logic)

**Strategic Context:**
3. [What_Matters_Now_Priority_Ordering.md](What_Matters_Now_Priority_Ordering.md) — Why NDVI matters (after Stripe)
4. [10Day_Critical_Sprint_Feb22_Mar3.md](10Day_Critical_Sprint_Feb22_Mar3.md) — Overall timeline + dependencies

---

## Key Details (Quick Reference)

### Database
- Table: `property_claims`
- New columns: 9 (baseline, current, delta, 4 date windows, status, updated, error)
- Index: Async queue lookups by `ndvi_status = 'pending'`

### Backend
- Language: Python/FastAPI (stay in current stack)
- Async queue: RQ + Redis (simpler than Celery)
- GEE client: `google-earthengine` Python library
- Job: `compute_ndvi_job(claim_id, latitude, longitude)`
- Trigger: When user claims property (`POST /api/claims`)

### Frontend
- Hook: `useClaimNDVI(claimId)` with real-time Supabase updates
- Component: `NDVICard` shows pending/ready/error states
- Display: "NDVI: 0.642 (Nov 21 - Feb 21) vs 0.625 (Dec 18, 2021 - Mar 18, 2022)"
- Integration: In PropertyClaimFlow below SatelliteIcon

### Credentials
- Never hardcoded
- All via environment variables (GEE_PROJECT_ID, GEE_SERVICE_ACCOUNT_EMAIL, GEE_PRIVATE_KEY)
- .env not committed
- Production uses Secrets Manager

### Timeline
- Feb 22: Auth + schema + Python script tested
- Feb 23-24: Queue + endpoint wiring
- Feb 25-26: Frontend integration
- Feb 27-28: Testing + demo prep
- Mar 1: Show coordinator real NDVI data

---

## Questions Before You Start?

**Ask in the same format as your clarifications:**
1. Clear question
2. Options if applicable
3. What you need to proceed

Brad responds within hours during Sydney business hours.

---

**Owner:** Emergent  
**Start Date:** Feb 22, morning  
**First Sync:** Feb 22, EOD (confirm checklist complete)  
**Next Review:** Feb 23 morning (queue wiring)
