# P1: Parcel Boundary Rendering Specification

## Overview
Once DCDB integration is complete and users select a property, display the **actual parcel boundary** on the map with high visual contrast. This is the critical moment where users see "this polygon is MY land"—making the regeneration science tangible to their real property.

## Dependency
**Blocked by:** P1_DCDB_Integration_Spec  
**Unblocks:** P1_Grid_Highlight_Overlay_Spec, P1_Satellite_Icon_State_Logic_Spec

---

## Requirements

### 1. Visual Design

#### 1.1 Parcel Boundary Styling
When user claims a property, display the parcel boundary with maximum visual contrast:

| Property | Value | Rationale |
|----------|-------|-----------|
| **Stroke Color** | Pure white `rgba(255, 255, 255, 0.95)` | High contrast against satellite imagery |
| **Stroke Width** | 4px (native.tsx) or **3px** (web) | Clear and bold, not dominating |
| **Fill Color** | `rgba(34, 197, 94, 0.08)` | Subtle green tint (Land for Wildlife brand) |
| **Z-Index** | 1000 | Sits above grid cells and all other layers |
| **Animation** | Pulse 0.8s ease-in-out | Draws user attention on first load |

#### 1.2 Pulse Animation (First Load)
When parcel boundary first appears:
```tsx
// CSS or React Native Animated
@keyframes boundaryPulse {
  0% { opacity: 0.3; strokeWidth: 2; }
  50% { opacity: 1; strokeWidth: 4; }
  100% { opacity: 0.95; strokeWidth: 4; }
}

// Duration: 0.8s
// Repeat: Once on first render
```

#### 1.3 Parcel Boundary on Confirmation Screen
Show the boundary *while user is still in PropertyClaimFlow*, before final claim submission:

```
[Map with parcel boundary highlighted in white]

Your property:
123 Main Street, Brookfield QLD 4069
Parcel ID: 1234-567
Area: 5,000 m²

[Boundary is clearly visible in white on satellite imagery]

[Confirm Claim] [Cancel]
```

---

### 2. Frontend Implementation

#### 2.1 NativeMapView Enhancement
**File:** `frontend/src/components/NativeMapView.native.tsx`

**Add new prop:**
```tsx
interface NativeMapViewProps {
  grids: any[];
  onGridTap: (gridId: string) => void;
  mapCenter?: MapCenter | null;
  claimedParcel?: ParcelBoundary | null;  // NEW
}

interface ParcelBoundary {
  parcel_id: string;
  address: string;
  boundary: GeoJSON.Polygon;
  area_sqm: number;
}
```

**Add parcel boundary rendering:**
```tsx
// Inside NativeMapView component

// Memoized claimed parcel polygon
const claimedParcelPolygon = useMemo(() => {
  if (!claimedParcel?.boundary) return null;
  
  const coordinates = claimedParcel.boundary.coordinates[0].map((coord: number[]) => ({
    latitude: coord[1],
    longitude: coord[0],
  }));
  
  return (
    <Polygon
      key={`claimed-parcel-${claimedParcel.parcel_id}`}
      coordinates={coordinates}
      fillColor="rgba(34, 197, 94, 0.08)"
      strokeColor="rgba(255, 255, 255, 0.95)"
      strokeWidth={4}
      zIndex={1000}
      tappable={false}
    />
  );
}, [claimedParcel]);

// Inside <MapView>, add after grid polygons:
{mapReady && claimedParcelPolygon}

// Add label for claimed parcel
{mapReady && claimedParcel && (
  <Marker
    coordinate={{
      latitude: claimedParcel.boundary.centroid_lat || centerLat,
      longitude: claimedParcel.boundary.centroid_lng || centerLng,
    }}
    anchor={{ x: 0.5, y: 0.5 }}
    tracksViewChanges={false}
  >
    <View style={styles.claimedParcelLabel}>
      <Text style={styles.claimedParcelText}>Your property</Text>
    </View>
  </Marker>
)}
```

**Add styles:**
```tsx
claimedParcelLabel: {
  backgroundColor: 'rgba(34, 197, 94, 0.9)',
  paddingHorizontal: 12,
  paddingVertical: 6,
  borderRadius: 6,
  borderWidth: 1,
  borderColor: 'rgba(255, 255, 255, 0.8)',
},
claimedParcelText: {
  fontSize: 12,
  fontWeight: '600',
  color: '#FFFFFF',
},
```

