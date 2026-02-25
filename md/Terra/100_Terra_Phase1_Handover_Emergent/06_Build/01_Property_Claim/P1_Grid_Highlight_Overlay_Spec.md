# P1: Grid Highlight Overlay Specification

## Overview
Once the parcel boundary is rendered, overlay the grid cell(s) that intersect with the claimed property. This provides **landscape context**—users see both their legal parcel boundary *and* how it aligns with the TBLS grid system used for vegetation monitoring.

## Dependency
**Blocked by:** P1_Parcel_Boundary_Rendering_Spec  
**Unblocks:** P1_Satellite_Icon_State_Logic_Spec

---

## Requirements

### 1. Grid Cell Selection Logic

#### 1.1 Find Intersecting Grid Cells
When user claims a parcel, determine which grid cells overlap with the parcel boundary:

```python
# backend/services/dcdb_service.py (addition to existing service)

from shapely.geometry import shape, Polygon
import geopandas as gpd

async def find_intersecting_grid_cells(
    parcel_boundary: Dict,
    all_grids: List[Dict]
) -> List[str]:
    """
    Input: Parcel GeoJSON boundary + all available grid cells
    Process: Spatial intersection using Shapely/GeoPandas
    Output: List of grid_ids that overlap with parcel
    
    Example:
      Input: Parcel boundary (5,000 m²) + 50 grid cells
      Output: ["T_E1_N2", "T_E2_N2", "T_E1_N3"] (3 grid cells overlap)
    """
    
    # Convert parcel boundary to Shapely polygon
    parcel_poly = shape(parcel_boundary)
    
    # Find all grid cells that intersect
    intersecting_grids = []
    for grid in all_grids:
        if not grid.get('geojson'):
            continue
        
        grid_poly = shape(grid['geojson'])
        
        # Check if grid intersects parcel
        if parcel_poly.intersects(grid_poly):
            intersecting_grids.append({
                'grid_id': grid['grid_id'],
                'priority_index': grid['priority_index'],
                'overlap_percentage': (
                    parcel_poly.intersection(grid_poly).area / parcel_poly.area * 100
                )
            })
    
    return intersecting_grids
```

#### 1.2 Return Intersecting Grids in DCDB Response
**Update:** `POST /api/dcdb/resolve-address` endpoint to include intersecting grids:

```json
{
  "status": "resolved",
  "parcel": {
    "id": "1234-567",
    "address": "123 Main Street, Brookfield QLD 4069",
    "boundary": { "type": "Polygon", "coordinates": [...] },
    "area_sqm": 5000
  },
  "intersecting_grids": [
    {
      "grid_id": "T_E1_N2",
      "priority_index": 78,
      "overlap_percentage": 60.5
    },
    {
      "grid_id": "T_E2_N2",
      "priority_index": 72,
      "overlap_percentage": 39.5
    }
  ]
}
```

---

### 2. Frontend Implementation

#### 2.1 Update PropertyClaimFlow State
**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

```tsx
interface PropertyClaimFlowProps {
  parcelBoundary?: ParcelBoundary | null;
  intersectingGrids?: Array<{
    grid_id: string;
    priority_index: number;
    overlap_percentage: number;
  }> | null;
}

export function PropertyClaimFlow({ 
  parcelBoundary, 
  intersectingGrids,
  ... 
}: PropertyClaimFlowProps) {
  // parcelBoundary is claimed property (white boundary)
  // intersectingGrids are TBLS cells that overlap (highlighted with bold stroke)
}
```

#### 2.2 Update NativeMapView to Highlight Intersecting Grids
**File:** `frontend/src/components/NativeMapView.native.tsx`

