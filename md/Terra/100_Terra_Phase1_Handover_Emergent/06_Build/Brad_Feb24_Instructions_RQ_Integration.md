# Brad's Feb 24 Instructions — NDVI RQ Integration

**From:** Brad  
**To:** Emergent Team  
**Date:** 23 February 2026 (Evening)  
**Priority:** Execute Tomorrow (Feb 24)  

---

## Action Items for Feb 24

### 1. Supabase SQL Migration

**Execute the NDVI schema migration:**

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
  ADD COLUMN IF NOT EXISTS ndvi_trend VARCHAR(20)
    CHECK (ndvi_trend IN ('improving', 'stable', 'declining')),
  ADD COLUMN IF NOT EXISTS ndvi_last_updated TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS ndvi_error_message TEXT;
```

**Verify with:**
```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'property_claims'
  AND (column_name LIKE 'ndvi%' OR column_name LIKE 'date_%')
ORDER BY column_name;
```

**Provide output** of verification query confirming all columns present.

---

### 2. Python RQ Queue Setup (NO Bull.js change)

**Use Python RQ, stay in current stack:**
- Install: `pip install rq redis`
- Use Redis connection from your environment
- Job workers: `rq worker default`
- Job queueing: `Job.create(...)` in FastAPI endpoint handler

---

### 3. NDVI Job Triggering on Claim Creation

**Requirements:**

#### A. Set initial status
- When claim created: `ndvi_status = 'pending'`
- Do NOT wait for job completion
- Return claim response immediately (fire-and-forget)

#### B. Queue background job
- Call `compute_ndvi_job(property_id, latitude, longitude)` in RQ worker
- Job runs asynchronously in background
- No blocking on HTTP response

#### C. Calculate NDVI delta + trend

**Important:** ndvi_trend is ONLY calculated when the job completes successfully. It represents the system's snapshot decision at that moment. If threshold logic changes later, historical records remain unchanged.

```python
def compute_ndvi_job(property_id: int, latitude: float, longitude: float) -> dict:
    """
    Compute NDVI baseline + current, calculate delta and trend.
    On error: update ndvi_status to 'error' but leave ndvi_trend NULL.
    """
    baseline_result = get_ndvi_baseline(latitude, longitude)
    current_result = get_ndvi_current(latitude, longitude)
    
    if baseline_result.get('status') == 'error' or current_result.get('status') == 'error':
        # Job failed: don't calculate trend
        return {
            'property_id': property_id,
            'ndvi_status': 'error',
            'ndvi_last_updated': datetime.now().isoformat(),
            'ndvi_error_message': baseline_result.get('error') or current_result.get('error')
        }
    
    # Calculate delta
    ndvi_baseline = float(baseline_result['ndvi_baseline'])
    ndvi_current = float(current_result['ndvi_current'])
    ndvi_delta = round(Decimal(str(ndvi_current - ndvi_baseline)), 3)
    
    # Calculate trend (ONLY when successful)
    if ndvi_delta > 0.02:
        ndvi_trend = 'improving'
    elif ndvi_delta < -0.02:
        ndvi_trend = 'declining'
    else:
        ndvi_trend = 'stable'
    
    # Job completed successfully: return all values
    return {
        'property_id': property_id,
        'ndvi_baseline': baseline_result['ndvi_baseline'],
        'ndvi_current': current_result['ndvi_current'],
        'ndvi_delta': ndvi_delta,
        'ndvi_trend': ndvi_trend,  # Locked in at this moment
        'date_baseline_start': baseline_result['date_baseline_start'],
        'date_baseline_end': baseline_result['date_baseline_end'],
        'date_current_start': current_result['date_current_start'],
        'date_current_end': current_result['date_current_end'],
        'ndvi_status': 'ready',
        'ndvi_last_updated': datetime.now().isoformat(),
        'ndvi_error_message': None
    }
```

---

## Frontend Requirements

### 1. Pending Status Display

**While `ndvi_status = 'pending'` or `'processing'`:**

Show spinner with message:
```
"Downloading your site's vegetation health from satellite…"
```

- Position: Card/widget showing satellite icon area
- Duration: Display until status changes to 'ready' or 'error'
- No interaction blocking

### 2. Status Change Animation

**When status changes to `'ready'`:**

- Briefly animate satellite icon (e.g., pulse, rotate, scale)
- Duration: 500-800ms
- Purpose: Draw user attention to newly available data
- No blocking—show ready state immediately after

### 3. Trend Display

**Once ready, show:**

```
NDVI Trend: [Improving | Stable | Declining]
Delta: +0.071 (current - baseline)

Baseline: 0.6613 (Jan 21 - Apr 21, 2023)
Current:  0.7326 (Nov 21 - Feb 21, 2026)
```

**Color coding (optional for Feb 24):**
- Improving → Green
- Stable → Gray/Neutral
- Declining → Orange/Warning

---

## Architecture Notes

**Fire-and-Forget Pattern:**
1. User creates claim
2. HTTP endpoint sets `ndvi_status='pending'`, queues RQ job
3. Return 201 Created immediately
4. RQ worker processes job in background
5. Job updates `property_claims` with results
6. Frontend polls/subscribes to status changes (Supabase real-time)

**Why this matters:**
- Claim creation latency stays low (<500ms)
- NDVI computation can take 5-15s (no user wait)
- Better user experience (instant confirmation + async processing)

---

## Success Criteria (Feb 24 EOD)

✅ SQL schema deployed to Supabase (10 NDVI columns visible)  
✅ RQ job executes on claim creation  
✅ NDVI delta calculated and stored  
✅ NDVI trend calculated (improving/stable/declining)  
✅ Claim response returns immediately (no blocking)  
✅ Background job updates table asynchronously  
✅ Test: Create claim → wait 15s → verify NDVI values in database  
✅ Test: Frontend shows spinner while pending, data when ready  

---

## Timeline

- **Feb 24 Morning:** SQL schema + RQ setup (1-2 hours)
- **Feb 24 Afternoon:** Job implementation + integration (2-3 hours)
- **Feb 24 Evening:** E2E test + verification (1 hour)
- **Feb 24 EOD:** Ready for frontend animation + spinner (next)

---

## Notes

- **Reliability > Polish:** Get the async flow working correctly first
- **No visual polish yet:** Spinner text + basic animation only (Feb 25+)
- **Real-time updates:** Frontend should use Supabase subscription to listen for status changes
- **Error handling:** If job fails, set `ndvi_status='error'` + `ndvi_error_message`

---

## Reference

- **NDVI Code:** [02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md](02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md)
- **Schema:** [02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md](02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md)
- **Progress:** [01_Property_Claim_Flow/Week_1_Progress_Tracking.md](01_Property_Claim_Flow/Week_1_Progress_Tracking.md)

---

**Status:** Ready to execute (Feb 24)  
**Owner:** Emergent  
**Blocker:** None  
**Next:** Frontend animation + Supabase real-time subscription (Feb 25)
