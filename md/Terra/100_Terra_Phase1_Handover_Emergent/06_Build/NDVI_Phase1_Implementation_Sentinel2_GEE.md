# Phase 1 NDVI Implementation: Sentinel-2 via Google Earth Engine

**Status:** Ready for implementation (fastest path confirmed)  
**Timeline:** Feb 22-28 (7 days)  
**Owner:** Emergent (backend + async job) + Brad (data validation)

---

## Overview

**Goal:** Every claimed property returns real NDVI data from Sentinel-2 satellite imagery.

**Flow:**
```
User claims property
  ↓
POST /api/claims (start-trial endpoint)
  ↓
Queue async job: compute_ndvi_baseline(claim_id, latitude, longitude)
  ↓
Frontend shows: "Refining data (up to 24 hours)"
  ↓
Backend queries Google Earth Engine
  ↓
Store results: ndvi_baseline, ndvi_current, ndvi_delta, date_baseline, date_current, ndvi_status, last_updated
  ↓
Frontend updates: "NDVI: 0.62 (Sentinel-2, Sep 21 - Feb 21)"
  ↓
Icon color remains driven by monitoring_state (trial_active / subscribed / paused)
NDVI is data layer, not state layer.
```

---

## Database Schema

### Option A: Columns on property_claims (Simpler)

Add 8 columns to `property_claims` table:

```sql
ALTER TABLE property_claims ADD COLUMN (
  ndvi_baseline          DECIMAL(5,3),           -- Baseline value (e.g., 0.625)
  ndvi_current           DECIMAL(5,3),           -- Most recent value
  ndvi_delta             DECIMAL(5,3),           -- current - baseline
  date_baseline          DATE,                   -- When baseline was captured (e.g., 2025-09-21)
  date_current           DATE,                   -- Most recent data date
  ndvi_status            VARCHAR(20),            -- 'pending' | 'ready' | 'error'
  ndvi_last_updated      TIMESTAMP,              -- When we last queried GEE
  ndvi_error_message     TEXT NULLABLE           -- If status='error', store reason
);

-- Create index for queries
CREATE INDEX idx_ndvi_status ON property_claims(ndvi_status) WHERE ndvi_status = 'pending';
```

**Pros:**
- No new table
- All claim data in one place
- Simpler queries

**Cons:**
- property_claims grows to 19 fields

### Option B: Separate property_ndvi Table (Normalized)

```sql
CREATE TABLE property_ndvi (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  claim_id              UUID NOT NULL UNIQUE REFERENCES property_claims(id) ON DELETE CASCADE,
  ndvi_baseline         DECIMAL(5,3),
  ndvi_current          DECIMAL(5,3),
  ndvi_delta            DECIMAL(5,3),
  date_baseline         DATE,
  date_current          DATE,
  ndvi_status           VARCHAR(20),           -- 'pending' | 'ready' | 'error'
  ndvi_last_updated     TIMESTAMP,
  ndvi_error_message    TEXT NULLABLE,
  gee_request_date      TIMESTAMP DEFAULT NOW(),
  created_at            TIMESTAMP DEFAULT NOW(),
  updated_at            TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_ndvi_claim ON property_ndvi(claim_id);
CREATE INDEX idx_ndvi_status ON property_ndvi(ndvi_status);
```

**Pros:**
- Normalized schema
- Can store multiple NDVI snapshots per property (historical)
- Cleaner separation of concerns

**Cons:**
- One extra JOIN on queries

**Recommendation:** Use **Option A** for Phase 1 simplicity. Move to Option B if we need historical NDVI snapshots (future feature).

---

## Backend Implementation

### 1. Environment Variables

