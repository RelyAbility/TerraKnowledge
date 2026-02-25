# P1: Real QLD DCDB Integration Specification

## Overview
Integrate Queensland's Digital Cadastral Database (DCDB) to enable **real address-to-parcel resolution** within the 4.2 km Gondwana pilot area. This is the foundation for the visual trust loop—users must see their *actual legal parcel*, not just a grid approximation.

## Pilot-Only Scope (Critical)

**DO NOT:**
- ❌ Build national cadastral registry architecture
- ❌ Over-engineer caching for 100,000+ parcels
- ❌ Design multi-region DCDB fallback chains
- ❌ Create complex data synchronization pipelines

**DO:**
- ✅ Make Brookfield + Moggill Creek work cleanly
- ✅ ~200-300 parcels in 4.2 km pilot area
- ✅ Simple, direct DCDB API calls
- ✅ Cache only pilot area parcels (single Redis hash)
- ✅ Hardcode 4.2 km boundary; reject requests outside it

**Timeline Distinction:**
- **Now (P1):** Pilot works perfectly for 4.2 km
- **Later (Phase 2+):** If scaling to multiple regions, refactor architecture

This keeps implementation lean and fast.

## Phase 1 Context
- **Goal:** Complete property visualization trust loop before Stripe integration
- **Dependency:** This must be complete before parcel boundary rendering and satellite icon logic
- **Timeline:** Implement before proceeding to P2 (Stripe)
- **Data Scope:** Brookfield Estate and 4.2 km pilot zone ONLY
- **Product Promise:** "Your regeneration is visible from space" (visual proof, not payment)
- **Architecture Scope:** Pilot-only implementation. Do NOT over-engineer for national scaling yet.

---

## Requirements

### Product Promise
Before you start: Remember why this matters.

**Product Promise:** "Your regeneration is visible from space."

The user's value is **visual proof** that their land is improving, measured by satellite. DCDB integration is the "real world binding"—it connects the satellite signal to their *actual legal property*, not an approximation.

This is why DCDB (not mock data) matters before Stripe. Users need to see *their real parcel* changing color on the map. That builds trust in the system. Payment comes *after* they believe the science.

---

### 1. DCDB API Integration
**Objective:** Establish connection to QLD cadastral data source (CKAN, WFS, or QSpatial).

#### 1.1 Cadastral Data Source Options
Choose one (recommend **QSpatial WFS**):

| Source | API Type | Pros | Cons |
|--------|----------|------|------|
| **QSpatial WFS** | OGC WFS | Official QLD, real-time, spatial filtering | Auth required |
| **CKAN/QGIS Server** | REST + GeoJSON | Free, well-documented | May lag behind |
| **Land Titles Office** | REST API | Authoritative | License required |

**Implementation Choice:** Use **QSpatial WFS** with fallback to cached GeoJSON for 4.2 km pilot.

#### 1.2 Backend Integration Point
**File:** `backend/services/dcdb_service.py` (new)

**Core Functions:**
```python
# Address → Parcel GeoJSON
async def resolve_address_to_parcel(
    address: str,
    lat: float,
    lng: float
) -> Dict:
    """
    Input: User's searching address + GPS fallback
    Process: Hit DCDB/QSpatial WFS with spatial filter (4.2 km radius)
    Output:
    {
        "parcel_id": "1234-567",
        "address": "123 Main St, Brookfield QLD 4069",
        "owner": "Jane Doe",
        "area_sqm": 5000,
        "boundary": {
            "type": "Polygon",
            "coordinates": [[[lng, lat], ...]]
        },
        "confidence": 0.95
    }
    """
    pass

# Reverse lookup: point → parcel
async def resolve_point_to_parcel(lat: float, lng: float) -> Dict:
    """
    Input: GPS coordinate
    Process: Spatial query against DCDB
    Output: Parcel GeoJSON + metadata
    """
    pass

# Bulk cache for pilot area (initialization)
async def cache_pilot_area_parcels() -> None:
    """
    On startup, load all DCDB parcels within 4.2 km pilot area
    Store in Redis with spatial index for fast queries
    """
    pass
```

#### 1.3 API Endpoint: Address Resolution
**Endpoint:** `POST /api/dcdb/resolve-address`

**Request:**
```json
{
  "address": "123 Main St, Brookfield",
  "latitude": -27.4902,
  "longitude": 152.9156
}
```