**Add animation on mount:**
```tsx
// At component level, add useEffect
useEffect(() => {
  if (claimedParcel && !boundaryClaimed) {
    // Trigger pulse animation
    Animated.sequence([
      Animated.timing(boundaryOpacity, {
        toValue: 1,
        duration: 800,
        useNativeDriver: false,
      }),
    ]).start();
    setBoundaryClaimed(true);
  }
}, [claimedParcel]);
```

#### 2.2 PropertyClaimFlow Enhancement
**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

**Update state management:**
```tsx
interface PropertyClaimFlowProps {
  // ... existing props
  parcelBoundary?: ParcelBoundary | null;  // NEW: from DCDB
  onParcelSelected?: (parcel: ParcelBoundary) => void;  // NEW callback
}

export function PropertyClaimFlow({ parcelBoundary, onParcelSelected, ... }: PropertyClaimFlowProps) {
  // ...
  
  useEffect(() => {
    // When parcel is received from AddressSearch, show it on map
    if (parcelBoundary) {
      onParcelSelected?.(parcelBoundary);
    }
  }, [parcelBoundary, onParcelSelected]);
  
  // Render confirmation screen with parcel details
  return (
    <Modal visible={true} animationType="slide">
      <SafeAreaView>
        {/* Pass claimed parcel to NativeMapView */}
        <NativeMapView 
          grids={grids}
          onGridTap={handleGridTap}
          claimedParcel={parcelBoundary}
        />
        
        {/* Confirmation card */}
        <View style={styles.confirmationCard}>
          <Text style={styles.cardTitle}>Confirm Your Property</Text>
          <Text style={styles.cardText}>{parcelBoundary?.address}</Text>
          <Text style={styles.cardMeta}>
            Parcel ID: {parcelBoundary?.parcel_id}
          </Text>
          <Text style={styles.cardMeta}>
            Area: {(parcelBoundary?.area_sqm / 10000).toFixed(2)} hectares
          </Text>
          
          <Button 
            title="Yes, This Is My Land"
            onPress={handleConfirmClaim}
            style={styles.confirmButton}
          />
          <Button 
            title="Cancel"
            onPress={handleCancel}
            style={styles.cancelButton}
          />
        </View>
      </SafeAreaView>
    </Modal>
  );
}
```

#### 2.3 Parent Component Integration (PropertyClaimScreen)
**File:** `frontend/src/screens/PropertyClaimScreen.tsx` or equivalent

**Connect DCDB response to PropertyClaimFlow:**
```tsx
export function PropertyClaimScreen() {
  const [selectedParcel, setSelectedParcel] = useState<ParcelBoundary | null>(null);
  
  const handleAddressResolved = (dcdbResponse: any) => {
    // Called when DCDB returns parcel data
    setSelectedParcel({
      parcel_id: dcdbResponse.parcel.id,
      address: dcdbResponse.parcel.address,
      boundary: dcdbResponse.parcel.boundary,
      area_sqm: dcdbResponse.parcel.area_sqm,
    });
  };
  
  return (
    <>
      {/* AddressSearch passes resolved parcel data */}
      <AddressSearch 
        onLocationSelect={handleAddressResolved}
      />
      
      {/* Show claim flow with parcel boundary on map */}
      {selectedParcel && (
        <PropertyClaimFlow 
          parcelBoundary={selectedParcel}
          onParcelSelected={(p) => console.log('Parcel selected:', p)}
          onClaimConfirmed={(claim) => {
            // Create claim with parcel_id + boundary
            createClaim(claim);
          }}
        />
      )}
    </>
  );
}
```

---

### 3. Data Flow

```
DCDB Integration (previous spec) returns:
  {
    parcel_id: "1234-567",
    address: "123 Main St, Brookfield",
    boundary: { type: "Polygon", coordinates: [...] },
    area_sqm: 5000
  }
  ↓
PropertyClaimFlow receives claimedParcel prop
  ↓
NativeMapView renders Polygon with:
  - Stroke: white, 4px
  - Fill: subtle green rgba(34, 197, 94, 0.08)
  - Z-Index: 1000 (above grid cells)
  ↓
Pulse animation plays on first render (0.8s)
  ↓
User sees confirmation screen with:
  - Map with white boundary
  - Address + parcel ID + area
  - "Confirm" button
  ↓
User confirms → claim created with parcel_id + boundary
```

---

### 4. Rendering Strategy

#### 4.1 Polygon vs. Boundary Line
Two approaches:

| Approach | Render | Pros | Cons |
|----------|--------|------|------|
| **Filled Polygon** (recommended) | Polygon with fill + stroke | User sees entire property area shaded | May cover satellite details |
| **Boundary Lines Only** | Just the stroke, no fill | Preserves satellite imagery underneath | Less obvious property boundary |

