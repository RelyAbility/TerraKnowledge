# NDVI Architecture Decision + GEE Setup Instructions

**Date:** 21 February 2026 (Evening)  
**Status:** Architecture confirmed. GEE setup in progress.

---

## Architecture Decision: Python-Native (Stay FastAPI)

**Brad's directive:**
> "Let's stay in Python/FastAPI for Phase 1 — please do NOT add a Node/Bull.js service. Use a Python-native queue (Celery+Redis or RQ+Redis)."

### Decision: Use **Celery + Redis** (or RQ + Redis)

**Why:**
- No stack fragmentation (Python → Python async jobs)
- Simpler deployment (no separate Node service)
- Mature, battle-tested (Celery especially)
- Idempotent job execution (retry-safe)
- Supabase webhook compatible

**Stack:**
```
FastAPI (sync endpoints)
  ↓
Celery Worker (async NDVI job)
  ↓
Redis (job queue)
  ↓
Python GEE Client (Sentinel-2 query)
  ↓
Supabase (store results)
```

**Celery vs RQ:**
- **Celery:** More features, overkill for one job type, heavier
- **RQ:** Simpler, faster, perfect for single-queue use case (NDVI job)

**Recommendation:** Start with **RQ** (simpler). If needs grow, migrate to Celery later.

---

## Logic Correction: 3-Year NDVI Baseline

**Brad's logic override:**
> "Current = median NDVI over the last ~90 days (or a defined seasonal window)"
> "Baseline = median NDVI over the same window 3 years ago"
> "Delta = current − baseline"
> "Store the date windows used so we can display 'Sentinel-2 (window vs window)'"

### Updated NDVI Computation

Instead of:
```
Baseline = Sentinel-2 best image from 365 days back
Current = Sentinel-2 best image from 30 days back
```

Now:
```
Baseline Window: Same 90-day period 3 years ago
  Example: Feb 21 → 0 years = Dec 21 - Mar 21 window (2025)
           Feb 21 → -3 years = Dec 18 - Mar 18 window (2021)
  Result: Median NDVI across all Sentinel-2 images in Dec-Mar 2021 window

Current Window: Last 90 days
  Example: Feb 21 = Nov 21 - Feb 21 (2025)
  Result: Median NDVI across all Sentinel-2 images in Nov-Feb 2025 window

Delta: current_median - baseline_median
```

### Database Schema Update

```sql
ALTER TABLE property_claims ADD COLUMN (
  ndvi_baseline          DECIMAL(5,3),           -- Median NDVI baseline period
  ndvi_current           DECIMAL(5,3),           -- Median NDVI current period
  ndvi_delta             DECIMAL(5,3),           -- current - baseline
  
  -- DATE WINDOWS (for credibility display)
  baseline_window_start  DATE,                   -- Start of 90-day window 3 years ago
  baseline_window_end    DATE,                   -- End of 90-day window 3 years ago
  current_window_start   DATE,                   -- Start of current 90-day window
  current_window_end     DATE,                   -- End of current 90-day window
  
  ndvi_status            VARCHAR(20),            -- 'pending' | 'ready' | 'error'
  ndvi_last_updated      TIMESTAMP,
  ndvi_error_message     TEXT NULLABLE
);

-- Index for async job queue
CREATE INDEX idx_property_claims_ndvi_status ON property_claims(ndvi_status) WHERE ndvi_status = 'pending';
```

### Frontend Display

**Before:**
```
"NDVI Baseline: 0.625 (Sep 21, 2025)"
"NDVI Current: 0.642 (Feb 21, 2026)"
```

**After (with date windows):**
```
"Sentinel-2 NDVI Comparison"

Current: 0.642 (Nov 21, 2025 - Feb 21, 2026)
         [👆 Last 3 months]

Baseline: 0.625 (Dec 18, 2021 - Mar 18, 2022)
          [3 years prior]

Change: +0.017 ↑ (Improved by 2.7%)
```

**Credibility benefit:**
Users see exact date windows → "This is real data with real dates" → Trust

---

## Google Earth Engine Setup Instructions

### Step 1: Create GCP Project (If You Don't Have One)

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create new project: **"Terra NDVI"**
3. Enable billing (free tier covers us for Phase 1)

### Step 2: Enable Earth Engine API