**Response:**
```json
{
  "status": "resolved",
  "parcel": {
    "id": "1234-567",
    "address": "123 Main Street, Brookfield QLD 4069",
    "boundary": {
      "type": "Polygon",
      "coordinates": [[[152.915, -27.490], ...]]
    },
    "accuracy": "exact",
    "source": "qspatial_wfs"
  },
  "grid_cell": {
    "grid_id": "T_E1_N2",
    "priority_index": 78
  }
}
```

**Response Codes:**
- `200` - Parcel resolved successfully
- `400` - Invalid address
- `404` - No parcel found (outside pilot area)
- `503` - DCDB service unavailable (use cache)

#### 1.4 API Endpoint: Point-to-Parcel Lookup
**Endpoint:** `POST /api/dcdb/resolve-point`

**Request:**
```json
{
  "latitude": -27.4902,
  "longitude": 152.9156
}
```

**Response:**
```json
{
  "status": "resolved",
  "parcel": {
    "id": "1234-567",
    "address": "123 Main Street, Brookfield QLD 4069",
    "boundary": { "type": "Polygon", "coordinates": [...] }
  }
}
```

---

### 2. Frontend Integration Point

#### 2.1 AddressSearch Component Enhancement
**File:** `frontend/src/components/AddressSearch.tsx`

**Changes:**
- Replace mock address autocomplete with real DCDB address resolution
- Call `POST /api/dcdb/resolve-address` on each address selection
- Pass parcel boundary + grid cell back to PropertyClaimFlow

**Code Reference:**
```tsx
// Before (mock):
const handleAddressSelect = (address: string) => {
  onLocationSelect(lat, lng, mockGridId);
};

// After (real DCDB):
const handleAddressSelect = async (address: string) => {
  const response = await fetch('/api/dcdb/resolve-address', {
    method: 'POST',
    body: JSON.stringify({
      address,
      latitude: currentLat,
      longitude: currentLng
    })
  });
  
  const { parcel, grid_cell } = await response.json();
  
  // Pass parcel boundary to map
  setParcelBoundary(parcel.boundary);
  
  // Select grid cell
  onLocationSelect(parcel.boundary.centroid_lat, parcel.boundary.centroid_lng, grid_cell.grid_id);
};
```

#### 2.2 PropertyClaimFlow Enhancement
**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

**Changes:**
- Accept parcel boundary from AddressSearch
- Display parcel boundary on map (see P1_Parcel_Boundary_Rendering_Spec)
- Display actual address + parcel ID on confirmation screen
- Store parcel_id + boundary in claim creation request

**Confirmation Screen Update:**
```tsx
// Show user their actual parcel before confirming claim
<Text>Claiming: {parcel.address}</Text>
<Text>Parcel ID: {parcel.id}</Text>
<Text>Area: {parcel.area_sqm} m²</Text>
```

#### 2.3 Backend Claims Endpoint Update
**File:** `backend/server.py` - `POST /api/claims`

**Updated Request Schema:**
```json
{
  "user_id": "user_123",
  "property_address": "123 Main Street, Brookfield QLD 4069",
  "parcel_id": "1234-567",
  "grid_id": "T_E1_N2",
  "boundary": {
    "type": "Polygon",
    "coordinates": [[[152.915, -27.490], [152.916, -27.490], ...]]
  }
}
```

**Database Update:**
Add columns to `property_claims` table:
```sql
ALTER TABLE property_claims ADD COLUMN parcel_id VARCHAR(50);
ALTER TABLE property_claims ADD COLUMN boundary JSONB;
ALTER TABLE property_claims ADD COLUMN dcdb_address VARCHAR(255);
```

---

### 3. Data Flow Diagram

```
User enters address in AddressSearch
    ↓
frontend/src/components/AddressSearch.tsx calls POST /api/dcdb/resolve-address
    ↓
backend/services/dcdb_service.py queries QSpatial WFS
    ↓
DCDB returns parcel GeoJSON + metadata
    ↓
Response: { parcel, grid_cell }
    ↓
PropertyClaimFlow receives parcel boundary + grid cell
    ↓
User confirms claim
    ↓
POST /api/claims with parcel_id + boundary + address
    ↓
Claims table stores parcel_id, boundary, dcdb_address
    ↓
P1_Parcel_Boundary_Rendering displays boundary on map
```