```bash
# .env file
GEE_PROJECT_ID=your-gee-project-id
GEE_SERVICE_ACCOUNT_EMAIL=your-service-account@gee-project.iam.gserviceaccount.com
GEE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"  # Multiline key as base64 or direct JSON

# Redis for async job queue (already have?)
REDIS_URL=redis://localhost:6379

# Or use Bull.js with in-memory queue for Phase 1 (simpler)
USE_BULL_QUEUE=true

# Sentinel-2 date range defaults
SENTINEL2_BASELINE_WINDOW_DAYS=365  # Look back 1 year for baseline
SENTINEL2_CLOUD_COVER_MAX=20        # Max cloud cover %
```

### 2. Google Earth Engine Authentication

**Setup (one-time, before Phase 1):**

1. Create GCP project (or use existing)
2. Enable Earth Engine API
3. Create service account with Earth Engine roles
4. Download JSON credentials
5. Store in environment variables (do not commit)

**Python Client (for local testing):**

```python
import ee
import os

def init_gee():
    """Initialize Earth Engine with service account."""
    credentials_dict = {
        "type": "service_account",
        "project_id": os.getenv("GEE_PROJECT_ID"),
        "private_key_id": os.getenv("GEE_PRIVATE_KEY_ID"),
        "private_key": os.getenv("GEE_PRIVATE_KEY").replace("\\n", "\n"),
        "client_email": os.getenv("GEE_SERVICE_ACCOUNT_EMAIL"),
        # ... other fields from service account JSON
    }
    
    credentials = ee.ServiceAccountCredentials(
        email=credentials_dict["client_email"],
        key_data=credentials_dict["private_key"]
    )
    ee.Initialize(credentials)
    return True
```

### 3. Core NDVI Computation Function

```python
# backend/services/ndvi_service.py

import ee
import os
from datetime import datetime, timedelta
from supabase import create_client
import asyncio

def get_ndvi_baseline(latitude, longitude, baseline_window_days=365):
    """
    Query Sentinel-2 NDVI for a location over a date range.
    
    Returns: {
        'ndvi_baseline': 0.625,
        'ndvi_current': 0.642,
        'ndvi_delta': 0.017,
        'date_baseline': '2024-09-21',
        'date_current': '2025-02-21',
        'status': 'ready',
        'error': None
    }
    """
    
    try:
        # Point of interest (parcel centroid)
        poi = ee.Geometry.Point([longitude, latitude])
        
        # Time range for baseline (1 year ago)
        end_date = datetime.now()
        start_date = end_date - timedelta(days=baseline_window_days)
        
        # Sentinel-2 L2A (atmospherically corrected)
        collection = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED") \
            .filterBounds(poi) \
            .filterDate(start_date.strftime("%Y-%m-%d"), end_date.strftime("%Y-%m-%d")) \
            .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", int(os.getenv("SENTINEL2_CLOUD_COVER_MAX", 20))))
        
        # Sort by cloud cover, get clearest image
        image_baseline = collection.sort("CLOUDY_PIXEL_PERCENTAGE").first()
        
        # Most recent image (within 30 days)
        recent_start = (end_date - timedelta(days=30)).strftime("%Y-%m-%d")
        image_current = collection.filterDate(recent_start, end_date.strftime("%Y-%m-%d")) \
            .sort("CLOUDY_PIXEL_PERCENTAGE").first()
        
        # NDVI = (NIR - Red) / (NIR + Red)
        # Sentinel-2: B8=NIR, B4=Red
        def compute_ndvi(image):
            return image.normalizedDifference(["B8", "B4"]).rename("NDVI")
        
        ndvi_baseline = compute_ndvi(image_baseline)
        ndvi_current = compute_ndvi(image_current)
        
        # Sample at point
        baseline_value = ndvi_baseline.sample(poi, 10).first().get("NDVI").getInfo()
        current_value = ndvi_current.sample(poi, 10).first().get("NDVI").getInfo()
        
        # Get image dates
        baseline_timestamp = image_baseline.get("system:time_start").getInfo()
        current_timestamp = image_current.get("system:time_start").getInfo()
        
        baseline_date = datetime.fromtimestamp(baseline_timestamp / 1000).date()
        current_date = datetime.fromtimestamp(current_timestamp / 1000).date()
        
        return {
            "ndvi_baseline": round(float(baseline_value), 3) if baseline_value else None,
            "ndvi_current": round(float(current_value), 3) if current_value else None,
            "ndvi_delta": round(float(current_value - baseline_value), 3) if (baseline_value and current_value) else None,
            "date_baseline": baseline_date.isoformat(),
            "date_current": current_date.isoformat(),
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
            "date_baseline": None,
            "date_current": None
        }
```