```tsx
interface NativeMapViewProps {
  grids: any[];
  onGridTap: (gridId: string) => void;
  mapCenter?: MapCenter | null;
  claimedParcel?: ParcelBoundary | null;
  highlightedGridIds?: string[] | null;  // NEW: grid IDs to highlight
}

export function NativeMapView({ 
  grids, 
  onGridTap, 
  mapCenter,
  claimedParcel,
  highlightedGridIds  // NEW
}: NativeMapViewProps) {
  
  // Modify grid polygon rendering
  const gridPolygons = useMemo(() => {
    return grids.map((grid: any) => {
      if (!grid.geojson?.coordinates?.[0]) return null;
      
      // Check if this grid is highlighted (intersects claimed parcel)
      const isHighlighted = highlightedGridIds?.includes(grid.grid_id) ?? false;
      const isSelected = grid.grid_id === selectedGridId;
      
      const coordinates = grid.geojson.coordinates[0].map((coord: number[]) => ({
        latitude: coord[1],
        longitude: coord[0],
      }));
      
      // Apply different styling based on state
      let strokeWidth: number;
      let strokeColor: string;
      let zIndex: number;
      
      if (isSelected) {
        // User tapped this cell
        strokeWidth = 3;
        strokeColor = 'rgba(255, 255, 255, 0.95)';
        zIndex = 100;
      } else if (isHighlighted) {
        // This cell overlaps claimed parcel
        strokeWidth = 2.5;
        strokeColor = 'rgba(22, 163, 74, 0.85)'; // Darker green for emphasis
        zIndex = 50;
      } else {
        // Normal cell
        strokeWidth = 1.5;
        strokeColor = 'rgba(255, 255, 255, 0.55)';
        zIndex = 1;
      }
      
      const colors = getEarthToneColor(grid.priority_index || 0, isSelected);
      
      return (
        <Polygon
          key={grid.grid_id}
          coordinates={coordinates}
          fillColor={colors.fill}
          strokeColor={isHighlighted ? strokeColor : colors.stroke}
          strokeWidth={strokeWidth}
          tappable={true}
          onPress={() => handleGridSelect(grid.grid_id)}
          zIndex={zIndex}
        />
      );
    });
  }, [grids, selectedGridId, highlightedGridIds]);
  
  // Parcel boundary renders on top (z-index: 1000)
  // Highlighted grids render with strong green stroke (z-index: 50)
  // Normal grids render with faint white stroke (z-index: 1)
  // Selected grid renders with white stroke (z-index: 100)
}
```

#### 2.3 Visual Hierarchy

```
Rendering Order (Z-Index):
  1000 ← Parcel boundary (white, 4px, on top)
   100 ← Selected grid cell (if tapped)
    50 ← Highlighted grid cells (green, 2.5px, overlaps parcel)
     1 ← Normal grid cells (white, 1.5px)
```

---

### 3. Visual Design

#### 3.1 Highlighted Grid Cell Styling
When a grid cell overlaps the claimed parcel:

| Property | Value | Rationale |
|----------|-------|-----------|
| **Stroke Color** | `rgba(22, 163, 74, 0.85)` | Darker green (Land for Wildlife brand color) |
| **Stroke Width** | 2.5px | Visible but not as bold as selected |
| **Z-Index** | 50 | Above normal cells, below parcel |
| **Fill Color** | Keep original earth tone | Emphasize via stroke, not fill |

#### 3.2 Example Visual Sequence

```
User searches "123 Main St, Brookfield"
  ↓
DCDB returns parcel boundary + intersecting grid cells
  ↓
PropertyClaimFlow renders:
  - Parcel boundary: Pure white polygon on satellite
  - Overlapping grids: Green stroke (darker than default)
  - Other grids: Normal faint white stroke
  ↓
User sees:
  [Satellite map with white parcel boundary]
  [2 grid cells highlighted with green stroke]
  [All other grids with normal white stroke for context]
  ↓
User confirms claim
```

---

### 4. Implementation Details

#### 4.1 Pass Intersecting Grids from Parent
**File:** `frontend/src/screens/PropertyClaimScreen.tsx`

```tsx
const handleAddressResolved = (dcdbResponse: any) => {
  // DCDB returns parcel + intersecting grids
  setSelectedParcel({
    parcel_id: dcdbResponse.parcel.id,
    address: dcdbResponse.parcel.address,
    boundary: dcdbResponse.parcel.boundary,
    area_sqm: dcdbResponse.parcel.area_sqm,
  });
  
  // Extract intersecting grid IDs for highlighting
  setHighlightedGridIds(
    dcdbResponse.intersecting_grids?.map(g => g.grid_id) || []
  );
};

return (
  <>
    <AddressSearch onLocationSelect={handleAddressResolved} />
    {selectedParcel && (
      <PropertyClaimFlow 
        parcelBoundary={selectedParcel}
        intersectingGrids={dcdbResponse.intersecting_grids}
        highlightedGridIds={highlightedGridIds}
        onClaimConfirmed={createClaim}
      />
    )}
  </>
);
```

#### 4.2 Update Claim Creation
Store which grids were intersected in the claim:

**Backend schema update:**
```sql
ALTER TABLE property_claims ADD COLUMN intersecting_grid_ids TEXT[];
-- Stores: ["T_E1_N2", "T_E2_N2"]
```

**Claim request:**
```json
{
  "user_id": "user_123",
  "property_address": "123 Main St, Brookfield",
  "parcel_id": "1234-567",
  "boundary": { "type": "Polygon", "coordinates": [...] },
  "intersecting_grid_ids": ["T_E1_N2", "T_E2_N2"]
}
```

---

### 5. Why Grid Context Matters

**Problem:** User sees white parcel boundary, but doesn't understand how it relates to TBLS monitoring grid.

**Solution:** Highlight the 2-3 grid cells that overlap their property. This shows:
- "My land spans parts of these landscape units"
- "My NDVI score is aggregated from these grid cells"
- "My regeneration is measured within this ecological context"

**Result:** User understands the science is grounded in real landscape segments, not arbitrary pixels.

---

### 6. Testing Checklist

#### Unit Tests
- [ ] `test_find_intersecting_grid_cells_returns_overlaps()` - Correct grids identified
- [ ] `test_calculate_overlap_percentage()` - Percentages accurate
- [ ] `test_highlighted_grids_render_with_green_stroke()` - Green stroke applied
- [ ] `test_highlighted_grid_z_index_above_normal()` - Z-index 50 > 1

#### Integration Tests
- [ ] DCDB response includes `intersecting_grids` array
- [ ] PropertyClaimFlow receives intersecting_grids
- [ ] NativeMapView receives highlightedGridIds
- [ ] Highlighted grids render with darker green stroke
- [ ] Normal grids still render with faint white stroke
- [ ] Parcel boundary renders on top of all grids

#### E2E Tests
- [ ] User searches address that overlaps 2 grid cells
- [ ] Parcel boundary appears in white
- [ ] 2 intersecting grid cells appear with green stroke
- [ ] Other grid cells remain with faint white stroke for context
- [ ] Confirmation screen shows all elements correctly
- [ ] User confirms claim
- [ ] Database stores `intersecting_grid_ids` correctly
- [ ] Claim is associated with correct grids for future NDVI aggregation

#### Visual QA
- [ ] Green stroke is clearly visible and distinct from white
- [ ] Highlighted grids don't obscure parcel boundary
- [ ] Parcel boundary remains sharp and visible on top
- [ ] Visual hierarchy is clear (white > green > faint)

---

### 7. Success Criteria

- [ ] DCDB service identifies all grid cells that intersect claimed parcel
- [ ] Intersecting grids returned in DCDB response with overlap percentage
- [ ] Highlighted grid cells render with darker green stroke (rgba(22, 163, 74, 0.85))
- [ ] Green stroke is 2.5px wide, clearly visible
- [ ] Parcel boundary remains pure white on top (z-index: 1000)
- [ ] Z-index hierarchy renders correctly (parcel > selected > highlighted > normal)
- [ ] Claim stores intersecting_grid_ids for future reference
- [ ] All E2E test cases pass

---

### 8. Files to Modify

| File | Changes |
|------|---------|
| `backend/services/dcdb_service.py` | Add `find_intersecting_grid_cells()` function |
| `backend/server.py` | Update `/api/dcdb/resolve-address` to return intersecting_grids |
| `frontend/src/components/NativeMapView.native.tsx` | Add `highlightedGridIds` prop, modify grid rendering logic, apply green stroke |
| `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx` | Accept intersecting_grids, pass to NativeMapView |
| `frontend/src/screens/PropertyClaimScreen.tsx` | Extract highlighted grid IDs from DCDB response, pass to PropertyClaimFlow |
| `backend/database.py` | Add `intersecting_grid_ids` column to property_claims table |

---

### 9. Timeline Estimate
- Backend intersection detection: **1-2 hours**
- Frontend grid highlighting logic: **1-2 hours**
- Styling + Z-index tuning: **1 hour**
- Testing + QA: **2 hours**
- **Total: 5-7 hours**

---

## Next Steps (After Grid Overlay Complete)

Once this spec is implemented:
1. Proceed to **P1_Satellite_Icon_State_Logic_Spec** - Icon visibility based on monitoring_state
2. Then **P2_Stripe_Integration_Spec** - Payment lifecycle tied to real parcel + grid context