---

### 4. Pilot Area Scope

**Geographic Bounds:**
- Center: -27.4902, 152.9156 (Gondwana Pilot)
- Radius: 4.2 km
- Postcodes: 4069 (Brookfield area)
- Estimated parcels: ~200-300

**Cache Strategy:**
1. On backend startup: Load all parcels within 4.2 km radius into Redis
2. Cache key: `dcdb:within:4.2km`
3. TTL: 24 hours
4. Fallback: If DCDB unavailable, use cached GeoJSON

**Performance Target:**
- Address resolution: < 500ms
- Point-to-parcel lookup: < 200ms (from cache)

---

### 5. Testing Checklist

#### Unit Tests
- [ ] `test_resolve_address_returns_valid_parcel()` - Valid address resolves to parcel
- [ ] `test_resolve_address_outside_pilot_area()` - Errors gracefully if outside 4.2 km
- [ ] `test_resolve_point_returns_parcel()` - GPS point maps to parcel
- [ ] `test_cache_pilot_area_parcels()` - Startup cache loads correctly
- [ ] `test_dcdb_fallback_to_cache()` - Uses cached GeoJSON if API down

#### Integration Tests
- [ ] Address search in frontend calls backend correctly
- [ ] Parcel boundary is returned in response
- [ ] PropertyClaimFlow receives and displays parcel metadata
- [ ] Claim creation request includes parcel_id + boundary

#### E2E Tests
- [ ] User can search "123 Main St, Brookfield"
- [ ] Parcel boundary appears on map (see P1_Parcel_Boundary_Rendering_Spec)
- [ ] Confirmation screen shows correct address + parcel ID
- [ ] Claim is created with parcel_id + boundary in database

#### Manual QA
- [ ] Test 10 real addresses within Brookfield pilot area
- [ ] Verify boundaries render correctly on map
- [ ] Verify accuracy of DCDB data vs. real property
- [ ] Test outside pilot area (should return 404)

---

### 6. Success Criteria

- [ ] DCDB service fetches real parcel data for Brookfield pilot area
- [ ] Address search resolves to actual legal parcels (not mock data)
- [ ] Parcel GeoJSON boundaries are accurate and complete
- [ ] API response time < 500ms for address resolution
- [ ] Fallback cache prevents outages during DCDB downtime
- [ ] PropertyClaimFlow displays actual address + parcel ID on confirmation
- [ ] Claims table stores parcel_id + boundary for each claim
- [ ] All 10 E2E test cases pass

---

### 7. Dependencies & Blockers

**External Dependencies:**
- QSpatial API credentials (request from QLD)
- DCDB WFS endpoint documentation

**Internal Dependencies:**
- `backend/database.py` - Schema updates for parcel_id, boundary, dcdb_address columns
- `frontend/src/components/AddressSearch.tsx` - Must be updated to call real DCDB endpoint
- `backend/server.py` - New `/api/dcdb/resolve-address` and `/api/dcdb/resolve-point` endpoints

**No Blockers:** This work is independent and can proceed immediately.

---

### 8. Next Steps (After DCDB Complete)

Once this spec is implemented:
1. Proceed to **P1_Parcel_Boundary_Rendering_Spec** to display parcel boundary on map
2. Proceed to **P1_Grid_Highlight_Overlay_Spec** for grid cell highlight
3. Then **P1_Satellite_Icon_State_Logic_Spec** for icon visibility
4. Finally, **P2_Stripe_Integration_Spec** (do NOT implement Stripe against mock data)

---

## Files to Create/Modify

| File | Type | Purpose |
|------|------|---------|
| `backend/services/dcdb_service.py` | NEW | DCDB API integration |
| `backend/server.py` | MODIFY | Add `/api/dcdb/resolve-address`, `/api/dcdb/resolve-point` endpoints |
| `frontend/src/components/AddressSearch.tsx` | MODIFY | Call real DCDB endpoint |
| `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx` | MODIFY | Accept + display parcel metadata |
| Database schema | MODIFY | Add parcel_id, boundary, dcdb_address columns |

---

## Timeline Estimate
- DCDB API integration: **2-3 hours**
- Frontend integration: **2-3 hours**
- Testing + QA: **2-3 hours**
- **Total: 6-9 hours**

This is foundational work. Get it right before moving to visual rendering.