### 4. Async Job Handler

**Using Bull.js (Node.js):**

```javascript
// backend/jobs/ndvi-job.js

import Queue from "bull";
import { supabase } from "../lib/supabase.js";
import { spawn } from "child_process";  // Call Python script

const ndviQueue = new Queue("ndvi-computation", {
  redis: {
    host: process.env.REDIS_HOST || "localhost",
    port: process.env.REDIS_PORT || 6379
  }
});

// Process NDVI jobs
ndviQueue.process(async (job) => {
  const { claimId, latitude, longitude } = job.data;
  
  try {
    // Update status to pending
    await supabase
      .from("property_claims")
      .update({ ndvi_status: "pending", ndvi_last_updated: new Date() })
      .eq("id", claimId);
    
    // Call Python service (can be in-process or separate service)
    const ndviResult = await computeNDVIViaGEE(latitude, longitude);
    
    // Store results
    await supabase
      .from("property_claims")
      .update({
        ndvi_baseline: ndviResult.ndvi_baseline,
        ndvi_current: ndviResult.ndvi_current,
        ndvi_delta: ndviResult.ndvi_delta,
        date_baseline: ndviResult.date_baseline,
        date_current: ndviResult.date_current,
        ndvi_status: ndviResult.status,
        ndvi_error_message: ndviResult.error,
        ndvi_last_updated: new Date()
      })
      .eq("id", claimId);
    
    return { success: true, claimId };
  } catch (err) {
    // Store error
    await supabase
      .from("property_claims")
      .update({
        ndvi_status: "error",
        ndvi_error_message: err.message,
        ndvi_last_updated: new Date()
      })
      .eq("id", claimId);
    
    throw err;
  }
});

// Handle job completion
ndviQueue.on("completed", (job) => {
  console.log(`NDVI ready for claim ${job.data.claimId}`);
});

ndviQueue.on("failed", (job, err) => {
  console.error(`NDVI failed for claim ${job.data.claimId}: ${err.message}`);
});

export { ndviQueue };
```

### 5. Update Claims Endpoint

**POST /api/claims (create new claim):**

```javascript
// When user submits claim
POST /api/claims
{
  "user_id": "uuid",
  "parcel_id": "string",
  "latitude": 40.123,
  "longitude": -75.456,
  "boundary": { ...geojson },
  "address": "123 Main St"
}

// Response
{
  "id": "claim_uuid",
  "monitoring_state": "trial_active",
  "ndvi_status": "pending",
  "ndvi_baseline": null,
  "date_baseline": null,
  ...
}
```

**In handler:**
```javascript
const claimId = newClaim.id;

// Queue NDVI computation
await ndviQueue.add(
  { claimId, latitude, longitude },
  { 
    delay: 1000,  // 1s delay to ensure claim persisted
    attempts: 3,  // Retry 3 times if fails
    backoff: { type: "exponential", delay: 2000 }
  }
);
```

### 6. Get Claim Endpoint (with NDVI)

**GET /api/claims/:id**