**Recommendation:** Use filled polygon with **low opacity fill** (8%) so satellite imagery remains visible but property boundary is obvious.

#### 4.2 Performance: Multiple Parcels
If user navigates to select *another* property:
- Previous parcel boundary should fade out
- New parcel boundary should fade in (pulse animation)
- Grid cells return to normal styling

```tsx
// On parcel change:
if (previousParcelId !== claimedParcel.parcel_id) {
  // Fade out old boundary
  Animated.timing(oldBoundaryOpacity, {
    toValue: 0,
    duration: 300,
    useNativeDriver: false,
  }).start();
  
  // Fade in new boundary
  Animated.sequence([
    Animated.timing(newBoundaryOpacity, {
      toValue: 1,
      duration: 300,
      useNativeDriver: false,
    }),
  ]).start();
}
```

---

### 5. Testing Checklist

#### Unit Tests
- [ ] `test_parcel_boundary_renders_with_correct_colors()` - White stroke, green fill
- [ ] `test_pulse_animation_triggers_on_first_load()` - 0.8s animation plays
- [ ] `test_parcel_marked_above_grid_cells()` - Z-Index 1000 respected
- [ ] `test_label_marker_positions_correctly()` - "Your property" text at parcel centroid

#### Integration Tests
- [ ] AddressSearch returns parcel boundary GeoJSON
- [ ] PropertyClaimFlow receives parcel and passes to NativeMapView
- [ ] NativeMapView renders polygon from GeoJSON coordinates
- [ ] Confirmation screen shows address + parcel ID + area
- [ ] Previous parcel boundary clears when selecting new property

#### E2E Tests
- [ ] User searches "123 Main St, Brookfield"
- [ ] Parcel boundary appears on map in white
- [ ] Boundary has green fill overlay (subtle, not dominating)
- [ ] "Your property" label appears at parcel center
- [ ] Pulse animation plays on first load
- [ ] Confirmation screen shows correct address + area
- [ ] User confirms claim
- [ ] Claim is created with parcel_id + boundary in database
- [ ] If user selects different property, old boundary fades, new boundary appears

#### Visual QA
- [ ] White boundary is clearly visible on satellite map
- [ ] Green fill doesn't obscure satellite imagery
- [ ] Text label is readable on all backgrounds
- [ ] Animation is smooth (no jank on 60fps devices)
- [ ] Boundary renders correctly for all parcel shapes (irregular, concave, etc.)

---

### 6. Success Criteria

- [ ] Parcel boundary renders immediately after user selects property
- [ ] Stroke is pure white, 4px wide, high contrast
- [ ] Fill is subtle green (alpha 0.08), doesn't cover satellite data
- [ ] Animation pulses smoothly on first load (0.8s)
- [ ] Label "Your property" appears at parcel centroid
- [ ] Confirmation screen displays before claim submission
- [ ] Previous boundary clears when switching properties
- [ ] All E2E test cases pass

---

### 7. Files to Modify

| File | Changes |
|------|---------|
| `frontend/src/components/NativeMapView.native.tsx` | Add `claimedParcel` prop, render Polygon, add pulse animation, add label Marker |
| `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx` | Accept `parcelBoundary` prop, display confirmation screen, pass to NativeMapView |
| `frontend/src/screens/PropertyClaimScreen.tsx` | Connect DCDB response to PropertyClaimFlow, manage parcel state |
| `frontend/src/components/AddressSearch.tsx` | Return full parcel object (boundary + metadata) from DCDB response |

---

### 8. Timeline Estimate
- Native map polygon rendering: **1-2 hours**
- Animation + styling: **1 hour**
- Confirmation screen UI: **1-2 hours**
- Testing + QA: **2 hours**
- **Total: 5-7 hours**

---

## Why This Matters

**This is the moment users see their land.**

No mock boundaries, no approximations. The white polygon on the satellite map is their *actual legal property*. The green tint shows it's enrolled in regeneration. The pulse animation draws attention to the boundary.

This visual moment—seeing your real land changing color from satellite data—is the **emotional anchor** that makes users trust the system enough to pay for it.

---

## Next Steps (After Parcel Boundary Complete)

Once this spec is implemented:
1. Proceed to **P1_Grid_Highlight_Overlay_Spec** - Overlay grid cells on top of parcel boundary for landscape context
2. Then **P1_Satellite_Icon_State_Logic_Spec** - Icon appears only when monitoring is active
3. Finally, **P2_Stripe_Integration_Spec** - Now users believe the science; time to monetize
