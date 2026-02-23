# Brad's GEE Setup Instructions — Feb 23

**From:** Brad  
**To:** Emergent Team  
**Date:** 23 February 2026  
**Priority:** 🟢 Ready to Execute  

---

## Environment Setup (Backend Runtime)

Backend is hosted/running on your side, so please set the GEE credentials as environment secrets in the backend runtime.

### Required Secrets

**Add to your backend environment:**

1. **`GEE_SERVICE_ACCOUNT_JSON`** (required)
   - Value: The full service account JSON (entire file contents)
   - Type: String (multiline)
   - Storage: Backend secret manager (not hardcoded, not in .env repo)
   - Format: Complete JSON object from GCP service account key file

2. **`GEE_PROJECT_ID`** (optional)
   - Value: The `project_id` field extracted from the JSON
   - Example: `"terra-phase1-gee"` or similar
   - Use: For debugging/logging reference

### Security Requirements

⚠️ **Do NOT commit any key material to the repo.**

---

## GCP Console Setup

### Service Account Access Grant

Ensure the service account (`client_email` from the JSON) is added in the **Earth Engine Code Editor** and granted appropriate access:

**Steps:**
1. Go to [Google Earth Engine Code Editor](https://code.earthengine.google.com/)
2. Navigate to **Assets** → **Earth Engine Developers** (or similar, depends on GCP UI)
3. Add the service account email (`client_email` from JSON) as a collaborator
4. Grant **Editor** access (sufficient for Phase 1)
5. **Do NOT skip this step** — `ee.Initialize()` will fail with correct JSON but no access grants

---

## Testing Procedure

Once environment secrets are in place:

### 1. Python Connectivity Test

```python
import ee
import os
import json

# Load credentials from environment
gee_json_str = os.getenv('GEE_SERVICE_ACCOUNT_JSON')
gee_credentials = json.loads(gee_json_str)

# Initialize Earth Engine
from google.oauth2.service_account import Credentials as ServiceAccountCredentials

credentials = ServiceAccountCredentials.from_service_account_info(
    gee_credentials,
    scopes=['https://www.googleapis.com/auth/cloud-platform', 
            'https://www.googleapis.com/auth/earthengine']
)

ee.Initialize(credentials)
print(f"✅ Earth Engine initialized successfully")
print(f"Service Account: {gee_credentials.get('client_email')}")
```

### 2. Load One Sentinel-2 Image (Confidence Check)

```python
# Test Sentinel-2 image load
sentinel2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
point = ee.Geometry.Point([142.8862, -36.7465])  # Gondwana coordinates
roi = point.buffer(1000)

image = (
    sentinel2
    .filterBounds(roi)
    .filterDate('2023-01-01', '2023-01-31')
    .first()
)

# Verify image loaded
info = image.getInfo()
if info:
    print(f"✅ Sentinel-2 image loaded successfully")
    print(f"Image ID: {info.get('id')}")
else:
    print(f"❌ Failed to load Sentinel-2 image")
```

**Expected Output:**
- No authentication errors
- Successfully loaded image metadata
- Confirmed service account email printed

---

## Next Steps

Once environment setup + connectivity test passes:

1. ✅ Confirm messaging: "GEE connectivity test passed"
2. ✅ Proceed with [NDVI Python implementation](GEE_Credentials_Ready_Feb23.md)
3. ✅ Code location: `backend/services/ndvi_service.py`
4. ✅ Reference code: All snippets provided in [GEE_Credentials_Ready_Feb23.md](GEE_Credentials_Ready_Feb23.md)

---

## Secure Credential Sharing

**JSON Key Delivery:**
- Brad will share the GEE service account JSON via **1Password** (secure channel)
- Do NOT request via email, Slack, or chat
- Do NOT screenshot or forward
- Do NOT commit to git history

---

## Timeline

- **Feb 23 (Today):** Environment setup + connectivity test
- **Feb 23-24:** NDVI Python implementation (`get_ndvi_baseline()`, `get_ndvi_current()`)
- **Feb 24:** RQ async queue integration
- **Feb 25:** Frontend NDVICard component integration

---

## Questions / Blockers

If connectivity test fails:
1. Verify `GEE_SERVICE_ACCOUNT_JSON` is set correctly (output first 100 chars to confirm format)
2. Verify service account email is added in Earth Engine Code Editor (with Editor access, not just Reader)
3. Check backend logs for initialization errors
4. Reach out to Brad with error message + environment confirmation

---

## Reference

- **Code Snippets:** [GEE_Credentials_Ready_Feb23.md](GEE_Credentials_Ready_Feb23.md)
- **NDVI Implementation:** [NDVI_Phase1_Implementation_Sentinel2_GEE.md](NDVI_Phase1_Implementation_Sentinel2_GEE.md)
- **Architecture Decision:** [NDVI_Architecture_Decision_GEE_Setup.md](NDVI_Architecture_Decision_GEE_Setup.md)
- **Schema Ready:** [NDVI_Supabase_Schema_Deployment_Feb23.md](NDVI_Supabase_Schema_Deployment_Feb23.md)

---

**Status:** 🟢 Ready for immediate execution  
**Owner:** Emergent (backend environment setup)  
**Blocker:** Waiting for JSON key via 1Password (Brad → Emergent secure channel)