```javascript
// Response includes NDVI fields
{
  "id": "claim_uuid",
  "user_id": "user_uuid",
  "parcel_id": "rp123456",
  "monitoring_state": "trial_active",
  "stripe_subscription_id": "sub_xyz",
  "trial_end_date": "2025-03-21",
  
  // NDVI Fields
  "ndvi_status": "ready",
  "ndvi_baseline": 0.625,
  "ndvi_current": 0.642,
  "ndvi_delta": 0.017,
  "date_baseline": "2024-09-21",
  "date_current": "2025-02-21",
  "ndvi_error_message": null,
  "ndvi_last_updated": "2025-02-21T14:32:00Z"
}
```

---

## Frontend Integration

### 1. NDVI Data Fetch

```typescript
// hooks/useClaimNDVI.ts

import { useEffect, useState } from "react";
import { supabase } from "@/lib/supabase";

interface NDVIData {
  baseline: number | null;
  current: number | null;
  delta: number | null;
  dateBaseline: string | null;
  dateCurrent: string | null;
  status: "pending" | "ready" | "error";
  errorMessage: string | null;
  lastUpdated: string | null;
}

export function useClaimNDVI(claimId: string) {
  const [ndvi, setNDVI] = useState<NDVIData | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Subscribe to real-time updates
    const subscription = supabase
      .from("property_claims")
      .on("UPDATE", (payload) => {
        if (payload.new.id === claimId) {
          setNDVI({
            baseline: payload.new.ndvi_baseline,
            current: payload.new.ndvi_current,
            delta: payload.new.ndvi_delta,
            dateBaseline: payload.new.date_baseline,
            dateCurrent: payload.new.date_current,
            status: payload.new.ndvi_status,
            errorMessage: payload.new.ndvi_error_message,
            lastUpdated: payload.new.ndvi_last_updated
          });
        }
      })
      .subscribe();

    // Fetch initial data
    fetchNDVI();

    return () => subscription.unsubscribe();
  }, [claimId]);

  async function fetchNDVI() {
    const { data } = await supabase
      .from("property_claims")
      .select(
        "ndvi_baseline, ndvi_current, ndvi_delta, date_baseline, date_current, ndvi_status, ndvi_error_message, ndvi_last_updated"
      )
      .eq("id", claimId)
      .single();

    if (data) {
      setNDVI({
        baseline: data.ndvi_baseline,
        current: data.ndvi_current,
        delta: data.ndvi_delta,
        dateBaseline: data.date_baseline,
        dateCurrent: data.date_current,
        status: data.ndvi_status,
        errorMessage: data.ndvi_error_message,
        lastUpdated: data.ndvi_last_updated
      });
    }

    setIsLoading(false);
  }

  return { ndvi, isLoading };
}
```

### 2. NDVI Display Component

```typescript
// components/NDVICard.tsx

import React from "react";
import { useClaimNDVI } from "@/hooks/useClaimNDVI";

interface NDVICardProps {
  claimId: string;
}

export function NDVICard({ claimId }: NDVICardProps) {
  const { ndvi, isLoading } = useClaimNDVI(claimId);

  if (isLoading) {
    return <div>Loading satellite data...</div>;
  }

  if (!ndvi) {
    return <div>No NDVI data</div>;
  }

  if (ndvi.status === "pending") {
    return (
      <div className="ndvi-card pending">
        <p>🛰️ Refining data (up to 24 hours)</p>
        <small>Sentinel-2 satellite imagery being analyzed...</small>
      </div>
    );
  }

  if (ndvi.status === "error") {
    return (
      <div className="ndvi-card error">
        <p>⚠️ Satellite data unavailable</p>
        <small>{ndvi.errorMessage}</small>
      </div>
    );
  }

  // Status: ready
  return (
    <div className="ndvi-card ready">
      <div className="ndvi-value">
        <span className="label">NDVI Baseline:</span>
        <span className="value">{ndvi.baseline?.toFixed(3)}</span>
      </div>

      <div className="ndvi-current">
        <span className="label">Current NDVI:</span>
        <span className="value">{ndvi.current?.toFixed(3)}</span>
      </div>

      {ndvi.delta !== null && (
        <div className="ndvi-delta">
          <span className="label">Change:</span>
          <span className={`value ${ndvi.delta > 0 ? "positive" : "negative"}`}>
            {ndvi.delta > 0 ? "+" : ""}{ndvi.delta?.toFixed(3)}
          </span>
        </div>
      )}

      <div className="date-range">
        <small>
          Sentinel-2 satellite imagery
          <br />
          Baseline: {ndvi.dateBaseline}
          <br />
          Current: {ndvi.dateCurrent}
        </small>
      </div>

      <div className="last-updated">
        <small>Updated: {new Date(ndvi.lastUpdated).toLocaleString()}</small>
      </div>
    </div>
  );
}
```

