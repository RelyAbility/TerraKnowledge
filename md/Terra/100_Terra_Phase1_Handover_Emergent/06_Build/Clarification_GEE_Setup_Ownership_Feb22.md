# Clarification: GEE Setup + Feb 22 Deliverable Confirmation

**Date:** 21 February 2026 (Evening)  
**Status:** Confirming workstreams and blockers

---

## Emergent's Feedback

> GitHub links aren't accessible yet (404). Based on your summary, here's my understanding:
>
> **Feb 22 Deliverables:**
> - GEE credentials working
> - 9 NDVI columns in Supabase
> - Python get_ndvi_baseline() function
> - Test with 5+ coordinates
>
> **GEE Setup - Your Action Required**
> 
> Before I can write the Python code, you need to complete the 8-step setup first.

---

## Brad's Response

### ✅ Confirmation: GEE Setup is Brad's Action

**Brad:** "You're correct. I will handle the GEE setup. You start with Supabase schema + Python function skeleton. When credentials are ready, you integrate."

**Timeline:**

| Item | Owner | Target Date | Status |
|------|-------|-------------|--------|
| GEE account setup (8 steps) | Brad | Feb 22 morning | 🔄 Starting now |
| Credentials passed to Emergent | Brad | Feb 22 noon | 🔄 Via env vars |
| Supabase schema update (9 columns) | Emergent | Feb 22 afternoon | ⏳ Ready to start |
| Python function skeleton + testing | Emergent | Feb 22 evening | ⏳ After credentials |

---

## Work Sequence (Not Parallel)

### Phase 1: Brad Sets Up GEE (Feb 22, Morning)

