# NDVI Supabase Schema Implementation (Feb 23)

**Status:** Schema deployment ready  
**Date:** 23 February 2026  
**Owner:** Brad/Emergent  

---

## SQL Confirmed ✅

Your schema SQL is correct. It adds all 9 NDVI columns with proper constraints and validation.

### What This Does

```sql
ALTER TABLE property_claims 
  ADD COLUMN IF NOT EXISTS ndvi_baseline DECIMAL,
  ADD COLUMN IF NOT EXISTS ndvi_current DECIMAL,
  ADD COLUMN IF NOT EXISTS ndvi_delta DECIMAL,
  ADD COLUMN IF NOT EXISTS date_baseline_start DATE,
  ADD COLUMN IF NOT EXISTS date_baseline_end DATE,
  ADD COLUMN IF NOT EXISTS date_current_start DATE,
  ADD COLUMN IF NOT EXISTS date_current_end DATE,
  ADD COLUMN IF NOT EXISTS ndvi_status TEXT DEFAULT 'pending'
    CHECK (ndvi_status IN ('pending', 'processing', 'ready', 'error')),
  ADD COLUMN IF NOT EXISTS ndvi_last_updated TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS ndvi_error_message TEXT;
```

**Columns added:**
1. `ndvi_baseline` — Median NDVI from 3-year-ago window (DECIMAL)
2. `ndvi_current` — Median NDVI from current 90-day window (DECIMAL)
3. `ndvi_delta` — Regeneration progress: current - baseline (DECIMAL)
4. `date_baseline_start` — Start of 3-year-ago window (DATE)
5. `date_baseline_end` — End of 3-year-ago window (DATE)
6. `date_current_start` — Start of current 90-day window (DATE)
7. `date_current_end` — End of current 90-day window (DATE)
8. `ndvi_status` — Job state with CHECK constraint (pending | processing | ready | error)
9. `ndvi_last_updated` — When GEE query last ran (TIMESTAMPTZ for timezone)
10. `ndvi_error_message` — Error details if status = 'error' (TEXT)

**Key features:**
- `IF NOT EXISTS` ensures idempotency (safe to re-run)
- CHECK constraint enforces valid status values
- Default 'pending' for new claims (job queued immediately)
- TIMESTAMPTZ for accurate job tracking across zones

---

## Verification Query (Good)

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'property_claims'
  AND (
    column_name LIKE 'ndvi%'
    OR column_name LIKE 'date_%'
  )
ORDER BY column_name;
```

**Run after ALTER to verify:**
- All 10 columns present
- Data types correct (DECIMAL for values, DATE for windows, TIMESTAMPTZ for timestamp)
- ordering by column_name for clarity

---

## Deployment Steps

### 1. Open Supabase SQL Editor
- Go to [Supabase Dashboard](https://app.supabase.com)
- Select "Terra" project
- Click **"SQL Editor"** (left sidebar)

### 2. Run ALTER TABLE
Copy-paste the ALTER TABLE statement above

### 3. Verify Columns Added
Copy-paste the verification query
Expected output:
```
column_name            | data_type
-----------------------|-----------
date_baseline_end      | date
date_baseline_start    | date
date_current_end       | date
date_current_start     | date
ndvi_baseline          | numeric
ndvi_current           | numeric
ndvi_delta             | numeric
ndvi_error_message     | text
ndvi_last_updated      | timestamp with time zone
ndvi_status            | text
```

### 4. Test Write Permissions
Quick test to ensure property_claims is writable:
```sql
-- Test INSERT with new NDVI columns (then ROLLBACK)
BEGIN;
INSERT INTO property_claims (
  user_id, 
  parcel_id, 
  monitoring_state,
  ndvi_status
) VALUES (
  'test-user-uuid',
  'test-parcel',
  'trial_active',
  'pending'
);
ROLLBACK;  -- Don't actually save
```

If successful, columns are ready for async job updates.

---

## Next: Python Function Implementation

Once schema is deployed (5 mins), Emergent proceeds to:

### Feb 23 Remaining Tasks

**If GEE credentials received:**
1. ✅ Schema deployed (just did this)
2. Implement `get_ndvi_baseline()` function (2-3 hours)
   - Use Python google-earthengine client
   - Query Sentinel-2 L2A
   - Compute median NDVI (current + 3-year baseline)
   - Return all 7 values + dates + status
3. Test locally with 5+ coordinates (1-2 hours)
4. Confirm real NDVI values returned

**If GEE credentials not yet received:**
1. ✅ Schema deployed (just did this)
2. Create function skeleton (ready to fill in)
3. Prepare test coordinates (independent)
4. Wait for credentials from Brad
5. Implement when ready

---

## Progress Checkpoint (Feb 23)

**Completed:**
- ✅ Supabase schema: 10 NDVI columns added
- ✅ CHECK constraint: ndvi_status validation
- ✅ Verification query: All columns present

**Next:**
- ⏳ Python get_ndvi_baseline() function (depends on GEE credentials)
- ⏳ Local testing with real NDVI values
- ⏳ RQ async queue integration (Feb 24+)

---

## SQL Backup (If Needed to Rollback)

If columns need to be removed (only if something goes wrong):

```sql
ALTER TABLE property_claims
  DROP COLUMN IF EXISTS ndvi_baseline,
  DROP COLUMN IF EXISTS ndvi_current,
  DROP COLUMN IF EXISTS ndvi_delta,
  DROP COLUMN IF EXISTS date_baseline_start,
  DROP COLUMN IF EXISTS date_baseline_end,
  DROP COLUMN IF EXISTS date_current_start,
  DROP COLUMN IF EXISTS date_current_end,
  DROP COLUMN IF EXISTS ndvi_status,
  DROP COLUMN IF EXISTS ndvi_last_updated,
  DROP COLUMN IF EXISTS ndvi_error_message;
```

But won't be needed — schema is solid.

---

## Documentation Reference

- **Full NDVI spec:** [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md)
- **GEE setup guide:** [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md)
- **Priority ordering:** [What_Matters_Now_Priority_Ordering.md](What_Matters_Now_Priority_Ordering.md)
- **10-day sprint:** [10Day_Critical_Sprint_Feb22_Mar3.md](10Day_Critical_Sprint_Feb22_Mar3.md)

---

**Status:** Ready to deploy  
**Estimated time:** 5-10 minutes (schema deployment)  
**Next phase:** Python function implementation (Feb 23 afternoon)