### 3. Integration into PropertyClaimFlow

Show NDVI card in property detail view (after icon):

```typescript
// PropertyClaimFlow.tsx component addition

function ClaimDetail({ claimId }: { claimId: string }) {
  return (
    <div className="claim-detail">
      {/* Icon (driven by monitoring_state) */}
      <div className="icon-section">
        <SatelliteIcon monitoring_state={claim.monitoring_state} />
        <p>{claim.monitoring_state}</p>
      </div>

      {/* NDVI Data (separate layer) */}
      <div className="ndvi-section">
        <NDVICard claimId={claimId} />
      </div>

      {/* Trial Status */}
      <div className="trial-section">
        <TrialStatus trialEndDate={claim.trial_end_date} />
      </div>
    </div>
  );
}
```

---

## Deployment

### Environment Variables (All Platforms)

```bash
# Supabase (already have)
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJxx
SUPABASE_SERVICE_ROLE_KEY=eyJxx

# Google Earth Engine
GEE_PROJECT_ID=my-gee-project-id
GEE_SERVICE_ACCOUNT_EMAIL=service-account@my-gee-project.iam.gserviceaccount.com
GEE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"

# Sentinel-2 Parameters
SENTINEL2_BASELINE_WINDOW_DAYS=365
SENTINEL2_CLOUD_COVER_MAX=20

# Redis (if using Bull.js)
REDIS_URL=redis://localhost:6379
```

### Vercel / Netlify Deployment

1. Add all env vars to platform dashboard
2. Python GEE service accessible from Node.js via subprocess or HTTP
3. Bull queue needs Redis (Upstash, Railway, or local)

For simplicity: **Use Upstash Redis** (serverless)

```bash
# Upstash setup
REDIS_URL=redis://default:password@host:port
```

---

## Testing Checklist

### Unit Tests

```
[ ] GEE authentication (credentials load correctly)
[ ] NDVI computation (returns valid NDVI values 0-1)
[ ] Date parsing (baseline and current dates correct)
[ ] Error handling (GEE API failures, network issues)
[ ] Cloud cover filtering (filters high-cloud images correctly)
```

### Integration Tests

```
[ ] Claim creation → NDVI job queued
[ ] NDVI job runs → Results stored in Supabase
[ ] Multiple claims → Async jobs run in parallel
[ ] Job failure → Status set to 'error', message stored
[ ] Job retry → Failed jobs retry up to 3 times
[ ] Long-running job → Status remains 'pending' for up to 24h
```

### End-to-End Tests

```
[ ] Claim property → Icon visible (monitoring_state = trial_active)
[ ] Claim property → NDVI status = 'pending'
[ ] ~5 minutes later → NDVI status = 'ready'
[ ] NDVI card displays → Shows baseline, current, delta, dates
[ ] Refresh page → NDVI data persists
[ ] Multiple properties → Each shows correct NDVI values
[ ] Error case → NDVI status = 'error', message shown to user
```

### Performance Tests

```
[ ] Single NDVI query → Completes in < 10 seconds (GEE API call)
[ ] 5 concurrent jobs → All complete within 30 seconds
[ ] Job queue depth → Handles 50+ queued jobs gracefully
[ ] Database queries → NDVI queries use indexes, < 50ms
[ ] UI real-time updates → Supabase subscription fires < 1s after GEE job done
```

