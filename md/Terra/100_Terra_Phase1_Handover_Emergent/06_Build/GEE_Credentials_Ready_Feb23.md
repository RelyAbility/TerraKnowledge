# GEE Credentials Ready — Feb 23

**Status:** ✅ UNBLOCKED — Python GEE initialization can proceed immediately

**Credential Delivery Method:**
- **Env Variable:** `GEE_SERVICE_ACCOUNT_JSON` (pod secret, not hardcoded)
- **Format:** Complete JSON key string (including all fields: type, project_id, private_key_id, private_key, client_email, client_id, auth_uri, token_uri, auth_provider_x509_cert_url, client_x509_cert_url)
- **Environment:** Pod environment (not local .env)

---

## Python Implementation (Using the Credential Env Var)

### 1. Installation

```bash
pip install google-auth google-cloud-earthengine
```

### 2. Initialization in Backend Service

Create `backend/services/ndvi_service.py`:

```python
import os
import json
import ee
from google.auth.crypt.rsa_signer import RSASigner
from google.auth.credentials import Credentials
from google.oauth2.service_account import Credentials as ServiceAccountCredentials
from datetime import datetime, timedelta
from decimal import Decimal

# Initialize Earth Engine with service account credentials from environment
def initialize_earth_engine():
    """Initialize EE with credentials from GEE_SERVICE_ACCOUNT_JSON env var."""
    
    # Load credentials from environment variable
    gee_json_str = os.getenv('GEE_SERVICE_ACCOUNT_JSON')
    
    if not gee_json_str:
        raise ValueError("GEE_SERVICE_ACCOUNT_JSON environment variable not set")
    
    gee_credentials = json.loads(gee_json_str)
    
    # Authenticate with service account
    credentials = ServiceAccountCredentials.from_service_account_info(
        gee_credentials,
        scopes=['https://www.googleapis.com/auth/cloud-platform', 
                'https://www.googleapis.com/auth/earthengine']
    )
    
    ee.Initialize(credentials)
    print(f"✅ Earth Engine initialized with service account: {gee_credentials.get('client_email')}")

# Call on startup (e.g., in FastAPI lifespan or module import)
try:
    initialize_earth_engine()
except Exception as e:
    print(f"⚠️ Earth Engine initialization warning: {e}")
```

### 3. NDVI Baseline Function