**Brad's tasks:**
1. Create GCP project ("Terra NDVI")
   - [GCP Console](https://console.cloud.google.com)
   - Create new project
   - Enable billing

2. Enable Earth Engine API
   - Search "Earth Engine API" in API library
   - Click **Enable**

3. Create service account
   - IAM & Admin → Service Accounts
   - Create: `terra-ndvi`
   - Create JSON key → Download

4. Grant GEE permissions
   - Earth Engine Console → Developer Settings
   - Add service account as Editor

5. Extract credentials
   - From JSON file:
     - `GEE_PROJECT_ID`
     - `GEE_SERVICE_ACCOUNT_EMAIL`
     - `GEE_PRIVATE_KEY`

6. Share with Emergent
   - Send 3 credential values via **secure channel** (not email)
   - Or store in **pod environment secrets** (preferred)

**Timeline:** Should complete by 10-11 AM Sydney time  
**Dependency:** Nothing (Brad's solo task)

---

### Phase 2: Emergent Starts Supabase + Function Skeleton (Feb 22, Parallel to Brad)

While Brad sets up GEE, Emergent can start:

**Emergent's parallel tasks (don't need credentials yet):**

1. **Supabase schema update** (30 mins, no dependencies)
   ```sql
   ALTER TABLE property_claims ADD COLUMN (
     ndvi_baseline DECIMAL(5,3),
     ndvi_current DECIMAL(5,3),
     ndvi_delta DECIMAL(5,3),
     baseline_window_start DATE,
     baseline_window_end DATE,
     current_window_start DATE,
     current_window_end DATE,
     ndvi_status VARCHAR(20) DEFAULT 'pending',
     ndvi_last_updated TIMESTAMP,
     ndvi_error_message TEXT
   );
   ```

2. **Python function skeleton** (30 mins, no credentials needed yet)
   ```python
   # backend/services/ndvi_service.py
   
   def get_ndvi_baseline(latitude: float, longitude: float) -> dict:
       """
       TODO: Query Sentinel-2 median NDVI via GEE.
       
       3-year baseline logic:
       - Current: Median NDVI from last 90 days
       - Baseline: Median NDVI from same 90-day window 3 years ago
       - Delta: current - baseline
       
       Returns: {
           'ndvi_baseline': float,
           'ndvi_current': float,
           'ndvi_delta': float,
           'baseline_window_start': str (DATE),
           'baseline_window_end': str (DATE),
           'current_window_start': str (DATE),
           'current_window_end': str (DATE),
           'status': 'ready' | 'error',
           'error': str or None
       }
       """
       # TODO: Implement when GEE credentials available
       pass
   ```

**Ready by:** 11-12 PM Sydney time (while Brad finishes credentials)

---

### Phase 3: Emergent Integrates Credentials + Tests (Feb 22, Afternoon)

Once Brad provides credentials:

1. Add to environment variables
   ```bash
   export GEE_PROJECT_ID="..."
   export GEE_SERVICE_ACCOUNT_EMAIL="..."
   export GEE_PRIVATE_KEY="..."
   ```

2. Implement `get_ndvi_baseline()` function (from [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md))

3. Test locally with 5+ Australian coordinates
   ```python
   test_coords = [
       (-40.1234, 149.5678),
       (-33.8688, 151.2093),
       (-37.8136, 144.9631),
       (-41.1448, 145.1099),
       (-32.5732, 152.6077)
   ]
   
   for lat, lon in test_coords:
       result = get_ndvi_baseline(lat, lon)
       print(f"{lat}, {lon}: {result}")
   ```

**Expected output:**
```
-40.1234, 149.5678: {'ndvi_baseline': 0.612, 'ndvi_current': 0.638, 'ndvi_delta': 0.026, ...}
-33.8688, 151.2093: {'ndvi_baseline': 0.548, 'ndvi_current': 0.621, 'ndvi_delta': 0.073, ...}
... (5 total)
```

**Ready by:** 5-6 PM Sydney time (EOD)

---

## GitHub Links (404 Explanation)

The documents exist in git history (commits c318b46, b426588, 88a62e5) but may not be visible on GitHub web UI yet due to:
- Refresh delay
- Branch not fully sync'd
- URL format difference

**Workaround:** Read directly from terminal

```bash
cd /Users/bradparsons/Documents/Code/TerraKnowledge

# View GEE setup guide
cat md/Terra/100_Terra_Phase1_Handover_Emergent/06_Build/NDVI_Architecture_Decision_GEE_Setup.md

# View implementation spec
cat md/Terra/100_Terra_Phase1_Handover_Emergent/06_Build/NDVI_Phase1_Implementation_Sentinel2_GEE.md

# View Emergent action items
cat md/Terra/100_Terra_Phase1_Handover_Emergent/06_Build/Emergent_NDVI_Ready_Feb22.md
```

All 3 documents are in the repo now.

---

## Feb 22 Confirmation Checklist

**Brad (GEE Setup):**
```
[ ] GCP project created ("Terra NDVI")
[ ] Earth Engine API enabled
[ ] Service account created (terra-ndvi)
[ ] JSON key downloaded (locally only, not committed)
[ ] GEE permissions granted (service account as Editor)
[ ] 3 credentials extracted (project_id, email, private_key)
[ ] Credentials ready to share
```

**Emergent (Supabase + Function):**
```
[ ] 9 NDVI columns added to property_claims table
[ ] Index created on ndvi_status
[ ] Python function skeleton created (get_ndvi_baseline)
[ ] Test coordinates prepared (5+ Australian locations)
```

**Daily Sync (EOD Feb 22):**
- Brad: "GEE credentials ready"
- Emergent: "Supabase schema + function skeleton done, ready for credentials"
- Brad: Share credentials (via secure method or pod secrets)
- Emergent: Integrate + test locally

**Confirm next morning (Feb 23):**
- Emergent: "get_ndvi_baseline() returns real NDVI values for 5+ test coordinates" ✓
- If yes → Proceed to RQ queue integration (Feb 23-24)
- If no → Debug + retry

---

## No Blocking Dependencies

**Key point:** Emergent does NOT need to wait for GEE credentials to start.

Emergent can:
1. ✅ Update Supabase schema (independent)
2. ✅ Create Python function skeleton (structure ready)
3. ✅ Prepare test coordinates (no auth needed)
4. ⏳ Implement GEE queries (when credentials available)
5. ⏳ Test with real data (when credentials available)

**Parallel work saves 2-3 hours.**

---

## Timeline Impact

**Original plan:** GEE setup → Python testing (sequential)  
**Actual execution:** GEE setup (Brad) + Supabase (Emergent) in parallel  

**Result:** Still finish Feb 22 EOD, with parallel work starting NOW.

---

## Brad's GEE Setup (Detailed Steps)

See: [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md)

**Steps 1-8 (8-step process):**
1. Create GCP project
2. Enable Earth Engine API
3. Create service account
4. Generate JSON key
5. Grant Earth Engine permissions
6. Extract credentials (3 values)
7. Set environment variables
8. Test Python connection

**Estimated time:** 30-45 minutes (mostly waiting for API enablement + permission propagation)

---

**Owner:** Brad (GEE) + Emergent (Supabase + function)  
**Status:** Ready to start now (both teams can work in parallel)  
**Next sync:** Feb 22 EOD (confirm checklist complete)  
**Blocker:** None (parallel execution)