1. Go to [Earth Engine Console](https://code.earthengine.google.com/)
2. Click **"Accept the Terms of Service"** (creates GEE account linked to GCP project)
3. Return to Google Cloud Console
4. Search for **"Earth Engine API"** in the API library
5. Click **"Enable"**

### Step 3: Create Service Account

1. In Google Cloud Console, go to **IAM & Admin → Service Accounts**
2. Click **"Create Service Account"**
3. Fill in:
   - **Service Account Name:** `terra-ndvi`
   - **Service Account ID:** `terra-ndvi` (auto-filled)
4. Click **"CREATE AND CONTINUE"**
5. Skip optional steps, click **"DONE"**

### Step 4: Create and Download JSON Key

1. Click on the newly created service account (`terra-ndvi@...`)
2. Go to **"Keys"** tab → **"Add Key"** → **"Create new key"**
3. Choose **"JSON"**
4. Click **"Create"** (downloads `terra-ndvi-...json`)

### Step 5: Grant Earth Engine Permissions

The service account must be granted Editor role in Earth Engine:

1. Go back to [Earth Engine Console](https://code.earthengine.google.com/)
2. Click **"Scripts"** (top left menu)
3. Click **"Developer Settings"** (bottom left)
4. Under **"Collaborators,"** add the service account email:
   - Email format: `terra-ndvi@terra-ndvi.iam.gserviceaccount.com`
   - Role: **"Editor"**
5. Click **"Add"**

### Step 6: Extract Credentials for Environment

Open the downloaded JSON file. Extract these values:

```json
{
  "type": "service_account",
  "project_id": "terra-ndvi",                          // ← GEE_PROJECT_ID
  "private_key_id": "abc123...",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIE...",  // ← GEE_PRIVATE_KEY (keep newlines as \n)
  "client_email": "terra-ndvi@terra-ndvi.iam.gserviceaccount.com",  // ← GEE_SERVICE_ACCOUNT_EMAIL
  "client_id": "1234567890",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/..."
}
```

### Step 7: Set Environment Variables

**For local testing:**

```bash
export GEE_PROJECT_ID="terra-ndvi"
export GEE_SERVICE_ACCOUNT_EMAIL="terra-ndvi@terra-ndvi.iam.gserviceaccount.com"
export GEE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQE...
(paste entire private_key value, keeping newlines)
-----END PRIVATE KEY-----"
```

**For production (pod environment):**

Add to **Secrets Manager** (not .env):
```
GEE_PROJECT_ID = "terra-ndvi"
GEE_SERVICE_ACCOUNT_EMAIL = "terra-ndvi@..."
GEE_PRIVATE_KEY = "[entire JSON private key value]"
```

Retrieve in Python:
```python
import os
import json

credentials_dict = {
    "type": "service_account",
    "project_id": os.getenv("GEE_PROJECT_ID"),
    "private_key": os.getenv("GEE_PRIVATE_KEY").replace("\\n", "\n"),
    "client_email": os.getenv("GEE_SERVICE_ACCOUNT_EMAIL"),
    # ... (add other required fields)
}
```

### Step 8: Test Connection

```python
import ee
import os

# Initialize with service account
credentials_dict = {
    "type": "service_account",
    "project_id": os.getenv("GEE_PROJECT_ID"),
    "private_key": os.getenv("GEE_PRIVATE_KEY").replace("\\n", "\n"),
    "client_email": os.getenv("GEE_SERVICE_ACCOUNT_EMAIL"),
    # ... (other fields from JSON)
}

credentials = ee.ServiceAccountCredentials(
    email=credentials_dict["client_email"],
    key_data=credentials_dict["private_key"]
)
ee.Initialize(credentials)

# Test query
image = ee.Image("COPERNICUS/S2_SR_HARMONIZED/20250221T143751_20250221T144851_T18SVD")
print(image.bandNames().getInfo())  # Should print band list
print("✅ GEE connection successful!")
```

---

## Feb 22 Action Items (In Order)

### Morning (Feb 22)

**Task 1: Emergent Sets Up GEE Account**
```
[ ] Follow steps 1-8 above
[ ] Test connection with Python script ✓
[ ] Confirm env vars set (GEE_PROJECT_ID, GEE_SERVICE_ACCOUNT_EMAIL, GEE_PRIVATE_KEY)
[ ] Send Brad confirmation: "GEE ready"
```

**Deliverable:** GEE credentials working locally

---

### Afternoon (Feb 22)

**Task 2: Update Database Schema**
```sql
-- Add 9 NDVI columns to property_claims
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

**Deliverable:** Supabase table updated

---

**Task 3: Test Python NDVI Query Script**
```python
# backend/services/ndvi_service.py (Python version with 3-year logic)

import ee
import os
from datetime import datetime, timedelta

def get_ndvi_baseline(latitude: float, longitude: float) -> dict:
    """
    Query Sentinel-2 NDVI using 3-year baseline logic.
    
    Returns:
    {
        'ndvi_baseline': 0.625,
        'ndvi_current': 0.642,
        'ndvi_delta': 0.017,
        'baseline_window_start': '2021-12-18',
        'baseline_window_end': '2022-03-18',
        'current_window_start': '2025-11-21',
        'current_window_end': '2026-02-21',
        'status': 'ready',
        'error': None
    }
    """
    
    try:
        poi = ee.Geometry.Point([longitude, latitude])
        
        # Define windows
        today = datetime.now()
        current_end = today
        current_start = today - timedelta(days=90)
        
        # 3 years ago, compute baseline window
        baseline_end = today - timedelta(days=365*3)
        baseline_start = baseline_end - timedelta(days=90)
        
        # Query Sentinel-2 for both windows
        collection = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED") \
            .filterBounds(poi) \
            .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
        
        # Baseline window collection
        baseline_collection = collection \
            .filterDate(baseline_start.strftime("%Y-%m-%d"), 
                       baseline_end.strftime("%Y-%m-%d"))
        
        # Current window collection
        current_collection = collection \
            .filterDate(current_start.strftime("%Y-%m-%d"), 
                       current_end.strftime("%Y-%m-%d"))
        
        # Compute median NDVI for each window
        def compute_median_ndvi(img_collection):
            ndvi = img_collection.map(
                lambda img: img.normalizedDifference(["B8", "B4"]).rename("NDVI")
            )
            return ndvi.median().sample(poi, 10).first().get("NDVI").getInfo()
        
        baseline_ndvi = compute_median_ndvi(baseline_collection)
        current_ndvi = compute_median_ndvi(current_collection)
        
        return {
            "ndvi_baseline": round(float(baseline_ndvi), 3) if baseline_ndvi else None,
            "ndvi_current": round(float(current_ndvi), 3) if current_ndvi else None,
            "ndvi_delta": round(float(current_ndvi - baseline_ndvi), 3) if (baseline_ndvi and current_ndvi) else None,
            "baseline_window_start": baseline_start.date().isoformat(),
            "baseline_window_end": baseline_end.date().isoformat(),
            "current_window_start": current_start.date().isoformat(),
            "current_window_end": current_end.date().isoformat(),
            "status": "ready",
            "error": None
        }
    
    except Exception as e:
        return {
            "status": "error",
            "error": str(e),
            "ndvi_baseline": None,
            "ndvi_current": None,
            "ndvi_delta": None,
            "baseline_window_start": None,
            "baseline_window_end": None,
            "current_window_start": None,
            "current_window_end": None
        }


# Test locally
if __name__ == "__main__":
    result = get_ndvi_baseline(-40.1234, 149.5678)  # Example coords (NSW)
    print(result)
```

**Test with ~5 test coordinates:**
```python
test_coords = [
    (-40.1234, 149.5678),  # NSW
    (-33.8688, 151.2093),  # Sydney area
    (-37.8136, 144.9631),  # Melbourne area
    # ... add more
]

for lat, lon in test_coords:
    result = get_ndvi_baseline(lat, lon)
    print(f"{lat}, {lon}: {result['ndvi_baseline']} → {result['ndvi_current']} (Δ {result['ndvi_delta']})")
```

**Deliverable:** Python script returns real NDVI values for test coordinates

---

### End of Day (Feb 22)

**Confirm with Brad:**
```
✅ GEE account created + credentials set
✅ Database schema updated (9 new columns)
✅ Python NDVI script tested (5+ coordinates returning real values)
✅ Ready for tomorrow: Celery queue + async job wiring
```

---

## Next Steps (Feb 23+)

- **Feb 23:** Celery/RQ queue setup + endpoint wiring
- **Feb 24:** FastAPI claim endpoint triggers NDVI job
- **Feb 25:** Frontend hook + NDVICard component
- **Feb 26:** Integration into PropertyClaimFlow
- **Feb 27-28:** Testing + error handling

---

## GEE Free Tier Limits (Phase 1)

Google Earth Engine free tier includes:
- ✅ Unlimited image requests (Sentinel-2 queries)
- ✅ 100+ simultaneous processes
- ✅ 40GB storage (for exports)

**Phase 1 usage estimate:**
- ~50 claims per day (soft launch)
- ~2-3 Sentinel-2 queries per claim
- **Total: ~150 queries/day = well within free tier**

**No cost for Phase 1.** Yay!

---

## Security Checklist

```
[ ] GEE_PRIVATE_KEY never appears in code (env vars only)
[ ] .env file not committed to git (.gitignore includes .env)
[ ] Production uses Secrets Manager (not .env)
[ ] Service account permissions scoped to Earth Engine (not full GCP Admin)
[ ] Credentials rotated annually (GCP best practice)
```

---

**Owner:** Emergent (GEE setup + Python script) + Brad (review)  
**Timeline:** Complete by Feb 22, 17:00 UTC  
**Next document:** Celery Queue Integration (Feb 23)