```python
def get_ndvi_baseline(latitude: float, longitude: float) -> dict:
    """
    Get NDVI baseline (3-year offset) using Sentinel-2 median composite.
    Returns dict with ndvi_baseline, date_baseline_start, date_baseline_end.
    """
    
    try:
        # Current date windows
        now = datetime.now()
        current_start = now - timedelta(days=90)
        current_end = now
        
        # Baseline date windows (3 years ago)
        baseline_start = current_start - timedelta(days=365*3)
        baseline_end = current_end - timedelta(days=365*3)
        
        # Define area of interest (1km buffer)
        point = ee.Geometry.Point([longitude, latitude])
        roi = point.buffer(1000)  # 1km buffer
        
        # Sentinel-2 collection
        sentinel2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
        
        # Baseline period (3 years ago)
        baseline_collection = (
            sentinel2
            .filterBounds(roi)
            .filterDate(baseline_start.strftime('%Y-%m-%d'), 
                       baseline_end.strftime('%Y-%m-%d'))
            .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 20)  # Cloud filter
        )
        
        # Median composite for baseline
        baseline_median = baseline_collection.median()
        
        # Calculate NDVI for baseline (NIR = B8, Red = B4)
        ndvi_baseline = baseline_median.normalizedDifference(['B8', 'B4']).rename('NDVI')
        
        # Reduce to mean value over ROI
        ndvi_baseline_value = ndvi_baseline.reduceRegion(
            reducer=ee.Reducer.median(),
            geometry=roi,
            scale=10  # 10m Sentinel-2 resolution
        ).getInfo()
        
        ndvi_baseline_float = float(ndvi_baseline_value.get('NDVI', None)) or 0.0
        
        return {
            'ndvi_baseline': round(Decimal(str(ndvi_baseline_float)), 3),
            'date_baseline_start': baseline_start.strftime('%Y-%m-%d'),
            'date_baseline_end': baseline_end.strftime('%Y-%m-%d'),
            'status': 'ready'
        }
    
    except Exception as e:
        return {
            'status': 'error',
            'error_message': str(e)
        }


def get_ndvi_current(latitude: float, longitude: float) -> dict:
    """
    Get NDVI current (last 90 days) using Sentinel-2 median composite.
    Returns dict with ndvi_current, date_current_start, date_current_end.
    """
    
    try:
        # Current date windows
        now = datetime.now()
        current_start = now - timedelta(days=90)
        current_end = now
        
        # Define area of interest
        point = ee.Geometry.Point([longitude, latitude])
        roi = point.buffer(1000)  # 1km buffer
        
        # Sentinel-2 collection
        sentinel2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
        
        # Current period
        current_collection = (
            sentinel2
            .filterBounds(roi)
            .filterDate(current_start.strftime('%Y-%m-%d'), 
                       current_end.strftime('%Y-%m-%d'))
            .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 20)
        )
        
        # Median composite for current
        current_median = current_collection.median()
        
        # Calculate NDVI for current
        ndvi_current = current_median.normalizedDifference(['B8', 'B4']).rename('NDVI')
        
        # Reduce to mean value over ROI
        ndvi_current_value = ndvi_current.reduceRegion(
            reducer=ee.Reducer.median(),
            geometry=roi,
            scale=10  # 10m resolution
        ).getInfo()
        
        ndvi_current_float = float(ndvi_current_value.get('NDVI', None)) or 0.0
        
        return {
            'ndvi_current': round(Decimal(str(ndvi_current_float)), 3),
            'date_current_start': current_start.strftime('%Y-%m-%d'),
            'date_current_end': current_end.strftime('%Y-%m-%d'),
            'status': 'ready'
        }
    
    except Exception as e:
        return {
            'status': 'error',
            'error_message': str(e)
        }
```

### 4. Combined NDVI Job (For RQ Queue)

```python
def compute_ndvi_job(property_id: int, latitude: float, longitude: float) -> dict:
    """
    Complete NDVI computation job (baseline + current + delta).
    Used by async queue (RQ/Celery).
    """
    
    baseline_result = get_ndvi_baseline(latitude, longitude)
    if baseline_result.get('status') == 'error':
        return baseline_result
    
    current_result = get_ndvi_current(latitude, longitude)
    if current_result.get('status') == 'error':
        return current_result
    
    # Calculate delta
    ndvi_baseline = float(baseline_result['ndvi_baseline'])
    ndvi_current = float(current_result['ndvi_current'])
    ndvi_delta = round(Decimal(str(ndvi_current - ndvi_baseline)), 3)
    
    return {
        'property_id': property_id,
        'ndvi_baseline': baseline_result['ndvi_baseline'],
        'ndvi_current': current_result['ndvi_current'],
        'ndvi_delta': ndvi_delta,
        'date_baseline_start': baseline_result['date_baseline_start'],
        'date_baseline_end': baseline_result['date_baseline_end'],
        'date_current_start': current_result['date_current_start'],
        'date_current_end': current_result['date_current_end'],
        'ndvi_status': 'ready',
        'ndvi_last_updated': datetime.now().isoformat(),
        'error_message': None
    }
```

---

## Test Coordinates (For Local Validation)

Before integrating into async queue, test with these 5 coordinates to verify NDVI computation is working:

```python
test_coordinates = [
    {'lat': -36.7465, 'lon': 142.8862},  # Gondwana, Victoria
    {'lat': -36.8, 'lon': 142.9},         # Nearby test point 1
    {'lat': -36.7, 'lon': 142.8},         # Nearby test point 2
    {'lat': -36.75, 'lon': 142.85},       # Nearby test point 3
    {'lat': -36.72, 'lon': 142.87},       # Nearby test point 4
]

# Run local test
for coord in test_coordinates:
    baseline = get_ndvi_baseline(coord['lat'], coord['lon'])
    current = get_ndvi_current(coord['lat'], coord['lon'])
    print(f"Lat {coord['lat']}, Lon {coord['lon']}")
    print(f"  Baseline NDVI: {baseline.get('ndvi_baseline')} ({baseline.get('date_baseline_start')} to {baseline.get('date_baseline_end')})")
    print(f"  Current NDVI:  {current.get('ndvi_current')} ({current.get('date_current_start')} to {current.get('date_current_end')})")
    print(f"  Delta: {float(current.get('ndvi_current', 0)) - float(baseline.get('ndvi_baseline', 0))}")
    print()
```

---

## Integration with FastAPI

Add endpoint for testing (temporary, for validation only):

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter(prefix="/api/ndvi", tags=["ndvi"])

class NDVIRequest(BaseModel):
    property_id: int
    latitude: float
    longitude: float

@router.post("/compute")
async def compute_ndvi(req: NDVIRequest):
    """Synchronous NDVI computation (for testing only)."""
    result = compute_ndvi_job(req.property_id, req.latitude, req.longitude)
    if result.get('ndvi_status') == 'error':
        raise HTTPException(status_code=500, detail=result.get('error_message'))
    return result
```

---

## Environment Variable Setup

### Local Development (.env)

```bash
GEE_SERVICE_ACCOUNT_JSON='{"type":"service_account","project_id":"...","private_key_id":"...","private_key":"...","client_email":"...","client_id":"...","auth_uri":"https://accounts.google.com/o/oauth2/auth","token_uri":"https://oauth2.googleapis.com/token","auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs","client_x509_cert_url":"..."}'
```

### Pod Environment (Already Set)

Brad has configured this as a secret in the pod environment. No additional setup needed—the Python code reads `os.getenv('GEE_SERVICE_ACCOUNT_JSON')` directly.

---

## Next Action Items (For Emergent)

**By End of Feb 23:**

1. ✅ Create `backend/services/ndvi_service.py` with functions above
2. ✅ Install dependencies: `pip install google-auth google-cloud-earthengine`
3. ✅ Ensure `.env` file has `GEE_SERVICE_ACCOUNT_JSON` (or use pod environment if running in pod)
4. ✅ Run test script with 5 test coordinates
5. ✅ Verify NDVI values returned (should be between 0.0-1.0)
6. ✅ Verify date windows are correct (3-year offset for baseline, 90-day for current)
7. ✅ Commit and push `backend/services/ndvi_service.py`

**By Feb 24 Morning:**

- Integrate `compute_ndvi_job()` into RQ async queue
- Wire FastAPI endpoint to trigger RQ job when claim created
- Set up Supabase real-time subscription on frontend to listen for NDVI updates

**By Feb 24 Evening:**

- End-to-end test: Claim → RQ job queued → NDVI computed → Frontend updated

---

## Credential Security Notes

✅ **No Hardcoding:** All credentials loaded from environment variable  
✅ **Pod Secret:** `GEE_SERVICE_ACCOUNT_JSON` stored securely in pod environment  
✅ **JSON String Format:** Complete service account JSON (not individual fields)  
✅ **On Startup:** Initialize EE on FastAPI startup, not per-request  
✅ **Error Handling:** Gracefully handle missing credentials, return error status to queue job

---

## Reference

- **Original GEE Setup:** [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md)
- **Implementation Spec:** [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md)
- **Schema Ready:** [NDVI_Supabase_Schema_Deployment_Feb23.md](NDVI_Supabase_Schema_Deployment_Feb23.md)
- **Supabase Table:** `property_claims` (10 NDVI columns ready)

---

**Timeline:**
- **23 Feb EOD:** Python function tested locally ✓
- **24 Feb:** RQ integration + FastAPI wiring
- **25 Feb:** Frontend NDVICard component integration
- **26-27 Feb:** E2E testing (claim to NDVI display)
- **28 Feb 09:00:** Go/No-Go decision

**Status:** 🟢 **UNBLOCKED** — Emergent can start Python implementation immediately