---

## Timeline & Milestones

| Day | Task | Deliverable |
|-----|------|------------|
| 22 Feb | GEE account + auth setup + Python script testing | Working `get_ndvi_baseline()` function |
| 23 Feb | Async job queue + Supabase schema update | Bull.js queue processing NDVI jobs |
| 24 Feb | Claims endpoint + job queueing logic | POST /api/claims triggers NDVI job |
| 25 Feb | Frontend NDVI hook + component | NDVICard displays pending/ready/error states |
| 25 Feb | PropertyClaimFlow integration | Icon + NDVI card both visible in claim view |
| 26 Feb | End-to-end testing (5+ test properties) | All claims have real NDVI baseline values |
| 27 Feb | Error handling + edge cases | Network failures, cloud cover issues handled |
| 28 Feb | Demo environment + documentation | Ready for March 1 Coordinator demo |

---

## Success Criteria (Feb 28)

```
[ ] GEE credentials configured (no hardcoded secrets)
[ ] 5+ test property claims in Supabase
[ ] Each claim has real ndvi_baseline (e.g., 0.625)
[ ] NDVI status transitions: pending → ready (visible in UI)
[ ] NDVI card shows: "NDVI: 0.625 (Sentinel-2, Sep 21 - Feb 21)"
[ ] "Refining data" spinner appears while pending
[ ] No NDVI computation delays claim creation (async job)
[ ] Icon state independent of NDVI status (they're separate)
[ ] Supabase real-time updates UI when NDVI ready
[ ] All claims survive restart with NDVI data intact
```

---

## Known Risks

1. **GEE API quotas** — Free tier has limits. Monitor usage.
   - *Mitigation:* Cache results, don't re-query same parcel unless requested
   
2. **Sentinel-2 data latency** — Newest data may be 5-7 days old.
   - *Mitigation:* Document in UI. Show date range clearly.
   
3. **Cloud cover** — Cloudy/rainy regions may have no clear images in 365 days.
   - *Mitigation:* Extend date range fallback, or show "Data unavailable for this location"
   
4. **Long processing time** — First job may take 30+ seconds (GEE cold start).
   - *Mitigation:* UI shows "up to 24 hours" - sets expectation. Actually faster.
   
5. **Redis dependency** — Bull.js requires Redis. If Redis goes down, jobs queue locally.
   - *Mitigation:* Use managed Redis (Upstash). Monitor queue depth.

---

## What's NOT Included (Phase 2+)

- ❌ Daily NDVI updates
- ❌ NDVI trending (3-year delta is stored, but no frontend trend chart)
- ❌ Multi-spectral indices (NDMI, NDBI, etc.)
- ❌ Cloud-free composite (use best available)
- ❌ Seasonal decomposition
- ❌ Machine learning predictions
- ❌ Historical NDVI snapshots (store one value per claim)

All of these are **Phase 2**, after March 1 soft launch.

---

## Env Var Setup (Copy-Paste for Emergent)

```bash
# GEE Service Account (from GCP console)
export GEE_PROJECT_ID="my-gee-project"
export GEE_SERVICE_ACCOUNT_EMAIL="ndvi-service@my-gee-project.iam.gserviceaccount.com"
export GEE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQE...
-----END PRIVATE KEY-----"

# Sentinel-2 parameters
export SENTINEL2_BASELINE_WINDOW_DAYS=365
export SENTINEL2_CLOUD_COVER_MAX=20

# Redis (Upstash example)
export REDIS_URL="redis://default:password@upstash-endpoint:port"

# Test: Check env vars loaded
node -e "console.log(process.env.GEE_PROJECT_ID)"
```

---

**Owner:** Emergent  
**Status:** Ready for implementation  
**Questions?** Ask Brad  
**Target Go-Live:** Feb 28, 18:00 UTC
